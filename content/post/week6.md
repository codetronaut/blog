---
title: "GSoC (Week 6)"
date: "2021-07-02"
author: "Anmol"
tags: 
  - gsoc
  - agl
---

**Mentee Name: Anmol**



**Mentors: Jan-Simon Möller and Marco Solieri**


**Gist about advancements in this week:**


This week I tried the QEmulated image again with the queues/jailhouse branch kernel but with different Kernel configuration i.e 5.10.xx, and It worked. But again when i tried to spin the root cell it’s again showing errors related to addressing. So, me and Jan-Simon sir tried to resolve it, by changing the static addressing configurations, it’s default was `0x3a000000` so we tried to change it with the available memory by looking at `/proc/iomem`, and then replaced it in the `qemu-x86.c` root cell configuration and also changed the address of debug console to available one. But it didn’t work out.

Another thought of mine was, manually cloning the `jailhouse` repo apart from the yocto build and then compiling the root cell there and `scp` it to the QEmulated image to get it work, but when i tried to run the `$ systemctl --type=service --all`, I found that `sshd.service` was not there. So, Jan sir helped me in spinning the network and ssh services and now it’s working fine. Note: by default `core-image-minimal` does not come with `sshd.service` loaded and enabled, we have to  manually install the `openssh` by modifying the `conf/local.conf`.


Also some of the things we analyzed: that the memmap value in the `*-qemuboot.conf` always shows `memmap=82M\$0x22000000` on first build, and Jan-Simon sir told me that `\` should not be there by default, so now i will try to patch this as soon as possible so that in the next build this will not occur.

Also, next tasks would be to first figure out the proper memory addressing for the cell configurations and also manually type the address manually in the configuration of the cells, and then will try to spin the root cell.  


Thanks for reading.


**-- Anmol**
