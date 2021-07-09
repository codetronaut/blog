---
title: "GSoC (Week 7)"
date: "2021-07-09"
author: "Anmol"
tags: 
  - gsoc
  - agl
---

**Mentee Name: Anmol**



**Mentors: Jan-Simon Möller and Marco Solieri**


**Gist about advancements in this week:**



This week I tried to fix the error and freezing of the `jailhouse enable qemu-x86.cell`, so first I tried to look at its configuration file tried with comparing that configuration to some other configurations like [this](https://git.automotivelinux.org/AGL/meta-agl-devel/tree/meta-agl-jailhouse/recipes-extended/jailhouse/files/qemu-agl.c), it’s the root cell configurations which was modified last year, and I also looked at the root cell configuration present in the `jailhouse-images` repository which fetched to the build directory of it. After some analysis I found some possible areas for solution, some of these are:

- I first checked the `Memory Flags`.

For this, I looked at the jailhouse mailing list for the important follow-ups and looked at some of the related threads, i.e which memory flags are required for the root cell configurations. I found some solutions, but need some more research on it.

- Then I checked the `Interrupt`.

I looked at the configuration and haven’t found anything related to missing interrupts, although there was some interrupt I found in the root cell configuration for arm64, and it was also looking very relevant but again, I need to research more about this.

I also found something different in the configuration comparison with the `jailhouse-images`, below is the code:

File: qemu-x86.c(jailhouse-images)
```sh
		.platform_info = {
			.pci_mmconfig_base = 0xb0000000,
			.pci_mmconfig_end_bus = 0xff,
			.x86 = {
				.pm_timer_address = 0x608,
				.vtd_interrupt_limit = 256,
				.iommu_units = {
					{
						.type = JAILHOUSE_IOMMU_INTEL,
						.base = 0xfed90000,
						.size = 0x1000,
					},
				},
			},
		},
```
In the `.header` of `struct jailhouse_system` data type, I saw some differences, i.e the `.iommu_units` in the `.platform_info` is defined differently in `.x86`. I am not very sure about this difference and will research more about it and give more concrete conclusions next week.

Also, due to my unit tests, I was not able to work for two days, So I will try to accommodate those on weekends, and will spend some time further on this.


Thanks for reading.


**-- Anmol**