# Packet Tracer: Examine NAT on a Wireless Router

A write-up of how I completed the Cisco Packet Tracer lab on NAT with a wireless router. I connected four PCs using DHCP, looked at the router's internal and external addressing, and used Simulation mode to watch NAT change the source IP on packets going out to a web server.

## Goals

- Examine the NAT configuration on a wireless router
- Connect 4 PCs to the router using DHCP
- Examine traffic that crosses the network using NAT

## What I did

### Part 1: Checking the external (Internet) side

I added one PC and connected it to the wireless router with a straight-through cable. I waited for the link lights to turn green, then went to **Desktop > IP Configuration** and switched the PC to **DHCP** so it got an address from the router.

From there I wrote down the **default gateway** address, typed it into the PC's web browser, and logged into the router's admin page with `admin` / `admin`. Under **Status > Router**, I scrolled to the **Internet Connection** section to see the address the ISP gave the router's Internet port.

The first time I looked it showed `0.0.0.0`. The router was still getting an address from the ISP's DHCP server, so I closed the window, waited a few seconds, and reopened it.

**Result:** the Internet port address is a **public** address. It comes from the ISP and is not in a private range.

### Part 2: Checking the internal (LAN) side

On the same Status page I opened **Local Network**. I looked at the router's LAN address and then scrolled down to the **DHCP server** section to see the range of addresses it hands out to connected hosts.

**Result:** these are **private** addresses (RFC 1918, `192.168.x.x` range). They can't be routed across the Internet, which is why NAT is needed.

### Part 3: Adding the other three PCs

I added three more PCs, connected each to the router with a straight-through cable, and again waited for green link lights. On each one I set the IP configuration to **DHCP**, then opened **Command Prompt** and ran `ipconfig /all` to confirm the settings.

All four PCs received private addresses from the router's DHCP range, with the router's LAN IP as their default gateway.

### Part 4: Watching NAT in Simulation mode

I switched to **Simulation** mode and set up the traffic I wanted to see:

1. Clicked **Show All/None** to hide all events.
2. Opened **Edit Filters > Misc** and checked only **TCP** and **HTTP**.
3. Added a **Complex PDU** (open envelope icon) and picked one of the PCs as the source.

Complex PDU settings I used:

| Setting | Value |
|---|---|
| Application | HTTP |
| Destination | `ciscolearn.nat.com` server |
| Source Port | 1000 |
| Simulation Setting | Periodic |
| Interval | 120 seconds |

After clicking **Create PDU**, I unlocked the simulation panel by double-clicking it so I could move it and see the whole topology, then pressed **Play** and sped up the animation with the slider. When the **Buffer Full** message appeared I clicked **View Previous Events**.

### Part 5: Looking at the packet headers

I double-clicked the **3rd event** in the list, then clicked the envelope that appeared in the workspace to open the PDU details. I compared the **Inbound PDU Details** and **Outbound PDU Details** tabs and looked at the source and destination IPs in each.

What I found:

| | Inbound PDU | Outbound PDU |
|---|---|---|
| **Source IP** | PC's private address | Router's public (ISP) address |
| **Destination IP** | Web server | Web server (unchanged) |

The source IP changed between inbound and outbound, which is NAT replacing the PC's private address with the router's public one. I clicked through a few other events to see the headers at other points, then used **Check Results** to confirm the activity was complete.

## My addresses

| Item | Value |
|---|---|
| Router Internet IP (public) | `47.145.214.53` |
| Router LAN IP / default gateway | `10.0.2.1` |
| DHCP range | `_10.0.2.1 - 10.0.2.254` |
| PC1 | `10.0.2.4' |
| PC2 | `10.0.2.5` |
| PC3 | `_10.0.2.6` |
| PC4 | `10.0.2.7` |
