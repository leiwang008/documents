# How to config Kafka broker in SSL mode?

Please refer to [Run kafka in SSL mode](https://github.com/leiwang008/documents/blob/main/kafka/how_to_run_kafka_in_ssl_mode.md), it talks about both broker configuration and client(consumer/producer) configuration.

This only talks about the broker configuration, but this explains step by step how to generate the keystore, CA-certificate, 


Followed the document of https://kafka.apache.org/documentation/#security_ssl

I just cannot believe the kafka document is a shit! 不推荐使用kafka的文档，可参考我下面的三个链接，都是好文章。

1. create keystore

```bash
# you will not be able to add the signed certificate into keystore of '-storetype pkcs12'
# keytool -keystore server.keystore.jks -alias localhost -validity 365 -genkey -keyalg RSA -storetype pkcs12
keytool -keystore server.keystore.jks -alias localhost -validity 365 -genkey -keyalg RSA

# A big pit, when you are asked the following question like this, make sure you input the "localhost" or the broker's FQDN
# don't be stupid to write your name, haha.
What is your first and last name?
  [Unknown]:  localhost
```

2. generate certificate signing requests (CSR)

```bash
# no parameter -destkeystoretype for keytool
# keytool -keystore server.keystore.jks -alias localhost -validity 365 -genkey -keyalg RSA -destkeystoretype pkcs12 -ext SAN=DNS:<you.pc.com>,IP:<your ip>
# you will not be able to add the signed certificate into keystore of '-storetype pkcs12'
# keytool –keystore server.keystore.jks –alias localhost -validity 365         –keyalg RSA        -storetype pkcs12 –certreq –file server.csr
keytool -keystore server.keystore.jks -alias localhost -validity 365         -keyalg RSA   -certreq -file server.csr
```

3. create serial.txt, index.txt and openssl-ca.cnf file

```bash
    echo 01 > serial.txt
    touch index.txt
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

4. generate your CA (Certificate Authority)

```bash
# there is no openssl command in DOS, you have to run it in the 'git bash'
openssl req -x509 -config openssl-ca.cnf -newkey rsa:4096 -sha256 -nodes -out cacert.pem -outform PEM
```


5. add the generated CA to the **clients' truststore** so that the clients can trust this CA, also add it to server truststore.

```txt
keytool -keystore client.truststore.jks -alias CARoot -import -file cacert.pem
keytool -keystore server.truststore.jks -alias CARoot -import -file cacert.pem
```


6. Signing the your CA

```bash
openssl ca -config openssl-ca.cnf -policy signing_policy -extensions signing_req -out server.cert -infiles server.csr
```

​
7. Import both the certificate of the CA and the signed certificate into the keystore:

```bash
keytool -keystore server.keystore.jks -alias CARoot -import -file cacert.pem
keytool -keystore server.keystore.jks -alias localhost -import -file server.cert
```


8. Modify the server.properties file

```bash
listeners=SSL://localhost:9093
advertised.listeners=SSL://localhost:9093

ssl.keystore.location=/sdk/kafka_2.13-3.7.0/ssl_certs/server.keystore.jks
ssl.keystore.password=*******
ssl.truststore.location=/sdk/kafka_2.13-3.7.0/ssl_certs/server.truststore.jks
ssl.truststore.password=*******
ssl.key.password=*******
ssl.client.auth=required
```

9. Then start the zookeeper and kafka-server in different consoles, now the kafka server is setup correctly and running

```bash
zookeeper-server-start.bat .\config\zookeeper.properties
kafka-server-start.bat .\config\server.properties
```


我遇到问题 

https://stackoverflow.com/questions/78584196/how-to-use-kafka-in-sasl-ssl-mode 

相关文章 

https://mbd.baidu.com/ma/s/rtsaeDlf  讲解的最明白，值得一看

https://blog.51cto.com/u_16099181/9912712 有脚本，可以自动生成 keystore 和 trustkeystore, 非常好用

https://blog.51cto.com/u_16213683/9782122 有客户端的java code 可以参考

​