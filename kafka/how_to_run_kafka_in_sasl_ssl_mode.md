# How to run kafka in SASL_SSL Mode?

**The authentication is different** between **SASL_SSL** mode and **SSL**  mode, the **SASL_SSL** will use its own way to authenticate like  **user/password**, **oauthbearer** etc. while the **SSL** mode will use the **keystore** (holding the client's **private key**, client's **certificate signed by CA**) to authenticate. For SSL mode please refer to [How to run kafka in SSL Mode](ttps://github.com/leiwang008/documents/blob/main/kafka/how_to_run_kafka_in_ssl_mode.md)

1. Generate the '**keystore**' and '**truststore**' on your kafka broker
- Copy the following script into a file like '**setup_ssl_broker.sh**'

```bash
#!/bin/bash
################################## setup environments ##############################
export MSYS_NO_PATHCONV=1 #avoid the git bash adds "C:\Program Files\Git\" in front of slash /
export CERT_OUTPUT_PATH="." # path holding certificates and stores
export PASSWORD=yourpass
export STORE_TYPE=pkcs12 # Public Key Crypto Standard 12, by default it is 'jks' type before JDK 9.
export KEY_STORE="$CERT_OUTPUT_PATH/kafka.keystore" # Kafka keystore file, holding the private key, signed-certificate and CA-certificate
export TRUST_STORE="$CERT_OUTPUT_PATH/kafka.truststore" # Kafka truststore file, holding CA certificate and public key, used by client to verify broker
export KEY_PASSWORD=$PASSWORD # keystore key password
export STORE_PASSWORD=$PASSWORD # keystore store password
export TRUST_KEY_PASSWORD=$PASSWORD # truststore key password
export TRUST_STORE_PASSWORD=$PASSWORD # truststore store password
export CA_CONFIG="$CERT_OUTPUT_PATH/openssl-ca.cnf" #CA config file
export CERT_KEY_FILE="$CERT_OUTPUT_PATH/ca-key" # CA key, private key to sign a cert or to decrypt
export CERT_AUTH_FILE="$CERT_OUTPUT_PATH/ca-cert.pem" # CA certificate, holding the public key to encrypt, and the signature to sign a cert or to verify a broker by client
export CLUSTER_NAME=localhost # alias for broker, it should be "localhost" or broker's FQDN
export CLUSTER_CERT_FILE="$CERT_OUTPUT_PATH/${CLUSTER_NAME}-cert.pem" # broker's certificate file 
export CLUSTER_CSR_FILE="${CLUSTER_CERT_FILE}.csr" # the CSR (certificate signing request) to get broker's certificate
export DAYS_VALID=365 # key valid time in days
export D_NAME="CN=${CLUSTER_NAME}, OU=game, O=org, L=beijing, ST=beijing, C=cn" # distinguished name
#############################################################################
 
echo "1. create keystore with the private key"
keytool -keystore $KEY_STORE -storetype $STORE_TYPE -alias $CLUSTER_NAME -validity $DAYS_VALID -genkey -keyalg RSA -storepass $STORE_PASSWORD -keypass $KEY_PASSWORD -dname "$D_NAME"

echo "2. generate your CA (Certificate Authority), certificate and public key "
openssl req -new -x509  -config $CA_CONFIG -keyout $CERT_KEY_FILE -out "$CERT_AUTH_FILE" -days "$DAYS_VALID" -passin pass:"$PASSWORD" -passout pass:"$PASSWORD"

echo "3. import CA into client truststore will be used by client to verify server"
keytool -keystore "$TRUST_STORE" -storetype $STORE_TYPE -alias CARoot -validity $DAYS_VALID -import -file "$CERT_AUTH_FILE" -storepass "$TRUST_STORE_PASSWORD" -keypass "$TRUST_KEY_PASSWORD" -noprompt

echo "4. generate certificate-signing-requests (CSR) from keystore ......"
keytool -keystore "$KEY_STORE" -storetype $STORE_TYPE -alias "$CLUSTER_NAME" -certreq -file "$CLUSTER_CSR_FILE" -storepass "$STORE_PASSWORD" -keypass "$KEY_PASSWORD" -noprompt

echo "5. sign the CSR by CA(certificate and key) and get the signedcertificate "
openssl x509 -req -CA "$CERT_AUTH_FILE" -CAkey $CERT_KEY_FILE -in "$CLUSTER_CSR_FILE" -out "${CLUSTER_CERT_FILE}" -days "$DAYS_VALID" -CAcreateserial -passin pass:"$PASSWORD"

echo "6. import CA into keystore "
keytool -keystore "$KEY_STORE" -storetype $STORE_TYPE -alias CARoot -import -file "$CERT_AUTH_FILE" -storepass "$STORE_PASSWORD" -keypass "$KEY_PASSWORD" -noprompt
 
echo "7. import signed certificate into keystore......"
keytool -keystore "$KEY_STORE" -storetype $STORE_TYPE -alias "${CLUSTER_NAME}" -import -file "${CLUSTER_CERT_FILE}" -storepass "$STORE_PASSWORD" -keypass "$KEY_PASSWORD" -noprompt
```

- Copy the following CA Config file into the file '**openssl-ca.cnf**' in the same folder as the above script

```bash
HOME            = .
RANDFILE        = $ENV::HOME/.rnd

####################################################################
[ ca ]
default_ca    = CA_default      # The default ca section

[ CA_default ]

base_dir      = .
certificate   = $base_dir/cacert.pem   # The CA certificate
private_key   = $base_dir/cakey.pem    # The CA private key
new_certs_dir = $base_dir              # Location for new certs after signing
database      = $base_dir/index.txt    # Database index file
serial        = $base_dir/serial.txt   # The current serial number

default_days     = 1000         # How long to certify for
default_crl_days = 30           # How long before next CRL
default_md       = sha256       # Use public key default MD
preserve         = no           # Keep passed DN ordering

x509_extensions = ca_extensions # The extensions to add to the cert

email_in_dn     = no            # Don't concat the email in the DN
copy_extensions = copy          # Required to copy SANs from CSR to cert

####################################################################
[ req ]
default_bits       = 4096
default_keyfile    = cakey.pem
distinguished_name = ca_distinguished_name
x509_extensions    = ca_extensions
string_mask        = utf8only
prompt             = no

####################################################################
[ ca_distinguished_name ]
countryName         = cn
#countryName_default = cn

stateOrProvinceName         = beijing
#stateOrProvinceName_default = beijing

localityName                = beijing
#localityName_default        = beijing

organizationName            = org
#organizationName_default    = org

organizationalUnitName         = game
#organizationalUnitName_default = game

commonName         = localhost
#commonName_default = localhost

emailAddress         = rabbit@org
#emailAddress_default = rabbit@org

####################################################################
[ ca_extensions ]

subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always, issuer
basicConstraints       = critical, CA:true
keyUsage               = keyCertSign, cRLSign

####################################################################
[ signing_policy ]
countryName            = optional
stateOrProvinceName    = optional
localityName           = optional
organizationName       = optional
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional

####################################################################
[ signing_req ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints       = CA:FALSE
keyUsage               = digitalSignature, keyEncipherment
```

- Then run the script in the Git Bash shell

```bash
.\setup_ssl_broker.sh
```

- Then you will have the '**kafka.keystore**' and '**kafka.truststore**' in this folder. 
2. Now let us config the kafka server properties file as below, now you config the kafka in SASL_SSL mode on port 9093 and PLAINTEXT mode on port 9092

```conf
listeners=PLAINTEXT://localhost:9092, SASL_SSL://localhost:9093
advertised.listeners=PLAINTEXT://localhost:9092, SASL_SSL://localhost:9093
security.inter.broker.protocol=SASL_SSL
sasl.mechanism.inter.broker.protocol=PLAIN
sasl.enabled.mechanisms=PLAIN

# ssl configurations
ssl.keystore.location=/path_to/kafka.keystore
ssl.keystore.type=pkcs12
ssl.keystore.password=yourpass
ssl.key.password=yourpass
ssl.truststore.location=/path_to/kafka.truststore
ssl.truststore.type=pkcs12
ssl.truststore.password=yourpass
# ssl.client.auth is probably not needed in SASL_SSL mode
# ssl.client.auth=required
```

- **Be careful with the store type settings**, we must set them as we generated the store in format '**pkcs12**'. If we don't sepcify them, the default type should be '**jks**' and you will meet error

```bash
ssl.keystore.type=pkcs12
ssl.truststore.type=pkcs12
```

- **Also be careful with the client auth setting** '**ssl.client.auth**', NOT like SSL mode, as we use the **SASL (user/password, token etc.)** to authenticate, so we probably don't need to set '**ssl.client.auth**'.
3. Now let set the jass config file, Create a file '**kafka_jaas.conf**' under the folder 'config' with the following content

```bash
sasl_ssl.KafkaServer{
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="admin-secret"
    user_admin="admin-secret"
    user_alice="alice-secret";
};
```

4. Modify the **kafka-server-start.bat/kafka-server-start.sh** script to set the JVM parameter '**java.security.auth.login.config**' by environment JAAS_OPTS

```bash
// .bat script
set JAAS_OPTS=-Djava.security.auth.login.config=file:%~dp0../../config/kafka_jaas.conf
// .sh script
export JAAS_OPTS="-Djava.security.auth.login.config=file:$base_dir/../config/kafka_jaas.conf"
```

5. Modify the **kafka-run-class.bat/kafka-run-class.sh** script to create java command with parameter %JAAS_OPTS%

```bash
// .bat script
set COMMAND=%JAVA% %KAFKA_HEAP_OPTS% %KAFKA_JVM_PERFORMANCE_OPTS% %KAFKA_JMX_OPTS% %KAFKA_LOG4J_OPTS% %JAAS_OPTS% -cp "%CLASSPATH%" %KAFKA_OPTS% %*
// .sh script
exec "$JAVA" $KAFKA_HEAP_OPTS $KAFKA_JVM_PERFORMANCE_OPTS $KAFKA_GC_LOG_OPTS $KAFKA_JMX_OPTS $KAFKA_LOG4J_CMD_OPTS $JAAS_OPTS $JAAS_OPTS -cp "$CLASSPATH" $KAFKA_OPTS "$@"
```

6. Then start the zookeeper and kafka-server in different consoles, now the kafka server is setup correctly and running

```bash
// .bat script
zookeeper-server-start.bat .\config\zookeeper.properties
kafka-server-start.bat .\config\server.properties
// .sh script
./bin/zookeeper-server-start.sh ./config/zookeeper.properties
./bin/kafka-server-start.sh ./config/server.properties
```


7. Next we need to modify the **consumer.properties/producer.properties** to allow connecting to port 9093 with protocol SASL_SSL, you can aslo copy the following content to a file 'client.properties' for kafka-topics.bat to use.

```bash
bootstrap.servers=localhost:9092, localhost:9093
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="alice" password="alice-secret";
security.protocol=SASL_SSL
sasl.mechanism=PLAIN

#ssl configurations
ssl.truststore.location=/path_to/kafka.truststore
ssl.truststore.type=pkcs12
ssl.truststore.password=yourpass
#the following keystore setting are probably not needed
#ssl.keystore.location=/path_to/kafka.keystore
#ssl.keystore.type=pkcs12
#ssl.keystore.password=yourpass
```

8. Test the ssl connection with the following command

```bash
openssl s_client -connect localhost:9093 -tls1_2
```

if everything runs correctly, you should be able to get something as below

```bash
Connecting to 20.36.258.36
CONNECTED(00000194)
```

9. Create and List topic with port 9093 in SASL_SSL mode

```bash
# .bat script
kafka-topics.bat --create --topic gaming-events --bootstrap-server localhost:9093 --command-config ./config/client.properties
kafka-topics.bat --list --bootstrap-server localhost:9093 --command-config ./config/client.properties
# .sh script
./bin/kafka-topics.sh --create --topic gaming-events --bootstrap-server localhost:9093 --command-config ./config/client.properties
./bin/kafka-topics.sh --list --bootstrap-server localhost:9093 --command-config ./config/client.properties
```

10. Run Consumer with port 9093 in SASL_SSL mode

```bash
# .bat script
kafka-console-consumer.bat --topic gaming-events --from-beginning --bootstrap-server localhost:9093 --consumer.config ./config/consumer.properties
# .sh script
./bin/kafka-console-consumer.sh --topic gaming-events --from-beginning --bootstrap-server localhost:9093 --consumer.config ./config/consumer.properties
```

11. Run Producer with port 9093 in SASL_SSL mode

```bash
# .bat script
kafka-console-producer.bat --topic gaming-events --bootstrap-server localhost:9093 --producer.config ./config/producer.properties
# .sh script
./bin/kafka-console-producer.sh --topic gaming-events --bootstrap-server localhost:9093 --producer.config ./config/producer.properties
```

Now you are good to communicate between producer and consumer on secured port 9093 in SAL_SSL mode.

​