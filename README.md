Wiki: Setup Completo Ambiente IoT con MQTT, Node-RED e ThingsBoard
==================================================================

Indice
------

1.  Configurazione No-IP e DUC
2.  Configurazione Router
3.  Generazione Certificati SSL
4.  Setup Container Docker
5.  Configurazione Node-RED,MQTTX

1\. Configurazione No-IP e DUC
------------------------------

### 1.1 Creazione Account No-IP

1.  Vai su [www.noip.com](https://www.noip.com/)
2.  Registra un nuovo account gratuito
3.  Accedi al pannello di controllo

### 1.2 Creazione Hostname

1.  Nel dashboard No-IP, vai su "Dynamic DNS"
2.  Clicca "Add Hostname"
3.  Scegli un hostname (es: `tuoserver.ddns.net`)
4.  Seleziona "Free Domain"
5.  Clicca "Create Hostname"

### 1.3 Installazione DUC

1.  Scarica DUC da [download.noip.com](https://www.noip.com/download)
2.  Installa DUC.
3.  Configura DUC con le tue credenziali No-IP
4.  Avvia il servizio DUC

### 2.2 Port Forwarding

1.  Trova la sezione "Port Forwarding" o "Virtual Server"
2.  Aggiungi le seguenti regole:
    -   Porta 1883 (MQTT non-SSL)
    -   Porta 8883 (MQTT SSL)

    `Nome: MQTT Porta Esterna: 1883
    Porta Interna: 1883
    Protocollo: TCP
    IP Destinazione: [IP del tuo computer]

    Nome: MQTT-SSL
    Porta Esterna: 8883
    Porta Interna: 8883
    Protocollo: TCP
    IP Destinazione: [IP del tuo computer]`

3\. Generazione Certificati SSL
-------------------------------

### 3.1 Preparazione Directory
1. Cancellare tutto il contenuto della cartella certs ( poichè bisogna associare il nuovo hostname) 

### 3.2 Creazione File di Configurazione OpenSSL

1. Aprendo il terminale, posizionandosi nella cartella certs copiare il codice di seguito , andando debitamente a sostituire State, Locality , Organization e tuoserver.ddns.net
 
# Crea file openssl-san.cnf
cat > openssl-san.cnf << EOF
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
req_extensions     = req_ext
x509_extensions    = v3_ca
prompt            = no

[ req_distinguished_name ]
C  = IT
ST = State
L  = Locality
O  = Organization
CN = tuoserver.ddns.net

[ req_ext ]
subjectAltName = @alt_names

[ v3_ca ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = tuoserver.ddns.net
DNS.2 = localhost
EOF

### 3.3 Generazione Certificati

1. Sempre nel terminale e nella cartella certs copiare il seguente codice 
`# Genera CA  openssl genrsa -out certs/ca.key 2048  openssl req -x509 -new -nodes -key certs/ca.key -sha256 -days 1024 -out certs/ca.crt -subj "/CN=My CA"
`# Genera certificato server  openssl genrsa -out certs/server.key 2048  openssl req -new -key certs/server.key -out certs/server.csr -config openssl-san.cnf`
openssl x509 -req -in certs/server.csr -CA certs/ca.crt -CAkey certs/ca.key -CAcreateserial -out certs/server.crt -days 365 -sha256 -extfile openssl-san.cnf -extensions req_ext`

4\. Setup Container Docker
--------------------------
### 4.1 Avvio Container

1. nel terminale posizionandosi sulla cartella di mosquitto copiare il seguente codice : 
docker-compose up -d

5\. Node-red, MQTTX e Thingsboard
--------------------------

1.Una volta che il container è in fase di run , su Node-red nella connection inserire un nuovo server ( l'host name generato da no-ip ) , come porta 8883, seleziore TLS e fare l'upload dei certificati di sicurezza presi dalla cartella certs. 
2.impostare una nuova connessione su mqttx con gli stessi settaggi di node-red. 
