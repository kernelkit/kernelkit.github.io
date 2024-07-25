---
icon: fas fa-microchip
order: 3
---

Infix can run on many different types of architectures and boards, much
thanks to Linux and Buildroot.  Currently the focus is on 64-bit ARM
devices with switching fabric supported by Linux switchdev.

The [following boards][1] are fully supported:

 - Marvell CN9130 CRB
 - Marvell EspressoBIN
 - Microchip SparX-5i PCB135 (eMMC)
 - NanoPi R2S


An x86_64 build is also available, primarily intended for development
and testing using [Qemu][2], but can also be used for evaluation and
demo purposes in [GNS3][3].  For more information, see: [Infix in
Virtual Environments][4].

[1]: https://github.com/kernelkit/infix/blob/main/board/aarch64/README.md
[2]: https://www.qemu.org/
[3]: https://www.gns3.com/
[4]: https://github.com/kernelkit/infix/blob/main/doc/virtual.md
