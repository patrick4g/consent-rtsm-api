# Authentication
## RTSM
```shell
curl --request POST "https://{sponsor-url}.4gclinical.com/_/api/oauth/token" \
  --json '{
      "client_id": "{partner code}",
      "client_secret": "{partner secret}",
      "grant_type": "bearer"
  }'
```

> Make sure to replace `{partner code}` and `{partner secret}` with your credentials.

> The above command returns JSON structured like this:

```json
{
    "access_token": "fsVjdlCDAfwenfr43klfkl3...2er23",
    "token_type": "bearer",
    "expires_in": 1800,
    "scope": "econsent"
}
```

The RTSM API uses OAuth2's Client Credentials Flow to authenticate requests.

4G Clinical will assign unique `partner code` and `partner secret` for approved partners.  Separate credentials will be created for different environments (i.e. dev, testing, production, etc.)

### HTTP Request

`POST https://sponsor-url.4gclinical.com/_/api/oauth/token`

To authenticate, the partner should send `partner code` and `partner secret` to the <b>Oauth Token</b> endpoint.
The API verifies the credentials and return an `access token`, valid for a limited time (usually ~30 minutes).
A renewed `access token` can be retrieved at any time by calling the Oauth Token endpoint again.

The API expects a valid `access token` to be included in all subsequent API requests in a HTTP header:

`Authorization: Bearer {access token}`

<aside class="notice">
You must replace <code>{access token}</code> with the token returned by the oauth endpoint.
</aside>

## ECONSENT

TBD (Basic Authentication?)
