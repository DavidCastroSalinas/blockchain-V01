# Configuración de una red HyperLedger
## Código parte del proyecto: Red blockchain para libros eclesiales de la iglesia católica en Chile
### contacto: dcastros@gmail.com
### Equipo de trabajo {Carlos Araya;  [Nicolás Ormeño](https://github.com/normeno/simple-hyperledger)  ; [David Castro](https://github.com/DavidCastroSalinas/blockchain-V01) }

# 1. Instalación de pre-requisitos

###1.1. Dar permiso de ejecución al archivo de prerequisitos

```console
chmod +x install-prereq.sh

```

###1.2. Ejecutación de script con prerequisitos, o correr los comandos que existen dentro del script

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



# 2. Preparación material criptográfico

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
    - Name: DIOCESIS01 #Arquidiocesis de Santiago
      Domain: diocesis01.libroeclesial.info
      EnableNodeOUs: true
      Template:
        Count: 1
        SANS:
          - localhost
      Users:
        Count: 1

    - Name: DIOCESIS02 #Diocesis de Melipilla
      Domain: diocesis02.libroeclesial.info
      EnableNodeOUs: true
      Template:
        Count: 1
        SANS:
          - localhost
      Users:
        Count: 1

    - Name: DIOCESIS03 #Diocesis de Chillán
      Domain: diocesis03.libroeclesial.info
      EnableNodeOUs: true
      Template:
        Count: 1
        SANS:
          - localhost
      Users:
        Count: 1

# Definición de las demás diocesis
- Diocesis_Arica
- Diocesis_Iquique
- Arquidiocesis_Antofagasta
- Diocesis_Calama
- Diocesis_Copiapo
- ArquiDiocesis_La Serena
- Prelatura_Illapel
- Diocesis_San Felipe
- Diocesis_Valparaíso
-
- Diocesis_Melipilla
- Arquidiocesis_Santiago
- Obispado_Castrense
- Diocesis_SanBernardo
- Diocesis_Rancagua
- Diocesis_Talca
- Diocesis_Linares
- Diocesis_Chillan
- ArquiDiocesis_Concepción
-  
- Diocesis_SantaMaríadelosAngeles
- Diocesis_Temuco
- Diocesis_Villarrica
- Diocesis_Valdivia
- Diocesis_Osorno
- ArquiDiocesis_PuertoMontt
- Diocesis_Ancud
- Vicariato_Aysén
- Diocesis_Punta_Arenas
- Prelatura_Personal_Opus_Dei
```

2.2. Ejecutar Generar el material criptográfico de organizaciones y usuarios
*El comando **cryptogen** es parte de los elementos instalados en los prerequisitos, y al ejecutarla utilizará el archivo **crypto-config.yaml** para generar todo el material criptográfico necesario para nuestra red blockchain dentro de la carpeta /channel-artifacts*

```console
cryptogen generate --config=./crypto-config.yaml
```



# 3. Configuración de canales de comunicación de la red

3.1. Configurar de variables globales
*configuramos las variables globales del proyecto que seran utilizandas dentro den entorno bash de la consola*

```console
EXPORT CHANNEL_SYSTEM=system-channel
EXPORT CHANNEL_ID=marketplace
EXPORT RT_PROFILE=LibroEclesialChannel
EXPORT RT_PROFILE_ORDERER=LibroEclesialOrdererGenesis
```


3.2 Creación del bloque géneris
*Este archivo corresponde al primer bloque de la redblockchain*
```console
configtxgen \
  -profile $RT_PROFILE_ORDERER \
  -channelID system-channel \
  -outputBlock ./channel-artifacts/genesis.block
```

3.3 Creación del canal de comunicaciones
*Ahora generamos fichero que para el canal de comunicaciones de la red interna*
```console
configtxgen \
  -profile $RT_PROFILE \
  -channelID $CHANNEL_ID \
  -outputCreateChannelTx ./channel-artifacts/channel.tx
```

3.4 Configuración de los AnchorsPeer
*La red estará conformada por cada unas de las organizaciones, que en este caso serán organizadas por cada una de las diócesis de Chile. (en este archivo sólo se mostrará la gestión con 3 organizaciones ya que los demás sólo serán repetidos y el podrán ser construidas en la personalización del proyecto)*
```console
configtxgen \
  -profile $RT_PROFILE \
  -channelID $CHANNEL_ID \
  -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx \
  -asOrg Org1MSP
