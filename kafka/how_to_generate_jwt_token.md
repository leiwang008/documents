# How to generate a CORRECT JWT?

JWT uses the Base64url Encoding to encode the token and there is an online encoder/decorder as below:

[base64url-encoder](https://simplycalc.com/base64url-encode.php)

[base64-decoder](https://simplycalc.com/base64-decode.php)

If we want to generate an un-secure (note that [the algorithm is "none"](https://datatracker.ietf.org/doc/html/rfc7515#appendix-A.5)) token of Header/Payload as below

**Header:**

```bash
{ "alg":"none", "typ" : "JWT" }
```

and you will get the encoded string like

```bash
eyAiYWxnIjoibm9uZSIsICJ0eXAiIDogIkpXVCIgfQ==
```

**Payload**

```bash
{ "sub":"alice", "iat" : 1629883000, "exp" : 9999999999999}
```

and you get the encoded string for the payload

```bash
eyAic3ViIjoiYWxpY2UiLCAiaWF0IiA6IDE2Mjk4ODMwMDAsICJleHAiIDogOTk5OTk5OTk5OTk5OX0=
```

then you concatenate the 2 encoded string with point as below

```bash
eyAiYWxnIjoibm9uZSIsICJ0eXAiIDogIkpXVCIgfQ==.eyAic3ViIjoiYWxpY2UiLCAiaWF0IiA6IDE2Mjk4ODMwMDAsICJleHAiIDogOTk5OTk5OTk5OTk5OX0=.
```

Then you think that it is good to be used in your program. NOOOOOOO...., a big PIT, this confused me a few days, you have to remove the padding equal signs as below, and the token is GOOD now.

```bash
eyAiYWxnIjoibm9uZSIsICJ0eXAiIDogIkpXVCIgfQ.eyAic3ViIjoiYWxpY2UiLCAiaWF0IiA6IDE2Mjk4ODMwMDAsICJleHAiIDogOTk5OTk5OTk5OTk5OX0.
```

**WHY WHY WHY?**

A JWT uses Base64url Encoding ([RFC 4648 - The Base16, Base32, and Base64 Data Encodings](https://datatracker.ietf.org/doc/html/rfc4648#section-5)) which is a variant of Base64 where the 62nd and 63rd are different and also where the padding (equal signs at the end) are not mandatory so you won’t see padding in the encoding of JWT’s.

**Do we have any script to generate the token automatically?**

Run the following script in a bash shell, 

```bash
#!/bin/bash

header=`echo -n '{ "alg":"none", "typ" : "JWT" }' | base64 | tr -d '=' | tr '/+' '_-' | tr -d '\n'`  #Header
payload=`echo -n '{ "sub":"alice", "iat" : 1629883000, "exp" : 9999999999999}' | base64 | tr -d '=' | tr '/+' '_-' | tr -d '\n'`   # Payload
token=$header.$payload. #Token
echo $token
```

and you should be able to get the un-secure JsonWebToken.

```bash
eyAiYWxnIjoibm9uZSIsICJ0eXAiIDogIkpXVCIgfQ.eyAic3ViIjoiYWxpY2UiLCAiaWF0IiA6IDE2Mjk4ODMwMDAsICJleHAiIDogOTk5OTk5OTk5OTk5OX0.
```

**We can also use the following python script to generate the token, run it in the** [ python online compiler](https://www.programiz.com/python-programming/online-compiler/)

```python
import base64
import json
import time

def base64url_encode(data):
    return base64.urlsafe_b64encode(data).rstrip(b'=').decode('utf-8')

# Create JWT header
header = {
    "alg": "none",
    "typ": "JWT"
}

# Create JWT payload
payload = {
    "sub": "alice",
    "iat": int(time.time()),
    "exp": int(time.time()) + 9999999999999  # Token expires in 1 hour
}

# Encode header and payload
encoded_header = base64url_encode(json.dumps(header).encode('utf-8'))
encoded_payload = base64url_encode(json.dumps(payload).encode('utf-8'))

# Combine encoded header and payload to form the token
token = f"{encoded_header}.{encoded_payload}."

print(f"Unsecured JWT Token: {token}")
```

the generated token is

```bash
Unsecured JWT Token: eyJhbGciOiAibm9uZSIsICJ0eXAiOiAiSldUIn0.eyJzdWIiOiAiYWxpY2UiLCAiaWF0IjogMTcxODcwMDMzOCwgImV4cCI6IDEwMDAxNzE4NzAwMzM3fQ.

=== Code Execution Successful ===
```

That's it :-)