---
title: "GSoC (Week 4)"
date: "2021-06-18"
author: "Anmol"
tags: 
  - gsoc
  - agl
---

**Mentee Name: Anmol**



**Mentors: Jan-Simon Möller and Marco Solieri**


**Gist about advancements in this week:**


This week I tried to improve the booting of the agl-image and also tried to resolve the unknown behavior while QEmulating the agl-image. It’s showing very weird behavior in every boot, and it was also the same situation with loading of `root-cell` as well. So, I first started with the modification of the `recipes-kernel` present in the `meta-agl-devel/meta-agl-jailhouse/`, and for that, I tried the same kernel but with previous versions, like currently, the setup contains the `5.12-rc6 kernel` which is present in the `queues/jailhouse` branch with a specific [commit id](http://git.kiszka.org/?p=linux.git;a=commit;h=452bfd9102265f67afd5818024ca2664aa3afa60), but this time i used a previous commit which is [x86: Export lapic_timer_period](http://git.kiszka.org/?p=linux.git;a=commit;h=f7e28ebb9785f690253006912485cd8670aa3b42), I copied it’s commit id in `SRCREV`and tried to build the kernel up to that commit.

But it was still not looking good. 

```
[    0.007617] [Firmware Bug]: TSC_DEADLINE disabled due to Errata; please update microcode to version: 0xb2 (or later)
[    0.525808] kvm: already loaded the other module
[    0.567138] hdaudio hdaudioC0D0: Unable to bind the codec

Automotive Grade Linux 11.91.0 qemux86-64 ttyS0

qemux86-64 login: [   27.731494] virtio_gpu virtio0: [drm] *ERROR* fbdev: Failed to setup generic emulation (ret=-22)
[   27.841660] Out of memory: Killed process 161 (systemd-udevd) total-vm:12276kB, anon-rss:556kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:64kB oom_score_adj:0

qemux86-64 login: 
qemux86-64 login: 
qemux86-64 login: root



^C^C^C^C^C

[  236.927354] Out of memory: Killed process 167 ((agetty)) total-vm:8260kB, anon-rss:1488kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:48kB oom_score_adj:0
[  437.781132] Out of memory: Killed process 166 (systemd-udevd) total-vm:13548kB, anon-rss:1500kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:64kB oom_score_adj:0
[  457.378023] Out of memory: Killed process 170 ((agetty)) total-vm:8260kB, anon-rss:1488kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:48kB oom_score_adj:0
[  516.062574] Out of memory: Killed process 171 ((agetty)) total-vm:8260kB, anon-rss:1488kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:48kB oom_score_adj:0
QEMU 4.2.0 monitor - type 'help' for more information
(qemu) q
runqemu - INFO - Cleaning up
```
After that, I halted my work on this for some time and started to focus on the siemens image which I compiled in the server and copied its image and initramfs files locally. So, initially, I was getting some errors like `JAILHOUSE_ENABLE: Cannot allocate memory` while loading the `qemu-x86.cell`, so I tried to alter its memory by looking into `/proc/iomem` for reserved and available memory regions and tried some of the addresses. But yes, finally it worked and now I am able to run the root cell and also able to spin up the `apic-demo` and `linux-x86-demo.cell`.

----------> Loading the `qemu-x86.cell`

```
Initializing Jailhouse hypervisor v0.12 (0-g92db71f2-dirty) on CPU 3
Code location: 0xfffffffff0000050
Using x2APIC
Page pool usage after early setup: mem 47/975, remap 0/131072
Initializing processors:
 CPU 3... (APIC ID 3) OK
 CPU 1... (APIC ID 1) OK
 CPU 0... (APIC ID 0) OK
 CPU 2... (APIC ID 2) OK
Initializing unit: VT-d
DMAR unit @0xfed90000/0x1000
Reserving 24 interrupt(s) for device ff:00.0 at index 0
Initializing unit: IOAPIC
Initializing unit: Cache Allocation Technology
Initializing unit: PCI
Adding PCI device 00:01.0 to cell "QEMU-VM"
Adding PCI device 00:02.0 to cell "QEMU-VM"
Reserving 5 interrupt(s) for device 00:02.0 at index 24
Adding PCI device 00:1b.0 to cell "QEMU-VM"
Reserving 1 interrupt(s) for device 00:1b.0 at index 29
Adding PCI device 00:1f.0 to cell "QEMU-VM"
Adding PCI device 00:1f.2 to cell "QEMU-VM"
Reserving 1 interrupt(s) for device 00:1f.2 at index 30
Adding PCI device 00:1f.3 to cell "QEMU-VM"
Adding PCI device 00:1f.7 to cell "QEMU-VM"
Reserving 2 interrupt(s) for device 00:1f.7 at index 31
Adding virtual PCI device 00:0c.0 to cell "QEMU-VM"
Adding virtual PCI device 00:0d.0 to cell "QEMU-VM"
Adding virtual PCI device 00:0e.0 to cell "QEMU-VM"
Adding virtual PCI device 00:0f.0 to cell "QEMU-VM"
Page pool usage after late setup: mem 270/975, remap 65543/131072
Activating hypervisor
Created cell "apic-demo"
Page pool usage after cell creation: mem 285/975, remap 65543/131072
Cell "apic-demo" can be loaded
CPU 3 received SIPI, vector 100
```


----------> Loading the `apic-demo`.

```
Started cell "apic-demo"
Calibrated TSC frequency: 2399999.000 kHz
Calibrated APIC frequency: 2399999 kHz
Timer fired, jitter:  22827 ns, min:  22827 ns, max:  22827 ns
Timer fired, jitter:  13673 ns, min:  13673 ns, max:  22827 ns
Timer fired, jitter:  14590 ns, min:  13673 ns, max:  22827 ns
Timer fired, jitter:  12245 ns, min:  12245 ns, max:  22827 ns
Timer fired, jitter:  11930 ns, min:  11930 ns, max:  22827 ns
Timer fired, jitter:  12005 ns, min:  11930 ns, max:  22827 ns
.
.
.
.
Timer fired, jitter:   7757 ns, min:   5098 ns, max: 155077 ns
Stopped APIC demo
Closing cell "apic-demo"
Page pool usage after cell destruction: mem 272/975, remap 65543/131072
```
----------> Loading the `linux-x86-demo.cell`

```
CPU 3 received SIPI, vector 9a
Adding virtual PCI device 00:0c.0 to cell "linux-x86-demo"
Shared memory connection established, peer cells:
 "QEMU-VM"
Adding virtual PCI device 00:0d.0 to cell "linux-x86-demo"
Shared memory connection established, peer cells:
 "QEMU-VM"
Adding virtual PCI device 00:0e.0 to cell "linux-x86-demo"
Shared memory connection established, peer cells:
 "QEMU-VM"
Adding virtual PCI device 00:0f.0 to cell "linux-x86-demo"
Shared memory connection established, peer cells:
 "QEMU-VM"
Created cell "linux-x86-demo"
Page pool usage after cell creation: mem 363/975, remap 65543/131072
Cell "linux-x86-demo" can be loaded
CPU 2 received SIPI, vector 100
CPU 3 received SIPI, vector 100
Started cell "linux-x86-demo"
[    0.000000] Linux version 5.4.61 (builder@3c517acea803) (gcc version 8.3.0 (Debian 8.3.0-6)) #1 SMP PREEMPT Wed Jun 2 09:44:36 UTC 2021
[    0.000000] Command line: console=ttyS0 8250.nr_uarts=1 ip=192.168.19.2
[    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x008: 'MPX bounds registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x010: 'MPX CSR'
[    0.000000] x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
[    0.000000] x86/fpu: xstate_offset[3]:  832, xstate_sizes[3]:   64
[    0.000000] x86/fpu: xstate_offset[4]:  896, xstate_sizes[4]:   64
[    0.000000] x86/fpu: Enabled xstate features 0x1f, context size is 960 bytes, using 'compacted' format.
[    0.000000] BIOS-provided physical RAM map:
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x00000000000fffff] usable
[    0.000000] BIOS-e820: [mem 0x0000000000100000-0x0000000000100fff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000000200000-0x00000000048fffff] usable
[    0.000000] NX (Execute Disable) protection: active
[    0.000000] extended physical RAM map:
[    0.000000] reserve setup_data: [mem 0x0000000000000000-0x0000000000001fff] usable
[    0.000000] reserve setup_data: [mem 0x0000000000002000-0x000000000000212b] usable
[    0.000000] reserve setup_data: [mem 0x000000000000212c-0x00000000000fffff] usable
[    0.000000] reserve setup_data: [mem 0x0000000000100000-0x0000000000100fff] reserved
[    0.000000] reserve setup_data: [mem 0x0000000000200000-0x00000000048fffff] usable
[    0.000000] DMI not present or invalid.
[    0.000000] Hypervisor detected: Jailhouse
[    0.000000] tsc: Detected 2399.999 MHz processor
[    0.000124] last_pfn = 0x4900 max_arch_pfn = 0x400000000
[    0.000296] x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- WT  
.
.
.
.
.
.
.
Saving random seed: [    3.358939] random: dd: uninitialized urandom read (512 bytes read)
OK
Starting dropbear sshd: [    3.369505] random: dropbear: uninitialized urandom read (32 bytes read)
OK

Welcome to Buildroot
jailhouse login: 

[   13.525941] random: dropbear: uninitialized urandom read (32 bytes read)
[   13.602532] random: dropbear: uninitialized urandom read (32 bytes read)
Closing cell "linux-x86-demo"
``` 

Conclusion:

Now I have a setup to proceed further in the project until the issues with the agl-image are fixed properly. I am also planning something for the next week, i.e to use the `jailhouse-images` setup with the AGL-images, but yes this planning is a little vague and I will need to discuss everything properly with my mentors, before proceeding with that.





Thanks for reading.


**-- Anmol**

