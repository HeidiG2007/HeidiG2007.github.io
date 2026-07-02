---
layout: page
title: it-tools
---
# it-tools

## Goals with this project
* Have a set of self-hosted easily accessible tools for developers.

## The process
Made the following stack:

```
services:
  it-tools:
    container_name: IT-TOOLS
    image: corentinth/it-tools
    healthcheck:
     test: curl -f http://localhost:80/ || exit 1
    mem_limit: 2g
    cpu_shares: 768
    security_opt:
      - no-new-privileges:true
    ports:
      - 5545:80
    restart: unless-stopped
```