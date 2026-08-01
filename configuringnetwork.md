---
title: Configuring a Router, a Switch, and IPv6 in Cisco Packet Tracer
---

# Configuring a Router, a Switch, and IPv6 in Cisco Packet Tracer

Cisco Packet Tracer is a network simulation tool that lets you build, configure, and troubleshoot virtual topologies without needing physical routers and switches. It's a staple of the CCNA curriculum, and it's a great way to practice Cisco IOS commands in a low-stakes environment.

In this walkthrough, I'll go through a Skills Integration Challenge that asks you to act as a LAN technician, configuring a router that connects two LANs. The goal is full end-to-end connectivity — IPv4 and IPv6 — between a router, two switches, and their attached hosts.

*Note: only the College router and the Class-B switch are configurable in this activity. The Class-A switch is locked, so any traffic through it depends on getting the router and Class-B switch right.*

## Prerequisites

To follow along, you'll need:

- [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) installed 
- Basic familiarity with Cisco IOS command syntax.
- An understanding of IPv4/IPv6 addressing and subnetting.

## Scenario

My network manager wants me to demonstrate that I can configure a router connecting two LANs. The task list breaks down into four areas:

- Basic device configuration (hostnames, passwords, banners, password encryption).
- IPv4 and IPv6 addressing on the router and Class-B switch, per the Addressing Table.
- Completing IPv4/IPv6 addressing on the PC hosts.
- Verifying and troubleshooting end-to-end connectivity.

## Topology and Addressing Table

The topology is a router  connecting two VLANs — one behind the (locked) **Class-A** switch, one behind **Class-B** — with two student hosts on each side. Here's the Addressing Table for this instance:

| Device | Interface | IPv4 Address | IPv6 Address | Link-Local | Default Gateway |
|---|---|---|---|---|---|
| College | G0/0 | 10.10.10.1 /24 | 2001:DB8:ACAD:100::1 /64 | FE80::2 | N/A |
| College | G0/1 | 10.10.11.1 /24 | 2001:DB8:ACAD:200::1 /64 | FE80::3 | N/A |
| Class-A | VLAN1 | 10.10.10.100 /24 | N/A (not reachable over IPv6) | — | 10.10.10.1 |
| Class-B | VLAN1 | 10.10.11.100 /24 | 2001:DB8:ACAD:200::100 /64 | — | 10.10.11.1 |
| Student-1 | NIC | 10.10.10.101 /24 | 2001:DB8:ACAD:100::50 /64 | — | 10.10.10.1 |
| Student-2 | NIC | 10.10.10.102 /24 | 2001:DB8:ACAD:100::60 /64 | — | 10.10.10.1 |
| Student-3 | NIC | 10.10.11.101 /24 | 2001:DB8:ACAD:200::50 /64 | — | 10.10.11.1 |
| Student-4 | NIC | 10.10.11.102 /24 | 2001:DB8:ACAD:200::60 /64 | — | 10.10.11.1 |


Here is the network topology in Cisco Packet Tracer:
![[Screenshot 2026-08-01 150710.png]]

Student-1 and Student-2 sit behind Class-A on the 10.10.10.0/24 LAN; Student-3 and Student-4 sit behind Class-B on the 10.10.11.0/24 LAN. Note the explicit link-local addresses on the router's interfaces (FE80::2 and FE80::3) — Packet Tracer wants these set manually rather than auto-generated, which is easy to miss.

With that mapped out, let's start configuring.

## Step 1 – Configure Basic Router Settings

First, I named the router and locked down access with the required passwords and encryption with no plain text passwords. 

```
Router>enable
Router#configure terminal
Router(config)#hostname College
College(config)#enable secret class
College(config)#line console 0
College(config-line)#password cisco
College(config-line)#login
College(config-line)#exit
College(config)#line vty 0 15
College(config-line)#password cisco
College(config-line)#login
College(config-line)#exit
College(config)#service password-encryption
College(config)#banner motd $ Authorized Access Only $
```

A few things worth noting:

- `enable secret` is already stored encrypted, but `service password-encryption` is still needed to scramble the console/VTY passwords, which are stored in plaintext by default.
-For the banner, configure something reasonable, like a warning against unauthorized access.

## Step 2 – Configure Router Interfaces (IPv4 and IPv6)

Next, I addressed both router interfaces and added descriptions so the config is self-documenting.

```
College(config)#interface gigabitEthernet 0/0
College(config-if)#description Link to Class-A LAN
College(config-if)#ip address 10.10.10.1 255.255.255.0
College(config-if)#ipv6 address FE80::2 link-local
College(config-if)#ipv6 address 2001:DB8:ACAD:100::1/64
College(config-if)#no shutdown
College(config-if)#exit

College(config)#interface gigabitEthernet 0/1
College(config-if)#description Link to Class-B LAN
College(config-if)#ip address 10.10.11.1 255.255.255.0
College(config-if)#ipv6 address FE80::3 link-local
College(config-if)#ipv6 address 2001:DB8:ACAD:200::1/64
College(config-if)#no shutdown
College(config-if)#exit

College(config)#ipv6 unicast-routing
```

