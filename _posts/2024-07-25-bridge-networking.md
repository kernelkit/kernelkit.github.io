---
title: Basic Bridge Networking
author: troglobit
date: 2024-07-25 08:23:00 +0100
categories: [examples]
tags: [cli, networking, bridge]
pin: false
---

This is an example of how to set up a VLAN transparent bridge with a
DHCP assigned IP address.  We have a system with two interfaces, or
ports, named `eth0` and `eth1`.

```console
admin@example:/> show interfaces 
INTERFACE       PROTOCOL   STATE       DATA
lo              ethernet   UP          00:00:00:00:00:00
                ipv4                   127.0.0.1/8 (static)
                ipv6                   ::1/128 (static)
eth0            ethernet   UP          00:c0:ff:ee:00:01
                ipv6                   2001:db8:0:1:2c0:ffff:feee:1/64 (link-layer)
                ipv6                   fe80::2c0:ffff:feee:1/64 (link-layer)
eth1            ethernet   UP          00:c0:ff:ee:00:02
                ipv6                   fe80::2c0:ffff:feee:2/64 (link-layer)
```

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
admin@example:/config/> set dhcp-client client-if br0
admin@example:/config/> leave
admin@example:/>
```

Back in admin-exec mode we inspect the changes and notice the bridge has
already got a DHCP lease from the server.

```console
admin@example:/> show interfaces 
INTERFACE       PROTOCOL   STATE       DATA
lo              ethernet   UP          00:00:00:00:00:00
                ipv4                   127.0.0.1/8 (static)
                ipv6                   ::1/128 (static)
br0             bridge
│               ethernet   UP          00:c0:ff:ee:00:01
│               ipv4                   192.168.1.161/24 (dhcp)
├ eth0          bridge     FORWARDING
└ eth1          bridge     FORWARDING

admin@example:/> 
```

Remember to save your changes for next boot:

```console
admin@infix:/> copy running-config startup-config
```

> For more information about networking in Infix, see the [official
> documentation][0].
{: .prompt-info }

[0]: https://github.com/kernelkit/infix/blob/main/doc/networking.md
