---
title: Seamless Wi-Fi with Mesh Backhaul and Roaming
author: mattiaswal
date: 2026-07-02 08:00:00 +0200
last_modified_at: 2026-10-27 12:00:00 +0100
categories: [howto]
tags: [wifi, mesh, roaming, networking]
image:
  path: /assets/img/wifi-mesh-roaming-hero.svg
  alt: Seamless Wi-Fi with 802.11s mesh backhaul and 802.11r/k/v fast roaming
  show_in_post: false
---

Covering a building or a yard with Wi-Fi usually means several access
points sharing one network name, so a laptop or a phone keeps working as
it moves between them.  Two problems show up quickly:

1. Getting the network to every AP.  Pulling Ethernet to each AP is the
   reliable way, but it is not always possible.  Where you cannot run a
   cable, a wireless backhaul has to carry the traffic instead.
2. Moving clients between APs without dropping them.  A client that has
   to fully re-authenticate every time it changes AP will stutter on voice
   calls and long downloads.  Fast roaming fixes that.

Infix [v26.06][release] adds both: 802.11s mesh for the wireless
backhaul, and 802.11r/k/v for fast roaming.  Basic access point support
arrived in v26.01, and with this release several APs can work as one
network.  Below we build a three-node network with them, and at the end
look at the test that keeps it working.

The [Wi-Fi documentation][docs] has the reference material behind each
setting: bands, channels, security modes, and the full list of roaming
knobs.  This post shows how they fit together.  For a gentler start,
[Wi-Fi Made Easy with Infix OS][easy] sets up a single access point, and
then one per band, step by step.

## The design

Three nodes cover the area.  Each node does two jobs, so each needs two
radios:

- `radio0` is the mesh backhaul.  All three nodes join one 802.11s mesh on
  5 GHz.  The mesh carries traffic between the nodes, so only one node needs
  a wire back to the rest of the LAN.  The others reach it over the air.
- `radio1` is for clients.  Each node runs an access point on 2.4 GHz with
  the same SSID and the same passphrase.  This is the network clients see
  and roam across.

![Network topology](/assets/img/wifi-mesh-roaming-topology.svg){: width="800" }
_Three nodes bridge a 5 GHz mesh backhaul and a 2.4 GHz roaming AP; only gw1 has a wire._

The backhaul and the clients use different bands.  If the mesh and the AP
shared one radio they would also share one channel and take turns on the
air, and every client frame would compete with backhaul traffic.  With
5 GHz for the mesh and 2.4 GHz for clients, both run at full speed, and the
backhaul, which carries everyone's traffic, gets the wider channels.

A bridge (`br0`) on each node ties the mesh interface, the AP interface,
and (on the wired node) the uplink into one layer-2 segment.  With mesh
forwarding on, the mesh acts as a transparent backhaul: a client attached
to gw3's AP sits on the same LAN as something plugged into gw1, and the
frames cross the mesh without anyone configuring a route.

> A mesh point and an access point cannot share a radio; Infix rejects
> that combination.  Plan on two radios per node.
{: .prompt-info }

## Building it

The three nodes are configured almost identically.  Only gw1 has the
wired uplink; gw2 and gw3 are reachable only over the mesh.  The examples
below show gw1 in full, then the one difference for the others.

### 1. The shared secrets

Every node needs two keystore entries: one passphrase for the mesh (all
mesh links use WPA3-SAE) and one for the client AP.  Use the same two
secrets on all three nodes.

