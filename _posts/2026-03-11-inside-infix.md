---
title: "Inside Infix"
description: "Built on YANG, Built to Last"
author: troglobit
date: 2026-03-11 12:25:00 +0100
categories: [architecture]
tags: [design, architecture, yang, netconf, buildroot, sysrepo]
image:
  path: /assets/img/architecture-overview.svg
  alt: Infix architecture overview showing the management stack
  show_in_post: false
---

How does Infix work, what is it based on, and what is YANG?  These are a
few of the questions we get from time to time, and this time we will
attempt to answer them.  Let's dive in!

> If your questions are not answered here, please feel free to engage
> with us in our [Discussion Forum][1] or join us on [Discord][2]!
{: .prompt-info }

### Overview

Let's start by zooming out and identify the major components:

 - *sysrepo:* the engine which all other major components connect to,
   funnels all actions to callbacks in *confd*
 - *netopeer2:* provides the NETCONF interface (XML over SSH) for
   external management
 - *rousette:* provides the RESTCONF interface (JSON over HTTPS) for
   external management, this is also where the upcoming WebUI will
   connect to, meaning the same WebUI can be used to manage all Infix
   devices on the LAN
 - *klish:* provides the built-in Command Line Interface (CLI)
 - *confd:* gets callbacks for all changes from *sysrepo* which is
   translated into C-library API calls, configuration file changes in
   `/etc`, and network interface setup calls to `iproute2`

![](/assets/img/architecture-overview.svg)
_**Figure 1:** Infix Architecture Overview_

Infix runs on a [broad range of hardware][3] — from Raspberry Pi home lab
boards and compact dual-port routers like the NanoPi R2S, through
general-purpose ARM and RISC-V end devices such as the NXP i.MX8MP EVK
and StarFive VisionFive2, all the way up to enterprise switch platforms
like the Microchip SparX-5i.  It also runs on x86_64, making it easy to
spin up instances in [Qemu][6] or [GNS3][5] for development and testing without
any dedicated hardware.  The same OS, the same tooling, the same
management interfaces throughout.

From a bottom-up perspective, one of the critical design choices for
switch platforms is to rely on Linux *switchdev* for switch silicon
abstraction.  It is what makes it possible to configure actual hardware
switch cores using the common Linux bridge.  Underneath switchdev sits
DSA (Distributed Switch Architecture), a kernel sub-layer that models
the individual ports and internal links of a switch chip and translates
bridge operations into hardware-specific commands.  All operations on the
bridge are thus "offloaded" to the DSA driver, e.g., adding an interface
to a bridge enables hardware switching on that port, and adding a VLAN
enables VLAN filtering in the switch silicon.  On platforms without a
switch core, the same bridge model applies in software — the management
interface is identical regardless of whether forwarding happens in
silicon or in the kernel.

Unlike many other Linux-based network operating systems, Infix is not a
flavor of OpenWRT.  Instead it is built on top of the developer-friendly
[Buildroot][0], tracking its long-term support (LTS) releases.
Buildroot's LTS cadence is one release every two years (in February),
each supported for three years, with quarterly stable releases in
between.  This provides a solid base and forms the majority of all Open
Source packages.  A few of those are locally upgraded by the Infix team,
e.g., sysrepo and netopeer2, and another few are tailor made for Infix,
e.g., `confd`.

### YANG

The real hero, however, is YANG.

YANG (RFC 6020/7950) is a data modeling language designed specifically
for network devices.  At its core, YANG lets you formally describe what
configuration and state a device has — what knobs exist, what values
they accept, how they relate to each other — in a machine-readable way.
Think of it as a schema, but one that carries enough semantic weight for
tools to do genuinely useful things with it automatically.

In Infix, the entire system is modeled in YANG.  This has one profound
consequence: every management interface — the CLI, NETCONF, RESTCONF,
and the upcoming WebUI — is ultimately backed by the same models.  There
is no separate CLI grammar to maintain, no divergence between what the
web interface can do and what NETCONF can do.  When a new feature is
added to YANG, it appears everywhere at once.

Infix follows industry-standard IETF models wherever they exist.  So
`ietf-interfaces`, `ietf-routing`, `ietf-ip`, and friends describe
interfaces, routes, and addresses — the same models you would find on
any standards-compliant device.  Where no standard model exists, Infix
defines its own, prefixed with `infix-`.

Here is a small, simplified excerpt to give a flavour of what a YANG
module looks like.  Real modules are larger, but the structure is
always the same:

