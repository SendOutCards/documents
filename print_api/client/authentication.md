# SendOutCards Public API Tutorial

## Setup

First of all, you need to have a SendOutCards account. For that, you can go to https://app.sendoutcards.com/ and create it.

## Authentication

Authentication for the public api is done using OAuth2, more specifically the Authorization Code Flow with PKCE, specified in RFCs 6749 and 7636.

### Authorization Code Flow with PKCE

#### Prepare the required data

| Data                  | Process                                                                                                                                                                                                                                                                                                                      |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| code_verifier (PKCE)  | Generate a random 32-byte sequence and encode it using Base64URL into a 43-byte sequence. You should generate a new one for every authorization request                                                                                                                                                                      |
| code_challenge (PKCE) | Apply the SHA-256 hash to the code_verifier and encode it using Base64URL                                                                                                                                                                                                                                                    |
| client_id             | Get the client id of your application. For this, you have to contact us so we can create an Application for your system.                                                                                                                                                                                                     |
| redirect_uri          | Defines the URL the user will be redirected to with the authorization response. This should be an URI in your site (https://example.com/cb) or in case of a native app an URI that's registered for your app. You'll need to contact us so we can register the URI, but for testing purposes you can use http://localhost/cb |

#### Get the authorization code

Redirect the user to:
`https://www.sendoutcards.com/oauth2/authorize/?response_type=code&code_challenge={code_challenge}&code_challenge_method=S256&client_id={client_id}&redirect_uri={redirect_uri}`

### Get the Access Token

To get the `access_token`, make a POST request to `https://www.sendoutcards.com/oauth2/token/`, sending the following data as "Form URL Encoded":

- `client_id`
- `code`
- `code_verifier`
- `redirect_uri`: This must be the exact same as the original `redirect_uri`.
- `grant_type=authorization_code`

The response will be in JSON format and will include the following data:

- `token_type` and `access_token`: Can be used to authorize public API requests by setting the authorization header: `Authorization: {token_type} {access_token}`.
  
- `expires_in`: Lifetime of the `access_token` in seconds.
  
- `refresh_token`: Can be used to obtain new `access_tokens` without going through the whole flow after the current `access_token` has expired.

### Refresh the Token After It Has Expired

After the `access_token` has expired, you can refresh the token by sending a POST request to `https://www.sendoutcards.com/oauth2/token/` with the following data:

- `client_id`
- `grant_type=authorization_code`
- `refresh_token`

The response data will be the same as in the previous request.
