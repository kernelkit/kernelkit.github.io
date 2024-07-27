---
icon: fas fa-terminal
order: 2
---

The CLI configure context is generated from the loaded [YANG][1] models
and their corresponding [sysrepo][2] plugins.  The following is brief
example of how to set the IP address of an interface.

> The `<TAB>` shown here means press the Tab key to show possible
> command completions.
{: .prompt-tip }

```console
admin@infix-12-34-56:/> configure
admin@infix-12-34-56:/config/> edit interface eth0
admin@infix-12-34-56:/config/interface/eth0/> set ipv4 <TAB>
      address     autoconf bind-ni-name      enabled
      forwarding  mtu      neighbor
admin@infix-12-34-56:/config/interface/eth0/> set ipv4 address 192.168.2.200 prefix-length 24
admin@infix-12-34-56:/config/interface/eth0/> show
type ethernet;
ipv4 {
  address 192.168.2.200 {
    prefix-length 24
  }
}
ipv6
```

Whenever you've made a change in configure context, you can see inspect
the modifications with the `diff` command:


```diff
admin@infix-12-34-56:/config/interface/eth0/> diff
interfaces {
  interface eth0 {
+    ipv4 {
+      address 192.168.2.200 {
+        prefix-length 24
+      }
+    }
  }
}
```

To activate the changes you can issue the `leave` command anywhere.
(Use the `abort` command to cancel all changes.)  Inspect the changes
made, and remember to save your changes to `startup-config`.

```console
admin@infix-12-34-56:/config/interface/eth0/> leave
admin@infix-12-34-56:/> show interfaces
INTERFACE       PROTOCOL   STATE       DATA
lo              ethernet   UP          00:00:00:00:00:00
                ipv4                   127.0.0.1/8 (static)
                ipv6                   ::1/128 (static)
eth0            ethernet   UP          52:54:00:12:34:56
                ipv4                   192.168.2.200/24 (static)
                ipv6                   fe80::5054:ff:fe12:3456/64 (link-layer)
admin@infix-12-34-56:/> copy running-config startup-config
```

> In the CLI all changes are made to the `running-config`, providing you
> with a basic "undo" mechanism -- in case you make a change that locks
> you out, e.g., changing IP address on the same interface used to log
> in to the device -- you can simply reboot the device to get back to
> the previous state.
{: .prompt-tip }


Curious?  Continue reading:
  - [CLI Introduction][3]
  - [Networking Lego®][6]


[1]: https://datatracker.ietf.org/doc/html/rfc7950
[2]: https://www.sysrepo.org/
[3]: https://github.com/kernelkit/infix/tree/main/doc/cli
[6]: https://github.com/kernelkit/infix/blob/main/doc/networking.md
