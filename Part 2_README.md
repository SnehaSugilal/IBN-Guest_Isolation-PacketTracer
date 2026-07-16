# Part 2 — MAC Learning: Why the Switch Behaved Differently

This is Part 2 of the IBN Guest Isolation lab series.

[← Back to Part 1: IBN Guest Isolation](../README.md)

---

## The Puzzle

In Part 1, Corp-PC1 pinged Guest-PC1. Something interesting happened in the simulation.

The Corp switch flooded the packet to two ports. The Guest switch sent it to exactly one device.

Same network. Same ping. Two completely different forwarding behaviors.

This lab explains why.

---

## The Answer: MAC Learning

A switch doesn't know where anyone lives when it boots up. It learns by listening.

Every frame that arrives teaches it something:
- Source MAC address
- Which port it came from

It writes that down. That's its CAM table. Its map.

**No entry for the destination?** It does the only thing it can — sends the frame out every port and waits to see who answers. That's flooding. Not a bug. Just a switch working with incomplete information.

The moment the destination replies, the switch learns that MAC too. Next time, it delivers directly. No flooding needed.

---

## Why Corp Switch Flooded

Corp switch had never seen Guest-PC1's MAC address. Guest-PC1 lives on a different subnet — it had never sent any L2 traffic through the Corp switch before.

So when the routed packet arrived destined for Guest-PC1, Corp switch didn't know which port to use. It flooded to all ports in VLAN 10.

Corp-PC2 got a copy. Checked the destination IP. Realized it wasn't for it. Dropped it silently.

---

## Why Guest Switch Didn't

By the time the packet reached the Guest switch, Guest-PC1 had already sent traffic earlier in the session — specifically when Guest-PC1 pinged the internet (8.8.8.8).

That outbound traffic taught the Guest switch exactly which port Guest-PC1 was on. MAC learned. Table entry written.

So when the incoming packet arrived, Guest switch looked up the destination MAC, found the entry, and sent it directly. No flooding needed.

**Same network. Different table state. Completely different behavior.**

---

## Standalone Lab: Seeing MAC Learning in Action

This topology isolates MAC learning from routing and VLANs — pure Layer 2 behavior.

![MAC Learning Topology](screenshots/topology.png)

### Devices
- 1× Cisco 2960 switch
- 3× PC (PC-A, PC-B, PC-C)

### IP Plan

| Device | IP Address | Subnet Mask |
|---|---|---|
| PC-A | 192.168.1.10 | 255.255.255.0 |
| PC-B | 192.168.1.20 | 255.255.255.0 |
| PC-C | 192.168.1.30 | 255.255.255.0 |

No gateway needed — same subnet, no routing involved.

---

## What the Simulation Shows

### Act 1 — First ping: PC-A → PC-C

Switch has no entry for PC-C yet. Floods to Fa0/2 (PC-B) and Fa0/3 (PC-C) both.

PC-B gets a copy. Doesn't recognize the destination IP. Drops it.

![Flooding simulation](screenshots/flooding_simulation.png)

### Act 2 — MAC table after the flood

```
Switch# show mac address-table
```

PC-A's MAC now appears on Fa0/1. The switch learned it the moment PC-A's frame arrived.

![MAC table learned](screenshots/mac_table_learned.png)

### Act 3 — Second ping: PC-A → PC-C

PC-C replied in Act 1, teaching the switch PC-C's MAC on Fa0/3. Now the switch delivers directly to Fa0/3 only. PC-B gets nothing.

That contrast — flood vs direct delivery — is the entire story.

---

## One More Thing: Aging Time

MAC table entries aren't permanent. A switch forgets them after a period of inactivity.

Default aging time on Cisco switches: **300 seconds (5 minutes).**

Check it with:
```
Switch# show mac address-table aging-time
```

The learning never really stops.

---

## The Connection Back to Part 1

In the IBN lab, Guest-PC1 had already pinged the internet before Corp-PC1 tried to reach it. That prior traffic built the Guest switch's MAC table entry for Guest-PC1.

Corp switch had no such entry — Guest-PC1 had never sent traffic through the Corp side.

The ACL enforced the policy. But MAC learning explained the forwarding asymmetry. Two different mechanisms. One simulation.

---

## What's Next

Part 3 — MAC flooding weaponizes this exact behavior. An attacker deliberately fills the CAM table until the switch forgets everything it learned. Then the switch floods all traffic everywhere — and the attacker is sitting on one of those ports.

The fix: Port Security and BPDU Guard. Coming soon.

---

## Author

Sneha | Network Engineer | Nokia NRS I | Akamai Certified
[LinkedIn](https://www.linkedin.com/in/) · [GitHub](https://github.com/)
