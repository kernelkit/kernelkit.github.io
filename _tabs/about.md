---
icon: fas fa-info-circle
order: 1
---

Infix is a free, Linux-based, immutable[^1] operating system based on
[Buildroot][2] and completely modeled in YANG using [sysrepo][3].  This
allows for full remote control and monitoring using NETCONF or RESTCONF.
Initially focused on switches and routers, Infix has grown to be useful
for many other use-cases as well.

An immutable operating system greatly enhances security.  Configuration
and data, e.g, containers, is stored on separate partitions to ensure
complete separation from system files and allow for seamless backup,
restore, and provisioning.

### Core Values

- Runs from a squashfs image on a read-only partition
- Single configuration file on a separate partition
- Linux switchdev (DSA) provides open switch APIs
- Atomic upgrades using common A/B partitioning
- Highly security focused — LTS kernel + Buildroot

### YANG vs NETCONF vs RESTCONF

The entire system is modeled using [YANG][1] with standard IETF models
and dedicated models when needed to fully leverage Linux capabilities.
Meaning, not only is the system configuration derived from YANG, but
also system state and any operations (RPC/actions), like upgrade.

The *wire protocol* to interact with Infix devices is NETCONF (xml over
ssh) and RESTCONF (json over https).  The latter is particularly useful
for scripting (and demo) purposes, while the former has more tooling
available, e.g., [Clixon Controller][4], which is a NETCONF controller.

### Adaptability with Containers

In itself, Infix is perfectly suited for dedicated networking tasks and
native support for Docker containers provides a versatile platform that
can easily be adapted to any customer need.  Be it legacy applications,
network protocols, process monitoring, or edge data analysis, it can run
close to end equipment.  Either directly connected on dedicated Ethernet
ports or indirectly using virtual network cables to exist on the same
LAN as other connected equipment.

### Summary

The simple design of Infix provides complete control over both system
and data, minimal cognitive burden, and makes it incredibly easy to get
started.

----

[^1]: An immutable operating system is one with read-only file systems,
    atomic updates, rollbacks, declarative configuration, and workload
    isolation.  All to improve reliability, scalability, and security.
    For more information, see <https://ceur-ws.org/Vol-3386/paper9.pdf>
    and <https://www.zdnet.com/article/what-is-immutable-linux-heres-why-youd-run-an-immutable-linux-distro/>.

[1]: https://datatracker.ietf.org/doc/html/rfc7950
[2]: https://buildroot.org/
[3]: https://www.sysrepo.org/
[4]: https://github.com/clicon/clixon-controller
