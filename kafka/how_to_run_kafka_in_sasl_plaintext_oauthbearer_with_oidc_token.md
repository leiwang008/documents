# How to run kafka in SASL_PLAINTEXT+OAUTHBEARER mode with OIDC secure token?
​
Now kafka supports **Extend SASL/OAUTHBEARER with Support for OIDC** which has been documented in [KIP-768](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=186877575) and has been implemented in the jira ticket [KAFKA-13202](https://issues.apache.org/jira/browse/KAFKA-13202) , the implementation of KIP-768 provides a concrete implementation of the interfaces defined in [KIP-255](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=75968876) to allow Kafka to connect to an Open ID identity provider for authentication and token retrieval and it can be used in **production cases**.

Before configuring the broker, we need the [azure OIDC secure token](https://github.com/leiwang008/documents/blob/main/kafka/how_to_generate_azure_oidc_token.md)  
Suppose that we have registered the azure application and we have  

**Directory (tenant) ID** as **4568erfs3-583d-98f4-598e-34567233250dr43**, this will be used in the 'sasl.oauthbearer.token.endpoint.url' and 'sasl.oauthbearer.jwks.endpoint.url' and 'sasl.oauthbearer.expected.issuer'  
**Application (client) ID** as **454657fdef2-45r7-45r4-843e-3484038403324**, this will be used in 'clientId'  
**Client Secret** as **8934839jfoierno390sfdjlj$2039043!flj430990**, this will be used in 'clientSecret'  
**Application ID URI** as **api://454657fdef2-45r7-45r4-843e-3484038403324**, this will be used in 'sasl.oauthbearer.expected.audience' and 'scope'  
 
1. config the server.properties to run with SASL_PLAINTEXT+OAUTHBEARER mode with azure OIDC  

```bash
listeners=SASL_PLAINTEXT://safsdev03.na.sas.com:9093
advertised.listeners=SASL_PLAINTEXT://safsdev03.na.sas.com:9093
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=OAUTHBEARER
sasl.enabled.mechanisms=OAUTHBEARER

# SASL OAuthBearer secure token (OIDC) settings
# kafka broker configuration always need 'OAuthBearerValidatorCallbackHandler', 'jwks.endpoint.url' and 'expected.audience' to validate the oauthbearer token.
listener.name.sasl_plaintext.oauthbearer.sasl.server.callback.handler.class=org.apache.kafka.common.security.oauthbearer.OAuthBearerValidatorCallbackHandler
# listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.jwks.endpoint.url=https://login.microsoftonline.com/<Directory (tenant) ID>/discovery/v2.0/keys
listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.jwks.endpoint.url=https://login.microsoftonline.com/4568erfs3-583d-98f4-598e-34567233250dr43/discovery/v2.0/keys
# listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.expected.audience=<Application ID URI>
listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.expected.audience=api://454657fdef2-45r7-45r4-843e-3484038403324

# if 'sasl.mechanism.inter.broker.protocol' is set to 'OAUTHBEARER', then broker configuration will also need OAuthBearerLoginCallbackHandler and 'token.endpoint.url' to obtain oauthbearer token.
listener.name.sasl_plaintext.oauthbearer.sasl.login.callback.handler.class=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginCallbackHandler
# listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.token.endpoint.url=https://login.microsoftonline.com/<Directory (tenant) ID>/oauth2/v2.0/token
listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.token.endpoint.url=https://login.microsoftonline.com/4568erfs3-583d-98f4-598e-34567233250dr43/oauth2/v2.0/token

#listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.expected.issuer=https://sts.windows.net/<Directory (tenant) ID>/
#listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.expected.issuer=https://sts.windows.net/4568erfs3-583d-98f4-598e-34567233250dr43/

# Clock skew allowance
#listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.clock.skew.seconds=300

# OAuthBearerLoginCallbackHandler will need 'clientId' and 'clientSecret' to exchange the token from 'token.endpoint.url'
#listener.name.sasl_plaintext.oauthbearer.sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required \
#      clientId="<Application (client) ID>" \
#      clientSecret="<Client Secret>" \
#      scope="<Application ID URI>/.default";
listener.name.sasl_plaintext.oauthbearer.sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required \
      clientId="454657fdef2-45r7-45r4-843e-3484038403324" \
      clientSecret="8934839jfoierno390sfdjlj$2039043!flj430990" \
      scope="api://454657fdef2-45r7-45r4-843e-3484038403324/.default";
```

​
2. start the zookeeper and kafka server  

```bash
zookeeper-server-start.bat .\config\zookeeper.properties
kafka-server-start.bat .\config\server.properties
```

​
3. next create a file client.properties in the config folder for kafka-topic script to use  

```bash
# secure token configuration
security.protocol=SASL_PLAINTEXT
sasl.mechanism=OAUTHBEARER

sasl.login.callback.handler.class=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginCallbackHandler
# sasl.oauthbearer.token.endpoint.url=https://login.microsoftonline.com/<Directory (tenant) ID>/oauth2/v2.0/token
sasl.oauthbearer.token.endpoint.url=https://login.microsoftonline.com/4568erfs3-583d-98f4-598e-34567233250dr43/oauth2/v2.0/token

# OAuthBearerLoginCallbackHandler will need 'clientId' and 'clientSecret' to exchange the token from 'token.endpoint.url'
#sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required \
#      clientId="<Application (client) ID>" \
#      clientSecret="<Client Secret>" \
#      scope="<Application ID URI>/.default";
sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required \
      clientId="454657fdef2-45r7-45r4-843e-3484038403324" \
      clientSecret="8934839jfoierno390sfdjlj$2039043!flj430990" \
      scope="api://454657fdef2-45r7-45r4-843e-3484038403324/.default";
```

4. create topic 'gaming-events' by kafka-topic script  

```bash
kafka-topics.bat --create --topic gaming-events --bootstrap-server localhost:9093 --command-config .\config\client.properties
kafka-topics.bat --list --bootstrap-server localhost:9093 --command-config .\config\client.properties
```

5. next modify consumer.properties/producer.properties the same as client.properties  

```bash
# secure token configuration
security.protocol=SASL_PLAINTEXT
sasl.mechanism=OAUTHBEARER

sasl.login.callback.handler.class=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginCallbackHandler
# sasl.oauthbearer.token.endpoint.url=https://login.microsoftonline.com/<Directory (tenant) ID>/oauth2/v2.0/token
sasl.oauthbearer.token.endpoint.url=https://login.microsoftonline.com/4568erfs3-583d-98f4-598e-34567233250dr43/oauth2/v2.0/token

# OAuthBearerLoginCallbackHandler will need 'clientId' and 'clientSecret' to exchange the token from 'token.endpoint.url'
#sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required \
#      clientId="<Application (client) ID>" \
#      clientSecret="<Client Secret>" \
#      scope="<Application ID URI>/.default";
sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required \
      clientId="454657fdef2-45r7-45r4-843e-3484038403324" \
      clientSecret="8934839jfoierno390sfdjlj$2039043!flj430990" \
      scope="api://454657fdef2-45r7-45r4-843e-3484038403324/.default";
```

​

6. finally start the consumer and producer communicating through the topic 'gaming-events'  

```bash
kafka-console-consumer.bat --topic gaming-events --from-beginning --bootstrap-server localhost:9093 --consumer.config .\config\consumer.properties
kafka-console-producer.bat --topic gaming-events --bootstrap-server localhost:9093 --producer.config .\config\producer.properties
```


This is a secure OAUTHBEARER token (azure OIDC) setting for kafka broker and consumer/producer.

# Reference
https://docs.confluent.io/platform/current/kafka/authentication_sasl/authentication_sasl_oauth.html this article is out of date
https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols-oidc
https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols
[KIP-768](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=186877575) is the most useful article, supports the OAUTHBEARER/OIDC
[KIP-255](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=75968876) supports the unsecure token, default implementation of kafka