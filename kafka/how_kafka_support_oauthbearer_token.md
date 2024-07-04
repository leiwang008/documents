# How kafka supports OAUTHBEARER token?
​
At the first kafka supported the **OAuth Authentication via SASL/OAUTHBEARER** which has been documented in [KIP-255](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=75968876) and has been implemented in the jira ticket [KAFKA-6562](https://issues.apache.org/jira/browse/KAFKA-6562)  


Before configuring the broker, we need the [azure OIDC secure token](https://github.com/leiwang008/documents/blob/main/kafka/how_to_generate_azure_oidc_token.md)  
Suppose that we have registered the azure application and we have  

**Directory (tenant) ID** as **4568erfs3-583d-98f4-598e-34567233250dr43**, this will be used in the 'sasl.oauthbearer.token.endpoint.url' and 'sasl.oauthbearer.jwks.endpoint.url' and 'sasl.oauthbearer.expected.issuer'  
**Application (client) ID** as **454657fdef2-45r7-45r4-843e-3484038403324**, this will be used in 'clientId'  
**Client Secret** as **8934839jfoierno390sfdjlj$2039043!flj430990**, this will be used in 'clientSecret'  
**Application ID URI** as **api://454657fdef2-45r7-45r4-843e-3484038403324**, this will be used in 'sasl.oauthbearer.expected.audience' and 'scope'  
 
1. config the server.properties to run with SASL_PLAINTEXT+OAUTHBEARER mode with azure OIDC  

```txt
listeners=SASL_PLAINTEXT://safsdev03.na.sas.com:9093
advertised.listeners=SASL_PLAINTEXT://safsdev03.na.sas.com:9093
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=OAUTHBEARER
sasl.enabled.mechanisms=OAUTHBEARER

# SASL OAuthBearer secure token settings
listener.name.sasl_plaintext.oauthbearer.sasl.login.callback.handler.class=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginCallbackHandler
# listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.token.endpoint.url=https://login.microsoftonline.com/<Directory (tenant) ID>/oauth2/v2.0/token
listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.token.endpoint.url=https://login.microsoftonline.com/4568erfs3-583d-98f4-598e-34567233250dr43/oauth2/v2.0/token

listener.name.sasl_plaintext.oauthbearer.sasl.server.callback.handler.class=org.apache.kafka.common.security.oauthbearer.OAuthBearerValidatorCallbackHandler
# listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.jwks.endpoint.url=https://login.microsoftonline.com/<Directory (tenant) ID>/discovery/v2.0/keys
listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.jwks.endpoint.url=https://login.microsoftonline.com/4568erfs3-583d-98f4-598e-34567233250dr43/discovery/v2.0/keys
# listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.expected.audience=<Application ID URI>
listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.expected.audience=api://454657fdef2-45r7-45r4-843e-3484038403324

#listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.expected.issuer=https://sts.windows.net/<Directory (tenant) ID>/
#listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.expected.issuer=https://sts.windows.net/4568erfs3-583d-98f4-598e-34567233250dr43/

# Clock skew allowance
#listener.name.sasl_plaintext.oauthbearer.sasl.oauthbearer.clock.skew.seconds=300

#listener.name.sasl_plaintext.oauthbearer.sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required \
#      clientId="<Application (client) ID>" \
#      clientSecret="<Client Secret>" \
#      scope="<Application ID URI>/.default";

listener.name.sasl_plaintext.oauthbearer.sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required \
      clientId="454657fdef2-45r7-45r4-843e-3484038403324" \
      clientSecret="8934839jfoierno390sfdjlj$2039043!flj430990" \
      scope="api://454657fdef2-45r7-45r4-843e-3484038403324/.default";
```