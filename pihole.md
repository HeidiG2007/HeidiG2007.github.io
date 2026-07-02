---
layout: page
title: Pi-hole & Unbound
---
# Pi-hole

## Goals with this project:
* Do something more cybersecurity-related with containers


## The process:
I set up the following stack:

```
services:
  pihole:
    image: pihole/pihole
    container_name: Pi-Hole
    network_mode: host
    security_opt:
      - no-new-privileges:false
    restart: on-failure:5
    volumes:
      - /volume1/docker/pihole/dnsmasq.d:/etc/dnsmasq.d:rw
      - /volume1/docker/pihole/pihole:/etc/pihole:rw
    environment:
      FTLCONF_webserver_api_password: Xxxx
      FTLCONF_webserver_port: 8080
      FTLCONF_dns_listeningMode: all
      TZ: Xxxx/Xxxx
      DNSMASQ_USER: pihole
      PIHOLE_UID: 130
      PIHOLE_GID: 137
    cap_add:
      - SYS_TIME
      - SYS_NICE
```

After trying to set this up a few times and failing, I did some research and learned that on Ubuntu machines, there's a
service that runs on port 53 called systemd-resolved that you have to turn off in order to run Pi-hole. Went and edited 
the configuration file and tried again, but that still didn't work. Realized I forgot to restart the service so the 
changes hadn't taken effect. Pi-hole successfully deployed after that.

## Testing

I used the following site to test the adblocker's efficacy.

[adblock.turtlecute.org](https://adblock.turtlecute.org/)

Since the network I'm using already has an adblocker, I decided to compare the difference 
between just the current adblocker vs the combination of both the adblocker and the pi-hole.

* Sans Pi-hole: 85%
* Both adblocker and Pi-hole with default list: 92%

Because the test gets a higher number of ads blocked with the Pi-hole, it confirms that the Pi-hole is functional.

# Unbound

## Goals with this project:
* Familiarize myself with networks on Portainer

## The process:
I created a new internal bridge network named proxy via Portainer's UI. Then I modified the Pihole stack, adding the 
Unbound service and putting both on the new internal network:

```
networks:
  proxy:
    external: true

services:
  pihole:
    container_name: pihole
    hostname: pihole
    image: pihole/pihole:latest
    networks:
      proxy:
        ipv4_address: 172.26.0.7
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8380:8380/tcp"
    #- "443:443/tcp"
    environment:
      TZ: 'America/New_York'
      FTLCONF_dns_upstreams: '172.26.0.8#53'
      FTLCONF_webserver_api_password: ${WEBSERVERPASS}
      FTLCONF_webserver_port: 8380
      FTLCONF_dns_listeningMode: all
      DNSMASQ_USER: pihole
      PIHOLE_UID: 130
      PIHOLE_GID: 137
    volumes:
      - '/home/ubuntu/docker/pihole/etc-pihole/:/etc/pihole/'
      - '/home/ubuntu/docker/pihole/etc-dnsmasq.d/:/etc/dnsmasq.d/'
    cap_add:
      - SYS_TIME
      - SYS_NICE
    restart: unless-stopped

  unbound:
    container_name: unbound
    image: mvance/unbound:latest
    networks:
      proxy:
        ipv4_address: 172.26.0.8
    volumes:
      - /home/ubuntu/docker/unbound:/opt/unbound/etc/unbound
    ports:
      - "5053:53/tcp"
      - "5053:53/udp"
    restart: unless-stopped
```

Then I went into the VM, navigated to /home/ubuntu/docker/unbound and created 3 empty files: 
`a-records.conf`, `srv-records.conf`, and `forward-records.conf`.

Created root.hints via command `curl -o root.hints https://www.internic.net/domain/named.root` on Ubuntu VM.

