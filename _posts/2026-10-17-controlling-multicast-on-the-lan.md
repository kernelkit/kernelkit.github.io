---
title: Controlling Multicast on the LAN
description: How to manage IPv4 and IPv6 multicast on your LAN with Infix OS
author: troglobit
date: 2026-10-17 09:00:00 +0200
last_modified_at: 2026-10-26 12:00:00 +0100
categories: [howto]
tags: [multicast, igmp, mld, bridge, mcd]
image:
  path: /assets/img/mcd-social.png
  alt: mcd, IGMP and MLD querier for the Linux bridge
  show_in_post: false
---

Multicast lets one sender reach many receivers with a single stream: a
camera feeding several operator stations, a controller publishing data
to a group of devices, or a service announcing itself on the LAN.  The
sender addresses a group, and the network delivers a copy to every
receiver that has joined it.

This post covers how a switch decides where that copy goes, what IGMP
and MLD have to do with it, and how to set it up in Infix OS.  It ends
with what is new in Infix OS v26.10: a querier for IPv6, from [mcd][]
3.1.

### Multicast on a Switch

A multicast frame has a group as its destination, not a port.  A switch
that knows nothing more forwards it the way it forwards broadcast, out
of every port.  That is fine for the occasional announcement, but a
video stream then reaches every device on the LAN, including the ones
on slow links that never asked for it.

### Snooping

Hosts announce the groups they want with membership reports, [IGMP][igmp]
for IPv4 and [MLD][mld] for IPv6.  A snooping switch listens to these
reports, records which ports joined which groups in its multicast
forwarding database, the MDB, and forwards each group only to its
members.  [RFC 4541][rfc4541] describes how switches do this.

Ports with a multicast router behind them get all known groups, so the
router can forward them further.  Groups that nobody has joined are
flooded on the ports set to flood unknown multicast, which in Infix OS
is every port by default.  That keeps MAC multicast, and devices that
do not speak IGMP or MLD, working.

### The Querier

Memberships do not last forever.  Each one expires unless the host
reports again, and many hosts only report when asked.  The asking is
done by the querier, which sends a general query every query interval,
125 seconds by default, and the hosts answer with their memberships.

The querier role usually belongs to a multicast router.  A LAN without
one needs a switch to take it on, and this matters more than it may
seem: the Linux bridge, and so Infix OS, only filters multicast while
there is a querier on the LAN.  With none, it floods all multicast to
every port.  When the querier goes away, the bridge waits out its other
querier present interval, 255 seconds by default, and then starts
flooding.

