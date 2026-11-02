# Packet Tracer: Examine the ARP Table

A write-up of how I used Cisco Packet Tracer in Simulation mode to watch ARP at work. I captured an ARP request and reply, checked the MAC address tables on two switches, and then looked at how ARP behaves when a host talks to a device on a different network.

## Goals

- Examine an ARP request
- Examine a switch MAC address table
- Examine the ARP process in remote communications

## Addressing table

| Device | Interface | MAC Address | Switch Interface |
|---|---|---|---|
| Router0 | G0/0 | 0001.6458.2501 | G0/1 |
| Router0 | S0/0/0 | N/A | N/A |
| Router1 | G0/0 | 00E0.F7B1.8901 | G0/1 |
| Router1 | S0/0/0 | N/A | N/A |
| 10.10.10.2 | Wireless | 0060.2F84.4AB6 | F0/2 |
| 10.10.10.3 | Wireless | 0060.4706.572B | F0/2 |
| 172.16.31.2 | F0 | 000C.85CC.1DA7 | F0/1 |
| 172.16.31.3 | F0 | 0060.7036.2849 | F0/2 |
| 172.16.31.4 | G0 | 0002.1640.8D75 | F0/3 |

## Part 1: Examine an ARP request

### Step 1: Generating the ARP request

1. I clicked **172.16.31.2** and opened **Command Prompt**.
2. I cleared the ARP table so the PC would have to ask again:

   ```text
   PC> arp -d
   ```

3. I switched to **Simulation** mode and ran:

   ```text
   PC> ping 172.16.31.3
   ```

   Two PDUs appeared. The ping can't build its ICMP packet without the destination's MAC address, so the PC sent an ARP broadcast frame first.

4. I clicked **Capture/Forward** once. The ARP PDU moved to Switch1 and the ICMP PDU disappeared while it waited for the ARP reply. I opened the ARP PDU and read the destination MAC address.

| Question | Answer |
|---|---|
| Is the destination MAC in the addressing table? | **No.** It's `FFFF.FFFF.FFFF`, the broadcast address, which belongs to no device. |
| How many copies of the PDU did Switch1 make? | **3.** The switch flooded the broadcast out every port except the one it arrived on. |
| IP address of the device that accepted the PDU? | **172.16.31.3.** The other devices dropped it. |

5. I opened the PDU and looked at Layer 2.

| Question | Answer |
|---|---|
| What happened to the source and destination MAC addresses? | They **swapped roles** in the reply. The source became 172.16.31.3's MAC (`0060.7036.2849`) and the destination became 172.16.31.2's MAC (`000C.85CC.1DA7`), so the frame is now unicast. |

6. I kept clicking **Capture/Forward** until the PDU got back to 172.16.31.2.

| Question | Answer |
|---|---|
| How many copies did the switch make during the ARP reply? | **1.** The reply is unicast, and the switch already knew which port the destination was on. |

### Step 2: Examining the ARP table

1. The ICMP packet reappeared. I opened it and compared its MAC addresses.

| Question | Answer |
|---|---|
| Do the source and destination MACs align with their IP addresses? | **Yes.** Source is 172.16.31.2's MAC and destination is 172.16.31.3's MAC. |

2. I switched back to **Realtime** and the ping completed.
3. On 172.16.31.2 I ran:

   ```text
   PC> arp -a
   ```

| Question | Answer |
|---|---|
| Which IP does the MAC entry belong to? | **172.16.31.3** (`0060.7036.2849`). |
| When does an end device issue an ARP request? | When it needs the MAC address of a destination on its own network and doesn't already have it in its ARP table. |

## Part 2: Examine a switch MAC address table

### Step 1: Generating more traffic

1. From 172.16.31.2 I ran `ping 172.16.31.4`.
2. I clicked **10.10.10.2**, opened its Command Prompt, and ran `ping 10.10.10.3`.

| Question | Answer |
|---|---|
| How many replies were sent and received? | **4 replies** for each ping. The first request can time out while ARP resolves, so check the count in your own output. |

### Step 2: Checking the switches

On each switch I opened the **CLI** tab and ran:

```text
Switch# show mac-address-table
```

| Question | Answer |
|---|---|
| Switch1: do the entries match the table? | **Yes.** 172.16.31.2, .3, and .4 show on F0/1, F0/2, and F0/3, and Router1's `00E0.F7B1.8901` shows on G0/1. |
| Switch0: do the entries match the table? | **Yes.** The two wireless MACs show on F0/2 and Router0's `0001.6458.2501` shows on G0/1. |
| Why are two MAC addresses on one port? | Both wireless devices connect through the same access point, which is plugged into F0/2. The switch sees both of their MACs arriving on that one port. |

## Part 3: Examine ARP in remote communications

### Step 1: Generating the ARP traffic

1. On 172.16.31.2 I opened Command Prompt and ran `ping 10.10.10.1`, then `arp -a`.

| Question | Answer |
|---|---|
| IP address of the new ARP table entry? | **172.16.31.1**, the default gateway (Router1's G0/0 interface). |

2. I ran `arp -d` to clear the table, switched to **Simulation** mode, and repeated `ping 10.10.10.1`.

| Question | Answer |
|---|---|
| How many PDUs appeared? | **2**, one ARP and one ICMP. |

3. I clicked **Capture/Forward** and opened the PDU now at Switch1.

| Question | Answer |
|---|---|
| Target destination IP of the ARP request? | **172.16.31.1.** |
| Why isn't it 10.10.10.1? | `10.10.10.1` is on a **different network**. The PC can't ARP for it directly, so it sends the packet to its default gateway and ARPs for the gateway's MAC instead. |

### Step 2: Examining Router1

1. I switched to **Realtime**, opened Router1's **CLI** tab, entered privileged EXEC mode, and ran `show mac-address-table`.

| Question | Answer |
|---|---|
| How many MAC addresses are in the table? Why? | **None.** Routers don't keep a MAC address table because that is a switch feature. They use an ARP table instead. |

2. I ran `show arp`.

| Question | Answer |
|---|---|
| Is there an entry for 172.16.31.2? | **Yes.** Router1 learned 172.16.31.2's MAC from the ARP request the PC sent for the gateway. |
| What happens to the first ping when the router responds to the ARP request? | The first ping typically **times out or is delayed** while ARP completes. After the entries are in place, the remaining pings succeed. |

## What I took away

- ARP requests are broadcasts (`FFFF.FFFF.FFFF`), so the switch floods them. ARP replies are unicast, so the switch sends them to one port.
- A switch builds its MAC address table from the source MACs it sees, so two MACs can show on one port when an access point sits behind it.
- For a destination on another network, a host ARPs for its **default gateway**, not for the remote device.
- Routers use ARP tables, and switches use MAC address tables.
