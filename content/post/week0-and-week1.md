+++
title = "GSoC (Week 0 and 1)"
date = "2021-05-31"
author = "Anmol"
description = "Weekly Project Report (Community Bonding Period)"
tags = [
	"gsoc",
	"agl",
]
categories = [
	"gsoc",
]
+++

**Mentee Name: Anmol**

This and coming articles will explain and follow my progress for the Google Summer of Code 2021 project [`VirtIO-blk implementation with AGL on Jailhouse`](https://summerofcode.withgoogle.com/projects/#6432060576628736).

**Mentors: Jan-Simon Möller and Marco Solieri**


**Gist about the sub-goals for this week:**


I have introduced myself to the community on the zoom call and also learned about AGL’s  ecosystem of development. This project’s initial and main target is to implement and port the virtIO block base for the Jailhouse with AGL, and the initial sub-goal was to set up the working environment and run the particular image with the Jailhouse on QEmu for x86-64. 

For setting everything up I took reference from:

[Link 1](http://old-docs.automotivelinux.org/docs/en/icefish/getting_started/reference/getting-started/app-workflow-image.html) and [Link 2](https://docs.automotivelinux.org/en/master/#0_Getting_Started/2_Building_AGL_Image/0_Build_Process/)

Initially, I had to clone the AGL repository (branch: master) and set up it to utilize and download the meta-agl-jailhouse which comes under meta-agl-devel.

```bash
anmol@Debian-95-stretch-64-minimal:~/workspace_agl/master$ tree meta-agl-devel/ -L 1
meta-agl-devel/
├── docs
├── LICENSE
├── LICENSE.GPL-2.0-only
├── LICENSE.MIT
├── meta-agl-devel.md
├── meta-agl-drm-lease
├── meta-agl-jailhouse
├── meta-egvirt
├── meta-oem-production-readiness
├── meta-speech-framework
├── README.md -> meta-agl-devel.md
└── templates
```

First I started with the full-fledged agl-demo-platform common target, but it took a lot of time and gave me some unknown build errors, and due to these I shifted my focus to agl-image-minimal common target and updated some configurations in the following files:

**- build/conf/local.conf**

```sh

# Extra Options
# For Virtualization
DISTRO_FEATURES_append = " virtualization"

# For saving Downloads
SSTATE_DIR = "/home/anmol/backup-downloads/shared-sstate"
DL_DIR = "/home/anmol/backup-downloads/shared-download"

IMAGE_INSTALL_append = " vim "
INHERIT += "rm_work" 

#Kernel modules
MACHINE_ESSENTIAL_EXTRA_RRECOMMENDS += "kernel-modules"
```

**- conf/bblayers.conf**
```sh

# Up-to-date meta-arm depends on meta-arm-toolchain
BBLAYERS =+ "${METADIR}/bsp/meta-arm/meta-arm-toolchain"
```

**- meta-agl-devel/meta-agl-jailhouse/recipes-extended/jailhouse/jailhouse_git.bb**
```sh

TARGET_CC_ARCH += "${LDFLAGS}"
```
The change I made in the `jailhouse_git.bb` needs some explanation. So, I was getting an error:

```

QA Issue: File /usr/sbin/jailhouse in package jailhouse doesn't have GNU_HASH (didn't pass LDFLAGS?) [ldflags]

```

And saw on yocto documentation that it is related to `Default Linker Hash Style Changed` and I need to add the lines which I mentioned above in the recipe to pass QA issues. So this above fix works only when my recipe does some compilation, but if I need to work with the precompiled binaries above fix might not work and below should work:

```sh

#For dev packages only
INSANE_SKIP_${PN} = "ldflags"
INSANE_SKIP_${PN}-dev = "ldflags"
```
More about the above in this [ Link ](https://www.yoctoproject.org/docs/latest/ref-manual/ref-manual.html#migration-2.2-default-linker-hash-style-changed).

After all the above configurations, I finally managed to  build the image, but after booting it with QEmu for x86-64, it was booting very differently and slowly with some TSC errors(most probably processor’s microcode  issue) and Out of memory errors. I will try to figure out why this was happening, as a wild guess I think it’s a memory
issue for the latter as before running the Jailhouse we also needed to set the bootargs to prototype the `rootfs` and `mem` fields, but for that QEmu needs to run at least once which is not happening. But I will try to sort it out soon. And, yes that's all for this article, and thank you for reading :). 


**-- Anmol**

