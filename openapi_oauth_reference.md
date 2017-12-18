# Xref Open API OAuth Reference

### Authorization

#### GET https://login.xref.com/oauth/open-api/:client_id

| Name  | Required? | Description |
| ----- | --------- | ----------- |
| state | optional  | Unique string to be passed back upon completion |

#### GET https://partnerinc.com/integrations/xref

| Name  | Description |
| ----- | ----------- |
| code  | An authorization code you can use in the next call to get an access token for your user. This can only be used once and expires in 5 minutes.|
| state | The value of the state parameter you provided on the initial GET request. |
| error | Error code, if any error. |

### Token Issuance

#### POST https://api-open.xref.com/oauth/tokens

| Name          | Required? | Type | Description |
| ------------- | --------- | :-----:| :-----:|
| client_id     | required  | string | Issued when you created your application|
| client_secret | required  | string |Issued when you created your application|
| code          | required  | string | The value of the code returned previously |

Request:
``` curl
curl -X POST https://api-open.xref.com/oauth/tokens \
    --header "Content-Type: application/json" \
    --data-binary "{
        \"client_id\": \"YOUR_CLIENT_ID\",
        \"client_secret\": \"YOUR_CLIENT_SECRET\",
        \"code\": \"AUTHORIZATION_CODE\"
    }"
```

Response:
```json
{
  "access_token": "cb6f9d8c843c22e53db97f7fd2ff20ab58c91e31",
  "xref_account_id": "d1e01e1a36a09eda2a31b01caf62eb2e"
}
```

### Deauthorize

#### POST https://api-open.xref.com/oauth/deauthorize

Request:
``` curl
curl -X POST https://api-open.xref.com/oauth/deauthorize \
   -H "Authorization: Token ACCESS_TOKEN"
```

Response:
```json
{
  "access_token": "cb6f9d8c843c22e53db97f7fd2ff20ab58c91e31"
}
```
