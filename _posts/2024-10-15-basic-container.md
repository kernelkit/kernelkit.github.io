---
title: Gentle Container Introduction
author: troglobit
date: 2024-10-15 07:00:00 +0100
last_modified_at: 2026-02-27 12:00:00 +0100
categories: [showcase]
tags: [container, containers, networking, docker, podman]
image:
  path: /assets/img/docker.webp
  alt: Docker whale
  show_in_post: false
---

![Docker whale](/assets/img/docker.webp){: width="200" .right}

This is the fourth post in a series about containers in Infix.  This
time we go back to basics for a more gentle introduction into using
containers.

We will use one real interface, connected to the outside world, and one
VETH pair.  One end of the pair in the host and the other given to the
container.  The host end of the pair can be bridged or routed, this is
left as an exercise to the reader.

See the [first post][1] for a background and networking basics.

> This post assumes knowledge and familiarity with the [Infix
> Operating System](https://kernelkit.org/).  Ensure you have
> either a network connection or console access to your Infix system and
> can log in to it using SSH.  Recommended reading includes the
> [networking documentation][0].
{: .prompt-info }

---

## Introduction

Let's set up the [basic building blocks][0] used with most containers,
which is usually hidden from users.

![](/assets/img/basic-docker-veth.svg)

 1. You need the [*Latest Build*][7] of Infix.  Either on an actual device, or
    a Linux PC with the `x86_64` image for [testing with Qemu][6]
 2. In Qemu you need to activate separate `/var`, at least 256 MiB: `./qemu.sh -c`
 3. Start Infix: `./qemu.sh`

## Configuration

We start with the networking: a single port as WAN, connected to the
Internet, and a VETH pair where one end will be handed over to the
container.

Notice the *DHCP client* on interface `e1`, it is required since we need
Internet access to download the container image below.

```console
admin@infix:/> configure
admin@infix:/config/> edit interface e1
admin@infix:/config/interface/e1/> set ipv4 dhcp
admin@infix:/config/interface/e1/> end
admin@infix:/config/> edit interface veth0a
admin@infix:/config/interface/veth0a/> set veth peer veth0b
admin@infix:/config/interface/veth0a/> set ipv4 address 192.168.0.1 prefix-length 30
admin@infix:/config/interface/veth0a/> end
admin@infix:/config/> edit interface veth0b
admin@infix:/config/interface/veth0b/> set ipv4 address 192.168.0.2 prefix-length 30
admin@infix:/config/interface/veth0b/> set container-network
admin@infix:/config/interface/veth0b/> set container-network route 0.0.0.0/0 gateway 192.168.0.1
admin@infix:/config/interface/veth0b/> leave
```

Time for the container configuration, as usual we employ [curiOS][2]
containers.

```console
admin@infix:/> configure
admin@infix:/config/> edit container system
admin@infix:/config/container/system/> set image docker://ghcr.io/kernelkit/curios:latest
admin@infix:/config/container/system/> set hostname sys101
admin@infix:/config/container/system/> set privileged true
admin@infix:/config/container/system/> set network interface veth0b
admin@infix:/config/container/system/> set network dns 192.168.0.1
admin@infix:/config/container/system/> set volume etc target /etc
admin@infix:/config/container/system/> set volume var target /var
admin@infix:/config/container/system/> leave
```

> We don't have to `leave` after each of the above sections, we could
> just as easily kept going all through the new configuration.
{: .prompt-info }

## Firewall

To route traffic between the container and the WAN we first enable IP
forwarding on both ends of the VETH pair:

```console
admin@infix:/> configure
admin@infix:/config/> edit interface veth0a
admin@infix:/config/interface/veth0a/> set ipv4 forwarding true
admin@infix:/config/interface/veth0a/> end
admin@infix:/config/> edit interface veth0b
admin@infix:/config/interface/veth0b/> set ipv4 forwarding true
admin@infix:/config/interface/veth0b/> leave
```

Next, a [zone-based firewall][8] to protect the WAN port and let the
container reach the Internet via masquerade (NAT).  The `containers`
zone covers the VETH subnet and the policy routes traffic from there out
through the `public` zone on `e1`:

```console
admin@infix:/> configure
admin@infix:/config/> edit firewall
admin@infix:/config/firewall/> set default public
admin@infix:/config/firewall/> set zone public action reject
admin@infix:/config/firewall/> set zone public interface e1
admin@infix:/config/firewall/> set zone public service ssh
admin@infix:/config/firewall/> set zone containers action accept
admin@infix:/config/firewall/> set zone containers network 192.168.0.0/30
admin@infix:/config/firewall/> set policy container-access ingress containers
admin@infix:/config/firewall/> set policy container-access egress public
admin@infix:/config/firewall/> set policy container-access action accept
admin@infix:/config/firewall/> set policy container-access masquerade true
admin@infix:/config/firewall/> leave
```

If the container runs a service you want reachable from the WAN, add a
port-forward rule to the public zone.  Here we forward TCP port 8080 on
the WAN to port 80 in the container:

```console
admin@infix:/> configure
admin@infix:/config/> edit firewall
admin@infix:/config/firewall/> set zone public port-forward 8080 proto tcp to addr 192.168.0.2 port 80
admin@infix:/config/firewall/> leave
```

> Port forwarding only works when the destination zone is defined using
> `network`, not `interface`.  If the `containers` zone were set with
> `set zone containers interface veth0a` instead, forwarded packets
> would be dropped silently.
{: .prompt-warning }

## The Result

We should now have a running container.

```console
admin@infix:/> show container 
CONTAINER ID  IMAGE                            COMMAND     CREATED       STATUS        PORTS       NAMES
1cd99db1f518  ghcr.io/kernelkit/curios:latest              16 hours ago  Up 6 seconds              system
```

We can enter the container using:

```console
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
          inet addr:192.168.0.2  Bcast:192.168.0.3  Mask:255.255.255.252
          inet6 addr: fe80::d0a3:70ff:fe0d:5000/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:63 errors:0 dropped:9 overruns:0 frame:0
          TX packets:19 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:12867 (12.5 KiB)  TX bytes:3064 (2.9 KiB)

root@sys101:~$ exit
admin@infix:/>
```

## Fin

That concludes the fourth post about containers in Infix.  As usual,
remember to

```console
admin@infix:/> copy running-config startup-config
```

Take care! 🧡

---

[0]: https://kernelkit.org/infix/latest/networking/
[1]: /posts/containers/
[2]: https://github.com/kernelkit/curiOS/
[3]: https://en.wikipedia.org/wiki/Network_address_translation
[4]: https://kernelkit.org/infix/latest/cli/text-editor/
[5]: https://wiki.nftables.org/wiki-nftables/index.php/Main_Page
[6]: /posts/getting-started/
[7]: https://github.com/kernelkit/infix/releases/tag/latest
[8]: /posts/zone-based-firewall/
