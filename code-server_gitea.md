---
layout: page
title: Code-server and Gitea
---
# Code-server and Gitea

Goals with this project:
* Create a proof-of-concept self-hosted code server
* Self-host the git repository for that code server

## The Process:

### Code-server
I modified the code-server compose file from [Marius Hosting](https://mariushosting.com/synology-install-code-server-with-portainer/) from Step 12: I removed the Proxy-Domain because I don't have a domain name.
```
services:
  codeserver:
    container_name: Code-Server
    image: ghcr.io/linuxserver/code-server
    healthcheck:
     test: curl -f http://localhost:8443/ || exit 1
    mem_limit: 6g
    cpu_shares: 768
    security_opt:
      - no-new-privileges:false
    ports:
      - 8377:8443
    volumes:
      - /volume1/docker/codeserver:/config:rw
    environment:
      TZ: Xxxx/Xxxx
      PUID: 130
      PGID: 137
      PASSWORD: Xxxx
      SUDO_PASSWORD: Xxxx
      DEFAULT_WORKSPACE: /config/workspace
    restart: unless-stopped
```

After that:
* Installed openjdk 21.0.10
* Installed Extension Pack for Java

### Gitea

Created the following stack:
```
version: "3"

networks:
  gitea:
    external: false

services:
  server:
    image: gitea/gitea:1.15.10
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
    restart: always
    networks:
      - gitea
    volumes:
      - ./gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - 3030:3000
      - 222:22
```

### Postgres
Gitea works with SQLite, but SQLite isn't super secure. I created a postgreSQL database stack:
```
services:  
  db:
    container_name: PostgreSQL
    image: postgres
    mem_limit: 256m
    cpu_shares: 768
    healthcheck:
      test: ["CMD", "pg_isready", "-q", "-d", "heidi_DB", "-U", "root"]
    environment:
      POSTGRES_USER: Xxxx
      POSTGRES_PASSWORD: Xxxx
      POSTGRES_DB: heidi_DB
    volumes:
      - /volume1/docker/postgresql:/var/lib/postgresql:rw
    ports:
      - 2665:5432
    restart: on-failure:5
  
  pgadmin:
    container_name: pgAdmin
    image: dpage/pgadmin4:latest
    user: 0:0
    mem_limit: 256m
    cpu_shares: 768
    healthcheck:
      test: wget --no-verbose --tries=1 --spider http://localhost:5050
    environment:
      PGADMIN_DEFAULT_EMAIL: Xxxx
      PGADMIN_DEFAULT_PASSWORD: Xxxx
      PGADMIN_LISTEN_PORT: 5050
    ports:
      - 2660:5050
    volumes:
      - /volume1/docker/postgresadmin:/var/lib/pgadmin:rw
    restart: on-failure:5
```

After creating both stacks, I entered Gitea for the first time and set up the service. I had some difficulty because
the video I based my setup off of claimed that I could enter whatever I wanted into the database name field, so I
initially tried to set up the database without using the database's actual name. In turn, the software told me that the 
database didn't exist. I eventually caught this, entered heidi_DB, and successfully set it up.

