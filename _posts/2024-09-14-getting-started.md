---
title: Getting Started with Infix
author: troglobit
date: 2024-09-14 12:11:00 +0100
categories: [howto]
tags: [beginner]
pin: true
---

This is a guide for those who want to learn more about Infix using a
hands-on approach.  You need a Linux 🐧 system with Qemu installed, we
recommend [Debian][] based systems, like [Ubuntu][] and [Linux Mint][].

> For a pain-free experience we recommend enabling CPU virtualization in
> your BIOS/UEFI, which for many computers is disabled by default.

From this point we assume you have your x86_64/AMD64 based Linux system
up and running.  Time to start your favorite terminal application! 😃


### Installing Qemu

```bash
$ sudo apt install qemu-system-x86 virt-manager
```

### Download Infix

Go to https://github.com/kernelkit/infix/releases/latest and scroll down
to *Assets*.  Download the tarball named `infix-x86_64-VERSION.tar.gz`,
where VERSION is the year+month+patch release number.

It extracts to a subdirectory:


```bash
$ sudo apt install qemu-system-x86 virt-manager
```


### Running Infix

> Depending on how you have [set up your Linux][sudo] installation, the
> following may require being run with superuser privileges, i.e., you
> may need to prepend the command with `sudo`.

For more Ethernet ports in your emulated system you need to change the
Qemu configuration used for Infix.  This can be done using a menuconfig
interface, which requires the following extra package:

```bash
$ sudo apt install kconfig-frontends
```

We can now enter the configuration:

```bash
$ ./qemu.sh -c
```

Go down to *Networking*, select *TAP*, now you can change the *Number of
TAPs*, e.g. to 10.  Exit and save the configuration, then you can start
Qemu again:

   ./qemu.sh

> Make sure to do a factory reset from the CLI, otherwise you will be
> stuck with that single interface from before.

[sudo]:       https://troglobit.com/2016/12/11/a-life-without-sudo/
[Debian]:     https://www.debian.org/
[Ubuntu]:     https://ubuntu.com/
[Linux Mint]: https://linuxmint.com/
