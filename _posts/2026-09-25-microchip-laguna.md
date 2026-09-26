---
title: Microchip Laguna Support
author: troglobit
date: 2026-09-25 10:00:00 +0200
categories: [showcase]
tags: [boards]
image:
  path: /assets/img/laguna-evb.png
  alt: Microchip EV23X71A (Laguna) evaluation board
  show_in_post: false
---

Twenty-eight ports, one lonely Cortex-A53, and a switch core that does TSN +
FRER and HSR/PRP in hardware.  Say hello to Microchip *Laguna*, the LAN969x
family, now supported in Infix via the [EV23X71A][1] evaluation board!

![](/assets/img/laguna-evb.png){: #fig1}
_**Figure 1**: Microchip EV23X71A, a.k.a. the Laguna EVB.  Twenty-four RJ45s
and four SFP+ cages along the bottom edge._

### Hardware

The EVB is a big, green, no-nonsense board built around a LAN9696RED, the 60
Gbps member of the family, where *RED* stands for hardware RedBox support
(HSR/PRP).  As eval boards usually go, Microchip have crammed all the things
into this one, but the SoC comes in [many variants][6].  Common to all of them
is the built-in single Cortex-A53 @ 1 GHz.  The EVB also has 1 GiB DDR4, 4 GiB
eMMC, and 2 MiB QSPI NOR flash.  The latter is interesting if you want to try
out Microchip's [VelocityDRIVE][7], have a look at it!

The many ports are what you came for:

- 24 x 1 GbE copper, six QSGMII quad PHYs in a neat row
- 4 x 10 GbE SFP+
- 1 x 1 GbE RGMII management port

It is the smaller sibling of the [SparX-5i][2], and the same Linux `sparx5`
switchdev driver takes care of both, so everything you already know about
bridging and VLANs in Infix works just as expected, all fully offloaded to the
switch core, no CPU cycles involved.

### Support Status

The EVB starts out as Tier 3 in Infix, keeping the SparX-5i company.  That
sounds modest, and it is, but only because an evaluation board rarely ends
up in a rack.  What works today:

- switch core and SerDes, all 29 ports, with switchdev offload
- eMMC, I²C (including the SFP mux), SPI, and USB host
- watchdog, temperature sensor, status and SFP LEDs
- our own boot chain, from BL2 all the way to an A/B slot on the eMMC

That last bullet deserves a moment.  The U-Boot the board ships with cannot
boot Infix from eMMC, so we built our own FIP (Trusted Firmware + U-Boot)
from Microchip's trees.  Along the way we tripped over a boot ROM erratum
that made roughly one build in a hundred fail to authenticate, which was
*great* fun to track down.  The whole story is in the [board README][3].

Not there yet: TSN queueing (time aware shaping, PSFP) and HSR/PRP offload.
The hardware is more than ready, it's our turn now.

### Getting Started

Laguna support ships with the upcoming v26.09 release, and until then in
the [latest][4] builds.  The bootloader is built with `laguna_boot_defconfig`
and the rest is the standard aarch64 image, combined for eMMC using
`utils/mkimage.sh`.  The [board README][3] walks you through it all:
boot mode strapping, installing the FIP, netbooting, and (just in case)
debricking.

### Coming Soon: Novarq Tactical-1000

Why all this effort on an EVB?  Because it is the reference design for real
products, and the first one is already on the bench.  The [Novarq
Tactical-1000][5] is basically a cut-down Laguna EVB in a box: same 24 + 4 +
1 port layout, a LAN9696TSN, 2 GiB RAM, and a CPU fan that looks suspiciously
like it was borrowed from a Raspberry Pi 5.

![](/assets/img/novarq-tactical-1000.webp){: #fig2}
_**Figure 2**: Novarq Tactical-1000, the business end.  Photo: Novarq._

This one is going straight to **Tier 1**, i.e., included in the default
builds and tested in our regression test system.  
Stay tuned!

[1]: https://www.microchip.com/en-us/development-tool/ev23x71a
[2]: https://ww1.microchip.com/downloads/en/DeviceDoc/00002854B.pdf
[3]: https://github.com/kernelkit/infix/tree/main/board/aarch64/microchip-ev23x71a
[4]: https://github.com/kernelkit/infix/releases/tag/latest
[5]: https://novarq.com/pages/tactical-1000
[6]: https://github.com/kernelkit/infix/tree/main/board/aarch64/microchip-ev23x71a#switch-core
[7]: https://www.microchip.com/en-us/software-library/velocitydrive-sp
