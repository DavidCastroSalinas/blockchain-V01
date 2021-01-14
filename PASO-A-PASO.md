# Configuración de una red HyperLedger
## Código parte del proyecto: Red blockchain para libros eclesiales de la iglesia católica en Chile
### contacto: dcastros@gmail.com
### Equipo de trabajo RealTeam {Carlos Araya; Nicolás Ormeño; David Castro}

# 1. Instalar pre-requisitos

1.1. Permitir ejecutar el archivo

```console
chmod +x install-prereq.sh

```

1.2. Ejecutar script, o correr los comandos que existen dentro del script

```console
./install-prereq.sh
```

1.3. Ingresar a carpeta con el proyecto
```console
cd libroEclesial
```
1.4. Creación de carpeta para el material criptográfico de Certificados de Autenticación
```console
mkdir channel-artifacts
mkdir crypto-config
mkdir chaincode
mkdir base
mkdir scripts
```



# 2. Preparación Material Criptográfico

2.1. Crear crypto-config.yaml
En nuestro caso, consideraremos como organizaciones válidas a cada una de las diocesis de Chile 
fuente: http://iglesia.cl/

```yaml
OrdererOrgs:
    - Name: Orderer
      Domain: libroeclesial.info
      EnableNodeOUs: true
      Specs:
        - Hostname: orderer
          SANS:
            - localhost

PeerOrgs:
    - Name: Arquidiocesis_Santiago
      Domain: arquidiocesis.santiago.libroeclesial.info
      EnableNodeOUs: true
      Template:
        Count: 1
        SANS:
          - localhost
      Users:
        Count: 1

    - Name: Diocesis_Iquique
      Domain: diocesis.iquique.libroeclesial.info
      EnableNodeOUs: true
      Template:
        Count: 1
        SANS:
          - localhost
      Users:
        Count: 1

    - Name: Arquidiocesis_Antofagasta
      Domain: arquidiocesis.antofagasta.libroeclesial.info
      EnableNodeOUs: true
      Template:
        Count: 1
        SANS:
          - localhost
      Users:
        Count: 1

    - Name: P003
      Domain: p003.libroeclesial.info
      EnableNodeOUs: true    
      Template:
        Count: 1
        SANS:
          - localhost
      Users:
        Count: 1        
```

2.2. Ejecutar Generar el material criptográfico de organizaciones y usuarios
*El comando **cryptogen** es parte de los elementos instalados en los prerequisitos, y al ejecutarla utilizará el archivo **crypto-config.yaml** para generar todo el material criptográfico necesario para nuestra red blockchain dentro de la carpeta /channel-artifacts*

```console
cryptogen generate --config=./crypto-config.yaml
```

5. mkdir channel-artifacts

6. Crear blockchain-network/configtx.yaml

7. Ingresar a blockchain-network

8. Configurar canales 

```
configtxgen -profile ThreeOrgsOrdererGenesis -channelID system-channel  -outputBlock ./channel-artifacts/genesis.block
```

```
configtxgen -profile ThreeOrgsChannel -channelID marketplace -outputCreateChannelTx ./channel-artifacts/channel.tx
```

```
configtxgen -profile ThreeOrgsChannel -channelID marketplace -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -asOrg Org1MSP
```

```
configtxgen -profile ThreeOrgsChannel -channelID marketplace -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -asOrg Org2MSP
```

```
configtxgen -profile ThreeOrgsChannel -channelID marketplace -outputAnchorPeersUpdate ./channel-artifacts/Org3MSPanchors.tx -asOrg Org3MSP
```

9. Crear blockchain-network/base/peer-base.yaml

10. Crear blockchain-network/docker-compose-base-yaml

11. Crear blockchain-netowrk/docker-compose-cli-couchdb.yaml

12. Exportamos las variables que necesitamos

```
export CHANNEL_NAME=marketplace
export FABRIC_CFG_PATH=$WPD # debemos estar en blockchain-network
export VERBOSE=false
```

13. Levantamos docker-compose

```
CHANNEL_NAME=$CHANNEL_NAME docker-compose -f docker-compose-cli-couchdb.yaml up -d
```

14. Probamos que esté ok el contenedor

```
docker exec -it cli bash
```

15. Dentro del contenedor CLI, creamos la variable de ambiante con el nombre del canal

```
export CHANNEL_NAME=marketplace
```

16. Creamos el canal

```
peer channel create -o orderer.acme.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/acme.com/orderers/orderer.acme.com/msp/tlscacerts/tlsca.acme.com-cert.pem
```

17. Hacer que la organización sea parte del canal

```
peer channel join -b marketplace.block
```

