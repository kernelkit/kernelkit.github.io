---
title:  OSPF Unnumbered Interfaces
author: vatn
date:   2024-01-30 17:06:42 +0200
tags: [networking, ospf]
---

This post explains how to use *Unnumbered Interfaces* to simplify OSPF
network setup and maintenance in [Infix][1].


## Introduction

When using OSPF over Ethernet point to point links, the work to make
IP plans for the PtP connections can be tedious and limits flexibility
as configurations must be coordinated and matched with each neighbor. 
The figure below is used as illustration.

![](/assets/img/ospf-unnumbered-fig1.svg){: #fig1}
_**Figure 1**: OSPF network with point-to-point links._

The setup disregards any user networks attached to the routers, and
focus solely on establishing connectivity between routers. (Adding
host subnets is described [further below](#adding-host-subnets)).

With OSPF, router connectivity would be easy to configure if the links
can be treated as *unnumbered point-to-point OSPF interfaces*. With
this approach, each router can be configured individually with minimum
coordination with its neighbours. Ideally, R1 should have IP address
10.1.1.1 assigned to its loopback, advertise 10.1.1.1 via OSPF, and
then connect one port (say *eth0*) towards R2 and another (*eth1*)
towards R3.

In contrast, the custom way is to make an IP plan for each connection,
e.g., 10.0.1.0/30 between R1 and R2, assign address 10.0.1.1/30 to
eth0 on R1 and 10.0.1.2/30 to eth0 on R2. And do the same for subnet
10.0.1.4/30 between R1 and R3, etc. This work we wish to avoid.

> With OSPF unnumbered interfaces we can skip coordinating and
> configuring IP addresses on Ethernet LANs used as point-to-point
> links between routers.
{: .prompt-info }


## Unnumbered interfaces in a nutshell

1. Make an IP plan for your routers: In this example R1 is allocated
   the address 10.1.1.1, R2 address 10.1.1.2, etc.
2. Configure the allocated address as /32 to loopback and to all
   Ethernet interfaces intended as unnumbered interfaces
   On R1, we would configure 10.1.1.1/32 to *lo*, *eth0* and *eth1*. 
3. In OSPF configuration there are two settings (done per OSPF area)
   - enable OSPF on intended interfaces
   - set Ethernet interfaces as OSPF *point-to-point* networks
4. Plug the network together

The routers should detect each other, build routing tables for
connectivity.


## Simple setup, two nodes

It's always a good idea to start with the smallest setup to get a feel
for the system. So we start out with a simple network, just two nodes:

![](/assets/img/ospf-unnumbered-fig2.svg){: #fig2}
_**Figure 2**: OSPF network with point-to-point link between two routers._

The first step, the IP plan, is already shown in Figure 2.

### Setting hostname (R1, R2, etc.)

By default, Infix units are assigned the name `infix-12-34-56` where the
last part (`12-34-56`) matches the last 3 octets of the unit's base MAC
address.  In the examples below, we change the hostname to R1, R2, etc.
To change it, login to the unit (console or SSH), and issue `cli`
command to enter [Infix CLI][2].  Then change the hostname as shown
below.

```console
admin@infix-04-00-00:~$ 
admin@infix-04-00-00:~$ cli
admin@infix-04-00-00:/> configure 
admin@infix-04-00-00:/config/> set system hostname R1
admin@infix-04-00-00:/config/>
```

Apply changes by leaving the configuration context with `leave`, then
store changes to the startup configuration.

```console
admin@infix-04-00-00:/config/> leave
admin@R1:/> copy running-config startup-config 
admin@R1:/>
```

> Remember to copy the running configuration to the startup
> configuration to make the changes permanent. `copy running-config
> startup-config` is not shown in remaining examples.

### Configuring IP address (and enabling IP forwarding)

Below we show how to set the address on R1 using the [Infix CLI][2].
As the unit is intended to work as IPv4 router, ensure IP forwarding
is enabled on the Ethernet interface(s).

```console
admin@R1:/> 
admin@R1:/> configure
admin@R1:/config/> set interface lo ipv4 address 10.1.1.1 prefix-length 32
admin@R1:/config/> set interface eth0 ipv4 address 10.1.1.1 prefix-length 32
admin@R1:/config/> set interface eth0 ipv4 forwarding
```

To view the changes done so far, use the `diff` command.

```diff
admin@R1:/config/> diff
interfaces {
  interface eth0 {
+    ipv4 {
+      forwarding true;
+      address 10.1.1.1 {
+        prefix-length 32;
+      }
+    }
  }
  interface lo {
    ipv4 {
+      address 10.1.1.1 {
+        prefix-length 32;
+      }
    }
  }
}
```

Then issue `leave` to activate the changes:

```console
admin@R1:/config/> leave
admin@R1:/>
```

Status of IP address assignment can be viewed using `show interfaces` command.

```console
admin@R1:/> show interfaces 
INTERFACE       PROTOCOL   STATE       DATA                                     
eth0            ethernet   UP          0c:ec:d1:04:00:00                        
                ipv4                   10.1.1.1/32 (static)
                ipv6                   fe80::eec:d1ff:fe04:0/64 (link-layer)
eth1            ethernet   DOWN        0c:ec:d1:04:00:01                        
eth2            ethernet   DOWN        0c:ec:d1:04:00:02                        
eth3            ethernet   DOWN        0c:ec:d1:04:00:03                        
eth4            ethernet   DOWN        0c:ec:d1:04:00:04                        
eth5            ethernet   DOWN        0c:ec:d1:04:00:05                        
eth6            ethernet   DOWN        0c:ec:d1:04:00:06                        
eth7            ethernet   DOWN        0c:ec:d1:04:00:07                        
eth8            ethernet   DOWN        0c:ec:d1:04:00:08                        
eth9            ethernet   DOWN        0c:ec:d1:04:00:09                        
lo              ethernet   UP          00:00:00:00:00:00                        
                ipv4                   127.0.0.1/8 (static)
                ipv4                   10.1.1.1/32 (static)
                ipv6                   ::1/128 (other)
admin@R1:/>
```

 
### OSPF configuration

OSPF configuration takes place under the *routing,
control-plane-protocol and ospfv2* subcontext. In line with the IETF
OSPF YANG model, an OSPF instance name has to be given. The name has
local significance only; here the instance is called
*default*. Enabling OSPF per interface and configuring interface-type
are done within OSPF area context (here the backbone area 0.0.0.0 is
used).

```console
admin@R1:/config/> edit routing control-plane-protocol ospfv2 name default
admin@R1:/config/routing/control-plane-protocol/ospfv2/name/default/> set ospf area 0.0.0.0 interface lo enabled
admin@R1:/config/routing/control-plane-protocol/ospfv2/name/default/> set ospf area 0.0.0.0 interface eth0 enabled
admin@R1:/config/routing/control-plane-protocol/ospfv2/name/default/> set ospf area 0.0.0.0 interface eth0 interface-type point-to-point 
admin@R1:/config/routing/control-plane-protocol/ospfv2/name/default/>
```

Changes can be shown with the `diff` command.

```diff
admin@R1:/config/routing/control-plane-protocol/ospfv2/name/default/> diff
+routing {
+  control-plane-protocols {
+    control-plane-protocol ospfv2 name default {
+      ospf {
+        areas {
+          area 0.0.0.0 {
+            interfaces {
+              interface eth0 {
+                interface-type point-to-point;
+                enabled true;
+              }
+              interface lo {
+                enabled true;
+              }
+            }
+          }
+        }
+      }
+    }
+  }
+}
```

Then issue `leave` to activate the changes:

```console
admin@R1:/config/routing/control-plane-protocol/ospfv2/name/default/> leave
admin@R1:/>
```

When the above configuration is done on R1, and correspondingly on R2
with address 10.1.1.2 ([Figure 2](#fig2)), routing information will be
exchanged using OSPF.

```console
admin@R1:/> show ospf routes 
============ OSPF network routing table ============
N    10.1.1.1/32           [0] area: 0.0.0.0
                           directly attached to lo
N    10.1.1.2/32           [10] area: 0.0.0.0
                           via 10.1.1.2, eth0

============ OSPF router routing table =============

============ OSPF external routing table ===========

admin@R1:/>
```

## Larger setup

Expanding the simple setup, depicted in [Figure 2](#fig2), to the larger
in [Figure 1](#fig1), we should include the remaining interfaces on R1
and R2, and the do the corresponding configuration for R3-R5.

On R1, we need include eth1 for IP settings and OSPF.

Doing IP setting via Infix CLI is shown below (eth1 should get same
config as eth0 got before).

First setting IPv4 address and enable forwarding

```console
admin@R1:/> configure 
admin@R1:/config/> set interface eth1 ipv4 address 10.1.1.1 prefix-length 32
admin@R1:/config/> set interface eth1 ipv4 forwarding
```

Then OSPF configuration

```console
admin@R1:/config/> edit routing control-plane-protocol ospfv2 name default
admin@R1:/config/routing/control-plane-protocol/ospfv2/name/default/> set ospf area 0.0.0.0 interface eth1 enabled
admin@R1:/config/routing/control-plane-protocol/ospfv2/name/default/> set ospf area 0.0.0.0 interface eth1 interface-type point-to-point
```

OSPF configuration can be inspected

```console
admin@R1:/config/routing/control-plane-protocol/ospfv2/name/default/> show
ospf {
  areas {
    area 0.0.0.0 {
      interfaces {
        interface eth0 {
          interface-type point-to-point;
          enabled true;
        }
        interface eth1 {
          interface-type point-to-point;
          enabled true;
        }
        interface lo {
          enabled true;
        }
      }
    }
  }
}
admin@R1:/config/routing/control-plane-protocol/ospfv2/name/default/> leave
admin@R1:/>
```

Corresponding IP and OSPF configuration are then applied to all
routers in [Figure 1](#fig1).  OSPF is able to establish routes to all
routers. Here is the result at R1.

```console
admin@R1:/> show ospf routes 
============ OSPF network routing table ============
N    10.1.1.1/32           [0] area: 0.0.0.0
                           directly attached to lo
N    10.1.1.2/32           [10] area: 0.0.0.0
                           via 10.1.1.2, eth0
N    10.1.1.3/32           [10] area: 0.0.0.0
                           via 10.1.1.3, eth1
N    10.1.1.4/32           [20] area: 0.0.0.0
                           via 10.1.1.2, eth0
N    10.1.1.5/32           [20] area: 0.0.0.0
                           via 10.1.1.3, eth1
N    10.1.1.6/32           [30] area: 0.0.0.0
                           via 10.1.1.2, eth0
                           via 10.1.1.3, eth1

============ OSPF router routing table =============

============ OSPF external routing table ===========


admin@R1:/>
```


## Adding host subnets

In addition to the point-to-point links, the router likely has regular
LAN subnets where hosts can be connected. In Figure 3, host subnets
have been added to R1 and R6.

![](/assets/img/ospf-unnumbered-fig3.svg){: #fig3}
_**Figure 3**: LAN broadcast networks added at R1 and R6._

On R1, *eth2* is configured with IP address 10.0.1.1/24, and enabled
for OSPF within area 0.0.0.0. The eth2 interface is kept as (regular)
broadcast interface (not point-to-point), and as no neighbor OSPF
router is expected on this LAN, eth2 can be configured as a passive
OSPF interface.

```console
admin@R1:/> configure 
admin@R1:/config/> set interface eth2 ipv4 address 10.0.1.1 prefix-length 24
admin@R1:/config/> set interface eth2 ipv4 forwarding
admin@R1:/config/> set routing control-plane-protocol ospfv2 name default ospf area 0.0.0.0 interface eth2 enabled
admin@R1:/config/> set routing control-plane-protocol ospfv2 name default ospf area 0.0.0.0 interface eth2 passive
admin@R1:/config/> leave
admin@R1:/>
```

If corresponding setup is done on R6, OSPF would now establish the
following routing table

```console
admin@R1:/> show ospf routes 
============ OSPF network routing table ============
N    10.0.1.0/24           [10] area: 0.0.0.0
                           directly attached to eth2
N    10.0.6.0/24           [40] area: 0.0.0.0
                           via 10.1.1.2, eth0
                           via 10.1.1.3, eth1
N    10.1.1.1/32           [0] area: 0.0.0.0
                           directly attached to lo
N    10.1.1.2/32           [10] area: 0.0.0.0
                           via 10.1.1.2, eth0
N    10.1.1.3/32           [10] area: 0.0.0.0
                           via 10.1.1.3, eth1
N    10.1.1.4/32           [20] area: 0.0.0.0
                           via 10.1.1.2, eth0
N    10.1.1.5/32           [20] area: 0.0.0.0
                           via 10.1.1.3, eth1
N    10.1.1.6/32           [30] area: 0.0.0.0
                           via 10.1.1.2, eth0
                           via 10.1.1.3, eth1

============ OSPF router routing table =============

============ OSPF external routing table ===========


admin@R1:/>
```

Verify by letting PC1 ping PC2, where attached to R1 and R4
respectivly, see [Figure 3](#fig3).

```console
PC1> ping 10.0.6.26
84 bytes from 10.0.6.26 icmp_seq=1 ttl=60 time=2.536 ms
84 bytes from 10.0.6.26 icmp_seq=2 ttl=60 time=2.386 ms
84 bytes from 10.0.6.26 icmp_seq=3 ttl=60 time=2.629 ms
^C
PC1>
```

Done!

[1]: https://github.com/kernelkit/infix
[2]: https://github.com/kernelkit/infix/tree/main/doc/cli
