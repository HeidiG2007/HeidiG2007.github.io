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