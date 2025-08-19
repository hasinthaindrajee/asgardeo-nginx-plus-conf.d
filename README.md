# nginx-openid-connect with Asgardeo


## Configuring your IdP

  * Create an OpenID Connect client to represent your NGINX Plus instance
    * Choose the **authorization code flow**
    * Set the **redirect URI** to the address of your NGINX Plus instance (including the port number), with `/_codexch` as the path, e.g. `http://localhost:80/_codexch`
    * Ensure NGINX Plus is configured as a confidential client (with a client secret) or a public client (with PKCE S256 enabled)
    * If NGINX Plus is configured as a confidential client, choose the appropriate authentication method: **client_secret_basic** or **client_secret_post**.
    * Make a note of the `client ID` and `client secret` if set
    * Set the **post logout redirect URI** to the address of your NGINX Plus instance (including the port number), with `/_logout` as the path, e.g. `http://localhost:80/_logout`

  * If your IdP supports OpenID Connect Discovery (usually at the URI `/.well-known/openid-configuration`) then use the `configure.sh` script to complete configuration. In this case you can skip the next section. Otherwise:
    * Obtain the URL for `jwks_uri` or download the JWK file to your NGINX Plus instance
    * Obtain the URL for the **authorization endpoint**
    * Obtain the URL for the **token endpoint**
    * Obtain the URL for the **end session endpoint**

## Configuring NGINX Plus

Configuration can typically be completed automatically by using the `configure.sh` script.

Manual configuration involves reviewing the following files so that they match your IdP(s) configuration.

  * **openid_connect_configuration.conf** - this contains the primary configuration for one or more IdPs in `map{}` blocks
    * Modify all of the `map…$oidc_` blocks to match your IdP configuration
    * Modify the URI defined in `map…$oidc_logout_redirect` to specify an unprotected resource to be displayed after requesting the `/logout` location
    * Set a unique value for `$oidc_hmac_key` to ensure nonce values are unpredictable
    * If NGINX Plus is deployed behind another proxy or load balancer, modify the `map…$redirect_base` and `map…$proto` blocks to define how to obtain the original protocol and port number.

  * **frontend.conf** - this is the reverse proxy configuration
    * Modify the upstream group to match your backend site or app
    * Add proxy_set_header urn:aws:givennameclaim $jwt_claim_given_name so that you will get given name sent from Asgardeo as "urn:aws:givennameclaim"
   
  * **openid_connect.server_conf** - this is the NGINX configuration for handling the various stages of OpenID Connect authorization code flow
    * No changes are usually required here
    * Modify the `resolver` directive to match a DNS server that is capable of resolving the IdP defined in `$oidc_token_endpoint` and `$oidc_end_session_endpoint`
    * If using [`auth_jwt_key_request`](http://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt_key_request) to automatically fetch the JWK file from the IdP then modify the validity period and other caching options to suit your IdP

  * **openid_connect.js** - this is the JavaScript code for performing the authorization code exchange and nonce hashing
    * No changes are required unless modifying the code exchange or validation process

