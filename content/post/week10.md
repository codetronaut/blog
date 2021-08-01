---
title: "GSoC (Week 10)"
date: "2021-08-01"
author: "Anmol"
tags: 
  - gsoc
  - agl
---

***Mentee Name: Anmol***



***Mentors: Jan-Simon Möller and Marco Solieri***


***Gist about advancements in this week:***


### Week-10: Upstreaming some initial code changeset.


**Current Status**

This week I worked on creating a changeset for the Jailhouse cell and some parts of the meta-agl-jailhouse layer, and this patchset includes the following changes:

- Updated Jailhouse root-cell(qemu-agl.c), apic-demo(agl-apic-demo.c), ivshmem-demo(agl-ivshmem-demo.c).
- Jailhouse non-root cell(agl-linux-x86-demo.c) is also working but needs some more tweaks for UART redirection.
- Linux Kernel updated to the latest `queues/jailhouse` kernel branch.
- Updated the `recipes-kernel` structure to a more custom one.
- Updated conf/local.conf according to the new `recipes-kernel` structure.
- Removed the Linux Kernel patches from the `recipes-kernel` as the updated kernel already contains those patches.

[Here's the Link to the changeset](https://gerrit.automotivelinux.org/gerrit/c/AGL/meta-agl-devel/+/26546).

After sending patches I also received some useful feedback and reviews regarding the patch, like adding more readability and comments to some of the parts in the configuration files and many more related to this. 

I also did some changes in the `50_local.conf.inc`, and also did some mistakes by adding `IMAGE_INSTALL_append` and `EXTRA_IMAGE_FEATURES` changes as, these should not be in this template, and if needed I should add them at the initial build time, so removed these from the conf file. And to get this changeset merged I also need to work on the reviews from maintainers. Right now one review from Scott Murray sir is still unresolved and I am finding ways to resolve it. The issue is as follows: "I have used Kernel(queues/jailhouse from git.kiszka.org) which is a fork of a kernel that's not even released yet, as it's destined to be a maintenance hassle". I agree with the issue but right now we do not have a working kernel(other than queues/jailhouse) that is working perfectly with the AGL_Jaihouse, So yes I along with maintainers working on the right way to resolve this issue.

I also replied to the mail from the jailhouse community and this time I explained to them about the non-root Linux cell loading we are facing right now. Here’s the [Link](https://groups.google.com/g/jailhouse-dev/c/_Um9Zwz1uC4/m/dHAdFLT8AwAJ). In the meantime I am trying to get IVSHMEM-demo up in my local setup with `jailhouse-images`, I was able to successfully set up the SSH and also `scp`ied the required files to the QEmulated instance, but after two-three runs its kernel is not booting, I am not sure about the reason, but this is happening after replacing it’s kernel to the recent one i.e 5.14-rc2. I am working to resolve it as it’s the only setup I have in which the `linux-x86-demo` is working properly( even with 5.14-rc2 kernel but only able to boot 2-3 times after that it’s stuck).

**Next Steps**

Next week I am expecting a reply from the jailhouse community, and also expecting that the issue we are facing right now would be resolved, and from there we will start working on the virtio part.


Thanks for reading.


**-- Anmol**
