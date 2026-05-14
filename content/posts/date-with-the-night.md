---
title: A Date With the Night
date: 2026-05-14
---
+++
cover:
  image: "</media/pc-on-bench-at-night-edited.webp>"
  alt: "<alt text>"
  caption: "<text>"
  relative: false # To use relative path for cover image, used in hugo Page-bundles
+++

### Late nights with server upgrades and troubleshooting

![](/media/pc-on-bench-at-night-edited.webp)

When your homelab becomes a production server and your users come to expect those services, it’s hard to find a good time for maintenance. In addition to that, when server hardware is being upgraded and parts being swapped out, downtime is a bit more extensive than simply spinning up services on new hardware. That was the situation that I ran into.

Previous Hardware:

- Intel i3-9100
- 24GB DDR4
- ASRock motherboard

New Hardware:

- Intel Core 5 Ultra 235
- MSI Pro B860M-A WiFi motherboard
- 16GB DDR5 ($$$)

The Intel i3-9100 was getting long in the tooth for the services I wanted to run. I was also running baremetal Ubuntu and wanted to take Proxmox for a spin to add more flexibility to my setup once I was on new hardware with additional headroom. I chose to upgrade to an Intel Core 5 Ultra 235. It gave me some headroom and fit the budget. I paired it with an MSI Pro B860M-A WiFi motherboard and was prepared to swap it in to my existing system. I bought 16GB of RAM right as prices started creeping up. I could use some more but that is probably a sentiment that most people have these days.  

I anticipated my biggest issues would be getting Proxmox running as I only had played with it a little on an older box I had sitting around with limited resources so I wasn’t able to spin up too many things. Boy, was I wrong.

After finding a couple nights where downtime wouldn’t be much of an issue, I pulled the computer out of the rack, set it down on the table and started gutting. I pulled out the old CPU and motherboard and installed the new one. Got it all wired up and…nothing. It wouldn’t turn on. Dead as a doornail. After several hours of troubleshooting and research, doing the paperclip test, verify the power supply was still working, etc., I realized that my old ATX v2.x power supply just might not be up to the task of powering the Ultra 235 unlike a newer 3.x ATX power supply. The fans wouldn’t kick over. The LED’s on the motherboard wouldn’t light. There were no beeps from the motherboard (I installed a speaker). It appeared that the safeguards on the power supply for over current protection were likely being triggered by the quick current spikes the Ultra 235 was pulling. A day later, with a new power supply ready to go, I was able to get the machine booted up.

I had a rescuezilla image of the entire baremetal system. I was able to set up Proxmox, mount a rescuezilla ISO and a drive with the image and restore the image into the VM. I had to update some IP addressing but most of the services came back up after I had worked through hardware pass-through issues.

