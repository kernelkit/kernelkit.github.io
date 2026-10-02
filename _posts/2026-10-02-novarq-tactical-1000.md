---
title: Novarq Tactical-1000 Support
author: troglobit
date: 2026-10-02 09:10:00 +0200
categories: [showcase]
tags: [boards]
image:
  path: /assets/img/novarq-tactical-1000.webp
  alt: Novarq Tactical-1000, a 29-port LAN969x switch
  show_in_post: false
---

We said "stay tuned" at the end of the [Laguna post][laguna].  Here it
is: the [Novarq Tactical-1000][novarq], the first product built on the
Microchip LAN969x family to run Infix.  Supported from the
[v26.09][v26.09] release, as a [Tier 1](/platforms) board from day one.

![](/assets/img/tactical-front.jpeg){: .normal width="360" } ![](/assets/img/tactical-no-lid.jpeg){: .normal width="360" }
_**Figure 1**: Novarq Tactical-1000, front panel and with the lid off.  Photos: Novarq._

### Hardware

Built on the [Laguna][lan969x] reference design, in a compact enclosure:

- LAN9696TSN, the TSN variant of the family, one Cortex-A53 at 1 GHz
- 2 GiB DDR4, 16 GB eMMC, QSPI NOR flash
- 24 x 1 GbE copper, 4 x 10 GbE SFP+, 1 x 1 GbE management port
- USB-C serial console

The ports come up as `e1` to `e24`, `e25` to `e28` for the SFP+ cages,
and `mgmt`, matching the labels on the front panel.  The [board
README][readme] has the full port map and the LED assignment.

### Support Status

The switch core uses the same driver as the Laguna EVB, so bridging and
VLANs are offloaded to hardware on all 29 ports.  Infix runs from eMMC
with A/B slots, so upgrades are atomic with a fallback to the previous
image, and [unattended updates][unattended] work like on any other
board.

In progress: TSN support, e.g., time aware shaping, CBS, and PSFP.

### Getting Started

Use the Novarq U-Boot to netboot Infix once, then install to eMMC from
the running system.  The [Netboot HowTo][netboot] covers the DHCP and
TFTP side, and the [board README][readme] walks you through the rest.

From there on the unit upgrades like any other Infix system.

### Tier 1

The Tactical-1000 is part of the default `aarch64` build, so every
release and every `latest` build carries it, and it is being added to
our regression test system.

![](/assets/img/tactical-1000-unpacked.png){: #fig2}
_**Figure 2**: Just unpacked, next to a couple of tiny BPi-R3 routers.  Photo: J. Wiberg._

[lan969x]: https://www.microchip.com/en-us/product/lan9694
[laguna]: /posts/microchip-laguna/
[v26.09]: https://github.com/kernelkit/infix/releases/tag/v26.09.0
[novarq]: https://novarq.com/pages/tactical-1000
[readme]: https://github.com/kernelkit/infix/blob/main/board/aarch64/novarq-tactical-1000/README.md
[netboot]: https://kernelkit.org/infix/latest/netboot/
[unattended]: https://kernelkit.org/infix/latest/upgrade/#unattended-software-updates
