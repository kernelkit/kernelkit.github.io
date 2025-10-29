---
title: Introduction to Zone-Based Firewall
author: troglobit
date: 2025-10-29 08:10:00 +0100
categories: [howto]
tags: [firewall, networking, security, zbf]
---

As of Infix v25.10, a zone-based firewall (ZBF) built on [firewalld][2]
is included.  Exposing the most relevant functionality for your network
security.  Rather than managing rules on a per-interface basis, zones
group interfaces by trust level and policies control traffic flow
between zones.

![](/assets/img/fw-concept.svg){: #fig1 width="600" }
_**Figure 1**: Zone-based firewall concept._

### The Zone Concept

A zone defines a level of trust for network connections.  All interfaces or
networks assigned to a zone inherit that trust level.  For example, you might
have a trusted LAN zone for your internal network and an untrusted WAN zone
for the Internet connection.

Each zone has an action that determines what happens to traffic from that
zone destined for the local host:

- `accept`: Allow all traffic to the host
- `reject`: Deny traffic, send rejection response
- `drop`: Silently discard traffic

When the action is set to `reject` or `drop`, you can explicitly allow
specific services like SSH or DHCP.

### A Simple Example

Let's set up a basic home router with two zones: a trusted LAN and an
untrusted WAN.  We'll start by creating the zones and assigning interfaces
to them.

```console
admin@router:/> configure
admin@router:/config/> edit firewall
admin@router:/config/firewall/> set zone lan action accept
admin@router:/config/firewall/> set zone lan interface eth0
admin@router:/config/firewall/> set zone wan action drop
admin@router:/config/firewall/> set zone wan interface eth1
```

At this point, the LAN zone trusts all traffic to the host, while the WAN
zone drops everything by default.  However, we need to allow certain services
from the WAN side, like DHCPv6 for address assignment:

```console
admin@router:/config/firewall/> set zone wan service dhcpv6-client
```

Now we need a policy to allow LAN devices to access the Internet through
the WAN interface.  Policies control traffic flow between zones:

```console
admin@router:/config/firewall/> set policy lan-wan ingress lan
admin@router:/config/firewall/> set policy lan-wan egress wan
admin@router:/config/firewall/> set policy lan-wan action accept
admin@router:/config/firewall/> set policy lan-wan masquerade true
admin@router:/config/firewall/> leave
```

The `masquerade` option enables source NAT, replacing the source IP address
of LAN clients with the router's WAN address.

Notice that we didn't create a policy for WAN to LAN traffic.  By default,
all inter-zone traffic is blocked unless explicitly allowed by a policy.
Return traffic from established connections is automatically permitted
through connection tracking.

### Traffic Flow Types

The firewall handles three types of traffic:

**Host-destined traffic**: Traffic to the router itself, like SSH or web
management.  This is controlled by the zone's action and service list.

**Intra-zone traffic**: Traffic between interfaces in the same zone, such
as LAN devices talking to each other.  This is not forwarded by default and
requires a policy where both ingress and egress are set to the same zone.

**Inter-zone traffic**: Traffic between different zones, like LAN to WAN.
This requires an explicit policy and is blocked by default.

![](/assets/img/fw-zones.svg){: #fig2 width="600" }
_**Figure 2**: Traffic flow between zones._

### The Default Zone

Infix requires a default zone as a safety mechanism.  Any interface not
explicitly assigned to a zone automatically joins the default zone.  This
prevents accidentally leaving an interface unprotected.

To set a zone as the default:

```console
admin@router:/config/firewall/> set zone wan default true
```

### Beyond the Basics

The firewall supports additional features for more complex scenarios:

**Port forwarding**: DNAT rules to expose services in a DMZ to the Internet.
Traffic can be forwarded to a different IP address and port than the original
destination.

**Custom filters**: Additional rules can be inserted at specific points in
the netfilter pipeline for advanced filtering needs.

**Network-based zones**: Instead of assigning interfaces, zones can match
specific IP networks for forwarding traffic.

For detailed configuration examples, including DMZ setups and port forwarding,
see the [firewall documentation][1].

[1]: https://kernelkit.org/infix/latest/firewall/
[2]: https://firewalld.org
