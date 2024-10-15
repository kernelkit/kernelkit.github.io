---
title: Basic Container Setup
author: troglobit
date: 2024-10-15 07:00:00 +0100
categories: [showcase]
tags: [container, containers, networking, docker, podman]
---

![Docker whale](/assets/img/docker.webp){: width="200" .right}

This is the fourth post in a series about containers in Infix.  This
time we go back to basics for a more gentle introduction into using
containers.

See the [first post][1] for a background and networking basics.

> This post assumes knowledge and familiarity with the [Infix Network
> Operating System](https://kernelkit.github.io/).  Ensure you have
> either a network connection or console access to your Infix system and
> can log in to it using SSH.  Recommended reading includes the
> [networking documentation][0].
{: .prompt-info }

----


## Introduction

Let's set up the basic building blocks used with most containers, which
is usually hidden from users.

![](/assets/img/basic-docker-veth.svg)

 1. You need the [*Latest
    Build*](https://github.com/kernelkit/infix/releases/tag/latest) of
    Infix, use the `x86_64` for [testing with
    Qemu](https://github.com/kernelkit/infix/blob/main/doc/virtual.md)
 2. In Qemu you need to activate separate `/var`, at least 256 MiB: `./qemu.sh -c`
 3. Start Infix: `./qemu.sh`


## Configuration

The Infix configuration consists of two parts: networking setup and the
container.  We start with the networking, we want a single port as our
WAN port, connected to the Internet, and a VETH pair where one end will
be handed over to the container.

```console
admin@infix:/> configure
admin@infix:/config/> set dhcp-client client-if e1
admin@infix:/config/> edit interface veth0a
admin@infix:/config/interface/veth0a/> set veth peer veth0b
admin@infix:/config/interface/veth0a/> set ipv4 address 192.168.0.1 prefix-length 24
admin@infix:/config/interface/veth0a/> end
admin@infix:/config/> edit interface veth0a
admin@infix:/config/interface/veth0b/> set ipv4 address 192.168.0.2 prefix-length 24
admin@infix:/config/interface/veth0b/> set container-network
admin@infix:/config/interface/veth0b/> leave
```

Time for the container configuration, as usual we employ [curiOS][2]
containers.

```console
admin@infix:/> configure
admin@infix:/config> edit container system
admin@infix:/config/container/system/> set image docker://ghcr.io/kernelkit/curios:edge
admin@infix:/config/container/system/> set hostname sys101
admin@infix:/config/container/system/> set network interface veth0b
admin@infix:/config/container/system/> leave
```

> We don't have to `leave` after each of the above sections, we could
> just as easily kept going all through the new configuration.
{: .prompt-info }


## The Result

We should now have a running container.

```console
admin@infix:/> show container 
CONTAINER ID  IMAGE                          COMMAND     CREATED       STATUS        PORTS       NAMES
1cd99db1f518  ghcr.io/kernelkit/curios:edge              16 hours ago  Up 6 seconds              system
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
          inet addr:192.168.0.2  Bcast:192.168.0.255  Mask:255.255.255.0
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

Take care! <3

----

[0]: https://github.com/kernelkit/infix/blob/main/doc/networking.md
[1]: /posts/containers/
[2]: https://github.com/kernelkit/curiOS/
[3]: https://en.wikipedia.org/wiki/Network_address_translation
[4]: https://github.com/kernelkit/infix/blob/main/doc/cli/text-editor.md
[5]: https://wiki.nftables.org/wiki-nftables/index.php/Main_Page
