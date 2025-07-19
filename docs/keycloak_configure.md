
# Keycloak configuration

Here we assume you are running several subdomains for example:
- Keycloak: keycloak.domain.example
- Badgehub app: badgehub.domain.example

You can also run it locally using the `localhost` as the domain. 

## Local development

For local development you only need to:
- Create the client and redirect rules 
- Set the CORS settings

## CORS settings

For CORS settings, we add the header modifications inside keycloak instead of NGINX. We add them under:

`Realm settings -> Security defenses`

For example:

`frame-src 'self' https://keycloak.domain.example http://localhost:3000; default-src 'self'; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline'; img-src 'self'; connect-src 'self'; form-action 'self' https://badgehub.domain.example/ http://localhost:3000; frame-ancestors 'self'; object-src 'none';`


## Client creation 

Then under "Clients" you have to import `badgehub-local-client.json`. The secret of the client should match the one in .env.local for Keycloak.

We need to configure the client which will be used by the frontend.

1. In the left menu, select "Clients"
2. Below Client ID, select or create the client you use (see `KEYCLOAK_CLIENT_ID` in `.env` in the deployed folder)

- **Root URL:** https://badgehub.domain.example/
- **Base URL:** https://badgehub.domain.example/
- **Direct Access Grants**: OFF
- **Front-channel Logout** to enable logging out from the application: ON 
- **Client authentication**: off
- **Authorization**: off
- **Authentication flow**: check Standard flow, uncheck all others

### Valid Redirect URIs
Valid redirect URIs that we need are the URL of the BadgeHub frontend + '*'. Also add your local development URL
if you're developing locally, like `http://localhost:5173/*`. 

Below is every possible `redirect_uri` your front-end or API might send. Localhost is used for development:

http://localhost:3000/*
http://localhost:8081/*
http://localhost:8081/api/auth/callback/keycloak
http://localhost:5173/*
http://localhost:5174/*
https://badgehub.domain.example/*
https://badgehub.domain.example/api/auth/callback/*

- Valid post logout redirect URIs: same as Valid redirect URIs

### Web Origins

Web origins: same as Valid redirect URIs, but without a `*` and without a slash at the end. Domains allowed for CORS requests and silent token refresh:

http://localhost:8081
http://localhost:3000
http://localhost:5173
http://localhost:5174
https://badgehub.domain.example

### Advanced client settings

1. Go to Client settings (see above)
2. Select the Adveanced tab

- Proof Key for Code Exchange Code Challenge Method: S256

## Theming

To install the BadgeHub theme in Keycloak:

- Go to [keycloak-theme](https://github.com/BadgeHubCrew/badgehub-app/blob/6d2383819f68e00fd52a984dba8c81a27e0ea339/keycloak-theme/BadgeHub.zip) or [keycloak-theme](https://github.com/BadgeHubCrew/badgehub-app/blob/feature/keycloak-theme/keycloak-theme/BadgeHub.zip)
- Download BadgeHub.zip (see the download raw file icon)
- Unzip BadgeHub.zip
- Move the resulting BadgeHub directory to the Keycloak `themes` directory (`/opt/keycloak/themes`)
- Then, in Keycloak, in Realm settings on the Themes tab, you can select the BadgeHub theme

The theme is for the login page only.

## User registration

We allow user registration. This is enabled under: 

`Realm settings -> Login > tick "User registration" `

Users can then register accounts to be used in the BadgeHub application. Email in Keycloak has to be configured for this to work.

## Email setup

Note: If we want to test the SMTP connection we have to set the keycloak admin email to a real email. 

We use emails for user account verification. Keycloak can be set up to send emails. Settings are located under:

`Realm settings -> Email `

Under `Template` we enter the details of how the email sent looks like for verification. 

- From: An email from which it is sent from (obligatory)
- From display name: A user-friendly name for the 'From' address (optional).
- Reply-to: should be same as `From`
- Reply display name:  A user-friendly name for the 'Reply-To' address (optional).
- Envelope from: An email address used for bounces (optional).

Under the same page we have `Connection & Authentication`

- Host: Domain or IP of SMTP server
- Port: whichever is from the SMTP server we use, e.g. 587
- Encryption: Tick `Enable StartTLS`
- Authentication: username / password
