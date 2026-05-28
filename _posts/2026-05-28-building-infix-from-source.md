---
title: Building Infix from Source
description: From git clone to pinging the internet with make run
author: troglobit
date: 2026-05-28 06:00:00 +0100
categories: [howto]
tags: [beginner, buildroot, qemu]
---

There are two ways to take Infix for a spin on your own PC.  You can
grab a pre-built release and boot it straight away; that is what the
[Getting Started][gs] and [Infix in GNS3][gns3] posts cover.  Or you
can build it yourself from source and boot the result with a single
`make run`.  This post is the second path.

Building from source is the way to go when you want to change an Open
Source component, follow the bleeding edge on `main`, hack on Infix
itself, or simply understand how the image is put together.  It takes a
bit longer to get going the first time, but everything after that is a
quick edit-build-run loop.

You will need an **x86_64 Linux PC** with around 30 GiB of free disk
space.  We recommend [Debian][] based systems like [Ubuntu][] and [Linux
Mint][].

### Install the Build Tools

Buildroot, which Infix is built on, needs a handful of host tools to
bootstrap itself.  Most are already present on a typical desktop install;
the rest can be pulled in like this on Debian/Ubuntu based systems:

```bash
$ sudo apt install bc binutils build-essential bzip2 cpio  \
                   diffutils file findutils git gzip       \
                   libncurses-dev libssl-dev perl patch    \
                   python3 rsync sed tar unzip wget mtools \
                   autopoint bison flex autoconf automake
```

