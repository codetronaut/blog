---
title: "GSoC (Final Report AGL)"
date: "2021-08-21"
author: "Anmol"
tags: 
  - gsoc
  - agl
---

# Google Summer of Code 2021 Final Work Submission Report


**Name: Anmol**

**Organisation: The Linux Foundation**

**Sub-Organisation: Automotive Grade Linux (AGL)**

**Project: VirtIO-blk implementation with AGL on Jailhouse**

**Mentors: Jan-Simon Möller and Marco Solieri**


This is the final report for my project [`VirtIO-blk implementation with AGL on Jailhouse`](https://summerofcode.withgoogle.com/projects/#6432060576628736) in the Google Summer of Code 2021 under Automotive Grade Linux(The Linux Foundation).

## Project Deliverables

- Improve Jailhouse support for AGL Image.
- Improve the jailhouse Root-Cell and Non-Root Cells for the QEmulation.
- Add Support for spinning Virtio over the IVSHMEM Block with AGL Image + Jailhouse hypervisor.
- Document everything from "changes made" to "how to reproduce" the whole setup.

## Community Bonding

The community bonding period started on May 17, 2021, and my mentors and I started to work on the initials of the project such as its feasibility, missing parts, expected problems e.t.c and also discussed via zoom calls regarding the same. Over the weeks we discussed everything on either zoom call or via Signal. In the whole duration I attended two categories of meetings one is `AGL-Weekly-Developer-Call` and another is `AGL-Virtualization-Experts-Group-Meetings`, In the weekly developer meet I generally present what work I have completed over the week, and in the Virt-EG meeting I generally ask my doubts about the virtio-layer and discuss the progress of my project. Overall I would say that the weekly meets and interaction with the developers and community was the best part of this whole GSoC journey.

## Weekly Updates

Over the whole GSoC period I have documented all my progress through the weekly progress reports:

- [On My Blog](https://anmol.netlify.app/post/).
- [On AGL Mailing List](https://lists.automotivelinux.org/g/agl-dev-community/search?p=created%2C0%2C%2C1%2C2%2C0%2C0&q=anmol.karan123%40gmail.com).

## Commits / Change Requests

- [Gerrit #26484: meta-agl-jailhouse: Fix memmap command line parameter](https://gerrit.automotivelinux.org/gerrit/c/AGL/meta-agl-devel/+/26484) [Merged].
- [Gerrit #26546: meta-agl-jailhouse: Update the Jailhouse configuration files](https://gerrit.automotivelinux.org/gerrit/c/AGL/meta-agl-devel/+/26546) [Merged].
- [Gerrit #26567: [PATCH] meta-agl-jailhouse: Add support for virtio over IVSHMEM block](https://gerrit.automotivelinux.org/gerrit/c/AGL/meta-agl-devel/+/26567) [In-Review].


## Future Work

- In the initial proposal I have also planned to work on the RTOS(specifically Zephyr)-Guest inmates, but not able to because Jailhouse support for it is removed. But I will keep working on it after the GSoC period, maybe with some other RTOS.
  - Some Links to the relevant threads and commits for RTOS inmates which can be considered are:
    - [Zephyr as a Jailhouse inmate](https://groups.google.com/g/jailhouse-dev/c/2SPK29UYDO4/m/9ioyQaKWAgAJ).
    - [Running Zephyr on Jailhouse inmate Cell](https://github.com/zephyrproject-rtos/zephyr/blob/efccfcbfc91501cac2ede0541a48c338f2b0cae4/boards/arm64/nxp_ls1046ardb/doc/index.rst#programming-and-debugging).
    - [boards/x86_jailhouse: remove support for Jailhouse hypervisor](https://github.com/zephyrproject-rtos/zephyr/commit/f3611fdd0c8ca54a9f19bc56a14b4a2fdadaffe3#diff-bb9445fa64739ef6a5a6b59d520deb07).
    - [More Commits](https://github.com/zephyrproject-rtos/zephyr/search?p=2&q=jailhouse&type=issues).
- Current AGL+Jailhouse setup depends on the `queues/jailhouse` branch of the Linux Kernel from [git.kiszka.org](http://git.kiszka.org/?p=linux.git;a=shortlog;h=refs/heads/queues/jailhouse), and until they merge this branch into the master we need to regularly update the recipe files.

## Challenges
- During the initial phase I used to get a lot of build errors related to Yocto/AGL, but my mentor Jan-Simon-Möller helped me in resolving all those :).
- Gerrit's workflow is a bit different, but it was worth learning.
- Due to my mentors I was able to make a balance between Academics, GSoC, and a research project (Mentored by Prof. Marco Solieri), which was also related to Jailhouse but more inclined towards the Bare Metal Side.

## Learnings
- Learned about the AGL's applications, development workflow, and review process.
- Due to the involvement of the jailhouse Hypervisor, research work was increased, searching for relevant threads on the Jailhouse mailing list and asking related questions increased my understanding of Jailhouse hypervisor.
- I also covered the Virtio applications and their extensions, IVSHMEM(Inter-VM Shared Memory), and integration with various hypervisors such as ACRN.

## Acknowledgements
I would like to thank my mentors Jan-Simon Möller and Marco Solieri for their continued assistance and support during the past weeks. This project certainly wouldn’t have been possible without their valuable suggestions and the knowledge they shared. I’d also like to thank AGL-Developer community members.

It has been an amazing journey and I look forward to contributing to Automotive Grade Linux in the future.
