
# Azure AD Config

```

    #
    # AuthenticationChain contains 2 Authenticators. One for Clusterinternal Auth and technical users (db) and one for the human users using Azure AD Auth (pac4j)
    # 
    druid.auth.authenticatorChain=["db","pac4j"]

    #
    # Authentication via pac4j web user authenticator (OIDC)
    #
    druid.auth.authenticator.pac4j.name=pac4j
    druid.auth.authenticator.pac4j.type=pac4j
    druid.auth.authenticator.pac4j.authorizerName=pac4j

    # druid.auth.pac4j.oidc.oidcClaim=email
    druid.auth.pac4j.oidc.scope=openid profile email

    druid.auth.pac4j.cookiePassphrase=
    druid.auth.pac4j.oidc.clientID=
    druid.auth.pac4j.oidc.clientSecret=
    druid.auth.pac4j.oidc.discoveryURI=https://login.microsoftonline.com/<TENANT>/.well-known/openid-configuration
```