```

```console
configtxgen \
    -profile ThreeOrgsChannel \
    -channelID $CHANNEL_ID \
    -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx  \
    -asOrg Org2MSP
```

```console
configtxgen \
    -profile ThreeOrgsChannel  \
    -channelID $CHANNEL_ID  \
    -outputAnchorPeersUpdate ./channel-artifacts/Org3MSPanchors.tx \
    -asOrg Org3MSP 
```


# 4. Configuración de Imagenes en Docker-Compose

4.1. Configurar archivo **./base/peer-base.yaml**  
*configuramos las variables globales del proyecto que seran utilizandas dentro den entorno bash de la consola*

```yaml
version: '2'
services:
  peer-base:
    image: hyperledger/fabric-peer:2.2.0
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      # the following setting starts chaincode containers on the same
      # bridge network as the peers
      # https://docs.docker.com/compose/networking/
      # ---CHANGED---
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=libroeclesial-network_basic
      - FABRIC_LOGGING_SPEC=INFO
     # - FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start

```


4.2. Configurar archivo **Docker-compose-base.yaml**  
*configuramos las variables globales del proyecto que seran utilizandas dentro den entorno bash de la consola*

```yaml

version: '2'

services:

  # ---CHANGED--- The orderer name is taken from the name generated by the "cryptogen" certs – it indicates the orderer orgs one and only orderer
  orderer.libroeclesial.info:
    # ---CHANGED--- The container name is a copy of the orderer name
    container_name: orderer.libroeclesial.info
    image: hyperledger/fabric-orderer:2.2.0
    environment:
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
      # enabled TLS
      - ORDERER_GENERAL_TLS_ENABLED=true
      - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: orderer
    volumes:
    - ../channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
    # ---CHANGED--- the path is different to reflect our company's domain
    - ../crypto-config/ordererOrganizations/libroeclesial.info/orderers/orderer.libroeclesial.info/msp:/var/hyperledger/orderer/msp
    # ---CHANGED--- the path is different to reflect our company's domain
    - ../crypto-config/ordererOrganizations/libroeclesial.info/orderers/orderer.libroeclesial.info/tls/:/var/hyperledger/orderer/tls
    ports:
      - 7050:7050

  # ---CHANGED--- The peer name is taken from the name generated by the "cryptogen" certs – it indicates the peer org 1 and one peer "peer0"
  peer0.main.libroeclesial.info:
    # ---CHANGED--- Container name – same as the peer name
    container_name: peer0.main.libroeclesial.info
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_ID=peer0.main.libroeclesial.info
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_ADDRESS=peer0.main.libroeclesial.info:7051
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.main.libroeclesial.info:7051
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.main.libroeclesial.info:7051      
      - CORE_PEER_LOCALMSPID=Org0MSP
    volumes:
        - /var/run/:/host/var/run/
        # ---CHANGED--- changed to reflect peer name, org name and our company's domain
        - ../crypto-config/peerOrganizations/main.libroeclesial.info/peers/peer0.main.libroeclesial.info/msp:/etc/hyperledger/fabric/msp
        # ---CHANGED--- changed to reflect peer name, org name and our company's domain
        - ../crypto-config/peerOrganizations/main.libroeclesial.info/peers/peer0.main.libroeclesial.info/tls:/etc/hyperledger/fabric/tls
    ports:
      - 7051:7051
      - 7053:7053



  # ---CHANGED--- The peer name is taken from the name generated by the "cryptogen" certs – it indicates the peer org 1 and one peer "peer0"
  peer0.p001.libroeclesial.info:
    # ---CHANGED--- Container name – same as the peer name
    container_name: peer0.p001.libroeclesial.info
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_ID=peer0.p001.libroeclesial.info
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_ADDRESS=peer0.p001.libroeclesial.info:7051
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.p001.libroeclesial.info:7051
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.p001.libroeclesial.info:7051      
      - CORE_PEER_LOCALMSPID=Org1MSP
    volumes:
        - /var/run/:/host/var/run/
        # ---CHANGED--- changed to reflect peer name, org name and our company's domain
        - ../crypto-config/peerOrganizations/p001.libroeclesial.info/peers/peer0.p001.libroeclesial.info/msp:/etc/hyperledger/fabric/msp
        # ---CHANGED--- changed to reflect peer name, org name and our company's domain
        - ../crypto-config/peerOrganizations/p001.libroeclesial.info/peers/peer0.p001.libroeclesial.info/tls:/etc/hyperledger/fabric/tls
    ports:
      - 8051:7051
      - 8053:7053

  # ---CHANGED--- The peer name is taken from the name generated by the "cryptogen" certs – it indicates the peer org 2 and one peer "peer0"
  peer0.p002.libroeclesial.info:
    # ---CHANGED--- Container name – same as the peer name
    container_name: peer0.p002.libroeclesial.info
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_ID=peer0.p002.libroeclesial.info
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_ADDRESS=peer0.p002.libroeclesial.info:7051
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.p002.libroeclesial.info:7051
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.p002.libroeclesial.info:7051
      # ---CHANGED--- ensure that the MSP ID is correctly set of Org2
      - CORE_PEER_LOCALMSPID=Org2MSP
    volumes:
        - /var/run/:/host/var/run/
        # ---CHANGED--- changed to reflect peer name, org name and our company's domain
        - ../crypto-config/peerOrganizations/p002.libroeclesial.info/peers/peer0.p002.libroeclesial.info/msp:/etc/hyperledger/fabric/msp
        # ---CHANGED--- changed to reflect peer name, org name and our company's domain
        - ../crypto-config/peerOrganizations/p002.libroeclesial.info/peers/peer0.p002.libroeclesial.info/tls:/etc/hyperledger/fabric/tls

    ports:
      - 9051:7051
      - 9053:7053

  # ---CHANGED--- The peer name is taken from the name generated by the "cryptogen" certs – it indicates the peer org 3 and one peer "peer0"
  peer0.p003.libroeclesial.info:
    # ---CHANGED--- Container name – same as the peer name
    container_name: peer0.p003.libroeclesial.info
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_ID=peer0.p003.libroeclesial.info
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_ADDRESS=peer0.p003.libroeclesial.info:7051
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.p003.libroeclesial.info:7051
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.p003.libroeclesial.info:7051
      # ---CHANGED--- ensure that the MSP ID is correctly set of Org3
      - CORE_PEER_LOCALMSPID=Org3MSP
    volumes:
        - /var/run/:/host/var/run/
        # ---CHANGED--- changed to reflect peer name, org name and our company's domain
        - ../crypto-config/peerOrganizations/p003.libroeclesial.info/peers/peer0.p003.libroeclesial.info/msp:/etc/hyperledger/fabric/msp
        # ---CHANGED--- changed to reflect peer name, org name and our company's domain
        - ../crypto-config/peerOrganizations/p003.libroeclesial.info/peers/peer0.p003.libroeclesial.info/tls:/etc/hyperledger/fabric/tls
    ports:
      - 10051:7051
      - 10053:7053

  # ---CHANGED--- The peer name is taken from the name generated by the "cryptogen" certs – it indicates the peer org 3 and one peer "peer0"
  peer0.p004.libroeclesial.info:
    # ---CHANGED--- Container name – same as the peer name
    container_name: peer0.p004.libroeclesial.info
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_ID=peer0.p004.libroeclesial.info
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_ADDRESS=peer0.p004.libroeclesial.info:7051
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.p004.libroeclesial.info:7051
      # ---CHANGED--- changed to reflect peer name, org name and our company's domain
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.p004.libroeclesial.info:7051
      # ---CHANGED--- ensure that the MSP ID is correctly set of Org3
      - CORE_PEER_LOCALMSPID=Org4MSP
    volumes:
        - /var/run/:/host/var/run/
        # ---CHANGED--- changed to reflect peer name, org name and our company's domain
        - ../crypto-config/peerOrganizations/p004.libroeclesial.info/peers/peer0.p004.libroeclesial.info/msp:/etc/hyperledger/fabric/msp
        # ---CHANGED--- changed to reflect peer name, org name and our company's domain
        - ../crypto-config/peerOrganizations/p004.libroeclesial.info/peers/peer0.p004.libroeclesial.info/tls:/etc/hyperledger/fabric/tls
    ports:
      - 11051:7051
      - 11053:7053

