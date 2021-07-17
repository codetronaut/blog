---
title: "GSoC (Week 8)"
date: "2021-07-17"
author: "Anmol"
tags: 
  - gsoc
  - agl
---

***Mentee Name: Anmol***



***Mentors: Jan-Simon Möller and Marco Solieri***


***Gist about advancements in this week:***


#### Week-08: Jailhouse Root cell now working with QEmulated AGL Image.

Root cell is now loading in the QEmulated setup and some of the changes in the configurations file which were not working before are now fixed by these changes:

- At first, the configuration file is generated from the QEmulated instance using.

```
$ jailhouse config create -c ttyS1 qemu-agl.c
```
- After that, some changes are made in the `serial` and `Memory` addressing of the configuration files, these are shown below:

`0x22000000-0x271fffff` is the Memory address range that is available to the root cell and other guest cells(according to /proc/iomem), so a value between this range is placed in the `.hypervisor_memory` in the root cell.

Changes related to `pio_regions` configuration in the non-root cells are also made.

- Some gist about the `pio_regions` changes: 

`ttyS0(0x3f8)`  is the default runqemu serial which is already included in the config files.
`ttyS1(0x2f8)` is a  jailhouse debug serial.

Now the next two are the changes for redirection from the loaded cells. Where `ttyS2(0x3e8)`  is available as the first cell console I only used this currently and `ttyS3(0x2e8)`  is available as 2nd cell console, and this one is not wired up in the runqemu/qemu boot arguments.

All the serial redirection is done using the `netcat` utility and running the below command in the separate terminals:
```
$ nc localhost <available_port>
```
And this is enforced by appending `-serial <>` and `-vga virtio`in `qb_opt_append` in `*qemuboot.conf` with ports `4321` and `5432` like this:

```
.
.
.
qb_opt_append = -vga vmware -show-cursor -usb -device usb-tablet -device virtio-rng-pci -device intel-iommu,intremap=on,x-buggy-eim=on -device intel-hda,addr=1b.0 -device hda-duplex -serial none -serial stdio -serial telnet:localhost:4321,server,nowait -serial telnet:localhost:5432,server,nowait -vga virtio
.
.
.
```

Note: 
- First Cell Serial: `-serial telnet:localhost:4321,server,nowait`
- Second Cell Serial: `-serial telnet:localhost:5432,server,nowait`

The above hack by Jan-Simon sir is a little new for me and I was not aware of redirection of console log using this way, I was only aware of the redirection by selecting the `tty` from a given console and then enter the `/dev/pts/<available_pseudo_terminal>` it to the `*qemuboot.conf`.

**Working Demo**

After all the above changes root cell is working fine and also `apic-demo` is also running fine, belo are the screenshots of these:

- Console `ttyS0`.

```
[    0.000000] [Firmware Bug]: TSC_DEADLINE disabled due to Errata; please update microcode to version: 0xb2 (or later)
[    0.516268] kvm: already loaded the other module
[    0.858907] hdaudio hdaudioC0D0: Unable to bind the codec

Automotive Grade Linux 11.91.0 qemux86-64 ttyS0

qemux86-64 login: root


qemux86-64:~# jailhouse enable /usr/share/jailhouse/cells/qemu-agl.cell 
qemux86-64:~# jailhouse cell create /usr/share/jailhouse/cells/agl-apic-demo.cell 
qemux86-64:~# jailhouse cell load apic-demo /usr/share/jailhouse/inmates/apic-demo.bin 
qemux86-64:~# jailhouse cell start 1
qemux86-64:~# jailhouse cell list  
ID      Name                    State             Assigned CPUs           Failed CPUs             
0       RootCell                running           0-2                                             
1       apic-demo               running/locked    3                                               
qemux86-64:~# jailhouse cell destroy 1

```
- First cell serial console

