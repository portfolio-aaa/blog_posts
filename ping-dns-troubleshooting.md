# Packet Tracer: Use the Ping Command to Find a Misconfigured PC

How I used `ping` in Cisco Packet Tracer to figure out why some PCs on a small business network couldn't reach a website. Every PC uses static IP addressing, and the cause turned out to be a wrong DNS server setting on the PCs that couldn't connect.

## The problem

A small business owner said some users couldn't open `www.cisco.pka`. My job was to use `ping` to find out why.

## How I did it

### 1. Testing each PC in the browser

I went to **Desktop > Web Browser** on every PC and entered `www.cisco.pka`. I gave the network about a minute to finish booting before testing, since the devices need time to start up.

Some PCs loaded the page and some didn't. The PCs that failed were: **PC 2**.

### 2. Pinging the site by name from a broken PC

On PC 2 I opened **Command Prompt** and ran:

```text
C:\> ping www.cisco.pka
```

The ping did **not** get a reply. The PC couldn't turn the name into an IP address, so there was no IP in the output.

### 3. Pinging the site by name from a working PC

I ran the same command on a PC that loaded the site fine:

```text
C:\> ping www.cisco.pka
```

This one **did** reply, and the reply showed the web server's IP address: **`<web server IP>`**. I used that address in the next step.

### 4. Pinging the server by IP from the broken PC

Back on the failing PC, I pinged the IP directly instead of the name:

```text
C:\> ping 192.15.2.10
```

This **worked**. So the PC could reach the server over the network and the only thing failing was name resolution. That pointed to a DNS problem on the PC, not a cable or IP addressing problem.

### 5. Comparing DNS settings

I ran `ipconfig /all` on a working PC and on a broken PC and compared the **DNS Servers** line:

```text
C:\> ipconfig /all
```

| | DNS server |
|---|---|
| Working PC | `192.15.2.5>` |
| Broken PC | `No DNS server displayed` |

The two **did not match**. The broken PCs were pointed at the wrong DNS server.

### 6. Fixing it

On each broken PC I went to **Desktop > IP Configuration** and changed the DNS server to the same address the working PCs use. Then I opened the **Web Browser** and loaded `www.cisco.pka` again. It came up, so the fix worked.


## What I took away

- If a site fails by name but pings by IP, the problem is DNS.
- Pinging the name and then the IP quickly separates a DNS problem from a connectivity problem.
- Comparing `ipconfig /all` on a working PC and a broken one shows the bad setting right away.