18. Hacemos lo mismo para la organización 2 y 3

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/users/Admin@org2.acme.com/msp/ CORE_PEER_ADDRESS=peer0.org2.acme.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/peers/peer0.org2.acme.com/tls/ca.crt peer channel join -b marketplace.block
```

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/users/Admin@org3.acme.com/msp/ CORE_PEER_ADDRESS=peer0.org3.acme.com:7051 CORE_PEER_LOCALMSPID="Org3MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/peers/peer0.org3.acme.com/tls/ca.crt peer channel join -b marketplace.block
```

19. Ahora debemos configurar el anchor peer para cada organización

```
peer channel update -o orderer.acme.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/acme.com/orderers/orderer.acme.com/msp/tlscacerts/tlsca.acme.com-cert.pem
```

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/users/Admin@org2.acme.com/msp/ CORE_PEER_ADDRESS=peer0.org2.acme.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/peers/peer0.org2.acme.com/tls/ca.crt peer channel update -o orderer.acme.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/acme.com/orderers/orderer.acme.com/msp/tlscacerts/tlsca.acme.com-cert.pem
```

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/users/Admin@org3.acme.com/msp/ CORE_PEER_ADDRESS=peer0.org3.acme.com:7051 CORE_PEER_LOCALMSPID="Org3MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/peers/peer0.org3.acme.com/tls/ca.crt peer channel update -o orderer.acme.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org3MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/acme.com/orderers/orderer.acme.com/msp/tlscacerts/tlsca.acme.com-cert.pem
```


20. Crear la carpeta chaincode/foodcontrol

21. Crear el archivo chaincode/foodcontrol/foodcontrol.go

22. Crear el archivo chaincode/foodcontrol/go.mod

23. Ingresamos al contanedor cli

```
docker exec -it cli bash
```

24. Dentro del contenedor verificamos que exista la ruta `/opt/gopath/src/github.com/chaincode/`, si no existe, es probable que sólo reiniciando el contenedor, tome la ruta

25. Dentro del contenedor, nos aseguramos de tener las siguientes variables

```
export CHANNEL_NAME=marketplace
export CHAINCODE_NAME=foodcontrol
export CC_RUNTIME_LANGUAGE=golang
export CC_SRC_PATH="../../../chaincode/$CHAINCODE_NAME/"
export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/acme.com/orderers/orderer.acme.com/msp/tlscacerts/tlsca.acme.com-cert.pem
export CHAINCODE_VERSION=3
export CHAINCODE_SEQUENCE=4
```

26. Hacer uso del ciclo de vida de chaincode, para empaquetar el proyecto

```
peer lifecycle chaincode package ${CHAINCODE_NAME}.tar.gz --path ${CC_SRC_PATH} --lang ${CC_RUNTIME_LANGUAGE} --label ${CHAINCODE_NAME}_${CHAINCODE_VERSION} >& log.txt
```

Esto debería generar algo como lo siguiente:

```
peer lifecycle chaincode package foodcontrol.tar.gz --path ../../../chaincode/$CHAINCODE_NAME/ --lang ${CC_RUNTIME_LANGUAGE} --label ${CHAINCODE_NAME}_${CHAINCODE_VERSION} >& log.txt
```

27. Revisamos que se cree correctamente, esto lo hacemos, `ls -l` para revisar que exista el archivo _foodcontrol.tar.gz_

- Aquí se nos puede generar el archivo go.sum (si es que no lo teníamos con anterioridad)

28. En la misma máquina del CLI, vamos a instalar el chaincode en las organizaciones

```
peer lifecycle chaincode install foodcontrol.tar.gz
```

29. Buscamos el identificador, que es el último mensaje, donde dice "Chaincode code package identifier", un ejemplo de identificador es:

_foodcontrol_1:d3511338dbe0363894823b9c0f5f098245af222a407e7dafdf78a8c53c1d2146_
_foodcontrol_2:a032e43a6be1450f0281f10ed42f4b8c2e6afdf4033e6ea1d0f8721d7f746204_
_foodcontrol_3:65b3cc9144708b16913e623542a98c2e26e8998e2e637c65e53d2ea59b93f746_

_* Esto es por que al instalarlo en otras organizaciones, nos debe dar el mismo identificador que el obtenido, y así nos aseguramente que todas las organizaciones instalen el paquete, y tengan el mismo paquete instalado_

30. Para instalarlo en otras organizaciones, debemos hacer lo siguiente:

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/users/Admin@org2.acme.com/msp/ CORE_PEER_ADDRESS=peer0.org2.acme.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/peers/peer0.org2.acme.com/tls/ca.crt peer lifecycle chaincode install ${CHAINCODE_NAME}.tar.gz
```

_foodcontrol_2:a032e43a6be1450f0281f10ed42f4b8c2e6afdf4033e6ea1d0f8721d7f746204_
_foodcontrol_3:65b3cc9144708b16913e623542a98c2e26e8998e2e637c65e53d2ea59b93f746_

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/users/Admin@org3.acme.com/msp/ CORE_PEER_ADDRESS=peer0.org3.acme.com:7051 CORE_PEER_LOCALMSPID="Org3MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/peers/peer0.org3.acme.com/tls/ca.crt peer lifecycle chaincode install ${CHAINCODE_NAME}.tar.gz
```

