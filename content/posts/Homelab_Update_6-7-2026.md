---
title: "How's My Homelab?"
date: 2026-06-07
draft: false
tags: ["homelab", "devops"]
cover:
    image: "/images/banner_homeassistant.png"
    alt: "Banner"
    relative: false
---

# Foreword
This is my first *real* post on my website. Since it feels special, I figured I'd post about how my home network/lab is set up. 

I will say most of my knowledge comes from learning out of Youtube, so here are a few channels I usually look at when I'm trying to figure something out.
- [Jim's Garage](https://www.youtube.com/@Jims-Garage)
- [Christian Lempa](https://www.youtube.com/@christianlempa)
- [Raid Owl](https://www.youtube.com/@RaidOwl)
- [Hardware Haven](https://www.youtube.com/@HardwareHaven)

---
# Overview
Here's a general list of what I'm running in my home lab

## Virtualization

- Proxmox cluster (multi-host, VLAN-aware networking, ZFS storage after RAIDZ1 migration)

## Storage

- TrueNAS (completed RAIDZ1 migration)

## Networking

- UniFi stack with full VLAN segmentation: server, IoT, DMZ, management VLANs
- UXG Fiber as the gateway
- U7 Pro AP in the basement
- WireGuard VPN server on the UXG
- Pi-hole for DNS

## Containerization / Reverse Proxy

- Docker across multiple hosts
- Traefik reverse proxy
- Gluetun VPN container (routing arr stack traffic)

## Security / Access

- Zone-based firewall policies in UniFi

---
# Afterword
Although this is a general overview, I intend on giving a more in depth write up of these technologies in the future. This will be for anyone who decided to look at this and for my own sanity and memory.

Later!