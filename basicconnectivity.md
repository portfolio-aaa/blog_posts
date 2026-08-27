---
title: Implementing Basic Connectivity in Cisco Packet Tracer — Switches, PCs, and Your First Pings
---

# Implementing Basic Connectivity in Cisco Packet Tracer

This one's a step down in complexity from a routed, dual-LAN topology, and that's the point. Before you touch a router, you need to be fluent in the basics: naming a switch, locking it down, giving it a management IP, and proving that two PCs on the same network can actually talk to each other. This walkthrough covers exactly that — a small flat network with two switches (S1, S2) and two PCs (PC1, PC2), all on a single 192.168.1.0/24 subnet.

## Prerequisites

To follow along, you'll need:

- [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) installed (free with a Networking Academy account).
- Basic familiarity with Cisco IOS command syntax.
- An understanding of IPv4 addressing and subnet masks.

## Scenario

The activity is broken into three parts:

- **Part 1** — basic configuration on both switches: hostname, passwords, encryption, a warning banner, and saving the config.
- **Part 2** — assigning static IP addresses to the two PCs and testing connectivity to the switches.
- **Part 3** — configuring each switch's VLAN1 interface with a management IP address, then verifying full connectivity across the network.

## Addressing Table

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| S1 | VLAN 1 | 192.168.1.253 | 255.255.255.0 |
| S2 | VLAN 1 | 192.168.1.254 | 255.255.255.0 |
| PC1 | NIC | 192.168.1.1 | 255.255.255.0 |
| PC2 | NIC | 192.168.1.2 | 255.255.255.0 |

Everything here lives on the same subnet — there's no router in this topology, so this activity is entirely about Layer 2 switching plus each device's own IP configuration, not routing between networks.

## Step 1 – Set the Hostname on S1

First thing on any device: give it a name that matches its role, so your config and your documentation actually mean something later.

```
Switch>enable
Switch#configure terminal
Switch(config)#hostname S1
S1(config)#
```

## Step 2 – Configure Console and Privileged EXEC Passwords

Next, I locked down local access to the CLI and the privileged (enable) mode.

```
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#enable secret class
```

`enable secret` stores the privileged EXEC password encrypted by default — that's different from the console password above, which sits in plaintext in the running config until you explicitly encrypt it (more on that in a second).

## Step 3 – Verify the Passwords

Before moving on, it's worth confirming both passwords actually took effect rather than just assuming the commands worked.

```
S1#show running-config
```

Looking at the output, the console line should show `password cisco` under `line console 0`, and you should see an `enable secret` line with an encrypted hash rather than plaintext. The real test, though, is functional: log out and back in with `exit`, then supply `cisco` at the console prompt and `class` when you type `enable`. If both prompts accept the password, the configuration is correct — checking the running-config alone only confirms the commands were entered, not that the password actually works as expected.

## Step 4 – Add a Warning Banner and Save the Configuration

With authentication in place, I added a login banner and encrypted any remaining plaintext passwords before saving.

```
S1(config)#service password-encryption
S1(config)#banner motd $ Authorized access only. Violators will be prosecuted to the full extent of the law. $
S1(config)#exit
S1#copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]
```

`copy running-config startup-config` is the command that actually persists your work — the running config lives in RAM and disappears on reload, so skipping this step means redoing everything after a power cycle.

## Step 5 – Repeat for S2

I ran the identical sequence on S2, just swapping the hostname:

```
Switch>enable
Switch#configure terminal
Switch(config)#hostname S2
S2(config)#line console 0
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#enable secret class
S2(config)#service password-encryption
S2(config)#banner motd $ Authorized access only. Violators will be prosecuted to the full extent of the law. $
S2(config)#exit
S2#copy running-config startup-config
```

## Step 6 – Configure the PCs

With both switches named and secured, I moved to the PCs. Each one needed a static IP address and subnet mask entered through its Desktop > IP Configuration screen:

- **PC1**: IP `192.168.1.1`, Subnet Mask `255.255.255.0`
- **PC2**: IP `192.168.1.2`, Subnet Mask `255.255.255.0`

Neither PC needs a default gateway here — there's no router in the topology, so nothing exists for the PCs to route toward.

## Step 7 – Test PC-to-Switch Connectivity

From PC1's command prompt, I pinged S1's VLAN1 address — except at this point, S1 doesn't have one yet, so this ping was expected to fail:

```
PC> ping 192.168.1.253
```

This came back unsuccessful, and that's the correct outcome for where the lab is at this stage. Switches are plug-and-play at Layer 2 — they'll forward frames between ports based on MAC address learning without any IP configuration at all. But an unconfigured switch has no IP interface to answer a ping with, so PC-to-switch connectivity is a different problem than PC-to-PC connectivity through the switch. That's exactly why Part 3 exists.

## Step 8 – Configure the Switch Management Interface

A switch doesn't strictly need an IP address to move traffic between its ports — but without one, you can't manage it remotely, run diagnostics against it, or use it as anything other than a black box you have to plug a console cable into. Giving it an address on VLAN1 is what makes it a manageable device rather than just a dumb forwarder.

```
S1#configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
S1(config)#interface vlan 1
S1(config-if)#ip address 192.168.1.253 255.255.255.0
S1(config-if)#no shutdown
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up
S1(config-if)#exit
```

The `no shutdown` command matters here for the same reason it does on a router interface: VLAN interfaces are administratively down by default, and won't pass traffic — or even show as "up" in interface status — until you explicitly bring them online.

I repeated the same steps on S2:

```
S2#configure terminal
S2(config)#interface vlan 1
S2(config-if)#ip address 192.168.1.254 255.255.255.0
S2(config-if)#no shutdown
S2(config-if)#exit
```

## Step 9 – Verify the IP Configuration

Before saving, I confirmed both interfaces came up as expected:

```
S1#show ip interface brief
S1#show running-config
```

`show ip interface brief` is the faster of the two for a quick sanity check — it lists every interface with its IP address and up/down status in one compact table, rather than scrolling through the full config.

## Step 10 – Save Both Configurations

```
S1#copy running-config startup-config
S2#copy running-config startup-config
```

## Step 11 – Verify Full Network Connectivity

With VLAN1 addressed on both switches, I re-ran the connectivity tests from PC1's command prompt:

```
PC> ping 192.168.1.2
PC> ping 192.168.1.253
PC> ping 192.168.1.254
```

This time, all three pings succeeded. If your very first ping in a batch comes back at 80% rather than 100%, that's normal — the initial ARP request to resolve the destination's MAC address costs one packet, and everything after it succeeds. A second ping to the same address should come back clean at 100%.

## Conclusion

By the end of this activity, both switches were named, password-protected, and reachable at their management addresses, and both PCs could ping each other and both switches successfully. The key takeaways:

- A switch will forward traffic between connected devices with zero configuration — Layer 2 switching doesn't depend on an IP address at all.
- An IP address on VLAN1 exists purely for managing the switch itself (remote access, diagnostics), not for forwarding data between ports.
- `no shutdown` is required on VLAN interfaces just like physical ones — they don't come up on their own.
- `enable secret` and `service password-encryption` protect different things: one is the privileged EXEC password itself, the other scrambles any passwords still stored in plaintext.
- A ping showing less than 100% success on the first attempt usually isn't a fault — it's ARP resolution happening on that first packet.
