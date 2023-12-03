Infix is a Linux Network Operating System (NOS) based on [Buildroot][1],
and [sysrepo][2].  A powerful mix that ease porting to different
platforms, simplify long-term maintenance, and provide made-easy
management using NETCONF[^1] (remote) or the built-in [command line
interface (CLI)][3]

## Example CLI Session

The CLI configure context is automatically generated from the loaded
YANG models and their corresponding [sysrepo][2] plugins.  The following
is brief example of how to set the IP address of an interface:

```
admin@infix-12-34-56:/> configure
admin@infix-12-34-56:/config/> edit interface eth0
admin@infix-12-34-56:/config/interface/eth0/> set ipv4 <TAB>
      address     autoconf bind-ni-name      enabled
	  forwarding  mtu      neighbor
admin@infix-12-34-56:/config/interface/eth0/> set ipv4 address 192.168.2.200 prefix-length 24
admin@infix-12-34-56:/config/interface/eth0/> show
type ethernet;
ipv4 {
  address 192.168.2.200 {
    prefix-length 24;
  }
}
ipv6
admin@infix-12-34-56:/config/interface/eth0/> diff
interfaces {
  interface eth0 {
+    ipv4 {
+      address 192.168.2.200 {
+        prefix-length 24;
+      }
+    }
  }
}
admin@infix-12-34-56:/config/interface/eth0/> leave
admin@infix-12-34-56:/> show interfaces
INTERFACE       PROTOCOL   STATE       DATA
eth0            ethernet   UP          52:54:00:12:34:56
                ipv4                   192.168.2.200/24 (static)
                ipv6                   fe80::5054:ff:fe12:3456/64 (link-layer)
lo              ethernet   UP          00:00:00:00:00:00
                ipv4                   127.0.0.1/8 (static)
                ipv6                   ::1/128 (static)
admin@infix-12-34-56:/> copy running-config startup-config
```

[Click here][3] for more details.

## Platform Support

Infix can run on many different types of architectures and boards, much
thanks to Linux and Buildroot.  Currently the focus is on 64-bit ARM
devices with switching fabric supported by Linux switchdev.

The [following boards][4] are fully supported:

 - Marvell CN9130 CRB
 - Marvell EspressoBIN
 - Microchip SparX-5i PCB135 (eMMC)

An x86_64 build is also available, primarily intended for development
and testing using [Qemu][5], but can also be used for evaluation and
demo purposes in [GNS3][5].  For more information, see: [Infix in
Virtual Environments][5].

[^1]: NETCONF or RESTCONF, <https://datatracker.ietf.org/doc/html/rfc8040>,
    for more information, see [Infix Variants][6].

[1]: https://buildroot.org/
[2]: https://www.sysrepo.org/
[3]: https://github.com/kernelkit/infix/blob/main/doc/cli/introduction.md
[4]: https://github.com/kernelkit/infix/blob/main/board/aarch64/README.md
[5]: https://github.com/kernelkit/infix/blob/main/doc/virtual.md
[6]: https://github.com/kernelkit/infix/blob/main/doc/variant.md
