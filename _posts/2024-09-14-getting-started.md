---
title: Getting Started with Infix
author: troglobit
date: 2024-09-14 12:11:00 +0100
last_modified_at: 2026-03-12 08:00:00 +0100
categories: [howto]
tags: [beginner]
---

This is a guide for those who want to learn more about Infix using a
hands-on approach.  You need a Linux 🐧 system with Qemu installed, we
recommend [Debian][] based systems, like [Ubuntu][] and [Linux Mint][].

> **Prefer a graphical environment or want to build multi-node topologies?**
>
> Infix is available in the [GNS3 Marketplace][GNS3] — install the
> appliance and you can drag, drop, and cable up entire networks without
> touching the command line.  See our [Infix in GNS3][gns3post] post to
> get started.
{: .prompt-tip }

### Installing Qemu

From this point we assume you have your x86_64/AMD64 based Linux system
up and running.  Time to start your favorite terminal application! 😃

```bash
$ sudo apt install qemu-system-x86 virt-manager
...
```

> For a pain-free experience we recommend enabling CPU virtualization in
> your BIOS/UEFI, which for many computers is disabled by default.
{: .prompt-tip }

### Download Infix

Go to https://github.com/kernelkit/infix/releases/latest and scroll down
to *Assets*.  Download the tarball named `infix-x86_64-VER.tar.gz`,
where VER is the `YY.MM.PATCH` release number, e.g. `26.02.1`.

It extracts to a subdirectory with the same name:

```bash
$ tar xf infix-x86_64-26.02.1.tar.gz
$ cd infix-x86_64-26.02.1/
```


### Running Infix

> Depending on how you have [set up your Linux][sudo] installation, the
> following may require being run with superuser privileges, i.e., you
> may need to prepend the command with `sudo`.
{: .prompt-warning }

```bash
$ ./qemu/run.sh
```

You should now see Infix booting.  When the login prompt appears, the
default credentials are `admin` / `admin`.


### More Ethernet Ports

For more Ethernet ports in your emulated system you need to change the
Qemu configuration used for Infix.  This can be done using a menuconfig
interface, which requires the following extra package:

```bash
$ sudo apt install kconfig-frontends
```

We can now enter the configuration:

```bash
$ ./qemu/run.sh -c
```

Go down to *Networking*, select *TAP*, now you can change the *Number of
TAPs*, e.g. to 10.  Exit and save the configuration, then start Qemu again:

```bash
$ ./qemu/run.sh
```

> Make sure to do a factory reset from the CLI, otherwise you will be
> stuck with that single interface from before.
{: .prompt-info }

For more advanced Qemu options — networking bridges, shared folders, and
multi-node setups — see the [virtual environments][virt] section of the
full documentation.

[sudo]:       https://troglobit.com/2016/12/11/a-life-without-sudo/
[Debian]:     https://www.debian.org/
[Ubuntu]:     https://ubuntu.com/
[Linux Mint]: https://linuxmint.com/
[GNS3]:       https://gns3.com/marketplace/appliances/infix
[gns3post]:   /posts/infix-in-gns3/
[virt]:       https://www.kernelkit.org/infix/latest/virtual/