<pre class="cli"><code>admin@gw1:/&gt; <b>configure</b>
admin@gw1:/config/&gt; <b>edit keystore symmetric-key mesh-secret</b>
admin@gw1:/config/keystore/.../mesh-secret/&gt; <b>set key-format passphrase-key-format</b>
admin@gw1:/config/keystore/.../mesh-secret/&gt; <b>edit cleartext-symmetric-key</b>
Passphrase: ************
Retype passphrase: ************
admin@gw1:/config/keystore/.../mesh-secret/&gt; <b>end</b>
admin@gw1:/config/&gt; <b>edit keystore symmetric-key wifi-secret</b>
admin@gw1:/config/keystore/.../wifi-secret/&gt; <b>set key-format passphrase-key-format</b>
admin@gw1:/config/keystore/.../wifi-secret/&gt; <b>edit cleartext-symmetric-key</b>
Passphrase: ************
Retype passphrase: ************
admin@gw1:/config/keystore/.../wifi-secret/&gt; <b>leave</b></code></pre>

### 2. The radios

The mesh refuses to start without a band, a channel, and a real country
code.  The country code is one setting for all radios, while `radio0`,
which carries the mesh, gets its own band and channel.  `radio1` carries
the AP on 2.4 GHz.

<pre class="cli"><code>admin@gw1:/&gt; <b>configure</b>
admin@gw1:/config/&gt; <b>set hardware wifi country-code SE</b>
admin@gw1:/config/&gt; <b>edit hardware component radio0 wifi-radio</b>
admin@gw1:/config/hardware/component/radio0/wifi-radio/&gt; <b>set band 5GHz</b>
admin@gw1:/config/hardware/component/radio0/wifi-radio/&gt; <b>set channel 36</b>
admin@gw1:/config/hardware/component/radio0/wifi-radio/&gt; <b>end</b>
admin@gw1:/config/&gt; <b>edit hardware component radio1 wifi-radio</b>
admin@gw1:/config/hardware/component/radio1/wifi-radio/&gt; <b>set band 2.4GHz</b>
admin@gw1:/config/hardware/component/radio1/wifi-radio/&gt; <b>set channel 1</b>
admin@gw1:/config/hardware/component/radio1/wifi-radio/&gt; <b>leave</b></code></pre>

> Before Infix v26.10, the country code was set per radio, with `set
> country-code SE` in each `wifi-radio` context.
{: .prompt-info }

All three nodes use the same mesh channel (36), since mesh peers must meet
on one channel, and the same AP channel (1), so a roaming client does not
have to change channel as it moves.

### 3. The mesh interface

`wifi0` is the 802.11s mesh point on `radio0`.  The `mesh-id` is the mesh
equivalent of an SSID: every node that should join uses the same one.
Leave `forwarding` at its default, `true`, so the mesh can be bridged.

<pre class="cli"><code>admin@gw1:/config/&gt; <b>edit interface wifi0</b>
admin@gw1:/config/interface/wifi0/&gt; <b>set wifi radio radio0</b>
admin@gw1:/config/interface/wifi0/&gt; <b>set wifi mesh-point mesh-id backhaul</b>
admin@gw1:/config/interface/wifi0/&gt; <b>set wifi mesh-point security secret mesh-secret</b>
admin@gw1:/config/interface/wifi0/&gt; <b>leave</b></code></pre>

### 4. The roaming access point

`wifi1` is the AP on `radio1`, and this step is mostly about its roaming
settings.  All three APs must agree on the SSID, the passphrase, and the
mobility domain.  Together they tell a client that the three APs are one
roaming network.

`mobility-domain hash` derives the domain from the SSID, so the three
APs land on the same domain without anyone copying a hex value around.

<pre class="cli"><code>admin@gw1:/config/&gt; <b>edit interface wifi1</b>
admin@gw1:/config/interface/wifi1/&gt; <b>set wifi radio radio1</b>
admin@gw1:/config/interface/wifi1/&gt; <b>set wifi access-point ssid campus</b>
admin@gw1:/config/interface/wifi1/&gt; <b>set wifi access-point security mode wpa2-wpa3-personal</b>
admin@gw1:/config/interface/wifi1/&gt; <b>set wifi access-point security secret wifi-secret</b>
admin@gw1:/config/interface/wifi1/&gt; <b>set wifi access-point roaming dot11r</b>
admin@gw1:/config/interface/wifi1/&gt; <b>set wifi access-point roaming dot11r mobility-domain hash</b>
admin@gw1:/config/interface/wifi1/&gt; <b>set wifi access-point roaming dot11k</b>
admin@gw1:/config/interface/wifi1/&gt; <b>set wifi access-point roaming dot11v</b>
admin@gw1:/config/interface/wifi1/&gt; <b>leave</b></code></pre>

