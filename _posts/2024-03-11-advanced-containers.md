---
title: Advanced Container Networking
author: troglobit
date: 2024-03-11 14:11:50 +0100
categories: [showcase]
tags: [container, containers, docker, podman]
---

![Docker whale](/assets/img/docker.webp){: width="200" .right}

This is the second post about containers in Infix, the focus this time
is on more advanced network setups.  In particular, containers with more
than one interface, and an Infix network configuration that includes
both VETH pairs and bridges with LLDP forwarding.

See the [first post][1] for a background and networking basics.

> This post assumes knowledge and familiarity with the [Infix Network
> Operating System](https://kernelkit.github.io/).  Ensure you have
> either a network connection or console access to your Infix system and
> can log in to it using SSH.  Recommended reading includes the
> [networking documentation][0].
{: .prompt-info }


----

## Introduction

The setup below aims to show how a container can have multiple network
connections.  Useful for applications where critical process data should
be isolated to prevent external access.

![](/assets/img/container-two-bridges.svg)
_**Figure 1:** Container with access to two physically separated networks._

We use two bridges for this example, we could easily have used a single
VLAN filtering bridge instead, but as a twist we want the left-most
bridge to forward LLDP[^1] traffic, and in that case it is better to use
two bridges instead.

> Depending on the underlying (switch) hardware, there are limitations
> to consider.  For example, a system with Marvel SOHO switch fabric,
> using the Linux kernel DSA driver, is limited to one regular bridge
> and multiple VLAN filtering bridges.  Our example with two non-VLAN
> filtering bridges means that they share the same MAC database.  In
> most use-case this is not a problem, but it is worth mentioning for
> networks where the same MAC address may appear in both bridges.
{: .prompt-info }

## System Container

For this case study to make sense we need a more competent container
image than the simple web server used in the first blog post.  To that
end we introduce [curiOS][2], a small container operating system built
on the same stable [Buildroot][3] foundation as Infix.  Specifically,
we'll be using the *system container* build, available as `docker pull`
from [GHCR][4] and as an [OCI tarball][5].

> Infix supports bundling OCI tarballs in the Infix image itself.  This
> can significantly simplify initial deployment of appliances running
> Infix with customer containers.
{: .prompt-tip }


## Configuration

Let's start by setting up the first bridge, `br0`:

```console
admin@infix:/> configure
admin@infix:/config/> edit interface br0
admin@infix:/config/interface/br0/> set bridge ieee-group-forward lldp
admin@infix:/config/interface/br0/> end
admin@infix:/config/> set interface eth1 bridge-port bridge br0
admin@infix:/config/> set interface eth2 bridge-port bridge br0
admin@infix:/config/> set interface eth3 bridge-port bridge br0
admin@infix:/config/> leave
```

The second bridge is similar:

```console
admin@infix:/> configure
admin@infix:/config/> edit interface br1
admin@infix:/config/interface/br1/> end
admin@infix:/config/> set interface eth4 bridge-port bridge br1
admin@infix:/config/> set interface eth5 bridge-port bridge br1
admin@infix:/config/> set interface eth6 bridge-port bridge br1
admin@infix:/config/> leave
```

Now, for the container we need to add two VETH pairs and attach one end
of each to each of the bridges.  We use the recommended nomenclature of
Infix to skip configuration steps and save time:

```console
admin@infix:/> configure
admin@infix:/config/> edit interface veth0a
admin@infix:/config/interface/veth0a/> set veth peer veth0b
admin@infix:/config/interface/veth0a/> end
admin@infix:/config/> edit interface veth1a
admin@infix:/config/interface/veth1a/> set veth peer veth1b
admin@infix:/config/interface/veth1a/> end
admin@infix:/config/> set interface veth0b bridge-port bridge br0
admin@infix:/config/> set interface veth1b bridge-port bridge br1
admin@infix:/config/> leave
```

Initial network setup is now complete (no IP addresses yet) and end
devices connected to ports in both bridges can now communicate with
each other, but not between bridges.

Time for IP networking!

> You could hold off on this until after you've created the container,
> but in that case the addresses for `veth0a` and `veth1a` will not be
> set until you manually restart the container.  In this documented
> order things will work from the first try.
{: .prompt-info }

```console
admin@infix:/> configure
admin@infix:/config/> edit interface br0
admin@infix:/config/interface/br0/> set ipv4 address 10.0.1.1 prefix-length 24
admin@infix:/config/interface/br0/> end
admin@infix:/config/> edit interface br1
admin@infix:/config/interface/br1/> set ipv4 address 192.168.1.1 prefix-length 24
admin@infix:/config/interface/br1/> end
admin@infix:/config/> edit interface veth0a 
admin@infix:/config/interface/veth0a/> set ipv4 address 10.0.1.2 prefix-length 24
admin@infix:/config/interface/veth0a/> end
admin@infix:/config/> edit interface veth1a 
admin@infix:/config/interface/veth1a/> set ipv4 address 192.168.1.2 prefix-length 24
admin@infix:/config/interface/veth1a/> leave
```

You can verify the setup with the following CLI command.

```console
admin@infix:/> show interfaces 
INTERFACE       PROTOCOL   STATE       DATA
lo              ethernet   UP          00:00:00:00:00:00
                ipv4                   127.0.0.1/8 (static)
                ipv6                   ::1/128 (static)
br0             bridge
│               ipv4                   10.0.1.1/24 (static)
├ eth1          bridge     FORWARDING
├ eth2          bridge     FORWARDING
├ eth3          bridge     FORWARDING
└ veth0b        bridge     FORWARDING
br1             bridge
│               ipv4                   192.168.1.1/24 (static)
├ eth4          bridge     FORWARDING
├ eth5          bridge     FORWARDING
├ eth6          bridge     FORWARDING
└ veth1b        bridge     FORWARDING
eth0            ethernet   UP          02:00:00:00:00:00
                ipv4                   169.254.1.3/16 (random)
                ipv6                   fe80::ff:fe00:0/64 (link-layer)
eth7            ethernet   UP          02:00:00:00:00:07
                ipv6                   fe80::ff:fe00:7/64 (link-layer)
eth8            ethernet   UP          02:00:00:00:00:08
                ipv6                   fe80::ff:fe00:8/64 (link-layer)
eth9            ethernet   UP          02:00:00:00:00:09
                ipv6                   fe80::ff:fe00:9/64 (link-layer)
veth0a          ethernet
                ipv4                   10.0.1.2/24 (static)
veth0b          ethernet   UP          42:c0:a2:f1:41:fa
veth1a          ethernet
                ipv4                   192.168.1.2/24 (static)
veth1b          ethernet   UP          da:d5:e8:0d:0b:55
```

Finally, the container configuration.  We start by marking the access
port ends of the VETH pairs as container networks, then we can refer
to them from the container.

```console
admin@infix:/> configure
admin@infix:/config/> set interface veth0a container-network
admin@infix:/config/> set interface veth1a container-network
admin@infix:/config> edit container system
admin@infix:/config/container/system/> set image docker://ghcr.io/kernelkit/curios:24.02.0
admin@infix:/config/container/system/> set hostname sys101
admin@infix:/config/container/system/> edit network
admin@infix:/config/container/system/network/> set interface veth0a
admin@infix:/config/container/system/network/> set interface veth1a
admin@infix:/config/container/system/network/> leave
```

> We don't have to `leave` after each of the above sections, we could
> just as easily kept going all through the new configuration.
{: .prompt-info }


## The Result

We should now have a running container that you can connect to and
inspect the results:

```console
admin@infix:/> show container
CONTAINER ID  IMAGE                             COMMAND     CREATED         STATUS         PORTS       NAMES
ccb2338b9f33  ghcr.io/kernelkit/curios:24.02.0              11 seconds ago  Up 10 seconds              system
admin@infix:/> container shell system
root@sys101:/# ifconfig
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

eth0      Link encap:Ethernet  HWaddr D2:A3:70:0D:50:00
          inet addr:10.0.1.2  Bcast:10.0.1.255  Mask:255.255.255.0
          inet6 addr: fe80::d0a3:70ff:fe0d:5000/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:63 errors:0 dropped:9 overruns:0 frame:0
          TX packets:19 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:12867 (12.5 KiB)  TX bytes:3064 (2.9 KiB)

eth1      Link encap:Ethernet  HWaddr E6:E8:40:2F:19:F0
          inet addr:192.168.1.2  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::e4e8:40ff:fe2f:19f0/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:41 errors:0 dropped:3 overruns:0 frame:0
          TX packets:19 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:5845 (5.7 KiB)  TX bytes:3064 (2.9 KiB)

root@sys101:~$ exit
admin@infix:/> 
```

The two interfaces, `eth0` and `eth1`, correspond to the access end of
the VETH pairs, `veth0a` and `veth1a`, respectively.  The order in which
they are added to the container configuration determines their names
inside the container.  This can of course be controlled from the
[network specific options][6] in the container configurations.


## Fin

That's the end of the second post about containers in Infix.  As usual,
remember to

```console
admin@infix:/> copy running-config startup-config
```

Take care! 🧡

----

[^1]: A standards compliant bridge by default does not forward IEEE
	reserved link-local multicast groups, in the 01:80:C2:00:00:0X
	range.  The Infix bridge supports forwarding select protocols.

[0]: https://github.com/kernelkit/infix/blob/main/doc/networking.md
[1]: /posts/containers/
[2]: https://github.com/kernelkit/curiOS/
[3]: https://buildroot.org/
[4]: https://github.com/kernelkit/curiOS/pkgs/container/curios/185490653?tag=24.02.0
[5]: https://github.com/kernelkit/curiOS/releases/tag/v24.02.0
[6]: https://github.com/kernelkit/infix/blob/main/doc/container.md#networking-and-containers
