---
icon: fas fa-microchip
order: 3
---

Infix can run on many different types of [architectures and boards][1],
much thanks to Linux and Buildroot.  Currently the focus is on 64-bit
ARM devices with switching fabric supported by Linux switchdev.

The following boards have been know to run Infix.  They are divided into
three tiers to denote level of support.

### Tier 1

Fully supported in default builds, with images included in releases:

 - [Marvell CN9130][5] CRB (ARM)
 - Qemu[^1] (x86_64)

### Tier 2

Separate `defconfig`, may use custom kernel and bootloader.  May
graduate to *Tier 1* support depending on community interest and
participation.

 - [StarFive VisionFive2][6] (RISC-V)
 - [NanoPi R2S][7] (ARM)

### Tier 3

Worked at one point but needs more attention to bring on par with the
Infix boot sequence, and testing:

 - [Microchip SparX-5i][8] PCB135 (ARM)
 - [Marvell EspressoBIN][9] (ARM)



------

**Footnotes**

[^1]: The [Qemu][2] x86_64 is primarily intended for development and
    testing, but can also be used for evaluation and demo purposes in
    [GNS3][3].  For more information, see: [Infix in Virtual
    Environments][4].

[1]: https://github.com/kernelkit/infix/tree/main/board
[2]: https://www.qemu.org/
[3]: https://www.gns3.com/
[4]: https://github.com/kernelkit/infix/blob/main/doc/virtual.md
[5]: https://www.marvell.com/content/dam/marvell/en/public-collateral/embedded-processors/marvell-infrastructure-processors-octeon-tx2-cn913x-product-brief.pdf
[6]: https://doc-en.rvspace.org/VisionFive2/Landing_Page/VisionFive_2/introduction.html
[7]: https://wiki.friendlyelec.com/wiki/index.php/NanoPi_R2S
[8]: https://ww1.microchip.com/downloads/en/DeviceDoc/00002854B.pdf
[9]: http://espressobin.net/
