---
title:  Infix Compatible Boards
author: troglobit
date:   2024-08-13 10:06:42 +0100
categories: [showcase]
tags: [boards]
---

Much thanks to the solid foundation curated by [Buildroot][1], Infix can
quite easily be ported to any system that supports Linux.  The only real
hardware requirement is "enough" RAM and storage, and if the board has a
built-in switch, that it is supported by switchdev.

Currently the following boards are fully supported.  Other boards have
been known to work, but have not been updated or tested continuously.

 - [Marvell CN9130][7] CRB (ARM)
 - [FriendlyELEC NanoPi R2S][9] (ARM)
 - [StarFive VisionFive2][8] (RISC-V)
 - [Qemu][0] (x86_64)

> Although not really a "board", Qemu can be quite useful for anyone who
> just want to understand what Infix is.  All releases, as well as the
> [*latest*][2] (nightly) builds, have an x86_64 image that can be run
> on any Linux PC with Qemu installed ([instructions][10]).


### Marvell CN9130 CRB

This *Customer Reference Board* is really expensive and not really
suited to everyone, even if you can get hold of one (!), but it remains
*the* main reference for Infix so far and has seen multiple customer
specific board spin-offs.

![](/assets/img/cn9130-crb.png){: #fig1}
_**Figure 1**: Marvell CN9130 CRB._

The CN9130 is a quad-core Cortex-A72 coupled with a Marvell 88E6393X
11-port switchcore (one port connected to the SoC).  Only 9 of the
remaining switch ports are accessible, however.

Thanks to Linux switchdev, when Infix runs on this board, all bridging
(switching) configuration, including VLANs, is fully offloaded to the
switchcore.  Allowing full wirespeed switching between switch ports.


### FriendlyELEC NanoPi R2S

In stark contrast to the CRB, the tiny little R2S is *very* cheap and
available from many sources.  It's nowhere near as powerful, of course,
but gives you much bang for [the buck][5]!

![](/assets/img/nanopi-r2s-board.png){: #fig2}
_**Figure 2**: NanoPi R2S._

Only two 1 Gbps Ethernet ports and no WiFi (on this version), and the
SoC is "only" a quad-core Cortex-A53.  Still, considering its size and
cost, a very capable little device.

Infix supports *all* features of this device:

 - routing between interfaces
 - usb port, for storage/logging
 - reset button, incl. factory reset at power-on
 - system LEDs to indicate bootup, factory-reset, and operating mode

The board has become so popular that they've now made an R2S *Plus* with
onboard eMMC, and WiFi over SDIO.  Support for the Plus is coming very
soon to Infix.

![](/assets/img/nanopi-r2s-overview.jpg){: #fig3}
_**Figure 3**: NanoPi R2S Plus Overview of functions._

> There are also spin-offs on the same theme with more powerful CPUs and
> 2.5 Gbps Ethernet: [R4S][11], [R5S][12], [R6S][13] ... all of which
> could easily be supported as well on Infix with a little bit of time
> and patience.


### StarFive VisionFive2

One of the most exciting things to witness, over past decade or so, is
the rise of RISC-V.  StarFive decided to make a follow-up to the first,
very popular VisionFive, with a bit more powerful CPU, quad-core U74
called [JH7110][3], yet keeping with the values of the original.  Not as
cheap as the R2S, it still brings a [lot of value][6].

The board is actually very similar to the RaspberryPi family, with the
distinct difference of having two Ethernet ports, making it suitable
for use as a home network router.

![](/assets/img/visionfive2-overview.png){: #fig4}
_**Figure 4**: StarFive VisionFive2._

Infix supports only a subset of all the features of this board.  As
always, the focus is on networking, but [PoE][4], eMMC support, and
the M.2 slot stand out as candidates for exploration.

[1]: https://buildroot.org
[2]: https://github.com/kernelkit/infix/releases/tag/latest
[3]: https://www.cnx-software.com/2022/08/29/starfive-jh7110-risc-v-processor-specifications/
[4]: https://bootlin.com/blog/power-over-ethernet-poe-support-into-the-official-linux-kernel/
[5]: https://www.aliexpress.com/w/wholesale-nanopi-r2s-metal-case.html?spm=a2g0o.detail.search.0
[6]: https://www.aliexpress.com/w/wholesale-visionfive2.html?spm=a2g0o.productlist.search.0
[7]: https://www.marvell.com/content/dam/marvell/en/public-collateral/embedded-processors/marvell-infrastructure-processors-octeon-tx2-cn913x-product-brief.pdf
[8]: https://doc-en.rvspace.org/VisionFive2/Landing_Page/VisionFive_2/introduction.html
[9]: https://wiki.friendlyelec.com/wiki/index.php/NanoPi_R2S
[0]: https://www.qemu.org/
[10]: https://kernelkit.org/infix/latest/virtual/
[11]: https://www.friendlyelec.com/index.php?route=product/product&product_id=284
[12]: https://www.friendlyelec.com/index.php?route=product/product&product_id=287&search=r5s&description=true&category_id=0&sub_category=true
[13]: https://www.friendlyelec.com/index.php?route=product/product&product_id=289
