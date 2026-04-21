# Attacking Authentication Mechanisms

## JWT

[JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519) is a format for transmitting cryptographically secure data. While JWTs are not directly tied to authentication in web applications, many web applications use JWTs as a stateless session token. These tokens, encoded as JSON objects, are a secure and efficient way to transmit information between a client and a server. JWTs consist of three main parts: a header, a payload, and a signature, enabling authentication, authorization, and stateless information exchange. They have become popular for implementing token-based authentication and authorization mechanisms due to their simplicity, flexibility, and widespread support across different programming languages and platforms.

A JWT is made up of three parts, which are separated by dots:

* ```
  JWT Header
  ```
* ```
  JWT Payload
  ```
* ```
  JWT Signature
  ```

Each part is a base64-encoded JSON object:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJIVEItQWNhZGVteSIsInVzZXIiOiJhZG1pbiIsImlzQWRtaW4iOnRydWV9.Chnhj-ATkcOfjtn8GCHYvpNE-9dmlhKTCUwl6pxTZEA
```

We will now look at these parts and discuss their function within the JWT.Header

### Header

The first part of the JWT is its header. It comprises metadata about the token itself, holding information that allows interpreting it. For instance, let us look at the header of our example token from before. After base64-decoding the first part, we are left with the following JSON object:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

As we can see, the header only consists of two parameters in this case. The `typ` parameter is usually set to `"JWT"`, and the `alg` parameter specifies the cryptographic signature or MAC algorithm the token is secured with. Valid values for this parameter are defined in the [JWA standard](https://datatracker.ietf.org/doc/html/rfc7518#section-3.1):

<table><thead><tr><th width="134">"alg" Parameter Value</th><th>Signature/MAC Algorithm</th></tr></thead><tbody><tr><td>HS256</td><td>HMAC using SHA-256</td></tr><tr><td>HS384</td><td>HMAC using SHA-384</td></tr><tr><td>HS512</td><td>HMAC using SHA-512</td></tr><tr><td>RS256</td><td>RSASSA-PKCS1-v1_5 using SHA-256</td></tr><tr><td>RS384</td><td>RSASSA-PKCS1-v1_5 using SHA-384</td></tr><tr><td>RS512</td><td>RSASSA-PKCS1-v1_5 using SHA-512</td></tr><tr><td>ES256</td><td>ECDSA using P-256 and SHA-256</td></tr><tr><td>ES384</td><td>ECDSA using P-384 and SHA-384</td></tr><tr><td>ES512</td><td>ECDSA using P-512 and SHA-512</td></tr><tr><td>PS256</td><td>RSASSA-PSS using SHA-256 and MGF1 with SHA-256</td></tr><tr><td>PS384</td><td>RSASSA-PSS using SHA-384 and MGF1 with SHA-384</td></tr><tr><td>PS512</td><td>RSASSA-PSS using SHA-512 and MGF1 with SHA-512</td></tr><tr><td>none</td><td>No digital signature or MAC performed</td></tr></tbody></table>

Our example token is secured with `HMAC using SHA-256` in this case. There are additional parameters that can be defined in the JWT header as defined in the [JWS standard](https://datatracker.ietf.org/doc/html/rfc7515#section-4.1), some of which we will take a look at in the upcoming sections.

### Payload

The JWT payload is the middle part and contains the actual data making up the token. This data comprises multiple `claims`. Base64-decoding the sample JWT's payload reveals the following JSON data:

```json
{
  "iss": "HTB-Academy",
  "user": "admin",
  "isAdmin": true
}
```

While registered claims are defined in the [JWT standard](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1), a JWT payload can contain arbitrary, user-defined claims. In our example case, the `iss` claim is a registered claim that specifies the identity of the JWT's issuer. The claims `user` and `isAdmin` are not registered and indicate that this particular JWT was issued for the user `admin`, who seems to be an administrator.

### Signature

The last part of the JWT is its signature, which is computed based on the JWT's header, payload, and a secret signing key, using the algorithm specified in the header. Therefore, the integrity of the JWT token is protected by the signature. If any data within the header, payload, or signature itself is manipulated, the signature will no longer match the token, thus enabling the detection of manipulation. Thus, knowledge of the secret signing key is required to compute a valid signature for a JWT.

### **Stateful Authentication**

Traditionally, the user presents a session token to the web application, which then proceeds to look up the user's associated data in some kind of database. Since the web application needs to keep track of the state, i.e., the data associated with the session, this approach is called **stateful** authentication.

<figure><img src="../../../.gitbook/assets/image (19) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### **Stateless Authentication**

On the other hand, in JWT-based authentication, the session token is replaced by a JWT containing user information. After verifying the token's signature, the server can retrieve all user information from the JWT's claims sent by the user. Because the server does not need to keep a state, this approach is called **stateless** authentication. Remember that the JWT's signature prevents an attacker from manipulating the data within it, and any manipulation would result in a failed signature verification.

<figure><img src="../../../.gitbook/assets/image (21) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## OAuth

[OAuth](https://datatracker.ietf.org/doc/html/rfc6749) is an open standard protocol that allows secure authorization and delegation of access between different web services without sharing user credentials. It enables users to grant third-party applications limited access to resources on other web services, such as social media accounts or online banking, without exposing their passwords. OAuth operates through token-based authentication processes, facilitating seamless interactions between service providers and consumers while maintaining user privacy and security. Widely adopted across various industries, OAuth has become the de facto standard for enabling secure API access and authorization in modern web and mobile applications.

***

## SAML

[Secure Assertion Markup Language (SAML)](https://wiki.oasis-open.org/security/FrontPage) is an XML-based open standard for exchanging authentication and authorization data between identity providers (IdPs) and service providers (SPs). SAML enables single sign-on (SSO), allowing users to access multiple applications and services with a single set of credentials. In the SAML workflow, the user's identity is authenticated by the IdP, which then generates a digitally signed assertion containing user attributes and permissions. This assertion is sent to the SP, which validates it and grants access accordingly. SAML is widely used in enterprise environments and web-based applications to streamline authentication processes and enhance security through standardized protocols and assertions.