For the current list of requirements, see the [Developer's Guide][dev].

### Get the Source

Clone the Infix tree and pull in its submodules — Buildroot itself is a
submodule, so this step is not optional:

```bash
$ mkdir -p ~/Projects && cd ~/Projects
$ git clone https://github.com/kernelkit/infix.git
...
$ cd infix/
$ git submodule update --init
...
```

### Configure and Build

Select the target you want to build for, then kick off the build.  We
use the `x86_64` target since it runs with acceleration in Qemu:

```bash
$ make x86_64_defconfig
$ make
...
```

The first build downloads all the sources and compiles the entire
system, so it may take the better part of an hour on a typical laptop —
now is a good time for a coffee ☕.

> Curious what else is available?  `make list-defconfigs` shows every
> supported target, and `make help` lists the most useful make targets.
{: .prompt-tip }

### First Boot with `make run`

When the build finishes you have a bootable image.  Launch it in Qemu
with:

```bash
$ make run
...
```

Infix boots in the same terminal.  When the login prompt appears, log in
with the default credentials, user/pass: `admin` / `admin`.  You land in
a plain shell — this is intentional, it is meant for scripting and
advanced debugging:

```
Run the command 'cli' for interactive OAM

admin@infix-c0-ff-ee:~$
```

Type `cli` to enter the Infix command line interface:

```
admin@infix-c0-ff-ee:~$ cli

See the 'help' command for an introduction to the system

admin@infix-c0-ff-ee:/>
```

To shut the VM down again, you can `poweroff` from inside Infix, or use
the Qemu escape from the terminal: press **Ctrl-a** then **x**.

### Look Around

You are now in the *admin-exec* context.  Tap `?` or `Tab` on an empty
line to see what is available, and use `show` to inspect the system:

```
admin@infix-c0-ff-ee:/> show interface
...
admin@infix-c0-ff-ee:/> show ip brief
...
admin@infix-c0-ff-ee:/> show running-config
...
```

To change something, enter the *configure* context with `configure`.
Changes are staged in a scratch copy and only applied when you type
`leave` (or thrown away with `abort`):

```
admin@infix-c0-ff-ee:/> configure
admin@infix-c0-ff-ee:/config/> edit system
admin@infix-c0-ff-ee:/config/system/> set hostname lab-switch
admin@infix-c0-ff-ee:/config/system/> leave
admin@lab-switch:/>
```

For a proper tour of the CLI, see the [CLI Introduction][cli] in the
documentation.

### Get It on the Network

A freshly booted Infix has its ports up but no IP addresses, and the
default user-mode networking sits behind a NAT — fine for a first boot,
but not much fun to interact with: no inbound SSH, no pinging the VM
from the host.  Far more interesting is to put it on a bridge, hand it
a DHCP address, and watch it reach the internet.

**1. Set up a bridge on your host.**  The easiest way is to install
[virt-manager][], which configures a libvirt NAT network on the bridge
`virbr0` — complete with a DHCP server and a route out to the internet:

```bash
$ sudo apt install virt-manager
```

**2. Switch Infix to bridged networking.**  The Qemu wrapper has its own
configuration; open it with:

```bash
$ make run-menuconfig
```

Go to **Networking → Network Mode**, select **Bridged**, and confirm the
**Bridge device** is `virbr0`.  Exit and save.

> If `make run` later complains that it is not allowed to use the bridge,
> add a line `allow virbr0` to `/etc/qemu/bridge.conf` (create the file if
> it does not exist) — this permits Qemu's bridge helper to attach to it.
{: .prompt-info }

**3. Boot again** with `make run`.  This time the first port, `eth0`, is
wired to `virbr0`.

**4. Enable the DHCP client** on `eth0` from the CLI:

```
admin@lab-switch:/> configure
admin@lab-switch:/config/> edit interface eth0
admin@lab-switch:/config/interface/eth0/> set ipv4 dhcp
admin@lab-switch:/config/interface/eth0/> leave
```

**5. Check the result.**  Within a second or two libvirt's DHCP server
hands out a lease (typically on the `192.168.122.0/24` network), which
you can confirm in the CLI with:

```
admin@lab-switch:/> show interface
...
```

**6. Reach the outside.**  With an address and a default route in place,
ping a public host:

```
admin@lab-switch:/> ping 1.1.1.1
...
```

That round trip goes from Infix, across `virbr0`, through your host's NAT,
and out to the internet — a virtual switch behaving exactly like the real
thing.

> These changes live in `running-config` and are gone on the next boot.
> To keep them, save with `copy running-config startup-config`.  To wipe
> the slate clean instead, `factory-reset` returns the system to defaults.
{: .prompt-info }

### Networking, Brick by Brick

What you just did, in Lego terms, was snap an *IP* brick (with DHCP)
onto the *Ethernet* brick called `eth0`.  That is the whole mental
model: in Infix, every networking object is a brick, and a network is
whatever you get by snapping bricks together.

![Linux networking bricks: bridge, ip, vlan, lag, eth, veth, lo](/assets/img/lego.svg)
_**Figure 1:** The networking bricks available in Infix._

There are only a handful of brick types: physical Ethernet (`eth`),
bridges, VLANs, IP, LAG (link aggregation), and `veth` pairs for
containers.  They connect in two ways.  A lower brick has *one* upper
brick on top of it — a port belongs to *one* bridge.  An upper brick
can have *many* lowers — a bridge can swallow several ports.

Those two rules are enough to describe a switch, a router, a
VLAN-tagged uplink, a bonded link, or a container with its own private
interface.

For a tour of every brick and how they fit together, see the
[Network Configuration][net] section of the documentation.

### Handy Make Targets

A few targets you will reach for often once you start hacking:

| Target                 | What it does                                        |
|------------------------|-----------------------------------------------------|
| `make run`             | Boot the built image in Qemu                        |
| `make run-menuconfig`  | Configure the Qemu wrapper (networking, ports, RAM) |
| `make menuconfig`      | Configure the Infix image itself (Buildroot)        |
| `make list-defconfigs` | List all supported targets                          |
| `make apply-dev`       | Enable `root` login, handy while developing         |
| `make foo-rebuild`     | Rebuild just package `foo` after changing it        |

### Where to Go Next

You now have a full Infix build you can boot, configure, and rebuild at
will.  A few directions from here:

 - [Getting Started with Infix][gs] — the pre-built release path, if you
   want the quick way too
 - [Infix in GNS3][gns3] — build multi-node topologies on a graphical
   canvas
 - [Inside Infix][inside] — how the system is put together, from YANG to
   the immutable root filesystem
 - [Developer's Guide][dev] and the [full documentation][docs] — the
   complete reference, including the [virtual environments][virt] section

Questions and feedback are always welcome in the [Discussion Forum][disc]
and on [Discord][discord]!

[gs]:          /posts/getting-started/
[gns3]:        /posts/infix-in-gns3/
[inside]:      /posts/inside-infix/
[cli]:         https://www.kernelkit.org/infix/latest/cli/introduction/
[net]:         https://www.kernelkit.org/infix/latest/networking/
[dev]:         https://www.kernelkit.org/infix/latest/developers-guide/
[docs]:        https://www.kernelkit.org/infix/
[virt]:        https://www.kernelkit.org/infix/latest/virtual/
[disc]:        https://github.com/orgs/kernelkit/discussions
[discord]:     https://discord.gg/6bHJWQNVxN
[virt-manager]: https://virt-manager.org/
[Debian]:      https://www.debian.org/
[Ubuntu]:      https://ubuntu.com/
[Linux Mint]:  https://linuxmint.com/
