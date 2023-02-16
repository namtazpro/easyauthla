Video: https://www.youtube.com/watch?v=tawHXz6qGQI&t=2s

Step 1: Create an AAD app for the client (App Registration) in AAD

Step 2: Create a client Token

Make an API call :

Body type: x-www-form-urlencoded

- Key: client_credentials
- client_id: YOUR-CLIENT-ID (App reg clientid)
-  client_secret: YOUR-CLIENT-SECRET (app reg secret)
- resource: https://management.azure.com

```
GET https://login.microsoftonline.com/{YOUR-TENANT-ID}/oauth2/token
```

Step 3: Inspect the token in jwt.io and get values:
   - audience: 'aud' => https://management.azure.com
   - issuer: 'iss' https://sts.windows.net/{YOUR-TENANT-ID}/
   - objectid: 'oid' YOUR-OBJECT-ID


Step 4: Configure EasyAuth:

```
PUT https://management.azure.com/subscriptions/{YOUR_SUBSCRIPTION-ID}/resourcegroups/{YOUR-RG}/providers/Microsoft.Web/sites/{YOUR-LOGICAPP-NAME}/config/authsettingsV2?api-version=2021-02-01&
```

Full body here: https://techcommunity.microsoft.com/t5/integrations-on-azure-blog/trigger-workflows-in-standard-logic-apps-with-easy-auth/ba-p/3207378

```
{
    "id": "/subscriptions/8e3330a8-f2fd-4db5-abb1-8cd2657ae28c/resourcegroups/rg-int-identity/providers/Microsoft.Web/sites/la-std-httprequest/config/authsettingsV2",
    "name": "authsettingsV2",
    "type": "Microsoft.Web/sites/config",
    "location": "westeurope",
    "tags": {},
    "properties": {
        "platform": {
            "enabled": true,
            "runtimeVersion": "~1"
        },
        "globalValidation": {
            "requireAuthentication": true,
            "unauthenticatedClientAction": "AllowAnonymous"
        },
        "identityProviders": {
            "azureActiveDirectory": {
                "enabled": true,
                "registration": {
                    "openIdIssuer": "{issuer}",
                    "clientId": "{objectid-(oid)}"
                },
                "login": {
                    "disableWWWAuthenticate": false
                },
                "validation": {
                    "jwtClaimChecks": {},
                    "allowedAudiences": [
                        "{audience}"
                    ],
                    "defaultAuthorizationPolicy": {
                        "allowedPrincipals": {
                            "identities": [
                                "{objectid-(oid)}"
                            ]
                        }
                    }
                }
```

Step 5: call logic apps with Token