```



4.3. Configurar archivo **docker-compose-cli-couchdb.yaml**  
*configuramos las variables globales del proyecto que seran utilizandas dentro den entorno bash de la consola*

```yaml
version: '2'

# ---CHANGED--- our network is called "basic"
networks:
  basic:
services:
  couchdb0:
    image: couchdb:3.1
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=adminpw
    ports: 
      - 5984:5984
    container_name: couchdb0
    networks:
      - basic

  couchdb1:
    image: couchdb:3.1
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=adminpw
    ports: 
      - 5985:5984
    container_name: couchdb1
    networks:
      - basic

  couchdb2:
    image: couchdb:3.1
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=adminpw
    ports: 
      - 5986:5984
    container_name: couchdb2
    networks:
      - basic

  couchdb3:
    image: couchdb:3.1
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=adminpw
    ports: 
      - 5987:5984
    container_name: couchdb3
    networks:
      - basic      

  couchdb4:
    image: couchdb:3.1
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=adminpw
    ports: 
      - 5988:5984
    container_name: couchdb4
    networks:
      - basic    



  # ---CHANGED--- The orderer name is taken from the name generated by the "cryptogen" certs – it indicates the orderer orgs one and only orderer
  orderer.libroeclesial.info:
    extends:
      file:   base/docker-compose-base.yaml
      # ---CHANGED--- refers to orderer name
      service: orderer.libroeclesial.info
    # ---CHANGED--- The container name is a copy of the orderer name
    container_name: orderer.libroeclesial.info
    networks:
      - basic

 # ---CHANGED--- The peer name is taken from the name generated by the "cryptogen" certs – it indicates the peer org 1 and one peer "peer0"
  peer0.main.libroeclesial.info:
    container_name: peer0.main.libroeclesial.info
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.main.libroeclesial.info
    environment:
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0:5984
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=adminpw
    depends_on:
      - orderer.libroeclesial.info
      - couchdb0
    networks:
      # ---CHANGED--- our network is called "basic"
      - basic

  # ---CHANGED--- The peer name is taken from the name generated by the "cryptogen" certs – it indicates the peer org 1 and one peer "peer0"
  peer0.p001.libroeclesial.info:
    # ---CHANGED--- Container name – same as the peer name
    container_name: peer0.p001.libroeclesial.info
    extends:
      file:  base/docker-compose-base.yaml
      # ---CHANGED--- Refers to peer name
      service: peer0.p001.libroeclesial.info
    environment:
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1:5985
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=adminpw
    depends_on:
      - orderer.libroeclesial.info
      - couchdb1
    networks:
      # ---CHANGED--- our network is called "basic"
      - basic

  # ---CHANGED--- The peer name is taken from the name generated by the "cryptogen" certs – it indicates the peer org 2 and one peer "peer0"
  peer0.p002.libroeclesial.info:
    # ---CHANGED--- Container name – same as the peer name
    container_name: peer0.p002.libroeclesial.info
    extends:
      file:  base/docker-compose-base.yaml
      # ---CHANGED--- Refers to peer name
      service: peer0.p002.libroeclesial.info
    environment:
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb2:5986
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=adminpw
    depends_on:
      - orderer.libroeclesial.info
      - couchdb2
    networks:
      # ---CHANGED--- our network is called "basic"
      - basic

  # ---CHANGED--- The peer name is taken from the name generated by the "cryptogen" certs – it indicates the peer org 3 and one peer "peer0"
  peer0.p003.libroeclesial.info:
    # ---CHANGED--- Container name – same as the peer name
    container_name: peer0.p003.libroeclesial.info
    extends:
      file:  base/docker-compose-base.yaml
      # ---CHANGED--- Refers to peer name
      service: peer0.p003.libroeclesial.info
    environment:
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb3:5987
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=adminpw
    depends_on:
      - orderer.libroeclesial.info
      - couchdb3
    networks:
      # ---CHANGED--- our network is called "basic"
      - basic

  # ---CHANGED--- The peer name is taken from the name generated by the "cryptogen" certs – it indicates the peer org 3 and one peer "peer0"
  peer0.p004.libroeclesial.info:
    # ---CHANGED--- Container name – same as the peer name
    container_name: peer0.p004.libroeclesial.info
    extends:
      file:  base/docker-compose-base.yaml
      # ---CHANGED--- Refers to peer name
      service: peer0.p004.libroeclesial.info
    environment:
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb4:5988
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=adminpw
    depends_on:
      - orderer.libroeclesial.info
      - couchdb4
    networks:
      # ---CHANGED--- our network is called "basic"
      - basic
      
  #CA for Org0	
  ca.main.libroeclesial.info:
    image: hyperledger/fabric-ca:1.4.8
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca.main.libroeclesial.info
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.main.libroeclesial.info-cert.pem
      - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/priv_sk
      - FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.main.libroeclesial.info-cert.pem
      - FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/priv_sk
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start -b admin:adminpw'
    volumes:
      - ./crypto-config/peerOrganizations/main.libroeclesial.info/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.main.libroeclesial.info
    networks:
      - basic

  cli:
    container_name: cli
    image: hyperledger/fabric-tools:2.2
    tty: true
    stdin_open: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_ID=cli
      # ---CHANGED--- peer0 from Org1 is the default for this CLI container
      - CORE_PEER_ADDRESS=peer0.main.libroeclesial.info:7051
      - CORE_PEER_LOCALMSPID=Org0MSP
      - CORE_PEER_TLS_ENABLED=true
      # ---CHANGED--- changed to reflect peer0 name, org1 name and our company's domain
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/main.libroeclesial.info/peers/peer0.main.libroeclesial.info/tls/server.crt
      # ---CHANGED--- changed to reflect peer0 name, org1 name and our company's domain
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/main.libroeclesial.info/peers/peer0.main.libroeclesial.info/tls/server.key
      # ---CHANGED--- changed to reflect peer0 name, org1 name and our company's domain
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/main.libroeclesial.info/peers/peer0.main.libroeclesial.info/tls/ca.crt
      # ---CHANGED--- changed to reflect peer0 name, org1 name and our company's domain
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/main.libroeclesial.info/users/Admin@main.libroeclesial.info/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    # ---CHANGED--- command needs to be connected out as we will be issuing commands explicitly, not using by any script
    # command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME}; sleep $TIMEOUT'
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        # ---CHANGED--- chaincode path adjusted
        - ./../chaincode/:/opt/gopath/src/github.com/chaincode
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
       # ---CHANGED--- reference to our orderer
      - orderer.libroeclesial.info
       # ---CHANGED--- reference to peer0 of Org1
      - peer0.main.libroeclesial.info
       # ---CHANGED--- reference to peer0 of Org1
      - peer0.p001.libroeclesial.info
       # ---CHANGED--- reference to peer0 of Org2
      - peer0.p002.libroeclesial.info
       # ---CHANGED--- reference to peer0 of Org3
      - peer0.p003.libroeclesial.info
       # ---CHANGED--- reference to peer0 of Org4
      - peer0.p004.libroeclesial.info
    networks:
      # ---CHANGED--- our network is called "basic"
      - basicñ
