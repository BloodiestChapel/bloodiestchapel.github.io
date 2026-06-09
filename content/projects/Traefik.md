---
title: "Traefik"
date: 2026-06-09
draft: false
tags: ["homelab", "traefik"]
---
This will my first project post. As such I'm posting about something which has become particularly important in my homelab. It's called ***Traefik*** and has helped me organize everything in my homelab and make sure things are secure.

The whole point of Traefik is that it can assign a website to your internal (or external with some extra work) services. This wasn't really an issue for a long time, but once I started having to remember 30 different IPs, I knew something had to change. Enter in Jim's Garage's [Traefik V3 tutorial](https://www.youtube.com/watch?v=CmUzMi5QLzI&t=1232s) and I knew immediately I needed this for my homelab. 

As it goes into in his tutorial, another aspect of this is the usage of Pi-Hole as a dns sinkhole and also a DNS resolver. I had been toying for Pi-Hole for a while and this seemed like the perfect way to seriously utilize it.

The main thing needed for Traefik to work is to have a domain name, so I went to Porkbun.com and registered a new one for dirt cheap, spun up a free cloudflare account, and got to work. 

The beauty of Traefik lies in it's dynamic config. You simply open up the config file in your editor of choice, add a service and a router.

Ex:
```yaml
http:
  routers:
    my-router:
      rule: "Host(`example.com`)"
      service: my-service
      entryPoints:
        - websecure
      tls:
        certResolver: myresolver

  services:
    my-service:
      loadBalancer:
        servers:
          - url: "http://localhost:8080"
```

Once this is all done, you can lock down port 80 on your router and only have things accessible from port 443 (HTTPS). Obviously, you can specify VLANs or specific machines to have permissions for internal port 80, but I won't go into that here. 

I can't overstate how much easier this is, especially for people who aren't as tech literate living in the same home. Instead of giving someone an IP and a port to remember, just point them to something like <mark>coolsite.dillonchappell.io</mark> and they'll remember it easily and with no fuss.