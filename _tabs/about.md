---
icon: fas fa-info-circle
order: 1
---

[Infix][0] is a free, Linux-based, immutable[^1] operating system built on
[Buildroot][2] and fully modeled in YANG using [sysrepo][3]. This enables
complete remote control and monitoring via NETCONF or RESTCONF.  Originally
designed for switches and routers, Infix now serves a broad range of use
cases, including edge devices and security-critical applications.

An immutable operating system significantly enhances security by design.
Configuration and application data, including containers, are stored on
separate partitions to ensure complete isolation from system files and
enable seamless backup, restore, and provisioning operations.

### Core Features

- Boots from a squashfs image on a read-only partition
- Full system modeling in YANG for standardized management
- Single configuration file stored on a separate partition
- Linux switchdev (DSA) provides open switch APIs
- Atomic upgrades using proven A/B partitioning
- Security-focused architecture — always LTS kernel and Buildroot
- Native Docker container support for workload isolation

### YANG, NETCONF, and RESTCONF Integration

The entire system is modeled using [YANG][1], incorporating both standard
IETF models and custom models designed to fully leverage Linux capabilities.
This means not only system configuration but also system state and
operations (RPC/actions) such as upgrades are all derived from YANG models.

The wire protocols for interacting with Infix devices are NETCONF (XML over
SSH) and RESTCONF (JSON over HTTPS). RESTCONF is particularly well-suited
for scripting and demonstration purposes, while NETCONF benefits from
extensive tooling support, including [Clixon Controller][4], a dedicated
NETCONF controller.

### Extensibility Through Containerization

While Infix excels at dedicated networking tasks, its native Docker container
support creates a versatile platform that adapts to diverse customer
requirements. Whether deploying legacy applications, implementing custom
network protocols, performing process monitoring, or conducting edge data
analysis, workloads can run close to end equipment. This can be achieved
either through direct connection via dedicated Ethernet ports or indirectly
using virtual network interfaces to participate in the same LAN as other
connected equipment.

### Summary

Infix's streamlined design provides comprehensive control over both system
and data layers while minimizing operational complexity. This makes it
exceptionally easy to deploy and manage in production environments.

### About KernelKit

KernelKit is an independent organization dedicated to developing and maintain
[Infix][0], [curiOS][5], and other innovative projects with a commitment to
keeping them forever open source.  Our mission is to advance secure, reliable,
and accessible operating system technologies for the broader community.

KernelKit is proudly sponsored by its founding developers and is commercially
backed by [Wires][6], which enables us to advance our development efforts for
commercial deployment and our community contributions.

<div align="center">
  <a href="https://wires.se"><img alt="Wires" src="https://raw.githubusercontent.com/wires-se/.github/main/profile/logo.png" width=300></a>
</div>

----

[^1]: An immutable operating system features read-only file systems,
    atomic updates, rollbacks, declarative configuration, and workload
    isolation—all designed to improve reliability, scalability, and security.
    For more information, see <https://ceur-ws.org/Vol-3386/paper9.pdf>
    and <https://www.zdnet.com/article/what-is-immutable-linux-heres-why-youd-run-an-immutable-linux-distro/>.

[0]: https://github.com/kernelkit/infix
[1]: https://datatracker.ietf.org/doc/html/rfc7950
[2]: https://buildroot.org/
[3]: https://www.sysrepo.org/
[4]: https://github.com/clicon/clixon-controller
[5]: https://github.com/kernelkit/curiOS
[6]: https://wires.se