The three roaming features do different jobs:

- `dot11r` (Fast BSS Transition) makes the handoff quick.  It pre-shares
  the keys so a roaming client skips the full WPA handshake, cutting the
  gap from roughly a second to under 50 ms.
- `dot11k` (Radio Resource Management) hands the client a neighbour list,
  so it already knows which APs to look at instead of scanning blindly.
- `dot11v` (BSS Transition Management) lets an AP nudge a client toward a
  better one.

### 5. The bridge

`br0` joins the mesh and the AP into one segment.  On gw1 the wired uplink
joins too, which gives the other nodes their path to the rest of the LAN.

<pre class="cli"><code>admin@gw1:/config/&gt; <b>set interface br0</b>
admin@gw1:/config/&gt; <b>set interface wifi0 bridge-port bridge br0</b>
admin@gw1:/config/&gt; <b>set interface wifi1 bridge-port bridge br0</b>
admin@gw1:/config/&gt; <b>set interface eth0 bridge-port bridge br0</b>
admin@gw1:/config/&gt; <b>leave</b></code></pre>

gw2 and gw3 are identical except they have no wired uplink, so drop the
`eth0` line.  Their `br0` carries only `wifi0` (mesh) and `wifi1` (AP), and
they reach the LAN over the mesh.

## Checking it works

**Did the mesh form?**  Each node should list the other two as peers:

<pre class="cli"><code>admin@gw1:/&gt; <b>show interface wifi0</b>
...
mode               : mesh-point
mesh-id            : backhaul
connected peers    : 2</code></pre>

**Is a client associated, and where?**  The AP lists the clients on it.  As
a client roams, it disappears from one node's list and shows up on another:

<pre class="cli"><code>admin@gw2:/&gt; <b>show interface wifi1</b>
...
mode               : access-point
ssid               : campus
connected stations : 1</code></pre>

**Does traffic cross the mesh?**  Attach a client to gw3's AP (the
node with no wire) and reach something on the wired LAN behind gw1.  If
that works, the mesh is carrying the traffic.

## The regression test

To make sure this setup keeps working, the Infix regression test suite
has a test for it, [wifi_mesh_roaming][test].  It sets up three nodes as
above, mesh backhaul on `radio0` and a roaming AP on `radio1`, all bridged,
plus a fourth node as the client.  It then checks that:

- the three nodes form a mesh (each sees its peers);
- a client associates to the `campus` SSID;
- traffic reaches the client across the mesh backhaul;
- when the AP the client is on goes away, the client roams to another node
  carrying the same SSID and mobility domain, and traffic recovers.

![Test topology](/assets/img/wifi-mesh-roaming-test.svg){: width="800" }
_The test topology.  Blue is the mesh backhaul, orange the access points the client roams between._

The test runs on virtual radios, `mac80211_hwsim`, and a small relay,
`wifimedium`, carries frames between the separate guests so they can hear
each other.  On real hardware the radios use the air instead, and the
configuration is the same.

In simulation every radio hears every other at the same strength, so there
is no signal gradient to drift a client from one AP to the next.  The test
forces the move instead.  It takes down the AP the client is on and
confirms the client lands on another one and keeps talking.  That is a
harder test than a slow drift between APs, since the second AP has to
accept the client and the backhaul has to re-converge on its own.

[docs]:    https://kernelkit.org/infix/latest/wifi/
[easy]:    /posts/mt7615-on-the-bpi-r4/
[release]: https://github.com/kernelkit/infix/releases/tag/v26.06.0
[test]:    https://github.com/kernelkit/infix/tree/main/test/case/interfaces/wifi_mesh_roaming
