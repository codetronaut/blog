---
title: "GSoC (Week 11)"
date: "2021-08-08"
author: "Anmol"
tags: 
  - gsoc
  - agl
---

***Mentee Name: Anmol***


***Mentors: Jan-Simon Möller and Marco Solieri***


***Gist about advancements in this week:***


#### Week-11: Non-Root Linux Cell working and initiated the virtio-block part.

**Current Status**

This week I managed to run Non-Root Linux cell and also managed to have serial redirection working. Now I have a working setup with the QEmulated AGL with Jailhouse.

#### Below are the steps to reproduce the whole working setup:

At first, we need to follow the initial steps required to download and build the AGL Image, So for initial steps just follow the [AGL documentation](https://docs.automotivelinux.org/en/master/#0_Getting_Started/2_Building_AGL_Image/2_Downloading_AGL_Software/), and at the step of initiating `aglsetup.sh Script` write below commands in the terminal.

```bash
$ source meta-agl/scripts/aglsetup.sh -m qemux86-64 -b build agl-devel agl-jailhouse
```

And then build this using `bitbake core-image-minimal`.

After building, run the below command to start the QEmu.

```bash
$ runqemu qemux86-64 slirp kvm publicvnc serial bootparams="verbose ipv6.disable=1 intel_iommu=off"
```

This will spin the QEmu, after logging in, run the `dhclient enp0s2` for the network and scp the `bzImage`, `rootfs.cpio` files into the QEmulated Image with Jailhouse.

After copying everything run below command to check if jailhouse is present or not.

```bash
qemux86-64:~# lsmod
Module                  Size  Used by
virtio_net             53248  0
virtio_gpu             65536  0
net_failover           16384  1 virtio_net
failover               16384  1 net_failover
virtio_dma_buf         16384  1 virtio_gpu
jailhouse              36864  0

```
As you can see it’s present, In case if it’s not present then run `modprobe jailhouse`, it will load the Jailhouse kernel module.

After loading the module now we have to enable the Jailhouse-Root-Cell, enable this by the below command:

```bash
qemux86-64:~# jailhouse enable /usr/share/jailhouse/cells/qemu-agl.cell
qemux86-64:~# jailhouse console
Initializing Jailhouse hypervisor v0.12 (279-g63000120) on CPU 2
Code location: 0xfffffffff0000050
Using xAPIC
Page pool usage after early setup: mem 49/975, remap 1/131072
Initializing processors:
 CPU 2... (APIC ID 2) OK
 CPU 3... (APIC ID 3) OK
 CPU 0... (APIC ID 0) OK
 CPU 1... (APIC ID 1) OK
Initializing unit: VT-d
DMAR unit @0xfed90000/0x1000
Reserving 24 interrupt(s) for device ff:00.0 at index 0
Initializing unit: IOAPIC
Initializing unit: Cache Allocation Technology
Initializing unit: PCI
Adding PCI device 00:00.0 to cell "AGL-Jailhouse-RootCell"
Adding PCI device 00:01.0 to cell "AGL-Jailhouse-RootCell"
Reserving 3 interrupt(s) for device 00:01.0 at index 24
Adding PCI device 00:02.0 to cell "AGL-Jailhouse-RootCell"
Reserving 3 interrupt(s) for device 00:02.0 at index 27
Adding PCI device 00:03.0 to cell "AGL-Jailhouse-RootCell"
Adding PCI device 00:04.0 to cell "AGL-Jailhouse-RootCell"
Reserving 2 interrupt(s) for device 00:04.0 at index 30
Adding PCI device 00:1b.0 to cell "AGL-Jailhouse-RootCell"
Reserving 1 interrupt(s) for device 00:1b.0 at index 32
Adding PCI device 00:1d.0 to cell "AGL-Jailhouse-RootCell"
Adding PCI device 00:1d.1 to cell "AGL-Jailhouse-RootCell"
Adding PCI device 00:1d.2 to cell "AGL-Jailhouse-RootCell"
Adding PCI device 00:1d.7 to cell "AGL-Jailhouse-RootCell"
Adding PCI device 00:1f.0 to cell "AGL-Jailhouse-RootCell"
Adding PCI device 00:1f.2 to cell "AGL-Jailhouse-RootCell"
Reserving 1 interrupt(s) for device 00:1f.2 at index 33
Adding PCI device 00:1f.3 to cell "AGL-Jailhouse-RootCell"
Adding virtual PCI device 00:0c.0 to cell "AGL-Jailhouse-RootCell"
Page pool usage after late setup: mem 205/975, remap 65545/131072
Activating hypervisor
```
After loading jailhouse root-cell, now we have to load the non-root linux cell, so for that run below commands:

```bash
qemux86-64:~# jailhouse cell linux /usr/share/jailhouse/cells/agl-linux-x86-demo.cell bzImage -i rootfs.cpio -w out.file -c "console=ttyS2,115200 earlycon earlyprintk"
``` 

In this command, when you add the `-w out.file` option then it will spit out some commands to start the non-root cell, if not then the cell will boot as usual. Those spitted out commands would look something like these below, you have to run it one by one:

```bash
qemux86-64:~# jailhouse cell create /usr/share/jailhouse/cells/agl-linux-x86-demo.cell
qemux86-64:~# jailhouse cell load linux-x86-demo linux-loader.bin -a 0x0 bzImage -a 0xffc600 rootfs.cpio -a 0x3d89000 out.file -a 0x1000
qemux86-64:~# jailhouse cell start linux-x86-demo

```
> Note: As you can see in the spit-out commands there is a `linux-loader.bin` is present, this is basically a tiny bootloader that is required to boot the Linux-inmate or Linux-non-root cell.

In the other serial console booting would look something like this:

```bash
anmol@Debian-95-stretch-64-minimal:~$ nc localhost 5432
��������[    0.000000] Linux version 5.14.0-rc2+ (codetronaut@Thinkpad) (gcc (GCC) 11.1.0, GNU ld (GNU Binutils) 2.36.1) #1 SMP PREEMPT Fri Jul 23 16:06:23 IST 2021
[    0.000000] Command line: console=ttyS2,115200 earlycon earlyprintk
[    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x008: 'MPX bounds registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x010: 'MPX CSR'
[    0.000000] x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
[    0.000000] x86/fpu: xstate_offset[3]:  832, xstate_sizes[3]:   64
[    0.000000] x86/fpu: xstate_offset[4]:  896, xstate_sizes[4]:   64
[    0.000000] x86/fpu: Enabled xstate features 0x1f, context size is 960 bytes, using 'compacted' format.
[    0.000000] signal: max sigframe size: 2032
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

.
.
.
[    1.380547] rodata_test: all tests were successful
[    1.381203] Run /init as init process
[    1.381772] process '/bin/busybox' started with executable stack
Starting syslogd: OK
Starting klogd: OK
Running sysctl: OK
Saving random seed: [    1.405727] random: dd: uninitialized urandom read (512 bytes read)
OK
Starting dropbear sshd: [    1.411374] random: dropbear: uninitialized urandom read (32 bytes read)
OK

Welcome to Buildroot
jailhouse login: root
# [   27.613449] clocksource: timekeeping watchdog on CPU0: acpi_pm retried 2 times before success
uname -a
uname -a
Linux jailhouse 5.14.0-rc2+ #1 SMP PREEMPT Fri Jul 23 16:06:23 IST 2021 x86_64 GNU/Linux
```
As you can see that cell is booting properly.

> Note: I have created the scripts for the above commands used for [QEmulation](https://gerrit.automotivelinux.org/gerrit/c/AGL/meta-agl-devel/+/26546/11/meta-agl-jailhouse/recipes-extended/jailhouse/files/helper-scripts/run-qemu-jailhouse.sh) and [loading](https://gerrit.automotivelinux.org/gerrit/c/AGL/meta-agl-devel/+/26546/11/meta-agl-jailhouse/recipes-extended/jailhouse/files/helper-scripts/linux-non-root-cell.sh) the Non-Root and Root cells and placed it in the `metal-jailhouse/recipes-extended/jailhouse/files/helper-scripts` in the patch sent to gerrit, to automate all this.

**Next Steps**

The next steps could be as follows: 
The first thing is we need to ask some questions again on jailhouse-mailing list, related to the virtio-block over IVSHMEM implementation, we might also need to port some patches or whole `queues/ivshmem2` kernel for the virtio-ivshmem-block support. Because I already compiled the `queues/ivshmem2` Kernel locally and found that some patches are not there in that branch, and are not updated recently as compared to the `queues/jailhouse` kernel.

Another way to achieve this is, taking the inspiration from `ACRN` hypervisor's implementation of virtio, i.e without using the jailhouse's implementation with IVSHMEM, and implement the whole thing from scratch(also by taking inspirations from various present implementations apart from ACRN). But yes this part needs more discussins regarding many things.


For the RTOS Non-Root-cell part, there are some things to do regarding the Zephyr as a Jailhouse-inmate guest and I am researching more on it, will update about this soon.



Thanks for reading.


**-- Anmol**