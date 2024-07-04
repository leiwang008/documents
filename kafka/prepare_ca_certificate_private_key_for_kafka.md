# How to generate CA-signed certificate and private key for kafka client (consumer/producer)?

If we run the kafka server in [SSL mode](https://github.com/leiwang008/documents/blob/main/kafka/how_to_run_kafka_in_ssl_mode.md) and we start the kafka server with parameter **ssl.client.auth=required**, then the kafka server will **authenticate** the client (consumer/producer) by its **certificate** and its **private key**.

​
Below are steps to generate the client's certificate and client's private key. We will use the **openssl** command, It is not easy to figure out how to use the **openssl** parameters at the first. Don't worry, it is simple, we will create them by script :-). 

​

1. As we need the **CA-signed-certificate** for the client, so we need the **CA certificate** and **CA private key** to sign, copy them from your kafka broker/server, let's suppose we have them as **ca-cert.pem** and **ca-key**. If you don't have the **CA certificate** and **CA private key,** please read [How to run kafka in SASL\_SSL Mode](https://github.com/leiwang008/documents/blob/main/kafka/how_to_run_kafka_in_sasl_ssl_mode.md), I have provided the script to generate them on kafka server/broker.

​

2. Then Let's copy the following content to a file '**openssl_ca.cnf**', it is the config file for **openssl** command

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
#stateOrProvinceName_default = beiijng

localityName                = beiijng
#localityName_default        = beiijng

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

​

3. Copy the following content into a file '**generate_client_key_certificate.sh**'

```bash
#!/bin/bash
################################## setup environments ##############################
export MSYS_NO_PATHCONV=1 #avoid the git bash adds "C:\Program Files\Git\" in front of slash /
export CERT_OUTPUT_PATH="." # path holding certificates and stores
export CA_CONFIG="$CERT_OUTPUT_PATH/openssl_ca.cnf" #CA config file
export CERT_KEY_FILE="$CERT_OUTPUT_PATH/ca-key" # CA key, private key to sign a cert or to decrypt
export CERT_AUTH_FILE="$CERT_OUTPUT_PATH/ca-cert.pem" # CA certificate, holding the public key to encrypt, and the signature to sign a cert or to verify a broker by client
export PASSWORD=yourpass # the password
export CLIENT_NAME=localhost # it should be "localhost" or client's FQDN
export CLIENT_PRIVATE_KEY="$CERT_OUTPUT_PATH/${CLIENT_NAME}_client_private.key" # client private key
export CLIENT_CERT_CSR="$CERT_OUTPUT_PATH/${CLIENT_NAME}_client_cert.csr" # the CSR (certificate signing request) to get client's certificate
export CLIENT_SIGNED_CERTIFICATE="$CERT_OUTPUT_PATH/${CLIENT_NAME}_client_cert.pem" # client signed certificate
export DAYS_VALID=365 # key valid time in days
#############################################################################

echo "===========================   generate client private key and csr (certificate signing request)  ==================================="
openssl req -newkey rsa:2048 -keyout $CLIENT_PRIVATE_KEY -out $CLIENT_CERT_CSR -passin pass:"$PASSWORD" -passout pass:"$PASSWORD" -config $CA_CONFIG
echo "verifying client private key ... "
openssl rsa -in $CLIENT_PRIVATE_KEY -check -passin pass:"$PASSWORD"

echo ""
echo "===========================   sign the client's csr by CA cert and CA key  ==================================="
openssl x509 -req -CA $CERT_AUTH_FILE -CAkey $CERT_KEY_FILE  -in $CLIENT_CERT_CSR -out $CLIENT_SIGNED_CERTIFICATE -days $DAYS_VALID -CAcreateserial -passin pass:"$PASSWORD"

echo ""
echo "===========================   verify the certificate in pem format  ==================================="
openssl x509 -in $CLIENT_SIGNED_CERTIFICATE -text -noout -passin pass:"$PASSWORD"
```

4. Finally you can run the script to generate the client **certificate** and **private key**

```bash
$ ./generate_client_key_certificate.sh
```

And the **client certificate**(localhost_client_cert.pem) and **private key**(localhost_client_private.key) will be ready for you :-).