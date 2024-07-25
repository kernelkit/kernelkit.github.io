---
title: Setting hostname using CLI
author: troglobit
date: 2024-07-24 12:18:00 +0100
categories: [examples]
tags: [cli]
---

Notice how the hostname in the prompt does not change until the change
is committed by issuing the leave command.

```
admin@host:/config/> edit system
admin@host:/config/system/> set hostname example
admin@host:/config/system/> leave
admin@example:/> 
```

The hostname is advertised over mDNS-SD in the `.local` domain.  If
another device already has claimed the example.local CNAME, in our case,
mDNS will advertise a "uniqified" variant, usually suffixing with an
index, e.g., `example-1.local`.  Use an mDNS browser to scan for
available devices on your LAN.

> Critical services like syslog, mDNS, LLDP, and similar that advertise
> the hostname, are restarted when the hostname is changed.
{: .prompt-info }