![Two panels of the same bridge, without a querier the camera stream floods every port, with mcd it reaches only the port that joined](/assets/img/mcd-why.svg){: #fig1 width="700" }
_**Figure 1**: Bridge without a querier, flooding a camera
stream to every port, and with mcd, where the stream reaches only the
port that joined._

If more than one device can be the querier, they hold an election, and
the lowest address wins.  A device without a usable address can still
send proxy queries, from `0.0.0.0` for IGMP, which never win.  Proxy
queries keep the memberships fresh, but some end devices ignore them.
Their memberships then expire, and a group whose only member did that
is unknown again, and flooded.

### In Infix OS

Enabling multicast on a bridge turns on snooping, and makes the bridge
a querier candidate that uses its own address in the election:

<pre class="cli"><code>admin@example:/> <b>configure</b>
admin@example:/config/> <b>edit interface br0</b>
admin@example:/config/interface/br0/> <b>set bridge multicast snooping</b>
admin@example:/config/interface/br0/> <b>set ipv4 address 192.168.2.1 prefix-length 24</b>
admin@example:/config/interface/br0/> <b>leave</b>
admin@example:/> <b>show ip multicast</b>
Multicast Overview
Query Interval (default): 125 sec
Router Timeout          : 255
Fast Leave Ports        :
Router Ports            :
Flood Ports             : e0, e1, e2, e3, e4, e5, e6, e7

<span class="header">Interface       VID  Proto  Querier                      State  Interval  Timeout  MRD  Ver</span>
br0                  IGMP   192.168.2.1                     Up       125     None  off    3
br0                  MLD    fe80::ff:fe00:1                 Up       125     None  off    2

<span class="header">Bridge          VID  Multicast Group       Ports                                      </span>
br0                  224.1.1.1             e3, e2
br0                  ff05::1234            e5
</code></pre>

The overview shows the bridge settings and the role of each port.  The
querier table has one row per protocol: the elected querier, its query
interval, and a timeout of `None` when the bridge itself is the
querier.  The last table is the MDB, with the groups that have been
joined and on which ports.

The defaults suit most networks.  The settings worth knowing are:

 - `querier`, under `bridge multicast`, is `auto` by default.  Set it
   to `proxy` to only send proxy queries when no other querier is
   present, or `off` to never query
 - `query-interval`, under `bridge multicast`, sets the time between
   general queries, 125 seconds by default
 - `bridge-port multicast router`, per port, is `auto` by default and
   detects multicast routers.  Set it to `permanent` for a router that
   does not announce itself
 - `bridge-port multicast fast-leave`, per port, drops a group as soon
   as the port reports a leave.  Use it on ports with a single end
   device
 - `bridge-port flood multicast`, per port, controls flooding of
   unknown groups, on by default
 - `bridge multicast-filters` holds static groups, for devices that do
   not report their memberships

On a VLAN filtering bridge, the same settings apply per VLAN, under
`bridge vlans vlan <VID> multicast`.  If the VLAN has an interface with
an address on top, the querier uses that address.  Otherwise it sends
proxy queries.

The [Multicast Filtering and Snooping][bridging] section of the user
guide has more examples, including VLAN filtering bridges and static
filters.  [Basic Bridge Networking][basic] covers setting up the bridge
itself.

### New in Infix OS v26.10: An MLD Querier

Up to now, the querier in Infix OS has been IGMP only.  Infix OS v26.10
comes with [mcd][] 3.1, which runs an MLD querier next to the IGMP one,
each with its own election.  IGMP queries are sent from the interface
address, and MLD queries from its link-local address, as
[RFC 2710][rfc2710] and [RFC 3810][rfc3810] require.  Both start at the
newest protocol version, IGMPv3 from [RFC 3376][rfc3376] and MLDv2, and
fall back while an older host is heard.  The configuration is unchanged,
a bridge that runs a querier now runs one for each protocol.

A host on port `e0` sees both queries, each from its own address:

```bash
$ tshark -n -i eth0 -Y 'igmp.type == 0x11 || icmpv6.type == 130'
  1 0.000000000 0.000000000  192.168.2.1 → 224.0.0.1    IGMPv3 50 Membership Query, general
  2 0.014043869 0.014043869 fe80::ff:fe00:1 → ff02::1   ICMPv6 90 Multicast Listener Query
```

mcd 3.1 came out a few days after this post was first published, and
Infix OS v26.10 ships it instead of 3.0.  It sends a general query as
soon as a bridge port starts forwarding, after link up, STP settling,
or a port being added, so hosts behind it are back in the MDB within
seconds instead of at the next periodic query.  Its timers now follow
the `query-interval` of each bridge, so a shorter interval also means
a faster takeover from a querier that has gone silent.  mcd can also
announce itself to other snooping switches with Multicast Router
Discovery, [RFC 4286][rfc4286].  Infix OS does not use that yet, so the
new MRD column in `show ip multicast` reads `off`.  The release also
has many robustness fixes, found by fuzzing and static analysis in a
Nix flake contributed by Dave Seddon.

mcd began in 2021 as [querierd][], written at [Westermo][] for WeOS 5.
Westermo's last release of querierd was v0.10, in May 2023.  In 2024
its original author adopted it as mcd, and today it is the multicast
querier in [Infix OS][infix].  mcd runs on any Linux system with a
bridge, and its [ChangeLog][changelog] lists everything new in 3.0 and
3.1.

[igmp]: https://www.rfc-editor.org/rfc/rfc3376.html
[mld]: https://www.rfc-editor.org/rfc/rfc3810.html
[rfc4541]: https://www.rfc-editor.org/rfc/rfc4541.html
[rfc2710]: https://www.rfc-editor.org/rfc/rfc2710.html
[rfc3810]: https://www.rfc-editor.org/rfc/rfc3810.html
[rfc3376]: https://www.rfc-editor.org/rfc/rfc3376.html
[rfc4286]: https://www.rfc-editor.org/rfc/rfc4286.html
[mcd]: https://github.com/kernelkit/mcd
[querierd]: https://github.com/westermo/querierd
[Westermo]: https://www.westermo.com/
[infix]: https://github.com/kernelkit/infix
[changelog]: https://github.com/kernelkit/mcd/blob/main/ChangeLog.md
[bridging]: https://kernelkit.org/infix/latest/bridging/#multicast-filtering-and-snooping
[basic]: /posts/bridge-networking/
