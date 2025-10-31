---
icon: fas fa-microchip
order: 3
---

Infix runs on many different types of [architectures and boards][1], thanks
largely to Linux and Buildroot. The project started out with a heavy network
focus, and that remains its strength, though it is now expanding to include
end devices like the Raspberry Pi.

The following boards are known to run Infix. The list is divided into three
tiers to denote level of support.

### Tier 1

Fully supported in default builds, verified continuously in regression test
system, images included in releases:

- [Marvell CN9130][5] CRB (ARM64)
- [GNS3][3]/Qemu[^1] (ARM64, x86_64)

### Tier 2

Possibly Separate `defconfig`, may even use custom kernel and bootloader.
Can graduate to *Tier 1* support by HW included in regression test system
and being part of default builds.

- [Raspberry Pi 4B][10] (ARM64)
- [Banana Pi-R3][12] (ARM64)
- [NanoPi R2S][7] (ARM64)

### Tier 3

Worked at one point but needs more attention to bring on par with the Infix
boot sequence and testing:

- [Microchip SparX-5i][8] PCB135 (ARM64)
- [Marvell EspressoBIN][9] (ARM64)
- [StarFive VisionFive2][6] (RISC-V)

---

### Footnotes

[^1]: The [Qemu][2] x86_64 support is primarily intended for development
    and testing, but can also be used for evaluation and demo purposes
    using the [Infix appliance][11] in [GNS3][3]. For more information,
    see: [Infix in Virtual Environments][4].

[1]: https://github.com/kernelkit/infix/tree/main/board
[2]: https://www.qemu.org/
[3]: https://www.gns3.com/
[4]: https://github.com/kernelkit/infix/blob/main/doc/virtual.md
[5]: https://www.marvell.com/content/dam/marvell/en/public-collateral/embedded-processors/marvell-infrastructure-processors-octeon-tx2-cn913x-product-brief.pdf
[6]: https://doc-en.rvspace.org/VisionFive2/Landing_Page/VisionFive_2/introduction.html
[7]: https://wiki.friendlyelec.com/wiki/index.php/NanoPi_R2S
[8]: https://ww1.microchip.com/downloads/en/DeviceDoc/00002854B.pdf
[9]: https://espressobin.net/
[10]: https://www.raspberrypi.com/products/raspberry-pi-4-model-b/
[11]: https://www.gns3.com/marketplace/appliances/infix
[12]: https://wiki.banana-pi.org/Banana_Pi_BPI-R3
