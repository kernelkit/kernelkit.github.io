---
icon: fas fa-terminal
order: 2
---

The CLI configure context is generated from the loaded [YANG][1] models
and their corresponding [sysrepo][2] plugins.  The following is brief
example of how to set the IP address of an interface.

> The <kbd>TAB</kbd> shown here means press the Tab key to show possible
> command completions.
{: .prompt-tip }

<div class="language-console">
<div class="code-header">
<span data-label-text="CLI"><i class="fas fa-code fa-fw small"></i></span>
<button aria-label="copy" data-title-succeed="Copied!"><i class="fas fa-terminal"></i></button></div>
<pre class="highlight"><code class="language-console">admin@infix-12-34-56:/> <b>configure</b>
admin@infix-12-34-56:/config/> <b>edit interface eth0</b>
admin@infix-12-34-56:/config/interface/eth0/> <b>set ipv4 <kbd>TAB</kbd></b>
      address     autoconf      bind-ni-name     dhcp
      enabled     forwarding    mtu              neighbor
admin@infix-12-34-56:/config/interface/eth0/> <b>set ipv4 address 192.168.2.200 prefix-length 24</b>
admin@infix-12-34-56:/config/interface/eth0/> <b>show</b>
type ethernet;
ipv4 {
  address 192.168.2.200 {
    prefix-length 24
  }
}
ipv6</code></pre></div>

Whenever you've made a change in configure context, you can see inspect
the modifications with the `diff` command:


<div class="language-console">
<div class="code-header">
<span data-label-text="diff"><i class="fas fa-code fa-fw small"></i></span>
<button aria-label="copy" data-title-succeed="Copied!"><i class="fas fa-terminal"></i></button></div>
<pre class="highlight"><code class="language-diff">admin@infix-12-34-56:/config/interface/eth0/> <b>diff</b>
interfaces {
  interface eth0 {
<span class="gi">+    ipv4 {
+      address 192.168.2.200 {
+        prefix-length 24
+      }
+    }</span>
  }
}</code></pre></div>

To activate the changes you can issue the `leave` command anywhere.
(Use the `abort` command to cancel all changes.)  Inspect the changes
made, and remember to save your changes to `startup-config`.

<div class="language-console">
<div class="code-header">
<span data-label-text="CLI"><i class="fas fa-code fa-fw small"></i></span>
<button aria-label="copy" data-title-succeed="Copied!"><i class="fas fa-terminal"></i></button></div>
<pre class="highlight"><code class="language-console">admin@infix-12-34-56:/config/interface/eth0/> <b>leave</b>
admin@infix-12-34-56:/> <b>show interfaces</b>
INTERFACE       PROTOCOL   STATE       DATA
lo              ethernet   UP          00:00:00:00:00:00
                ipv4                   127.0.0.1/8 (static)
                ipv6                   ::1/128 (static)
eth0            ethernet   UP          52:54:00:12:34:56
                ipv4                   192.168.2.200/24 (static)
                ipv6                   fe80::5054:ff:fe12:3456/64 (link-layer)
admin@infix-12-34-56:/> <b>copy running-config startup-config</b></code></pre></div>

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
[3]: /infix/latest/cli/introduction/
[6]: /infix/latest/networking/