_foodcontrol_2:a032e43a6be1450f0281f10ed42f4b8c2e6afdf4033e6ea1d0f8721d7f746204_
_foodcontrol_3:65b3cc9144708b16913e623542a98c2e26e8998e2e637c65e53d2ea59b93f746_

_* Si revisamos el hash obtenido, comprobaremos que es el mismo que nos dio con la primera organización_

31. Indicamos que sólo la org 1 y 3 puedan aprobar (uso la secuencia 2, pero podría ser que usemos la 1 si estamos corriéndolo por primera vez)

```
peer lifecycle chaincode approveformyorg --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name $CHAINCODE_NAME --version $CHAINCODE_VERSION --sequence 2 --waitForEvent --signature-policy "OR ('Org1MSP.peer', 'Org3MSP.peer')" --package-id foodcontrol_3:65b3cc9144708b16913e623542a98c2e26e8998e2e637c65e53d2ea59b93f746
```

Si vemos que dice _committed with status (VALID) at_, significa que el proceso fue correcto

33. Hacemos lo mismo para la organización 3, pero con variables específicas

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/users/Admin@org3.acme.com/msp/ CORE_PEER_ADDRESS=peer0.org3.acme.com:7051 CORE_PEER_LOCALMSPID="Org3MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/peers/peer0.org3.acme.com/tls/ca.crt peer lifecycle chaincode approveformyorg --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name $CHAINCODE_NAME --version $CHAINCODE_VERSION --sequence 2 --waitForEvent --signature-policy "OR ('Org1MSP.peer', 'Org3MSP.peer')" --package-id foodcontrol_3:65b3cc9144708b16913e623542a98c2e26e8998e2e637c65e53d2ea59b93f746
```

33. Para verificar las políticas

```
peer lifecycle chaincode checkcommitreadiness --channelID $CHANNEL_NAME --name $CHAINCODE_NAME --version $CHAINCODE_VERSION --sequence 2 --signature-policy "OR ('Org1MSP.peer', 'Org3MSP.peer')" --output json
```

34. Hacer commit para el peer de la primera y tercera organización

```
peer lifecycle chaincode commit -o orderer.acme.com:7050 --tls --cafile $ORDERER_CA --peerAddresses peer0.org1.acme.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.acme.com/peers/peer0.org1.acme.com/tls/ca.crt --peerAddresses peer0.org3.acme.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/peers/peer0.org3.acme.com/tls/ca.crt --channelID $CHANNEL_NAME --name $CHAINCODE_NAME --version $CHAINCODE_VERSION --sequence 2 --signature-policy "or ('Org1MSP.peer', 'Org3MSP.peer')"
```

35. Vamos a probar llamando al método set

```
peer chaincode invoke -o orderer.acme.com:7050 --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME -n $CHAINCODE_NAME -c '{"Args":["Set","dig:3","ricardo","banana"]}'
```

36. Vamos a probar el mismo id, pero con un nombre distinto, debería también darnos éxito

```
peer chaincode invoke -o orderer.acme.com:7050 --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME -n $CHAINCODE_NAME -c '{"Args":["Set","dig:3","pedro","banana"]}'
```

37. Podemos revisar los cambios en la siguiente url: http://localhost:5985/_utils/

38. Vamos a obtener los datos guardados, esto lo hacemos con el método query de nuestro chaincode

```
peer chaincode query --channelID $CHANNEL_NAME -n $CHAINCODE_NAME -c '{"Args":["Query","dig:3"]}'
```

39. Ahora probaremos con la organización 2. Esto nos arrojará un error, y esto es debido a que no le dimos permisos, por lo cual, es correcto que no nos deje.

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/users/Admin@org2.acme.com/msp/ CORE_PEER_ADDRESS=peer0.org2.acme.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/peers/peer0.org2.acme.com/tls/ca.crt peer chaincode invoke -o orderer.acme.com:7050 --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME -n $CHAINCODE_NAME -c '{"Args":["Set","dig:4","carlos","banana"]}'
```

40. Ahora en lugar de probar con la organización 2, lo haremos con la organización 3, y en esta ocasión será exitoso, dado que esta organización si tiene los permisos correspondientes

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/users/Admin@org3.acme.com/msp/ CORE_PEER_ADDRESS=peer0.org3.acme.com:7051 CORE_PEER_LOCALMSPID="Org3MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/peers/peer0.org3.acme.com/tls/ca.crt peer chaincode invoke -o orderer.acme.com:7050 --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME -n $CHAINCODE_NAME -c '{"Args":["Set","dig:4","carlos","banana"]}'
```
