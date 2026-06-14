# Heimdall

Goals with this container:
* Make it easier to keep track of all my links to different containers


## The process:
Created a stack using Portainer:
```
services:
 heimdall:
    image: ghcr.io/linuxserver/heimdall:latest
    container_name: Heimdall
    healthcheck:
      test: timeout 10s bash -c ':> /dev/tcp/127.0.0.1/80' || exit 1
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 90s
    restart: on-failure:5
    volumes:
      - /volume1/docker/heimdall:/config:rw 
    environment:
      TZ: Xxxx/Xxxx
      PGID: Xxxx
      PUID: Xxxx
    ports:
      - 8056:80
      - 7543:443
```