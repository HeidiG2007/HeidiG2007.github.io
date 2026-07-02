---
layout: page
title: Uptime Kuma
---
# Uptime Kuma

## Goals with this project
* Have a service that monitors the uptime of my other services

## The process

Made the following stack:

```
services:
  uptimekuma:
    image: louislam/uptime-kuma:2
    container_name: Uptime-Kuma
    hostname: uptimekuma
    mem_limit: 4g
    cpu_shares: 1024
    ports:
      - 3444:3001
    volumes:
      - /volume1/docker/uptimekuma:/app/data:rw
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      TZ: America/New_York
    restart: unless-stopped
    depends_on:
      uptime-kuma-db:
        condition: service_healthy

  uptime-kuma-db:
    image: mariadb:lts
    container_name: uptime-kuma-db
    environment:
      MYSQL_ROOT_PASSWORD: ${MARIADB_ROOT}
      MYSQL_USER: ${MARIADB_USER}
      MYSQL_PASSWORD: ${MARIADB_PASS}
      MYSQL_DATABASE: kuma
    volumes:
      - /volume1/docker/mariadb/mariadb-data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
```

Then I went to port 3444, selected MariaDB and entered the database information to connect the two. After that, 
I created an administrator account.