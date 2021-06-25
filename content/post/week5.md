---
title: "GSoC (Week 5)"
date: "2021-06-25"
author: "Anmol"
tags: 
  - gsoc
  - agl
---

**Mentee Name: Anmol**



**Mentors: Jan-Simon Möller and Marco Solieri**


**Gist about advancements in this week:**


This week I researched about the virtio-block integrations and also learned how I can take inspiration from already implemented ones. I looked at some of the implementations and these include `spdk` and `acrn`, and I learned many things from them, although I need to deep dive more into the remaining commits, and link to them are [`Link for ACRN`](https://github.com/projectacrn/acrn-hypervisor/search?q=virtio-block&type=commits), [`Link for dpdk`](https://github.com/spdk/spdk/search?o=asc&q=virtio-blk&s=committer-date&type=commits).

Some of the implementations are already present in the `queues/jailhouse` branch in the git.kiszka.org, and some detailed information is present in the [`inter-cell-communication`](https://github.com/siemens/jailhouse/blob/master/Documentation/inter-cell-communication.md). But everything which is written thhere it is not directly implementable in the `jailhouse-images`.


So as I am working with the `jailhouse-images`, there are some things I need to do before testing the [`virtio_ivshmem_block`](http://git.kiszka.org/?p=linux.git;a=blob;f=tools/virtio/virtio-ivshmem-block.c;h=c97aa5076a6d22ccd01862f3e4db0e12641825c3;hb=refs/heads/queues/jailhouse), because I can not directly try this driver with already upstream kernel in available with `jailhouse-images`, because it’s default branch is [master](http://git.kiszka.org/?p=linux.git;a=shortlog;h=refs/heads/master), which has kernel version 5.10.x, and this particular driver code is available in the `queues/jailhouse`, so I need to first modify the kernel recipe files, and then maybe I will be able to spin it. This whole procedure for spinning and testing the driver would be the first step and then after its testing results and analysis, I will start working on the RTOS one.

Also, for the QEmulation for the problem, I was facing for some week, now I have sent the mail regarding this in the jailhouse mailing list, and you can see it [here](https://groups.google.com/g/jailhouse-dev/c/_Um9Zwz1uC4).


Thanks for reading.


**-- Anmol**

