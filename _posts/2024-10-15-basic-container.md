---
title: Gentle Container Introduction
author: troglobit
date: 2024-10-15 07:00:00 +0100
last_modified_at: 2026-09-14 12:00:00 +0100
categories: [howto]
tags: [containers, networking, docker, podman]
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
> Operating System](https://www.kernelkit.org/).  Ensure you have
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

<pre class="cli"><code>admin@infix:/&gt; <b>configure</b>
admin@infix:/config/&gt; <b>edit interface e1</b>
admin@infix:/config/interface/e1/&gt; <b>set ipv4 dhcp</b>
admin@infix:/config/interface/e1/&gt; <b>end</b>
admin@infix:/config/&gt; <b>edit interface veth0a</b>
admin@infix:/config/interface/veth0a/&gt; <b>set veth peer veth0b</b>
admin@infix:/config/interface/veth0a/&gt; <b>set ipv4 address 192.168.0.1 prefix-length 30</b>
admin@infix:/config/interface/veth0a/&gt; <b>end</b>
admin@infix:/config/&gt; <b>edit interface veth0b</b>
admin@infix:/config/interface/veth0b/&gt; <b>set ipv4 address 192.168.0.2 prefix-length 30</b>
admin@infix:/config/interface/veth0b/&gt; <b>set container-network</b>
admin@infix:/config/interface/veth0b/&gt; <b>set container-network route 0.0.0.0/0 gateway 192.168.0.1</b>
admin@infix:/config/interface/veth0b/&gt; <b>leave</b></code></pre>

Time for the container configuration, as usual we employ [curiOS][2]
containers.

<pre class="cli"><code>admin@infix:/&gt; <b>configure</b>
admin@infix:/config/&gt; <b>edit container system</b>
admin@infix:/config/container/system/&gt; <b>set image docker://ghcr.io/kernelkit/curios:latest</b>
admin@infix:/config/container/system/&gt; <b>set hostname sys101</b>
admin@infix:/config/container/system/&gt; <b>set privileged true</b>
admin@infix:/config/container/system/&gt; <b>set network interface veth0b</b>
admin@infix:/config/container/system/&gt; <b>set network dns 192.168.0.1</b>
admin@infix:/config/container/system/&gt; <b>set volume etc target /etc</b>
admin@infix:/config/container/system/&gt; <b>set volume var target /var</b>
admin@infix:/config/container/system/&gt; <b>leave</b></code></pre>

> We don't have to `leave` after each of the above sections, we could
> just as easily kept going all through the new configuration.
{: .prompt-info }

## Firewall

To route traffic between the container and the WAN we first enable IP
forwarding on both ends of the VETH pair:

<pre class="cli"><code>admin@infix:/&gt; <b>configure</b>
admin@infix:/config/&gt; <b>edit interface veth0a</b>
admin@infix:/config/interface/veth0a/&gt; <b>set ipv4 forwarding</b>
admin@infix:/config/interface/veth0a/&gt; <b>end</b>
admin@infix:/config/&gt; <b>edit interface veth0b</b>
admin@infix:/config/interface/veth0b/&gt; <b>set ipv4 forwarding</b>
admin@infix:/config/interface/veth0b/&gt; <b>leave</b></code></pre>

Next, a [zone-based firewall][8] to protect the WAN port and let the
container reach the Internet via masquerade (NAT).  The `containers`
zone covers the VETH subnet and the policy routes traffic from there out
through the `public` zone on `e1`:

<pre class="cli"><code>admin@infix:/&gt; <b>configure</b>
admin@infix:/config/&gt; <b>edit firewall</b>
admin@infix:/config/firewall/&gt; <b>set default public</b>
admin@infix:/config/firewall/&gt; <b>set zone public action reject</b>
admin@infix:/config/firewall/&gt; <b>set zone public interface e1</b>
admin@infix:/config/firewall/&gt; <b>set zone public service ssh</b>
admin@infix:/config/firewall/&gt; <b>set zone containers action accept</b>
admin@infix:/config/firewall/&gt; <b>set zone containers network 192.168.0.0/30</b>
admin@infix:/config/firewall/&gt; <b>set policy container-access ingress containers</b>
admin@infix:/config/firewall/&gt; <b>set policy container-access egress public</b>
admin@infix:/config/firewall/&gt; <b>set policy container-access action accept</b>
admin@infix:/config/firewall/&gt; <b>set policy container-access masquerade true</b>
admin@infix:/config/firewall/&gt; <b>leave</b></code></pre>

If the container runs a service you want reachable from the WAN, add a
port-forward rule to the public zone.  Here we forward TCP port 8080 on
the WAN to port 80 in the container:

<pre class="cli"><code>admin@infix:/&gt; <b>configure</b>
admin@infix:/config/&gt; <b>edit firewall</b>
admin@infix:/config/firewall/&gt; <b>set zone public port-forward 8080 proto tcp to addr 192.168.0.2 port 80</b>
admin@infix:/config/firewall/&gt; <b>leave</b></code></pre>

> Port forwarding only works when the destination zone is defined using
> `network`, not `interface`.  If the `containers` zone were set with
> `set zone containers interface veth0a` instead, forwarded packets
> would be dropped silently.
{: .prompt-warning }

## The Result

We should now have a running container.

<pre class="cli"><code>admin@infix:/&gt; <b>show container</b>
<span class="header">NAME    STATUS        NETWORK  MEMORY (KiB)  CPU%</span>
system  Up 6 seconds  veth0b           3960  0.00</code></pre>

We can enter the container using:

<pre class="cli"><code>admin@infix:/&gt; <b>container shell system</b>
root@sys101:/# <b>ifconfig</b>
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

root@sys101:~$ <b>exit</b>
admin@infix:/&gt;</code></pre>

## Fin

That concludes the fourth post about containers in Infix.  As usual,
remember to

<pre class="cli"><code>admin@infix:/&gt; <b>copy running-config startup-config</b></code></pre>

Take care! 🧡

---

[0]: https://www.kernelkit.org/infix/latest/networking/
[1]: /posts/containers/
[2]: https://github.com/kernelkit/curiOS/
[3]: https://en.wikipedia.org/wiki/Network_address_translation
[4]: https://www.kernelkit.org/infix/latest/cli/edit/
[5]: https://wiki.nftables.org/wiki-nftables/index.php/Main_Page
[6]: /posts/getting-started/
[7]: https://github.com/kernelkit/infix/releases/tag/latest
[8]: /posts/zone-based-firewall/
