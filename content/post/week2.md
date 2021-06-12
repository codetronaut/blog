---
title: "GSoC (Week 2)"
date: "2021-06-04"
author: "Anmol"
description: "Weekly Project Report (Community Bonding Period)"
tags: 
  - gsoc
  - agl
---

**Mentee Name: Anmol**



**Mentors: Jan-Simon Möller and Marco Solieri**


**Gist about the sub-goals for this week:**

This week I tried to focus on spinning up a custom Jailhouse-image for x86_64, instead of the agl-image i was working on, because there is some issues with the emulation of qemu with agl-image and jailhouse:

```sh
[    0.008525] [Firmware Bug]: TSC_DEADLINE disabled due to Errata; please update microcode to version: 0xb2 (or later)
[FAILED] Failed to start Load Kernel Modules.
See 'systemctl status systemd-modules-load.service' for details.
         Starting Apply Kernel Variables...
[  OK  ] Started Apply Kernel Variables.
[  OK  ] Started Journal Service.
[  OK  ] Started udev Coldplug all Devices.
         Starting Helper to synchronize boot up for ifupdown...
[  OK  ] Mounted Kernel Debug File System.
[  OK  ] Started Helper to synchronize boot up for ifupdown.
[  OK  ] Mounted POSIX Message Queue File System.
[  OK  ] Started Remount Root and Kernel File Systems.
         Starting Flush Journal to Persistent Storage...
         Starting Regenerate sshd host keys...
         Starting Create System Users...
         Starting Load/Save Random Seed...
[  OK  ] Started Flush Journal to Persistent Storage.
[  OK  ] Started Load/Save Random Seed.
[  OK  ] Started Create System Users.
         Starting Create Static Device Nodes in /dev...
[  OK  ] Started Create Static Device Nodes in /dev.
         Starting udev Kernel Device Manager...
[  OK  ] Reached target Local File Systems (Pre).
[  OK  ] Reached target Local File Systems.
         Starting Create Volatile Files and Directories...
         Starting Raise network interfaces...
[  OK  ] Started Create Volatile Files and Directories.
         Starting Network Time Synchronization...
         Starting Update UTMP about System Boot/Shutdown...
[  OK  ] Started Network Time Synchronization.
[  OK  ] Started udev Kernel Device Manager.
[  OK  ] Started Raise network interfaces.
[ TIME ] Timed out waiting for device /dev/ttyS0.
[DEPEND] Dependency failed for Serial Getty on ttyS0.
[  OK  ] Stopped Network Time Synchronization.
         Starting Network Time Synchronization...
[  OK  ] Stopped Flush Journal to Persistent Storage.
         Stopping Flush Journal to Persistent Storage...
[  OK  ] Stopped Journal Service.
         Starting Journal Service...
[FAILED] Failed to start Network Time Synchronization.
See 'systemctl status systemd-timesyncd.service' for details.
[  OK  ] Stopped Network Time Synchronization.
         Starting Network Time Synchronization...
[FAILED] Failed to start Journal Service.
See 'systemctl status systemd-journald.service' for details.
[DEPEND] Dependency failed for Flus…Journal to Persistent Storage.
[FAILED] Failed to start Network Time Synchronization.
See 'systemctl status systemd-timesyncd.service' for details.
[  OK  ] Stopped Network Time Synchronization.
         Starting Network Time Synchronization...
[  OK  ] Stopped Journal Service.
         Starting Journal Service...
[FAILED] Failed to start Network Time Synchronization.
See 'systemctl status systemd-timesyncd.service' for details.
[FAILED] Failed to start Journal Service.
See 'systemctl status systemd-journald.service' for details.
[  OK  ] Stopped Journal Service.
         Starting Journal Service...
[FAILED] Failed to start Network Time Synchronization.
See 'systemctl status systemd-timesyncd.service' for details.
[  OK  ] Stopped Network Time Synchronization.
         Starting Network Time Synchronization...
[FAILED] Failed to start Network Time Synchronization.
See 'systemctl status systemd-timesyncd.service' for details.
[FAILED] Failed to start Regenerate sshd host keys.
See 'systemctl status sshd-regen-keys.service' for details.
[FAILED] Failed to start Journal Service.
See 'systemctl status systemd-journald.service' for details.

```

For the image provided by jailhouse I tried to build it with the given instruction on jailhouse-images repository, and steps involved are:

