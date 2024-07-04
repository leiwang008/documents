# How to run kafka in SASL_PLAINTEXT+OAUTHBEARER test mode.

At the first kafka supported the **OAuth Authentication via SASL/OAUTHBEARER** which has been documented in [KIP-255](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=75968876) and has been implemented in the jira ticket [KAFKA-6562](https://issues.apache.org/jira/browse/KAFKA-6562) , the implementation of KIP-255 provided a concrete example implementation that allowed clients to provide an **unsecured JWT access token** to the broker when initializing the connection only for ”**development scenarios**”, NOT for **production cases**.

Here we are going to config the kafka broker with this unsecure token way.  

1. config the server.properties to run with SASL OAUTHBEARER test mode as below, we only need to specify [OAuthBearerLoginModule](https://github.com/a0x8o/kafka/blob/master/clients/src/main/java/org/apache/kafka/common/security/oauthbearer/OAuthBearerLoginModule.java) for **sasl.jaas.  config** property, very simple.

```bash
listeners=SASL_PLAINTEXT://localhost:9093
advertised.listeners=SASL_PLAINTEXT://localhost:9093
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=OAUTHBEARER
sasl.enabled.mechanisms=OAUTHBEARER

# Specify the JAAS login context name for SASL/OAUTHBEARER
listener.name.sasl_plaintext.oauthbearer.sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required unsecuredLoginStringClaim_sub="alice";
```

2. start the zookeeper and kafka server

```bash
zookeeper-server-start.bat .\config\zookeeper.properties
kafka-server-start.bat .\config\server.properties
```

3. next create a file client.properties in the config folder for kafka-topic script to use

```bash
security.protocol=SASL_PLAINTEXT
sasl.mechanism=OAUTHBEARER
sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required unsecuredLoginStringClaim_sub="alice";
```

4. ccreate topic 'gaming-events' by kafka-topic script

```bash
kafka-topics.bat --create --topic gaming-events --bootstrap-server localhost:9093 --command-config .\config\client.properties
kafka-topics.bat --list --bootstrap-server localhost:9093 --command-config .\config\client.properties
```

5. next modify consumer.properties/producer.properties the same as client.properties

```bash
security.protocol=SASL_PLAINTEXT
sasl.mechanism=OAUTHBEARER
sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required unsecuredLoginStringClaim_sub="alice";
```

6. finally start the consumer and producer communicating through the topic 'gaming-events'

```bash
kafka-console-consumer.bat --topic gaming-events --from-beginning --bootstrap-server localhost:9093 --consumer.config .\config\consumer.properties
kafka-console-producer.bat --topic gaming-events --bootstrap-server localhost:9093 --producer.config .\config\producer.properties
```

This is a setting of OAUTHBEARER with unsecure token for kafka, never use it in the **production scenarios**.