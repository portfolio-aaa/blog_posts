# Packet Tracer: Connect to a Web Server

A write-up of how I used Cisco Packet Tracer to watch a client reach a web server across a network using IP addresses. I first tested connectivity with `ping`, then loaded the server's web page in a browser.

## Goal

Observe how packets are sent across the Internet using IP addresses.

## What I did

### Part 1: Verifying connectivity to the web server

1. I selected **PC0** and opened **Desktop > Command Prompt**.
2. I pinged the web server's IP address:

   ```text
   PC> ping 172.33.100.50
   ```

3. Output:

   ```text
   Pinging 172.33.100.50 with 32 bytes of data:

   Reply from 172.33.100.50: bytes=32 time=0ms TTL=127
   Reply from 172.33.100.50: bytes=32 time=0ms TTL=127
   Reply from 172.33.100.50: bytes=32 time=0ms TTL=127

   Ping statistics for 172.33.100.50:
       Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
   ```

   Replies from the server confirmed the client could reach it. One packet was lost, which is normal on the first ping. The first request can time out while the devices finish loading and ARP resolves the next hop's MAC address.

4. I closed only the command prompt window and left the PC0 window open.

### Part 2: Loading the web page

1. In PC0's **Desktop** tab I opened **Web Browser**.
2. I entered `172.33.100.50` in the URL field and clicked **Go**.
3. The browser connected to the server by IP address and opened the page.

**Question: What messages did you see after the web page finished loading?**

The page displayed **"Welcome to the Learn IP Web Site"**.

## Summary

| Step | Result |
|---|---|
| Ping `172.33.100.50` from PC0 | Replies received (3 of 4, first request timed out for ARP) |
| Browse to `http://172.33.100.50` | "Welcome to the Learn IP Web Site" page loaded |

## What I took away

- A successful ping shows the client has a working path to the server before I try the web page.
- The first ping can fail while ARP completes, so a single lost packet at the start isn't a problem.
- A web browser can reach a server by IP address alone, with no domain name or DNS needed.


