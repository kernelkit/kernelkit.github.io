---
icon: fas fa-info-circle
order: 1
---

Infix is a free, Linux based, immutable[^1] operating system for
networked equipment.  Although primarily focused on switches and
routers, its core values may be appealing for other use-cases as
well:

- Runs from a squashfs image on a read-only partition
- Single configuration file on a separate partition
- Linux switchdev provides open switch APIs
- Atomic upgrades to secondary partition
- Highly security focused

The entire system is modeled using [YANG][1] with standard IETF models
and dedicated models when needed to fully leverage Linux capabilities.
Meaning, not only is the system configuration derived from YANG, but
also system state and any operations (RPC/actions), like upgrade.

An immutable operating system greatly enhances security.  Configuration
and data, e.g, containers, is stored on separate partitions to ensure
complete separation from system files and allow for seamless backup,
restore, and provisioning.

In itself Infix is perfectly suited for dedicated networking tasks and
native support for Docker containers provides a versatile platform that
can easily be adapted to any customer need.  Be it legacy applications,
network protocols, process monitoring, or edge data analysis, it can run
close to end equipment.  Either directly connected on dedicated Ethernet
ports or indirectly using virtual network cables to exist on the same
LAN as other connected equipment.

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