```


### 4.4. Levantar imagenes docker configuradas: Configuración de variables de entorno
```console
export CHANNEL_NAME=marketplace
export FABRIC_CFG_PATH=$WPD 
export VERBOSE=false
```

### 4.5. Levantar imagenes docker configuradas: Iniciar Servicios

```console
CHANNEL_NAME=$CHANNEL_NAME docker-compose -f docker-compose-cli-couchdb.yaml up -d
```


# 5. Entorno de Administración de Peers
### 5.1. Ingresamos al bash del entorno *CLI* 

```console
docker exec -it cli bash
```

### 5.2. [CLI] Configuración de las variables del entorno *CLI*

```console
export CHANNEL_NAME=marketplace
```

### 5.3. [CLI] Creamos el canal

```console
peer channel create -o orderer.acme.com:7050 \
                    -c $CHANNEL_NAME crypto-config.yaml
                    -f ./channel-artifacts/channel.tx \
                    --tls true \
                    --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/acme.com/orderers/orderer.acme.com/msp/tlscacerts/tlsca.acme.com-cert.pem
```

### 5.4. [CLI] Integrar a la primera organización al canal 

```console
peer channel join -b marketplace.block
```

### 5.5. [CLI] Integrar a la segunda organización al canal
```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/users/Admin@org2.acme.com/msp/ CORE_PEER_ADDRESS=peer0.org2.acme.com:7051 \
    CORE_PEER_LOCALMSPID="Org2MSP" \
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/peers/peer0.org2.acme.com/tls/ca.crt \
    peer channel join \
    -b marketplace.block
