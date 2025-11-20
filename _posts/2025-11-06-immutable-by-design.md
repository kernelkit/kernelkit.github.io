---
title: Immutable by Design
author: troglobit
date: 2025-11-06 09:00:00 +0100
categories: [architecture]
tags: [immutable, embedded, linux, security, buildroot, containers]
---

We've talked before about how Infix OS focuses on being friendly, secure,
and immutable. But what does immutability actually mean to us?

It's not about locking things down for the sake of it. It's about
**predictability**. Every Infix release is a fully rebuilt, self-contained
OS image — not a patchwork of upgrades. We don't `apt upgrade` one package;
we rebuild everything from source with [Buildroot][1]. The result is a root
filesystem that's read-only, shipped as a squashfs.

Your configuration lives separately in `/cfg/startup-config.cfg`, and
anything you add — like Docker containers — goes into `/var`, which stays
writable. To us, immutability means you always know exactly what's running
— no drift, no surprises.

### What This Means in Practice

When you deploy an Infix device, you get:

**Consistent behavior across devices**: Every device running Infix v25.10
is running the exact same bits. No variations from incremental updates or
package conflicts.

**Atomic updates**: System upgrades replace the entire OS image. Either
the update succeeds completely, or it doesn't happen at all. No half-broken
systems from interrupted package installations.

**Easy rollback**: Because each release is a complete image, rolling back
to a previous version is straightforward. Your data and configuration in
`/var` and `/cfg` persist across updates.

**Reduced attack surface**: A read-only root filesystem means attackers
can't modify system binaries or inject persistent malware into core system
files.

### The Separation of Concerns

Infix maintains a clear separation between three types of data:

```
/           Read-only root filesystem (squashfs)
/cfg        Configuration data
/var        Variable data (logs, containers, etc.)
```

This separation ensures that:

- System binaries and libraries are protected from tampering
- Your network configuration persists across OS updates
- User data and containers remain independent of the OS version

### Building from Source

Every Infix release starts from a clean slate. [Buildroot][1] downloads,
compiles, and assembles every component — from the kernel to userspace
utilities. This approach has several advantages:

- Full control over what goes into the image
- Reproducible builds with known component versions
- Optimized for embedded targets
- No hidden dependencies or unexpected packages

### Updates Without Surprises

Traditional Linux systems often accumulate state over time. Package updates
can leave behind old configuration files, orphaned dependencies, and
conflicting libraries. Over months or years, systems drift from their
initial state.

With Infix, each update is a fresh start. The OS image is replaced entirely,
while your configuration and data remain untouched in their dedicated
partitions. What you test in development is what runs in production — no
hidden variables.

### Writable Where You Need It

Immutability doesn't mean inflexibility. The `/var` directory provides
persistent storage for:

- Container images and volumes
- Log files
- Dynamic application data
- Package caches

For containers, Infix uses standard OCI-compatible container runtimes. Pull
your images, run your services, and manage your applications as you would
on any container platform — all while the underlying OS remains protected.

### Conclusion

Immutability in Infix isn't a constraint; it's a foundation for reliability.
By separating the OS from configuration and data, and by rebuilding from
source for every release, we ensure that your network infrastructure behaves
predictably — today, tomorrow, and after the next update.

For more details about the Infix architecture, visit the [Infix documentation][2]
or explore the [source code on GitHub][3].

[1]: https://buildroot.org/
[2]: https://kernelkit.org/infix/
[3]: https://github.com/kernelkit/infix/
