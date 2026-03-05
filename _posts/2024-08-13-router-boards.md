---
title:  Infix Compatible Boards
author: troglobit
date:   2024-08-13 10:06:42 +0100
last_modified_at: 2026-03-05 16:00:00 +0100
categories: [showcase]
tags: [boards]
pin: true
---

Much thanks to the solid foundation curated by [Buildroot][1], Infix can
quite easily be ported to any system that supports Linux.  The only real
hardware requirement is "enough" RAM and storage, and if the board has a
built-in switch, that it is supported by switchdev.

Currently the following boards are fully supported.  Other boards have
been known to work, but have not been updated or tested continuously.

| **Board**                                           | **Arch** | **Since** | **Notes**                  |
|-----------------------------------------------------|----------|-----------|----------------------------|
| [Banana Pi BPi-R3](#banana-pi-bpi-r3)               | Aarch64  | v25.09    |                            |
| [Banana Pi BPi-R3 Mini](#banana-pi-bpi-r3-mini)     | Aarch64  | v26.02    |                            |
| [FriendlyELEC NanoPi R2S](#friendlyelec-nanopi-r2s) | Aarch64  | v24.02    | Fully supported in v24.08  |
| [Marvell CN9130 CRB](#marvell-cn9130-crb)           | Aarch64  | v23.06    |                            |
| [Microchip SAMA7G54-EK](#microchip-sama7g54-ek)     | Arm      | v26.02    |                            |
| [NXP i.MX8MP EVK](#nxp-imx8mp-evk)                  | Aarch64  | v25.02    |                            |
| [Raspberry Pi](#raspberry-pi) 4B, 3B, CM4           | Aarch64  | v25.05    | 3B and CM4 added in v25.10 |
| [Raspberry Pi](#raspberry-pi) 2B                    | Arm      | v25.11    |                            |
| [StarFive VisionFive2](#starfive-visionfive2)       | RISC-V   | v24.08    |                            |
| [Qemu](#qemu)                                       | x86_64   | v23.06    |                            |
{: style="margin: 0 auto; width: auto" }


### Banana Pi BPi-R3

The [BPi-R3][14] is an affordable (€130-€150) WiFi-capable router board
built around the MediaTek MT7986A (Filogic 830), a quad-core Cortex-A53
at 2.0 GHz with 2 GB DDR4 RAM.

![](/assets/img/bpi-r3-board.jpg){: #fig1}
_**Figure 1**: Banana Pi BPi-R3._

What makes this board stand out is its networking hardware: a MediaTek
MT7531A 5-port gigabit switch with full switchdev offload, plus two 2.5
Gbps SFP ports.  Storage options include microSD, eMMC, and M.2 NVMe.

Infix supports all hardware features:

 - routing between interfaces
 - built-in 5-port switch with switchdev offload
 - 2.5 Gbps Ethernet and SFP connectivity
 - USB 3.0 port, microSD, eMMC, and M.2 NVMe storage
 - system LEDs and reset button
 - dual-band 802.11ax WiFi (access point and station modes)

For detailed setup instructions, see the [BPi-R3 announcement][15].

 - <https://www.amazon.com/OpenSource-Wireless-Dual-Band-MediaTek-Bundle3-BPI/dp/B0BDG9VNJP>
 - <https://www.conrad.se/sv/p/banana-pi-bananapi-r3-banana-pi-bpi-r3-2-gb-2798410.html>
 - <https://www.aliexpress.com/w/wholesale-Banana-Pi-Bpi-R3.html>

### Banana Pi BPi-R3 Mini

The [BPi-R3 Mini][22] is a more compact variant of the BPi-R3, built
around the same MediaTek MT7986A (Filogic 830) SoC.  It trades the R3's
SFP ports and microSD slot for a smaller form factor with 2× 2.5 Gbps
RJ45 ports and 8 GB onboard eMMC.

![](/assets/img/bpi-r3-mini-board.jpeg){: #fig2}
_**Figure 2**: Banana Pi BPi-R3 Mini._

With 2 GB DDR4 RAM, dual-band WiFi 6 (802.11ax) across two radios, and
support for up to 16 virtual APs per radio, it is a natural fit as a
managed WiFi AP router or wireless repeater.

![Banana Pi BPi-R3 Mini](/assets/img/bpi-r3-mini.jpg){: width="300" .right}

Infix supports all hardware features:

 - routing between interfaces
 - 2× 2.5 Gbps Ethernet (RJ45) connectivity
 - 8 GB onboard eMMC storage
 - USB 2.0 port
 - system LEDs and reset button
 - dual-band 802.11ax WiFi (access point and station modes)

Support for the BPi-R3 Mini was added in Infix v26.02.

 - <https://www.amazon.com/OpenWRT-Wireless-Dual-Band-MediaTek-Filogic/dp/B0CH37C83X>

### FriendlyELEC NanoPi R2S

In stark contrast to the CRB, the tiny [NanoPi R2S][9] is *very* cheap
and available from many sources.  It's nowhere near as powerful, of
course, but gives you much bang for [the buck][5]!

![](/assets/img/nanopi-r2s-board.png){: #fig3}
_**Figure 3**: NanoPi R2S._

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

![](/assets/img/nanopi-r2s-overview.jpg){: #fig4}
_**Figure 4**: NanoPi R2S Plus Overview of functions._

 - <https://www.friendlyelec.com/index.php?route=product/product&product_id=282>
 - <https://www.amazon.com/FriendlyElec-NanoPi-R2S-Computer-Dual-Gbps/dp/B0869K2P8R>
 - <https://www.aliexpress.com/w/wholesale-FriendlyELEC-NanoPi-R2S.html>

> There are also spin-offs on the same theme with more powerful CPUs and
> 2.5 Gbps Ethernet: [R4S][11], [R5S][12], [R6S][13] ... all of which
> could easily be supported as well on Infix with a little bit of time
> and patience.


### Marvell CN9130 CRB

The [CN9130 CRB][7] is a *Customer Reference Board* — really expensive
and not easy to get hold of — but it remains *the* main reference for
Infix so far and has seen multiple customer specific board spin-offs.

![](/assets/img/cn9130-crb.png){: #fig5}
_**Figure 5**: Marvell CN9130 CRB._

The CN9130 is a quad-core Cortex-A72 coupled with a Marvell 88E6393X
11-port switchcore (one port connected to the SoC).  Only 9 of the
remaining switch ports are accessible, however.

Thanks to Linux switchdev, when Infix runs on this board, all bridging
(switching) configuration, including VLANs, is fully offloaded to the
switchcore.  Allowing full wirespeed switching between switch ports.


### Microchip SAMA7G54-EK

The [SAMA7G54-EK][23] is the evaluation kit for Microchip's SAMA7G54
SoC, an Arm Cortex-A7 processor.

Support for the SAMA7G54-EK was added in Infix v26.02.


### NXP i.MX8MP EVK

The [i.MX8MP EVK][24] is NXP's evaluation kit for the i.MX 8M Plus SoC,
a quad-core Cortex-A53 at up to 1.8 GHz.  Beyond its CPU cores, the SoC
integrates a 2.3 TOPS Neural Processing Unit, making it a platform of
interest for edge inference workloads.

The board ships with 6 GB LPDDR4 RAM, dual Gigabit Ethernet (one via the
SoC's FEC controller and one via the STMMAC/DWMAC IP block), two CAN FD
ports (FlexCAN), eMMC and SD card storage, USB 3.0 (USB-C with Power
Delivery), and HDMI output.

Infix supports all hardware features relevant to networking:

 - routing between the two Gigabit Ethernet interfaces
 - two CAN FD interfaces (FlexCAN)
 - eMMC and SD card storage
 - USB 3.0 (USB-C with Power Delivery)
 - HDMI output

Support for the i.MX8MP EVK was added in Infix v25.02.

 - <https://www.nxp.com/design/design-center/development-boards-and-designs/i-mx-evaluation-and-development-boards/evaluation-kit-for-the-i-mx-8m-plus-applications-processor:8MPLUSLPD4-EVK>


### Raspberry Pi

The [Raspberry Pi][16] family needs no introduction.  Infix supports
several models across both 32-bit and 64-bit ARM architectures:

**64-bit (aarch64):**

 - **[Pi 3B][25]**: BCM2837B0 quad-core Cortex-A53 @ 1.4 GHz, 1 GB RAM, 100 Mbps Ethernet
 - **[Pi 4B][26]**: BCM2711 quad-core Cortex-A72 @ 1.5 GHz, 1-8 GB RAM, Gigabit Ethernet
 - **[Compute Module 4][27]**: Same processor as Pi 4B, optional eMMC, compact form factor

**32-bit (aarch32):**

 - **[Pi 2B][28]**: BCM2836 quad-core Cortex-A7 @ 900 MHz, 1 GB RAM, 100 Mbps Ethernet

![](/assets/img/raspberrypi4b.png){: #fig6}
_**Figure 6**: Raspberry Pi 4 Model B._


All models include WiFi (dual-band on Pi 3B/4B) and Bluetooth.  The Pi 4B
and CM4 also support various carrier boards, including the [CM4 IoT Router
Board Mini][17] and [CM4 NVMe NAS][18] enclosures.

Infix provides DHCP-enabled Ethernet out of the box, with WiFi available
for client or access point modes.  However, these boards have limitations
for routing use cases: a single Ethernet port and CPU-based packet
processing (no hardware switching offload).

> **Note:** Pi 2B revision 1.2 uses a BCM2837 and is *not* supported.
> The supported Pi 2B uses BCM2836.

 - <https://www.raspberrypi.com/resellers/>

For installation, download an SD card image from the [latest bootloader][21]
builds and flash to a microSD card.  Default credentials are `admin`/`admin`
via SSH.  See the [64-bit README][19] or [32-bit README][20] for details.


### StarFive VisionFive2

One of the most exciting things to witness, over past decade or so, is
the rise of RISC-V.  StarFive decided to make a follow-up to the first,
very popular VisionFive, with a bit more powerful CPU, quad-core U74
called [JH7110][3], yet keeping with the values of the original.  Not as
cheap as the R2S, it still brings a [lot of value][6].

The [VisionFive2][8] is actually very similar to the RaspberryPi family,
with the distinct difference of having two Ethernet ports, making it
suitable for use as a home network router.

![](/assets/img/visionfive2-overview.png){: #fig7}
_**Figure 7**: StarFive VisionFive2._

Infix supports only a subset of all the features of this board.  As
always, the focus is on networking, but [PoE][4], eMMC support, and
the M.2 slot stand out as candidates for exploration.

 - <https://www.amazon.com/VisionFive2-StarFive-JH7110-64-bit-Integrated/dp/B0C7MV1RJ8>
 - <https://www.aliexpress.com/w/wholesale-visionfive2.html>


### Qemu

Although not really a "board", [Qemu][0] can be quite useful for anyone
who just wants to understand what Infix is.  All releases, as well as
the [*latest*][2] (nightly) builds, have an x86_64 image that can be
run on any Linux PC with Qemu installed ([instructions][10]).

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
[14]: https://wiki.banana-pi.org/Banana_Pi_BPI-R3
[15]: /posts/banana-pi-r3/
[16]: https://www.raspberrypi.com/
[17]: https://www.waveshare.com/wiki/CM4-IO-BASE-A
[18]: https://www.waveshare.com/wiki/CM4-NAS-Double-Deck
[19]: https://github.com/kernelkit/infix/blob/main/board/aarch64/raspberrypi-rpi64/README.md
[20]: https://github.com/kernelkit/infix/blob/main/board/aarch32/raspberrypi-rpi2/README.md
[21]: https://github.com/kernelkit/infix/releases/tag/latest-boot
[22]: https://wiki.banana-pi.org/Banana_Pi_BPI-R3_Mini
[23]: https://www.microchip.com/en-us/development-tool/ev21h18a
[24]: https://www.nxp.com/design/design-center/development-boards-and-designs/i-mx-evaluation-and-development-boards/evaluation-kit-for-the-i-mx-8m-plus-applications-processor:8MPLUSLPD4-EVK
[25]: https://www.raspberrypi.com/products/raspberry-pi-3-model-b/
[26]: https://www.raspberrypi.com/products/raspberry-pi-4-model-b/
[27]: https://www.raspberrypi.com/products/compute-module-4/
[28]: https://www.raspberrypi.com/products/raspberry-pi-2-model-b/