```

###  5.6. [CLI] Integrar a la tercera organización al canal (se deberá integrar a todas las organizaciones existentes)
```console
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/users/Admin@org3.acme.com/msp/ CORE_PEER_ADDRESS=peer0.org3.acme.com:7051 \
    CORE_PEER_LOCALMSPID="Org3MSP" \
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/peers/peer0.org3.acme.com/tls/ca.crt \
    peer channel join \
    -b marketplace.block
```

### 5.7. Ahora debemos configurar el anchor peer para cada organización
### 5.7.1 Organización 1
```console
peer channel update \
  -o orderer.acme.com:7050 \
  -c $CHANNEL_NAME \
  -f ./channel-artifacts/Org1MSPanchors.tx \
  --tls \
  --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/acme.com/orderers/orderer.acme.com/msp/tlscacerts/tlsca.acme.com-cert.pem
```

### 5.7.2 Organización 2
```console
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/users/Admin@org2.acme.com/msp/ \
  CORE_PEER_ADDRESS=peer0.org2.acme.com:7051 \
  CORE_PEER_LOCALMSPID="Org2MSP" \ 
  CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/peers/peer0.org2.acme.com/tls/ca.crt \ 
  peer channel update \
  -o orderer.acme.com:7050 \
  -c $CHANNEL_NAME \
  -f ./channel-artifacts/Org2MSPanchors.tx \
  --tls \
  --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/acme.com/orderers/orderer.acme.com/msp/tlscacerts/tlsca.acme.com-cert.pem
