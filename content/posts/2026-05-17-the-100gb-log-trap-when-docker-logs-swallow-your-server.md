---
title: "A 100GB Log Trap: When Docker Logs Swallow Your Server"
date: 2026-05-17
draft: true
cover:
  image: /media/AI-Logs image.png
---
I woke up the other day to a server that was acting strange. Some docker services appeared to be working fine while others were completely down and others still worked okay but reported as "unhealthy". It started with a couple select cameras in Frigate appearing offline, which usually means a quick container restart. But when I tried to kick Frigate back into gear to check the cameras, I was met with an error message along the lines of "Restart of container failed. No disk space available."

My 240GB VM storage drive, which usually sits comfortably with about 100GB of breathing room, was completely maxed out. Using `df -h` the system reported 0MB available on the VM OS drive.

### The Setup

My setup (at the time of writing) consists of an Ubuntu Server virtual machine hosted on Proxmox, configured with a 240GB virtual disk to manage storage requirements. This VM serves as the primary environment for a container stack focused on home automation and network monitoring. Within this virtualized environment, several key services are deployed as Docker containers, including Home Assistant for central control, Frigate for AI-powered video surveillance, Zigbee2MQTT to handle smart device communication, and about 35 other microservices.

### ~~Throwing Silicon at a Software Problem~~

The expectation was that my recent leap from an i3-9100 to the Intel Core 5 Ultra 235 meant I could stop worrying about "basic" overhead for a while. I figured with that much modern silicon, I could just throw containers at the wall and they’d stick. Also, in my migration to the Ultra 235

Docker logs were the silent killers of my homelab. Home Assistant is incredibly chatty. Over months of uptime, those tiny `.json` entries turned into a 100GB+ monster. When the OS drive hits 100%, Docker loses its mind. Containers start reporting as **unhealthy**, and because there is no room to write temporary files, even lightweight services just stop responding.

### Finding the Outlier

I knew something was eating the disk, but I needed to find the bloat without digging through every directory manually. I reached for `ncdu`, but with a catch: I have mass storage mounted for photo backups and database files, and I didn't want to wait an hour for a scan. I ran:

```
sudo ncdu --exclude /mnt
```

Navigating into `/var/lib/docker/overlay2/`, I found the black hole. A single folder was holding a massive `.json.log` file belonging to Home Assistant. I deleted the file via `ncdu`, but the space didn't come back immediately. Docker holds onto that file handle like a dog with a bone. I had to restart the Home Assistant container to finally release the storage. A quick `sudo df -h` confirmed the win, the drive was breathing again.

### Preventing the Log-pocalypse

To make sure this never happens again, I implemented log rotation. You can't just edit a running container; you have to stop it, kill it, and recreate it with specific flags to cap the file size.

I updated my run script to include the `--log-opt` flags. This tells Docker to "roll" the log once it hits a reasonable size and only keep a few backups.

```
--log-opt max-size=10m
--log-opt max-file=3
```

The `max-size=10m` keeps the files small, and `max-file=3` ensures I only ever have 30MB of logs at most.

### Final Thoughts

Moving from an **i3** to an **Ultra 235** gave me plenty of CPU cycles, but it didn't protect me from a runaway text file. No matter how much hardware you throw at a problem, software configuration is what keeps the lights on. If you haven't checked your `/var/lib/docker/` folder lately, do it before your smart home decides to take an unscheduled nap.