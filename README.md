# Xref Open API OAuth Integration Guide

With Xref Open API OAuth integration, your customers can easily connect their Xref account with your platform. OAuth 2 is a protocol that provides access to your end user's Xref account upon their approval, without knowing their specific credentials.

At Xref, multiple users can belong to one account. Our OAuth integration works at the account level instead of the user level, meaning that end users will authenticate and deauthenticate on behalf of their account. This means that it is only necessary for one individual end user from one account to go through the OAuth integration flow, since the account they belong to will be authenticated for all of the account's users.

Since this behavior deviates from traditional OAuth integrations done at the user level, we will clarify vocabulary before moving on:

- Partner: You, the reader.
- Platform: Your application that will have Xref Open API integration available. Your end users use this platform.
- End user: Your users.
- Integrated account: The Xref account of an end user. Note that multiple of your end users can belong to the same Xref account. You will receive an `access_token` that will serve as the API key for each integrated account, not for each end user.

## General overview of integration flow

When an end user wants to connect their Xref account to your platform, they’ll go through these steps:

1. On your website, the end user will click a link that takes them to Xref. The link will pass along your platform `client_id`.
2. Once on Xref’s website, the end user will be prompted to authorize your platform to manage resources on their behalf by either:
    - Logging into their existing account
    - If they are already logged in, simply authorizing the platform
3. The end user will then be redirected back to your site (specifically to your `redirect_url`), passing along an authorization `code`.
4. Using this `code`, your application will be able to create an `access_token` for the end user and store it in your data store.
5. Subsequent API requests can be made on behalf of the integrated account using the `access_token` in place of the `API_KEY` typically used for Xref API requests.

## Integration setup
Please email integrations@xref.com to register your platform, if you haven't already.

You will need to provide the following:

| Field | Description | Example |
|-------|-------------|---------|
| `name`| Name of your platform. | Partner Inc |
|`description`| A short description of your platform. | Partner Inc is awesome.  |
|`url`| The url of your platform. | https://partnerinc.com |
|`redirect_url` | A page on your platform to which the end user will be redirected after connecting their integrated account (or failing to, should that be the case). It must follow the HTTPS protocol. | https://partnerinc.com/integrations/xref |
|`webhook_url`|An endpoint to which we will send webhooks to when certain events occur. | https://partnerinc.com/webhooks/xref/incoming |
|`logo_url`| A url to the logo of your platform. | https://partnerinc.com/assets/logo.png |

The following will be provided to you when you register your platform:

- Your `client_id`, a unique identifier for your platform, generated by Xref.
- Your `client_secret`, a unique secret for your platform, generated by Xref.

The `client_secret` should not be shared.

## Integration
### End user authorization

With these pieces of information on hand, you’re ready to have your end users connect to Xref within your platform. We recommend showing a Connect button that sends them to our authorize url endpoint. 

```
https://login.xref.com/authorize/:client_id/open?state=UNIQUE_TOKEN
```

To prevent CSRF attacks, you can use the `state` parameter, passing along a unique token as the value. We’ll include the `state` you gave us when we redirect back.

### Token issuance

When the end user arrives at Xref, they’ll be prompted to allow or deny the connection to your platform, and will then be sent to your `redirect_url` page. In the URL, we’ll pass along an authorization code:

```
https://partnerinc.com/integrations/xref?state=UNIQUE_TOKEN&code=AUTHORIZATION_CODE
```

Using the `code` parameter, you should make a POST request to our tokens endpoint to retrieve the `access_token`:

```curl
curl -X POST https://api-open.xref.com/oauth/tokens \
    --header "Content-Type: application/json" \
    --data-binary "{
        \"client_id\": \"YOUR_CLIENT_ID\",
        \"client_secret\": \"YOUR_CLIENT_SECRET\",
        \"code\": \"AUTHORIZATION_CODE\"
    }"
```

Xref will return a response containing the `access_token` for the integrated account:

```json
{
  "access_token": "cb6f9d8c843c22e53db97f7fd2ff20ab58c91e31",
  "xref_account_id": "d1e01e1a36a09eda2a31b01caf62eb2e"
}
```

You’re done! The end user has now authorized your platform to manage the account they belong to.  You’ll want to store all of the returned information in your database for later use.

### Make API calls using user tokens

You can now make requests to the [Xref Open API](https://xrefopenapi.docs.apiary.io) using the `access_token`.

```curl
curl -X GET https://api-open.xref.com/auth \
    -H "Authorization: Token ACCESS_TOKEN"
```

### User deauthorization

If you want to disconnect access to an integrated account, you can POST to `https://api-open.xref.com/oauth/deauthorize`:

```curl
curl -X POST https://api-open.xref.com/oauth/deauthorize \
   -H "Authorization: Token ACCESS_TOKEN"
```

## Xref Open API OAuth Reference

See the [Xref Open API OAuth Reference](https://github.com/xrefdev/openapi-oauth-docs/blob/master/openapi_oauth_reference.md) for a complete guide on the Xref Open API OAuth.
