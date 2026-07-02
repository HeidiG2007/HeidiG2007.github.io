---
layout: page
title: Calibre
---
# Calibre

## Goals with this project:
* Familiarize myself with Portainer

## The process:
Created a stack using Portainer:

```
version: "2.1"
services:
  calibre:
    image: lscr.io/linuxserver/calibre:latest
    container_name: calibre
    environment:
      - PUID=130
      - PGID=137
      - TZ=Xxxx/Xxxx
    volumes:
      - /opt/docker/Calibre:/config
      - /opt/media/books:/books
    ports:
      - 8080:8080
      - 8081:8081
      - 8181:8181
    shm_size: "1gb"
    restart: unless-stopped
```