---
title: "GSoC (Week 9)"
date: "2021-07-23"
author: "Anmol"
tags: 
  - gsoc
  - agl
---

***Mentee Name: Anmol***



***Mentors: Jan-Simon Möller and Marco Solieri***


***Gist about advancements in this week:***


#### Week-09: Some progress with Jailhouse Non-Root Linux cell and local working setup.

**Current Status**


This week I worked on debugging the non-root Linux cell, updating the Linux kernel in the jailhouse root cell in the QEmulated AGL setup, and also on my local QEmulated setup with `jailhouse-images` because the jailhouse community has just updated its `master` and `queues/jailhouse` branches of the Linux Kernel and they also included some improvement commits for the jailhouse `x86`, like [Add simple debug console via the hypervisor](http://git.kiszka.org/?p=linux.git;a=commit;h=edc9cb2fad874889730b3844dbc1ae380f40ef2a) and, IVSHMEM. 


*Non-Root Cell*

When I first loaded the Non-Root Linux with the Kernel Image it was giving the errors like this below:

```
$ jailhouse cell linux /usr/share/jailhouse/cells/linux-x86-demo.cell ./vmlinux -i ./rootfs.cpio -c "console=ttyS0,115200"

Traceback (most recent call last):
  File "/usr/libexec/jailhouse/jailhouse-cell-linux", line 727, in <module>
    arch.setup(args, config)
  File "/usr/libexec/jailhouse/jailhouse-cell-linux", line 256, in setup
    self._zero_page = X86ZeroPage(self.kernel_image, args.initrd,
  File "/usr/libexec/jailhouse/jailhouse-cell-linux", line 601, in __init__
    self.setup_header.set_kernel_alignment(self.setup_header.pref_address)
  File "/usr/libexec/jailhouse/jailhouse-cell-linux", line 548, in set_kernel_alignment
    self.set_value_in_data('Q', 0x230, value)
  File "/usr/libexec/jailhouse/jailhouse-cell-linux", line 527, in set_value_in_data
    struct.pack_into(fmt, self.data, offset - X86SetupHeader.BASE_OFFSET,
struct.error: pack_into requires a buffer of at least 72 bytes for packing 8 bytes at offset 64 (actual buffer size is 18)
```
The above error occurred when I was using the `vmlinux` while loading the Linux non-root cell, and `vmlinux` was the reason for this error as it’s an uncompressed kernel image and when the `jailhouse-cell-linux` script tries to decompress it, then it just fails as it’s already in the uncompressed form.

The above error was fixed by using the `bzImage` instead, and for this, I compiled the kernel externally because non-root cells need some extra configurations, and also it should be minimal in configuration so that no extra probing will take place. 

For this first, I cloned the `queues/jailhouse` Linux Kernel from [here](http://git.kiszka.org/?p=linux.git;a=summary).
After that, I copied the kernel configuration from [here](https://github.com/siemens/jailhouse-images/blob/master/recipes-kernel/linux/files/amd64_defconfig_5.10).
Then I compiled the kernel and copied the `bzImage` into the QEmulated setup with AGL via scp.

After loading the non-root cell, again it was showing some error like this:

```
$ jailhouse cell linux /usr/share/jailhouse/cells/linux-x86-demo.cell bzImage -i rootfs.cpio -c "console=ttyS0,115200"

Page pool usage after late setup: mem 204/975, remap 65545/131072
Activating hypervisor
Adding virtual PCI device 00:0c.0 to cell "linux-x86-demo"
Adding virtual PCI device 00:0d.0 to cell "linux-x86-demo"
Adding virtual PCI device 00:0e.0 to cell "linux-x86-demo"
Adding virtual PCI device 00:0f.0 to cell "linux-x86-demo"
Created cell "linux-x86-demo"
Page pool usage after cell creation: mem 299/975, remap 65545/131072
FATAL: unsupported instruction (0xeb 0x00 0x00 0x00)
FATAL: Invalid MMIO/RAM read, addr: 0x000000003ec0b010 size: 0
RIP: 0xffffffffc00937ea RSP: 0xffff9873002cbe98 FLAGS: 246
RAX: 0x0000000000000000 RBX: 0xffff962682dc0ec8 RCX: 0x0000000000000040
RDX: 0x000000000000001c RSI: 0x0000000000000001 RDI: 0x000000000128b000
CS: 10 BASE: 0x0000000000000000 AR-BYTES: a09b EFER.LMA 1
CR0: 0x0000000080050033 CR3: 0x0000000005b0c002 CR4: 0x0000000000372ef0
EFER: 0x0000000000000d01
Parking CPU 0 (Cell: "RootCell")
```
After analyzing some threads from the jailhouse mailing list [threads](https://groups.google.com/g/jailhouse-dev/search?q=FATAL%3A%20unsupported%20instruction) I came to the conclusion of updating the non-root cell’s address of some IVSHMEM (demo) `.mem_regions` from `.phys_start = 0x3f0f[1,a,c,e]000` to `.phys_start = 0x2f0f[1,a,c,e]000`. And it fixed the issue. 

After this fix, there were still more errors occurring, so I and Jan-Simon sir decided to remove those parts from the non-root Linux cell like some regions in the `.memory_regions`.  And it worked and all the errors now are gone and the Non-root cell is now working but not properly, and it’s showing console logs something like this:

```
Adding virtual PCI device 00:0c.0 to cell "linux-x86-demo"
Adding virtual PCI device 00:0d.0 to cell "linux-x86-demo"
Adding virtual PCI device 00:0e.0 to cell "linux-x86-demo"
CPU 2 received SIPI, vector 9a
CPU 3 received SIPI, vector 9a
^C
```
Why I am saying that it’s not working properly because it’s expected to see the booting of the Linux non-root cell like this below:
(The below log is from the Linux non-root cell in my local setup.)
```
Created cell "linux-x86-demo"
Page pool usage after cell creation: mem 361/975, remap 65543/131072
Cell "linux-x86-demo" can be loaded
CPU 2 received SIPI, vector 100
CPU 3 received SIPI, vector 100
Started cell "linux-x86-demo"
[    0.000000] Linux version 5.10.31 (builder@6a5f65e19278) (gcc (Debian 8.3.0-6) 8.3.0, GNU ld (GNU Binutils for Debian) 2.31.1) #1 SMP PREEMPT Wed Jun 23 15:16:06 UTC 2021
[    0.000000] Command line: console=ttyS0 8250.nr_uarts=1 ip=192.168.19.2
[    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x008: 'MPX bounds registers'
0x0000000000200000-0x00000000048fffff] usable
[    0.000000] DMI not present or invalid.
[    0.000000] Hypervisor detected: Jailhouse

.
.
.
.
.
.
[    4.232407] rodata_test: all tests were successful
[    4.233358] Run /init as init process
Starting syslogd: OK
Starting klogd: OK
Running sysctl: OK
Saving random seed: [    4.278495] random: dd: uninitialized urandom read (512 bytes read)
OK
Starting dropbear sshd: OK

Welcome to Buildroot
jailhouse login: 

[   23.902732] random: crng init done
Closing cell "linux-x86-demo"
Page pool usage after cell destruction: mem 272/975, remap 65543/131072
CPU 2 received SIPI, vector 9a
CPU 3 received SIPI, vector 9a

```

But our root-cell Linux in the QEmulated AGL setup is not loading properly like the above one.

I also tried with similar configurations and kernel Image in the `jailhouse-images` setup in my local machine and it was working perfectly.

**Next Steps**

Now I have updated the kernel in the `jailhouse-images` in my local machine setup and will do the next proceeding with `virtio-blk` mostly from here, but I will also try to debug the non-root Linux cell in the agl-setup as well, and according to the threads I have read in the jailhouse mailing list, it’s probably due to the misconfiguration of the UART redirection.


Thanks for reading.


**-- Anmol**
