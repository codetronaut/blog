+++
title = "GSoC (Week 3)"
date = "2021-06-11"
author = "Anmol"
description = "Weekly Project Report (Coding Period)"
tags = [
	"gsoc",
	"agl",
]
categories = [
	"gsoc",
]
+++

**Mentee Name: Anmol**



**Mentors: Jan-Simon Möller and Marco Solieri**


**Gist about the sub-goals for this week:**

This week I along with my mentors worked on fixing the QEmulation of the agl-image for x86_64. At first, we were getting some kernel panics and unknown errors, that's why I was advised by my mentor that I should use a backup Image provided by siemens in [jailhouse-images](https://github.com/siemens/jailhouse-images) repository, and start working on it, I have already updated about this in my last week’s post.

So, after trying again with QEmulating the agl-image we again got some errors as expected, so we decided to change the kernel from 5.4.xx to a recent one which is [queues/jailhouse](http://git.kiszka.org/?p=linux.git;a=shortlog;h=refs/heads/queues/jailhouse), we also had some other [options](http://git.kiszka.org/?p=linux.git;a=summary), but we first tried with the former one. 

Initially, Jan-Simon sir generated and configured the kernel in the `workspace_agl/master/build/workspace/sources/linux-jailhouse-custom` for me, and after that, I started building again, and as expected `Errors` again, which were looking something like these below:

```sh
ERROR: linux-jailhouse-custom-5.12+git999-r0 do_compile_kernelmodules: oe_runmake failed
ERROR: linux-jailhouse-custom-5.12+git999-r0 do_compile_kernelmodules: Execution of '/home/anmol/workspace_agl/master/build/tmp/work/qemux86_64-agl-linux/linux-jailhouse-custom/5.12+git999-r0/temp/run.do_compile_kernelmodules.3052211' failed with exit code 1:
make: Entering directory '/home/anmol/workspace_agl/master/build/workspace/sources/linux-jailhouse-custom'
  DESCEND  objtool
  CALL    scripts/atomic/check-atomics.sh
  CALL    scripts/checksyscalls.sh
In file included from <command-line>:
././include/linux/kconfig.h:5:10: fatal error: generated/autoconf.h: No such file or directory
    5 | #include <generated/autoconf.h>
      |          ^~~~~~~~~~~~~~~~~~~~~~
compilation terminated.
make[1]: *** [Kbuild:48: missing-syscalls] Error 1
make: *** [Makefile:1235: prepare0] Error 2
make: *** Waiting for unfinished jobs....
.
.
.
.
.
ERROR: make-mod-scripts-1.0-r0 do_configure: oe_runmake failed
ERROR: make-mod-scripts-1.0-r0 do_configure: Execution of '/home/anmol/workspace_agl/master/build/tmp/work/qemux86_64-agl-linux/make-mod-scripts/1.0-r0/temp/run.do_configure.3052217' failed with exit code 1:
make: Entering directory '/home/anmol/workspace_agl/master/build/workspace/sources/linux-jailhouse-custom'
make[1]: Entering directory '/home/anmol/workspace_agl/master/build/tmp/work-shared/qemux86-64/kernel-build-artifacts'
  SYNC    /home/anmol/workspace_agl/master/build/workspace/sources/linux-jailhouse-custom/include/config/auto.conf.cmd
***
*** The source tree is not clean, please run 'make mrproper'
*** in /home/anmol/workspace_agl/master/build/workspace/sources/linux-jailhouse-custom
***
make[2]: *** [/home/anmol/workspace_agl/master/build/workspace/sources/linux-jailhouse-custom/Makefile:547: outputmakefile] Error 1
make[1]: *** [/home/anmol/workspace_agl/master/build/workspace/sources/linux-jailhouse-custom/Makefile:711: /home/anmol/workspace_agl/master/build/workspace/sources/linux-jailhouse-custom/include/config/auto.conf.cmd] Error 2
```
At first, I told Jan-Simon sir that, first I will try to resolve this error, after that if I failed to resolve this then I  will take your help. So I started again by looking at [Jailhouse-mailing lists](https://groups.google.com/g/jailhouse-dev) and tried some solutions which worked, but at the same time, I was again attacked by another unknown error. In the whole error resolving process, I also tried to patch the kernel as I was also getting some `SELinux` related issues. But at the end, I asked Jan-Simon sir to look into this problem, and after some tries and hacks in the linux-jailhouse-custom `recipes` we managed to QEmulate the agl-image:

```sh
[  OK  ] Started Permit User Sessions.
[  OK  ] Started Getty on tty1.
[  OK  ] Started Serial Getty on ttyS0.
[  OK  ] Reached target Login Prompts.
[  OK  ] Started Login Service.
[   70.017964] Out of memory: Killed process 183 ((agetty)) total-vm:8368kB, anon-rss:1508kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:52kB oom_score_adj:0
[   71.989351] Out of memory: Killed process 189 (systemd-udevd) total-vm:12808kB, anon-rss:1096kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:48kB oom_score_adj:0

Automotive Grade Linux 11.91.0 qemux86-64 ttyS0

qemux86-64 login: root
[   82.178431] Out of memory: Killed process 192 (systemd-udevd) total-vm:12808kB, anon-rss:1096kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:48kB oom_score_adj:0
[   86.478264] Out of memory: Killed process 191 (systemd-udevd) total-vm:12808kB, anon-rss:1096kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:48kB oom_score_adj:0
[   88.035583] Out of memory: Killed process 190 (systemd-udevd) total-vm:12808kB, anon-rss:1096kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:48kB oom_score_adj:0
[   93.008469] Out of memory: Killed process 193 ((agetty)) total-vm:8500kB, anon-rss:1688kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:52kB oom_score_adj:0
[   93.011007] Out of memory: Killed process 145 (systemd-network) total-vm:6692kB, anon-rss:440kB, file-rss:0kB, shmem-rss:0kB, UID:1005 pgtables:56kB oom_score_adj:0
[   93.520704] Out of memory: Killed process 195 ((agetty)) total-vm:8500kB, anon-rss:1688kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:52kB oom_score_adj:0
[   94.049963] Out of memory: Killed process 198 ((sd-pam)) total-vm:9344kB, anon-rss:1772kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:52kB oom_score_adj:0
qemux86-64:~# ls
app-data
qemux86-64:~# cd app-data/
qemux86-64:~/app-data# ls
qemux86-64:~/app-data#
```
Although it’s not yet very functional as we are getting some issues with the jailhouse cells part, and there were also some issues with memory, which is: not carving-out memory properly, but now it's fixed. But yes we are very close to fixing all those and proceed further.

Thanks for reading.


**-- Anmol**