```

### 5.7.3 Organización 3
```console
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/users/Admin@org3.acme.com/msp/ \
  CORE_PEER_ADDRESS=peer0.org3.acme.com:7051 \
  CORE_PEER_LOCALMSPID="Org3MSP" \
  CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/peers/peer0.org3.acme.com/tls/ca.crt \
  peer channel update \
  -o orderer.acme.com:7050  \
  -c $CHANNEL_NAME  \
  -f ./channel-artifacts/Org3MSPanchors.tx  \
  --tls  \
  --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/acme.com/orderers/orderer.acme.com/msp/tlscacerts/tlsca.acme.com-cert.pem
```
### 5.7.4 Volvemos al entorno de bash anterior
```console
  exit
```


# 6 Creación la carpeta chaincode (Smart Constract)

### 6.1. Crear el archivo chaincode/foodcontrol/foodcontrol.go

### 6.2. Crear el archivo chaincode/foodcontrol/go.mod

### 6.3. Ingresar al contanedor cli

```console
docker exec -it cli bash
```

24. Dentro del contenedor verificamos que exista la ruta `/opt/gopath/src/github.com/chaincode/`, si no existe, es probable que sólo reiniciando el contenedor, tome la ruta

25. Dentro del contenedor, nos aseguramos de tener las siguientes variables

```console
export CHANNEL_NAME=marketplace
export CHAINCODE_NAME=foodcontrol
export CC_RUNTIME_LANGUAGE=golang
export CC_SRC_PATH="../../../chaincode/$CHAINCODE_NAME/"
export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/acme.com/orderers/orderer.acme.com/msp/tlscacerts/tlsca.acme.com-cert.pem
export CHAINCODE_VERSION=3
export CHAINCODE_SEQUENCE=4
```

26. Hacer uso del ciclo de vida de chaincode, para empaquetar el proyecto

```console
peer lifecycle chaincode package ${CHAINCODE_NAME}.tar.gz \
      --path ${CC_SRC_PATH}  \
      --lang ${CC_RUNTIME_LANGUAGE}  \
      --label ${CHAINCODE_NAME}_${CHAINCODE_VERSION}  \
      >& log.txt
