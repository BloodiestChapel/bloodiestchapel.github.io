---
title: "Passing Through an Intel GPU on Proxmox"
date: 2026-07-17
draft: true
tags: ["homelab", "proxmox", "jellyfin", "gpu", "docker", "linux"]
description: "Getting hardware transcoding working on Jellyfin after moving from an NVIDIA card to an Intel Arc A770, and what it took to fix OpenCL initialization failures along the way."
---
# Passing Through an Intel GPU on Proxmox
## Why an Intel Card

My media stack has been running on Proxmox for a while, and up until recently it leaned on an NVIDIA GPU for hardware transcoding in Jellyfin. NVIDIA passthrough works, but it comes with baggage: proprietary driver installs on both the host and inside containers, licensing quirks around concurrent transcode sessions on consumer cards, and a driver stack that tends to break in small, annoying ways after kernel updates.

As I was going through the woes of NVIDIA GPU I took a look at an Intel Arc A770 16GB I picked up a while ago. It was mostly used as a work GPU for a PC in my basement and sometimes for light gaming but I felt it was being underutilized considering it is capable of Intel's QSV (Quick Sync Video), AV1, H.264 and HEVC transcoding. The drivers are also all open and mainlined into the kernel, and Jellyfin has solid support for it through VAAPI. On paper, this was what I've been looking for to replace my old 8GB NVIDIA card.

## The Move: VAAPI Instead of NVENC

The first part of the migration was mechanical. I updated the Docker Compose files across my media containers (Jellyfin, Plex, and a few supporting services) to drop the NVIDIA runtime and device reservations, and instead map in the Arc card's render node directly:

```yaml
devices:
  - /dev/dri/renderD128:/dev/dri/renderD128
group_add:
  - "104"  # render group GID
  - "44"   # video group GID
```

The important detail here is using the actual numeric GIDs for the `render` and `video` groups from the Proxmox host, not just the group names. Docker doesn't always resolve group names the same way inside the container as the host does as I quickly found out, so pinning the GIDs avoided a class of permission errors I was getting otherwise.

With that in place, Jellyfin picked up the device fine and VAAPI transcoding looked healthy on paper. But when I actually tried to use hardware-accelerated transcoding by playing the original masterpiece `House on Haunted Hill` or anything routed through the tone-mapping or OpenCL-based filters Jellyfin uses for certain HDR content, I got big fat errors on screen.

## The Failure: OpenCL Wouldn't Initialize

At this point we move to the logs which pointed to OpenCL failing to initialize on the Arc device. VAAPI itself was working, but the OpenCL layer that Jellyfin's ffmpeg build relies on for some of its more advanced transcoding paths (tone mapping in particular) couldn't find a usable OpenCL platform for the card.

After some research, I found this is a common gap with Arc GPUs: VAAPI support and OpenCL support are separate driver stacks, and having one working doesn't guarantee the other is installed or configured. The container had the render node mapped in and could see the device, but it was missing the actual OpenCL runtime packages needed to talk to it.

## The Fix: A Docker Mod

Rather than baking custom packages into a hand-rolled image and maintaining it myself, I used some Docker mods I found through the interwebs to inject the missing dependencies at container start. The packages that closed the gap were:

- `intel-opencl-icd` – the Intel Compute Runtime OpenCL ICD, which is the actual driver that lets OpenCL applications talk to the Arc GPU
- `libze-intel-gpu1` – the Level Zero driver for Intel GPUs, which some newer Intel media/compute paths depend on
- `libze1` – the base Level Zero loader library that `libze-intel-gpu1` needs to function

Once those were installed inside the container via the mod, OpenCL initialization succeeded and the tone-mapping and other OpenCL-dependent transcode paths started working correctly. Turns out VAAPI had been fine the whole time; it was specifically the OpenCL stack that needed the extra packages.

## Lessons Learned

A few things stuck with me from this one:

**"The GPU is visible" doesn't mean "everything that needs the GPU works."** VAAPI and OpenCL are separate stacks on Intel hardware, and each can succeed or fail independently. Just because `vainfo` or basic hardware transcoding looks fine doesn't mean every feature that touches the GPU will work.

**Docker mods are a clean way to handle driver-layer gaps.** Rather than maintaining a custom Jellyfin image just to bolt on a few packages, a mod lets you keep using the upstream image and layer in exactly what's missing at container start. Easier to maintain, easier to roll back.

**Pin your GIDs explicitly when passing through hardware devices in Docker.** It's a small thing, and may not be necessary for all deployments, but it removed an entire category of permission issues.

The end result is a media stack that transcodes faster than the old NVIDIA setup, draws less power, can support however many concurrent transcodes it can handle, and doesn't require chasing proprietary driver updates. Worth the afternoon it took to track down the OpenCL gap and definitely worth the quality/bandwidth improvements from the AV1 codec.

My next veture will be to implement this card for AI workloads as well. I will be exploring Felix Kjellberg's project [Odysseus](https://odysseusai.dev/) to play with AI in meaningful ways that accelerate how well I can work without relying on 3rd parties for the AI horsepower.