```yang
container interface {
  leaf name {
    type string {
      length "1..15";
      pattern '[a-zA-Z][a-zA-Z0-9_-]*';
    }
    description
      "Interface name, e.g. eth0 or br0.  Linux limits
       interface names to 15 characters.";
  }

  leaf description {
    type string {
      length "0..64";
    }
    description "Human-readable interface label.";
  }

  leaf enabled {
    type boolean;
    default true;
    description
      "Administrative state.  Set to false to bring the
       interface down without removing its configuration.";
  }

  leaf mtu {
    type uint16 {
      range "68..9000";
    }
    units "bytes";
    default 1500;
    description
      "Maximum transmission unit.  Standard Ethernet is 1500
       bytes; jumbo frames typically go up to 9000.";
  }
}
```

The `pattern` on `name` is a regular expression — only valid Linux
interface names are accepted.  The `range` on `mtu` means a value of
`42` is rejected before it ever reaches the device; the model itself
is the contract.  This is what makes *offline validation* possible: a
configuration file can be checked against the loaded YANG modules at
any point — on a developer's laptop, in CI, or in a management system
— and any constraint violation is caught immediately, before anything
is sent over the wire.

#### The CLI is generated from YANG

One of the most visible benefits is the CLI.  *klish* generates the
entire command hierarchy directly from the loaded YANG modules.  This
means `?` and `TAB` completion always reflect the actual model —
if a node is in YANG, it is in the CLI; if it is not, it is not.  There
is no hand-written command parser to drift out of sync.

#### Datastores

*sysrepo* organises configuration into a set of datastores:

 - `factory-config` — generated at first boot, holds per-device defaults
   such as a unique hostname and the correct number of ports; can be
   customised per product by OEMs
 - `failure-config` — also generated at boot; used when the system
   enters *Fail Secure Mode* to ensure the device remains accessible
   even if `startup-config` is broken
 - `startup-config` — created from `factory-config` if it does not yet
   exist; loaded as the active configuration on every boot
 - `running-config` — what is actually running right now; identical to
   `startup-config` at boot unless changes have been made since
 - `candidate-config` — a scratch space created from `running-config`
   when entering the configure context; changes here can be freely
   discarded with `abort` or committed with `leave`

The separation between `candidate` and `running` means you can stage a
set of changes, inspect the diff, and apply them atomically — or throw
them away without touching the live system.

### Immutable by Design

Infix runs from a read-only *SquashFS* root filesystem.  There is
nothing to corrupt, no package manager to leave the system in a
half-upgraded state, and no way for a bad configuration to break the OS
itself.  Configuration lives separately in a writable partition, and the
YANG datastore sits on top of that.

Software upgrades use an A/B partition scheme.  A new image is written
to the inactive slot while the system keeps running, and the bootloader
is only pointed at the new slot once the write completes successfully.
If the new image fails to boot, the system automatically falls back to
the previous slot.  Upgrades are therefore atomic: you either end up on
the new version or the old one — never in between.

### Built for Continuous Testing

The same design choices that make Infix robust in production also make
it straightforward to test thoroughly.  Because *sysrepo* is the
authoritative source of truth and configuration is fully decoupled from
the OS, the Infix regression suite can push NETCONF configuration to a
device under test (DUT), verify the resulting operational state via
RESTCONF, and reset it to a known-good baseline — all without touching
the filesystem directly and without any CLI scraping.

The test suite, *Infamy*, runs against both virtual topologies in [Qemu][6]
and real physical hardware using identical test cases.  Virtual
topologies make it cheap to catch regressions early in development;
physical runs ensure that hardware-specific paths — DSA offloads, WiFi,
switch silicon — are exercised regularly.

This level of automation means Infix is not constrained to a fixed
monthly release cadence.  When a fix or feature is ready and passes the
full suite, a release can go out the same day.  The result is a project
that moves quickly without sacrificing stability.


### Wrapping Up

Three ideas run through everything described here.  YANG gives the
system a single source of truth that every interface — CLI, NETCONF,
RESTCONF, WebUI — derives from automatically, and whose constraints
catch mistakes before they reach a device.  Buildroot gives it a
stable, well-understood foundation that is easy to audit and extend.
And the immutable, A/B filesystem means that neither a bad upgrade nor
a broken configuration can leave a device in an unrecoverable state.

Together they add up to a system that is easier to manage at scale, easier
to automate, and easier to trust in production — whether you are running
it on a $35 Raspberry Pi or a data-centre switch.

If you want to go deeper, the [full documentation][4] covers every
feature in detail.  Questions and feedback are always welcome in the
[Discussion Forum][1] and on [Discord][2]!

[0]: https://buildroot.org/lts.html
[1]: https://github.com/orgs/kernelkit/discussions
[2]: https://discord.gg/6bHJWQNVxN
[3]: /posts/router-boards/
[4]: https://www.kernelkit.org/infix/
[5]: /posts/infix-in-gns3/
[6]: /posts/getting-started/