```

Esto debería generar algo como lo siguiente:

```console
peer lifecycle chaincode package foodcontrol.tar.gz  \
      --path ../../../chaincode/$CHAINCODE_NAME/  \
      --lang ${CC_RUNTIME_LANGUAGE}  \
      --label ${CHAINCODE_NAME}_${CHAINCODE_VERSION}  \
      >& log.txt
```

27. Revisamos que se cree correctamente, esto lo hacemos, `ls -l` para revisar que exista el archivo _foodcontrol.tar.gz_

- Aquí se nos puede generar el archivo go.sum (si es que no lo teníamos con anterioridad)

28. En la misma máquina del CLI, vamos a instalar el chaincode en las organizaciones

```console
peer lifecycle chaincode install foodcontrol.tar.gz
```

29. Buscamos el identificador, que es el último mensaje, donde dice "Chaincode code package identifier", un ejemplo de identificador es:

_foodcontrol_1:d3511338dbe0363894823b9c0f5f098245af222a407e7dafdf78a8c53c1d2146_
_foodcontrol_2:a032e43a6be1450f0281f10ed42f4b8c2e6afdf4033e6ea1d0f8721d7f746204_
_foodcontrol_3:65b3cc9144708b16913e623542a98c2e26e8998e2e637c65e53d2ea59b93f746_

_* Esto es por que al instalarlo en otras organizaciones, nos debe dar el mismo identificador que el obtenido, y así nos aseguramente que todas las organizaciones instalen el paquete, y tengan el mismo paquete instalado_

30. Para instalarlo en otras organizaciones, debemos hacer lo siguiente:

```console
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/users/Admin@org2.acme.com/msp/  \
  CORE_PEER_ADDRESS=peer0.org2.acme.com:7051  \
  CORE_PEER_LOCALMSPID="Org2MSP"  \
  CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/peers/peer0.org2.acme.com/tls/ca.crt  \
  peer lifecycle chaincode install ${CHAINCODE_NAME}.tar.gz
```

_foodcontrol_2:a032e43a6be1450f0281f10ed42f4b8c2e6afdf4033e6ea1d0f8721d7f746204_
_foodcontrol_3:65b3cc9144708b16913e623542a98c2e26e8998e2e637c65e53d2ea59b93f746_

```console
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/users/Admin@org3.acme.com/msp/  \
  CORE_PEER_ADDRESS=peer0.org3.acme.com:7051  \
  CORE_PEER_LOCALMSPID="Org3MSP"  \
  CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/peers/peer0.org3.acme.com/tls/ca.crt  \
  peer lifecycle chaincode install ${CHAINCODE_NAME}.tar.gz
```

_foodcontrol_2:a032e43a6be1450f0281f10ed42f4b8c2e6afdf4033e6ea1d0f8721d7f746204_
_foodcontrol_3:65b3cc9144708b16913e623542a98c2e26e8998e2e637c65e53d2ea59b93f746_

_* Si revisamos el hash obtenido, comprobaremos que es el mismo que nos dio con la primera organización_

31. Indicamos que sólo la org 1 y 3 puedan aprobar (uso la secuencia 2, pero podría ser que usemos la 1 si estamos corriéndolo por primera vez)

```console
peer lifecycle chaincode approveformyorg  \
    --tls  \
    --cafile $ORDERER_CA  \
    --channelID $CHANNEL_NAME  \
    --name $CHAINCODE_NAME  \
    --version $CHAINCODE_VERSION  \
    --sequence 2  \
    --waitForEvent  \
    --signature-policy  \
    "OR ('Org1MSP.peer', 'Org3MSP.peer')"  \
    --package-id foodcontrol_3:65b3cc9144708b16913e623542a98c2e26e8998e2e637c65e53d2ea59b93f746
```

Si vemos que dice _committed with status (VALID) at_, significa que el proceso fue correcto

33. Hacemos lo mismo para la organización 3, pero con variables específicas

```console
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/users/Admin@org3.acme.com/msp/  \
  CORE_PEER_ADDRESS=peer0.org3.acme.com:7051  \
  CORE_PEER_LOCALMSPID="Org3MSP"  \
  CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/peers/peer0.org3.acme.com/tls/ca.crt  \
  peer lifecycle chaincode approveformyorg  \
    --tls  \
    --cafile $ORDERER_CA  \
    --channelID $CHANNEL_NAME  \
    --name $CHAINCODE_NAME  \
    --version $CHAINCODE_VERSION  \
    --sequence 2  \
    --waitForEvent  \
    --signature-policy "OR ('Org1MSP.peer', 'Org3MSP.peer')"  \
    --package-id foodcontrol_3:65b3cc9144708b16913e623542a98c2e26e8998e2e637c65e53d2ea59b93f746