```
anmol@Debian-95-stretch-64-minimal:~$ nc localhost 4321
��������Shutting down hypervisor
 Releasing CPU 2
 Releasing CPU 1
 Releasing CPU 3
 Releasing CPU 0

Initializing Jailhouse hypervisor v0.12 (273-g6d9c51d0-dirty) on CPU 2
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
Adding PCI device 00:00.0 to cell "RootCell"
Adding PCI device 00:01.0 to cell "RootCell"
Reserving 3 interrupt(s) for device 00:01.0 at index 24
Adding PCI device 00:02.0 to cell "RootCell"
Reserving 3 interrupt(s) for device 00:02.0 at index 27
Adding PCI device 00:03.0 to cell "RootCell"
Adding PCI device 00:04.0 to cell "RootCell"
Reserving 2 interrupt(s) for device 00:04.0 at index 30
Adding PCI device 00:1b.0 to cell "RootCell"
Reserving 1 interrupt(s) for device 00:1b.0 at index 32
Adding PCI device 00:1d.0 to cell "RootCell"
Adding PCI device 00:1d.1 to cell "RootCell"
Adding PCI device 00:1d.2 to cell "RootCell"
Adding PCI device 00:1d.7 to cell "RootCell"
Adding PCI device 00:1f.0 to cell "RootCell"
Adding PCI device 00:1f.2 to cell "RootCell"
Reserving 1 interrupt(s) for device 00:1f.2 at index 33
Adding PCI device 00:1f.3 to cell "RootCell"
Page pool usage after late setup: mem 204/975, remap 65545/131072
Activating hypervisor
Created cell "apic-demo"
Page pool usage after cell creation: mem 219/975, remap 65545/131072
Cell "apic-demo" can be loaded
Started cell "apic-demo"
CPU 3 received SIPI, vector 100
Closing cell "apic-demo"
Page pool usage after cell destruction: mem 205/975, remap 65545/131072
CPU 3 received SIPI, vector 9a
Shutting down hypervisor
 Releasing CPU 2
 Releasing CPU 0
 Releasing CPU 3
 Releasing CPU 1
```


- second cell serial console

```
anmol@Debian-95-stretch-64-minimal:~$ nc localhost 5432
��������Calibrated TSC frequency: 3408013.000 kHz
Calibrated APIC frequency: 3408013 kHz
Timer fired, jitter:  11295 ns, min:  11295 ns, max:  11295 ns
Timer fired, jitter:  11169 ns, min:  11169 ns, max:  11295 ns
Timer fired, jitter:   9553 ns, min:   9553 ns, max:  11295 ns
Timer fired, jitter:  11089 ns, min:   9553 ns, max:  11295 ns
Timer fired, jitter:  10744 ns, min:   9553 ns, max:  11295 ns
Timer fired, jitter:  10893 ns, min:   9553 ns, max:  11295 ns
Timer fired, jitter:  10870 ns, min:   9553 ns, max:  11295 ns
Timer fired, jitter:  10433 ns, min:   9553 ns, max:  11295 ns
Timer fired, jitter:  39342 ns, min:   9553 ns, max:  39342 ns
Timer fired, jitter:   9608 ns, min:   9553 ns, max:  39342 ns
Timer fired, jitter:  10490 ns, min:   9553 ns, max:  39342 ns
Timer fired, jitter:  22160 ns, min:   9553 ns, max:  39342 ns
Timer fired, jitter:  10307 ns, min:   9553 ns, max:  39342 ns
Timer fired, jitter:   8781 ns, min:   8781 ns, max:  39342 ns
Timer fired, jitter:  36739 ns, min:   8781 ns, max:  39342 ns
Timer fired, jitter:  48629 ns, min:   8781 ns, max:  48629 ns
Timer fired, jitter:  10072 ns, min:   8781 ns, max:  48629 ns
Timer fired, jitter:  10802 ns, min:   8781 ns, max:  48629 ns
Timer fired, jitter:  36913 ns, min:   8781 ns, max:  48629 ns
Timer fired, jitter:  35121 ns, min:   8781 ns, max:  48629 ns
Timer fired, jitter:  36728 ns, min:   8781 ns, max:  48629 ns
Timer fired, jitter:  88343 ns, min:   8781 ns, max:  88343 ns
Timer fired, jitter:  36653 ns, min:   8781 ns, max:  88343 ns
Timer fired, jitter:  72971 ns, min:   8781 ns, max:  88343 ns
.
.
.
Timer fired, jitter:  73341 ns, min:   7410 ns, max: 368574 ns
Timer fired, jitter:  72861 ns, min:   7410 ns, max: 368574 ns
Timer fired, jitter:  34449 ns, min:   7410 ns, max: 368574 ns
Stopped APIC demo
```
**Next Goal**

The next part which I am currently working on is the non-root Linux cell part. I have gathered all the configurations required for running this cell but it’s giving some error while loading it after the `ivshmem-demo` cell, and I am further researching it to get it resolved.


Thanks for reading.


**-- Anmol**
