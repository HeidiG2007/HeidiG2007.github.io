---
layout: page
title: Vaultwarden
---
# Vaultwarden

## The process
I created the following stack on Portainer:

```
services:
  db:
    image: postgres:18
    container_name: Vaultwarden-DB
    hostname: vaultwarden-db
    security_opt:
      - no-new-privileges:true
    healthcheck:
      test: ["CMD", "pg_isready", "-q", "-d", "vaultwarden", "-U", "vaultwardenuser"]
      timeout: 45s
      interval: 10s
      retries: 10
    volumes:
      - /volume1/docker/vaultwarden/db:/var/lib/postgresql:rw
    environment:
      POSTGRES_DB: vaultwarden
      POSTGRES_USER: vaultwardenuser
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    restart: unless-stopped
    
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: Vaultwarden
    hostname: vaultwarden
    security_opt:
      - no-new-privileges:true
    user: 130:137
    ports:
      - 4080:4020
    volumes:
      - /volume1/docker/vaultwarden/data:/data:rw
    environment:
      ROCKET_PORT: 4020
      DATABASE_URL: postgresql://vaultwardenuser:${POSTGRES_PASSWORD}@vaultwarden-db:5432/vaultwarden
      ADMIN_TOKEN: ${ADMIN_PASSWORD}
      DISABLE_ADMIN_TOKEN: false
      DOMAIN: https://vaultwarden.heidigee.duckdns.org:6443
      SMTP_HOST: smtp.gmail.com
      SMTP_FROM: ${EMAIL}
      SMTP_PORT: 587
      SMTP_SECURITY: starttls
      SMTP_USERNAME: ${EMAIL}
      SMTP_PASSWORD: ${SMTP_PASSWORD}
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
```

Then I created the proxy host vaultwarden.heidigee.duckdns.org and applied the TLS certificate. When I tried to access 
it, I received error 502 bad gateway from openresty. While solving this issue, I learned that I'm not supposed to set 
the IP address to https in NPM. After that, I was able to access the service and set up an account on Vaultwarden. After 
that, I went to the admin page, disabled new signups, and saved the changes to the settings.

For a more secure admin login, I generated a password hash through the Vaultwarden container CLI using the command 
`/vaultwarden hash` to produce a hash of a password, then putting the generated hash under admin token in the general 
settings and saving the changes. 