```

33. Para verificar las políticas

```console
peer lifecycle chaincode checkcommitreadiness  \
  --channelID $CHANNEL_NAME  \
  --name $CHAINCODE_NAME  \
  --version $CHAINCODE_VERSION  \
  --sequence 2  \
  --signature-policy "OR ('Org1MSP.peer', 'Org3MSP.peer')" \
   --output json
```

34. Hacer commit para el peer de la primera y tercera organización

```console
peer lifecycle chaincode commit  \
  -o orderer.acme.com:7050  \
  --tls  \
  --cafile $ORDERER_CA  \
  --peerAddresses peer0.org1.acme.com:7051  \
  --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.acme.com/peers/peer0.org1.acme.com/tls/ca.crt  \
  --peerAddresses peer0.org3.acme.com:7051  \
  --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/peers/peer0.org3.acme.com/tls/ca.crt  \
  --channelID $CHANNEL_NAME  \
  --name $CHAINCODE_NAME  \
  --version $CHAINCODE_VERSION  \
  --sequence 2  \
  --signature-policy "or ('Org1MSP.peer', 'Org3MSP.peer')"
```

35. Vamos a probar llamando al método set

```console
peer chaincode invoke  \
  -o orderer.acme.com:7050  \
  --tls  \
  --cafile $ORDERER_CA  \
  --channelID $CHANNEL_NAME  \
  -n $CHAINCODE_NAME  \
  -c '{"Args":["Set","dig:3","ricardo","banana"]}'
```

36. Vamos a probar el mismo id, pero con un nombre distinto, debería también darnos éxito

```console
peer chaincode invoke -o  \
  orderer.acme.com:7050  \
  --tls  \
  --cafile $ORDERER_CA  \
  --channelID $CHANNEL_NAME  \
  -n $CHAINCODE_NAME  \
  -c '{"Args":["Set","dig:3","pedro","banana"]}'
```

37. Podemos revisar los cambios en la siguiente url: http://localhost:5985/_utils/

38. Vamos a obtener los datos guardados, esto lo hacemos con el método query de nuestro chaincode

```console
peer chaincode query  \
  --channelID $CHANNEL_NAME  \
  -n $CHAINCODE_NAME  \
  -c '{"Args":["Query","dig:3"]}'
```

39. Ahora probaremos con la organización 2. Esto nos arrojará un error, y esto es debido a que no le dimos permisos, por lo cual, es correcto que no nos deje.

```console
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/users/Admin@org2.acme.com/msp/  \
  CORE_PEER_ADDRESS=peer0.org2.acme.com:7051  \
  CORE_PEER_LOCALMSPID="Org2MSP"  \
  CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.acme.com/peers/peer0.org2.acme.com/tls/ca.crt  \
    peer chaincode invoke  \
      -o orderer.acme.com:7050  \
      --tls  \
      --cafile $ORDERER_CA  \
      --channelID $CHANNEL_NAME  \
      -n $CHAINCODE_NAME  \
      -c '{"Args":["Set","dig:4","carlos","banana"]}'
```

40. Ahora en lugar de probar con la organización 2, lo haremos con la organización 3, y en esta ocasión será exitoso, dado que esta organización si tiene los permisos correspondientes

```console
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/users/Admin@org3.acme.com/msp/  \
  CORE_PEER_ADDRESS=peer0.org3.acme.com:7051  \
  CORE_PEER_LOCALMSPID="Org3MSP"  \
  CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.acme.com/peers/peer0.org3.acme.com/tls/ca.crt \
    peer chaincode invoke 
      -o orderer.acme.com:7050  \
       --tls  \
       --cafile $ORDERER_CA  \
       --channelID $CHANNEL_NAME  \
       -n $CHAINCODE_NAME  \
       -c '{"Args":["Set","dig:4","carlos","banana"]}'
```
