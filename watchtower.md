# Watchtower

## Goals with this project:
* Have a service that automatically updates my containers

## The process:
I created the following stack:
```
services:
  watchtower:
    image: nickfedor/watchtower:latest
    container_name: WATCHTOWER
    hostname: watchtower
    mem_limit: 512m
    mem_reservation: 128m
    cpu_shares: 512
    security_opt:
      - no-new-privileges=true
    read_only: true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      TZ: America/New_York
      WATCHTOWER_CLEANUP: true # Remove old images after updating
      WATCHTOWER_REMOVE_VOLUMES: false # Remove attached volumes after updating
      DOCKER_API_VERSION: 1.45 # SSH docker version 1.41 for Docker engine version 20.10 - 1.43 for Docker engine version 24 - 1.45 for Docker engine version 26.1
      WATCHTOWER_INCLUDE_RESTARTING: true # Restart containers after update
      WATCHTOWER_INCLUDE_STOPPED: false # Update stopped containers
      WATCHTOWER_SCHEDULE: "0 0 */2 * * *" # Update & Scan containers every 2 hours
      WATCHTOWER_LABEL_ENABLE: false
      WATCHTOWER_ROLLING_RESTART: false #or false if you are using containers that have depends_on.
      WATCHTOWER_TIMEOUT: 30s
      WATCHTOWER_LOG_FORMAT: pretty
    restart: on-failure:5
```

As soon as I created this stack, Nginx Proxy Manager went down. I stopped the Watchtower container in order to troubleshoot. 
Apparently, version 17 of postgres was out of date, and the switch to version 18 required me to back up the database, 
stop the original container, delete the container, update both the version and the mount location, spin up the new container, 
then restore the data from the backup. After that, I restarted Watchtower.