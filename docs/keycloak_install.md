# Keycloak install

To install keycloak, it is ran inside docker together with postgres. You may check and use the Docker compose files in [keycloak/](../keycloak). 

## Requirements

- Docker with docker compose support
- At least 1-2GB of RAM allocated indirectly for Keycloak + Postgres

## Setup 

Install by cloning this repository, change the .env.example to .env and populate with the relevant details. You may put realm exports into auth/import for Keycloak to configure itself automatically. The network within that Keycloak is reachable directly is `badgehub_network`, otherwise it is exposed to listen locally on port 8080. 

## Run

Then run docker-compose up -d (or whichever syntax is supported on your distribution).

If you want to use a custom env file like the one for local development, then run:

`docker compose --env-file .env.local up -d`

For production:

`docker compose up -d`

To configure Keycloak further, check [keycloak_configure.md](./keycloak_configure.md)


## Nginx reverse proxying

In one instance, we run Keycloak behind Nginx reverse proxy. This does not have to be used for local development. These are the settings used:

```nginx
server {
    listen 443 ssl;
    server_name keycloak.domain.example;
    ssl_certificate /etc/letsencrypt/live/cert_name_example/fullchain.pem; 
    ssl_certificate_key /etc/letsencrypt/live/cert_name_example/privkey.pem; 

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Scheme $scheme;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port $server_port;
    }
}


```