Note: 
Two easy things to miss here: the explicit `ipv6 address FE80::x link-local` command (the Addressing Table calls for specific link-local addresses, not whatever the router auto-generates), and `ipv6 unicast-routing` at the bottom — without it, the router won't forward IPv6 packets between interfaces at all, even if every address is configured correctly.

## Step 3 – Configure the Class-B Switch

Switches don't route between subnets, but they still need management addressing, security basics, and a description on VLAN1 so it's clear what the interface is for.

```
Switch>enable
Switch#configure terminal
Switch(config)#hostname Class-B
Class-B(config)#enable secret class
Class-B(config)#line console 0
Class-B(config-line)#password cisco
Class-B(config-line)#login
Class-B(config-line)#exit
Class-B(config)#line vty 0 15
Class-B(config-line)#password cisco
Class-B(config-line)#login
Class-B(config-line)#exit
Class-B(config)#service password-encryption

Class-B(config)#interface vlan 1
Class-B(config-if)#description Class-B Management Interface
Class-B(config-if)#ip address 10.10.11.100 255.255.255.0
Class-B(config-if)#ipv6 address 2001:DB8:ACAD:200::100/64
Class-B(config-if)#no shutdown
Class-B(config-if)#exit
Class-B(config)#ip default-gateway 10.10.11.1
```

The Class-A switch is locked in this activity, so I couldn't touch it directly — but its VLAN1 address (10.10.10.100) is already set, and per the lab notes it isn't reachable over IPv6 at all, only IPv4.

Note the `ip default-gateway` command — a switch needs this for IPv4 management traffic to leave the local subnet, since VLAN1 is just a Layer 3 interface for management, not a routed interface.

## Step 4 – Complete Host Addressing

The four student PCs came partially configured, so I filled in the missing IPv4 details and configured IPv6 from scratch on each — IP address, default gateway, and (for IPv6) the router's global unicast address as the gateway, since these hosts don't do SLAAC/DHCPv6 automatically in this activity.

Student-1 and Student-2 sit on the Class-A LAN, so both point to the College router's G0/0 address as their gateway:

- **Student-1**: IPv4 `10.10.10.101 /24`, Gateway `10.10.10.1` — IPv6 `2001:DB8:ACAD:100::50/64`, Gateway `2001:DB8:ACAD:100::1`
- **Student-2**: IPv4 `10.10.10.102 /24`, Gateway `10.10.10.1` — IPv6 `2001:DB8:ACAD:100::60/64`, Gateway `2001:DB8:ACAD:100::1`

Student-3 and Student-4 sit on the Class-B LAN, pointing to G0/1:

- **Student-3**: IPv4 `10.10.11.101 /24`, Gateway `10.10.11.1` — IPv6 `2001:DB8:ACAD:200::50/64`, Gateway `2001:DB8:ACAD:200::1`
- **Student-4**: IPv4 `10.10.11.102 /24`, Gateway `10.10.11.1` — IPv6 `2001:DB8:ACAD:200::60/64`, Gateway `2001:DB8:ACAD:200::1`

## Step 5 – Save the Configuration

 I saved the running config to NVRAM on both the router and the switch:

```
College#copy running-config startup-config
Class-B#copy running-config startup-config
```

## Step 6 – Verify Connectivity

With everything configured, I ran pings across the topology to confirm end-to-end reachability, both IPv4 and IPv6.

```
Student-1> ping 10.10.11.101
Student-1> ping 2001:DB8:ACAD:200::50

Class-B# ping 10.10.10.1
Class-B# ping 2001:DB8:ACAD:100::1
```

My first round of pings from Student-1 across to Student-3 failed over IPv6, which pointed me toward troubleshooting.

## Step 7 – Troubleshoot the IPv6 Issue

The IPv4 pings all succeeded, so I knew basic connectivity and routing were fine — the issue was isolated to IPv6. Checking the router config, I found the missing piece:

```
College#show ipv6 interface brief
```

The interfaces had addresses assigned, but IPv6 forwarding between them wasn't enabled. Adding `ipv6 unicast-routing` in global configuration mode (Step 2 above) resolved it — after that, `show ipv6 route` showed both connected LAN prefixes, and pings across the router succeeded in both directions.

That's the value of the "documentation" requirement in this lab: writing down exactly what wasn't working and why makes the fix (and the write-up) much faster.

## Conclusion

By the end of this activity, all devices — the College router, the Class-B switch, and all four student PCs — could reach each other over IPv4 and IPv6. The main lessons from this lab:

- `service password-encryption` and `enable secret` cover different passwords — you need both for a fully encrypted config.
- IPv4 routing between directly connected interfaces works as soon as you bring the interfaces up, but IPv6 forwarding needs `ipv6 unicast-routing` explicitly enabled on the router.
- Switches need `ip default-gateway` for their management interface to reach devices off the local subnet — a routed interface would use `ip route` instead, but VLAN1 on a switch is not that.
- Interface descriptions aren't just a formality — they're what makes a topology like this readable six months later, or to the next tech who has to troubleshoot it.
