---
title: Setting hostname using CLI
author: troglobit
date: 2024-07-24 12:18:00 +0100
categories: [examples]
tags: [cli]
---

A common task for initial setup of your operating system is to change
the hostname of a device.  The default is a unique name composed from
the built-in default name suffixed by the last three octets of the base
MAC address.

```console
admin@infix-c0-ff-ee-:/>
```

In this example the built-in name is `infix` and the last three octets
resemble my favorite morning drink.  The built-in name depends on the
Infix build and can be [tailored to your product][0].

The hostname can be changed from the system configuration context:

```console
admin@infix-c0-ff-ee:/> configure
admin@infix-c0-ff-ee:/config/> edit system
admin@infix-c0-ff-ee:/config/system/> set hostname example
admin@infix-c0-ff-ee:/config/system/> leave
admin@example:/> 
```

Notice how the hostname in the prompt does not change until the change
is committed by issuing the leave command.

There are a few format specifiers available:

 - `%h`: built-in hostname, here `infix`
 - `%i`: built-in identity, here `infix` (depends on branding)
 - `%m`: last three octets of base MAC address (may be 00-00-00)
 - `%%`: literal `%` character

One good reason to maintain a unique hostname, e.g., using `%m`, is that
it is advertised over mDNS-SD in the `.local` domain.  If another device
already has claimed the `example.local` CNAME, mDNS will notice this and
advertise a "uniqified" variant, usually suffixing with an index, e.g.,
`example-1.local`.  Use an mDNS browser to scan for available devices on
your LAN.

> Critical services like syslog, mDNS, LLDP, and similar that advertise
> the hostname, are restarted when the hostname is changed.
{: .prompt-info }


[0]: https://kernelkit.org/infix/latest/branding/