I first cloned the repository [Link](https://github.com/siemens/jailhouse-images), then i ran the script given in the repository  `build-images.sh` which after executing looks like this:


```sh
anmol@Debian-95-stretch-64-minimal:~/workspace_agl/master/build/jailhouse-images$ ./build-images.sh 
Available images demo images:
 1: QEMU/KVM Intel-x86 virtual target
 2: QEMU ARM64 virtual target
 3: Orange Pi Zero (256 MB edition)
 4: Intel NUC (NUC6CAY, 8 GB RAM)
 5: SIMATIC IPC127E (2 cores / 2 GB edition)
 6: Marvell ESPRESSObin (1 GB edition)
 7: Marvell MACCHIATObin
 8: LeMaker HiKey (Kirin 620 SoC, 2 GB edition)
 9: Avnet Ultra96 v1
 10: Avnet Ultra96 v2
 11: Raspberry Pi 4 (1-8 GB editions)
 12: Pine64+ (Allwinner A64, 2 GB edition)
 0: all (may take hours...)

Select images to build (space-separated index list): 1

```
I chose the `QEMU/KVM Intel-x86 virtual target` and it started building. After successfully building the image it produced `build` folder and it’s contents are:

```sh
anmol@Debian-95-stretch-64-minimal:~/workspace_agl/master/build/jailhouse-images$ tree build -L 2
build
├── bitbake-cookerdaemon.log
├── bitbake.lock
├── bitbake.sock
├── conf
│   ├── bblayers.conf
│   ├── local.conf
│   └── templateconf.cfg
├── downloads
│   ├── buildroot
│   ├── buildroot-2019.11.1.tar.bz2
│   ├── buildroot-2019.11.1.tar.bz2.done
│   ├── deb
│   ├── git
│   ├── ivshmem-demo.c
│   ├── ivshmem-demo.c.done
│   ├── linux-eb3ca54a7882348374d7ae32a749459c8bb4decd.tar.gz
│   └── linux-eb3ca54a7882348374d7ae32a749459c8bb4decd.tar.gz.done
└── tmp
    ├── cache
    ├── deploy
    ├── stamps
    └── work

10 directories, 12 files
```
Note: If you don't want to preface the docker command with sudo(it’s needed when you run the `build-images.sh` script) , create a Unix group called docker and add your USER to it. When the Docker daemon starts, it creates a Unix socket accessible by members of the docker group. The docker group grants privileges equivalent to the root user. [Ref.](https://docs.docker.com/engine/install/linux-postinstall/#:~:text=Manage%20Docker%20as%20a%20non%2Droot%20user&text=If%20you%20don't%20want,members%20of%20the%20docker%20group.&text=The%20docker%20group%20grants%20privileges%20equivalent%20to%20the%20root%20user.)

After this I tried to emulate the `jailhouse-image`, using the script `start-qemu.sh` (already present in the jailhouse-images repo.), but it didn’t boot. 

```sh
anmol@Debian-95-stretch-64-minimal:~/workspace_agl/master/build/jailhouse-images$ ./start-qemu.sh x86
QEMU 5.2.0 monitor - type 'help' for more information
(qemu) ALSA lib confmisc.c:767:(parse_card) cannot find card '0'
ALSA lib conf.c:4745:(_snd_config_evaluate) function snd_func_card_driver returned error: No such file or directory
ALSA lib confmisc.c:392:(snd_func_concat) error evaluating strings
ALSA lib conf.c:4745:(_snd_config_evaluate) function snd_func_concat returned error: No such file or directory
ALSA lib confmisc.c:1246:(snd_func_refer) error evaluating name
ALSA lib conf.c:4745:(_snd_config_evaluate) function snd_func_refer returned error: No such file or directory
ALSA lib conf.c:5233:(snd_config_expand) Evaluate error: No such file or directory
.
.
.
.
.
```

Hmm, I was not sure of the reason and the error output was not familiar to me, so I asked my mentor about this, and he replied : You can not run it on the server as it requires graphical output, you need to try jailhouse-images locally. So I tried to `scp` the `IMAGE_PREFIX` and `IMAGE_FILES` (according to the `start-qemu.sh` script) into my local machine, and tried QEmulating it with the below command:

```sh
╭─codetronaut@Thinkpad ~/jailhouse 
╰─$ qemu-system-x86_64 -drive file=build/tmp/deploy/images/qemu-amd64/demo-image-jailhouse-demo-qemu-amd64.ext4.img,discard=unmap,if=none,id=disk,format=raw  -kernel build/tmp/deploy/images/qemu-amd64/demo-image-jailhouse-demo-qemu-amd64-vmlinuz -append "root=/dev/sda intel_iommu=off memmap=82M\$0x3a000000 vga=0x305 serial=ttyS0,115200n8" -initrd build/tmp/deploy/images/qemu-amd64/demo-image-jailhouse-demo-qemu-amd64-initrd.img -cpu host,-kvm-pv-eoi,-kvm-pv-ipi,-kvm-asyncpf,-kvm-steal-time,-kvmclock -smp 4 -enable-kvm -machine q35,kernel_irqchip=split -serial null -device ide-hd,drive=disk
```

I created the above command by striping the `start-qemu.sh` script so that it can run with minimal arguments. And finally it worked.

![jailhouse-image](/../../jailhouse-images.png)


Thanks for reading.


**-- Anmol**

