---
title: Basic Bridge Networking
author: troglobit
date: 2024-07-25 08:23:00 +0100
last_modified_at: 2026-09-14 12:00:00 +0100
categories: [examples]
tags: [cli, networking, bridge]
image:
  path: /assets/img/bridge-topology.svg
  alt: Topology with an Infix switch bridging the office LAN and a PC
  show_in_post: false
pin: false
---

This is an example of how to set up a VLAN transparent bridge with a
DHCP assigned IP address.  We have a system with two interfaces, or
ports, named `eth0` and `eth1`.

![](/assets/img/bridge-topology.svg){: #fig1 width="700" }
_**Figure 1**: `eth0` faces the office LAN, `eth1` a PC.  Both become
ports of `br0`, which picks up a management address over DHCP._

```console
admin@example:/> show interface
⚑ INTERFACE       PROTOCOL      STATE       DATA
  lo              loopback      UP
                  ipv4                      127.0.0.1/8 (static)
                  ipv6                      ::1/128 (static)
  eth0            1000baseT     UP          duplex: full
                  ethernet                  00:c0:ff:ee:00:01
                  ipv6                      fe80::2c0:ffff:feee:1/64 (link-layer)
  eth1            1000baseT     UP          duplex: full
                  ethernet                  00:c0:ff:ee:00:02
                  ipv6                      fe80::2c0:ffff:feee:2/64 (link-layer)
```

Each interface is listed as a stack of protocol rows.  For a port with
link, the top row names the IEEE PMD type and the negotiated duplex,
with the MAC address on the `ethernet` row below it.

Creating a bridge and setting our interfaces as bridge ports is a
straight forward operation.

```console
admin@example:/> configure
admin@example:/config/> set interface br0
admin@example:/config/> set interface eth0 bridge-port bridge br0
admin@example:/config/> set interface eth1 bridge-port bridge br0
```

> Because it does not make much sense to have IP addresses on bridge
> ports, Infix takes care to disable IPv6 SLAAC on them automatically
> when we attach interfaces to a bridge.
{: .prompt-tip }

We can use the `diff` command to inspect the changes.  Notice how the
system has automatically set the bridge interface type for us.  This
is a feature of the CLI and not available over NETCONF or RESTCONF.

```diff
admin@example:/config/> diff
interfaces {
+  interface br0 {
+    type bridge;
+  }
  interface eth0 {
+    bridge-port {
+      bridge br0;
+    }
  }
  interface eth1 {
+    bridge-port {
+      bridge br0;
+    }
  }
}
```

Now we enable a DHCP client on `br0` and activate the changes.

```console
admin@example:/config/> edit interface br0
admin@example:/config/interface/br0/> set ipv4 dhcp
admin@example:/config/interface/br0/> leave
admin@example:/>
```

Back in admin-exec mode we inspect the changes and notice the bridge has
already got a DHCP lease from the server.

```console
admin@example:/> show interface
⚑ INTERFACE       PROTOCOL      STATE       DATA
  lo              loopback      UP
                  ipv4                      127.0.0.1/8 (static)
                  ipv6                      ::1/128 (static)
  br0             bridge
  │               ethernet                  00:c0:ff:ee:00:01
  │               ipv4                      192.168.1.161/24 (dhcp)
  │               ipv6                      fe80::2c0:ffff:feee:1/64 (link-layer)
  ├ eth0          bridge        FORWARDING
  └ eth1          bridge        FORWARDING

admin@example:/>
```

The ports are now listed beneath `br0`, each in the `FORWARDING` state,
and the addresses have moved from the ports to the bridge.

Remember to save your changes for next boot:

```console
admin@example:/> copy running-config startup-config
```

> For more information about networking in Infix, see the [official
> documentation][0].
{: .prompt-info }

[0]: https://www.kernelkit.org/infix/latest/networking/
