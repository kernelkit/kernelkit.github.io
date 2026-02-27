---
title: "Field Report: WiFi AP on BPi-R3"
author: troglobit
date: 2026-02-27 10:00:00 +0100
categories: [showcase]
tags: [boards, wifi, networking, firewall, wireguard]
---

![Banana Pi BPi-R3](/assets/img/spider-on-wall.jpeg){: width="400" .right}

WiFi access point support landed in Infix v26.01, and the engineer who
spent six months making it happen — Mattias Walström — wasted no time
putting it to the test.  He wrote up [his experience][ml] of replacing
his home network with a full Infix deployment on a [BPi-R3][bpi].

His setup runs six SSIDs across 2.4 GHz and 5 GHz, each on its own
isolated network: trusted devices, IoT (13+ smart home gadgets with DHCP
reservations), guest, and untrusted.  A [zone-based firewall][zbf] keeps
everything separated and a WireGuard tunnel handles remote access.  The
MT7976C chipset supports up to 16 virtual access points per radio, so
there's plenty of room to grow.

All of it running on a €150 router board, managed through the same
NETCONF/RESTCONF interfaces you'd use on data center gear.

Read [Mattias' article][ml] for the full story, then grab a [BPi-R3][ali],
[flash an image][flash], and have a look at the [WiFi documentation][docs].

---

[ml]:    https://www.linkedin.com/pulse/running-infix-banana-pi-r3-mattias-walstr%C3%B6m-lywcf
[bpi]:   /posts/banana-pi-r3/
[zbf]:   /posts/zone-based-firewall/
[flash]: /posts/flashing-sdcard/
[docs]:  https://kernelkit.org/infix/latest/
[ali]:   https://www.aliexpress.com/item/1005005827004033.html
