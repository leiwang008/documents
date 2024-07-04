# How to generate azure OIDC oauthbearer token?

1. First of all, we need to register an application in Azure ActiveDirectory.

    - Go to the [Azure Portal](https://portal.azure.com/#home) and find the `App registrations` from the top search bar.
    - Then we click at the `New registration` and fill in the application details and register the application.
    - After registration, take note of the `Application (client) ID` and `Directory (tenant) ID`, suppose we have them as **454657fdef2-45r7-45r4-843e-3484038403324** and **4568erfs3-583d-98f4-598e-34567233250dr43**, they are not real, don't try to hack me :-).
1. Next we need to create a Client Secret
    - In the registered application, navigate to `Certificates & secrets`. 
    - Create a new client secret and **SAVE** the value. Please **SAVE** it, we will never see it again once we leave this page!!! We'll need this secret for the OAuth flow. Suppose we have it as **8934839jfoierno390sfdjlj$2039043!flj430990**, this is a fake secret.
2. Next we need to add an `Application ID URI`, which is called identifier URI, is a globally unique URI used to identify the web API. This URI is the prefix for scopes in the Oauth protocol. 
    - Go to the `Expose an API` section
    - Find the `Application ID URI` on the top and click the `Add`, it will use your `Application (client) ID` by default, just accept it, it is something like **api://454657fdef2-45r7-45r4-843e-3484038403324**
3. Now we are ready to generate the OIDC oauthbearer token by the following command  

```bash
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" \
     -d "client_id=454657fdef2-45r7-45r4-843e-3484038403324" \
     -d "client_secret=8934839jfoierno390sfdjlj$2039043!flj430990" \
	 -d "scope=api://454657fdef2-45r7-45r4-843e-3484038403324/.default" \
     -d "grant_type=client_credentials" \
     https://login.microsoftonline.com/4568erfs3-583d-98f4-598e-34567233250dr43/oauth2/v2.0/token
```

we will get the token as below

```bash
{"token_type":"Bearer","expires_in":3599,"ext_expires_in":3599,"access_token":"eyJ0JDLAJFDKAIEJLKJ43890342JSLJKLSJF343KJKJDSFJKSJDFKI3.eyJAFLHJHUERHUADFU34HUDHS875HUSHOU34SHUI3426Y8976SDFGH324Y9SDFHKSDF834HSAJHFUH324Y8SDFH346FDSs.I-FASLHFJASJLH34285HUISDFHURW4390CVXVHIKH54308FGJH237565646SDFHIKEWRHJGSDFfjke5rs9fjlj54JH5HKSFHKhfkw"}
```

Finally we can verify the generated oauthbearer token in [https://jwt.io/](https://jwt.io/) to see the decoded token, we can see the field 'typ', 'alg', 'kid' for the header, 'aud', 'iss', 'sub' etc. for the payload, and we will see if the token's signature is verified or not.