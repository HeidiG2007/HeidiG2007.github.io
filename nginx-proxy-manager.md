# Nginx Proxy Manager

## Goals for this project:
* Learn to use environment variables in Portainer
* Get a domain name for easier remembering
* Get my other websites encrypted via TLS

## The process:

### 1. Set up Nginx Proxy Manager container:
Created a stack on Portainer:
```
services:
  app:
    image: 'jc21/nginx-proxy-manager:2.15.1'
    restart: unless-stopped
    ports:
      # These ports are in format <host-port>:<container-port>
      - '6080:80' # Public HTTP Port
      - '6443:443' # Public HTTPS Port
      - '6081:81' # Admin Web Port
    environment:
      TZ: "America/New_York"
      # Postgres parameters:
      DB_POSTGRES_HOST: 'db'
      DB_POSTGRES_PORT: '5432'
      DB_POSTGRES_USER: 'npm'
      DB_POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      DB_POSTGRES_NAME: 'npm'
      # Uncomment this if IPv6 is not enabled on your host
      # DISABLE_IPV6: 'true'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    depends_on:
      - db

  db:
    image: postgres:17
    environment:
      POSTGRES_USER: 'npm'
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: 'npm'
    volumes:
      - ./postgres_data:/var/lib/postgresql/data
```

I created the stack, and the Nginx proxy manager container instantly failed. I forgot to define the environment variable 
that I'd set. Went back in, defined it, updated the stack, and it successfully deployed. Had to change my 

### Domain name:

After some research, I got a free subdomain name through Duck DNS, then followed their install instructions on the 
Ubuntu VM to catch any IP changes.

### Let's Encrypt TLS certificate:

I went through NPM's certificate UI to generate a certificate via a DNS challenge. The first attempt timed out, so I put 125 under 
Propagation Seconds and reran. That cleared up the issue. After that, I made a proxy host, but was winding up with errors because 
I didn't specify the Nginx Proxy Manager port, as it isn't the default port 80 and 443. Upon specifying port 6443, I was able to successfully 
access the sites that I set up proxy hosts for.

### Updating PostgreSQL Container

I had to upgrade Postgres from version 17 to 18. Here's what I had to do:
1. Enter the CLI of the Postgres 17 container and create a backup of the database using `pg_dumpall -U npm > /var/lib/postgresql/data/backup_17_to_18.sql`, then `exit` the interface 
2. Stop and delete the Postgres container under both containers and also beneath the stack. 
3. Edit the stack, changing the version from 17 to 18 and changing the volume location from /var/lib/postgresql/data to /var/lib/postgresql
4. Enter the CLI of the Postgres 18 container and restore from the backup: `psql -U npm -f /var/lib/postgresql/data/backup_17_to_18.sql`
5. If you can't find the backup, `find /var/lib/postgresql -name "*.sql"`
6. Check that the backup worked: `psql -U npm -c "\l"`
7. Remove the backup: `rm /var/lib/postgresql/data/backup_17_to_18.sql`
