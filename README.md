# DNS-DHCP-configuration-
Packet Tracer lab configuring DNS and DHCP services from scratch — custom domain resolution and a 30-address DHCP scope, verified via browser test and client lease assignment.

# DNS and DHCP Configuration Lab (Packet Tracer: Standing Up Name Resolution and Dynamic Addressing)

`Cisco Packet Tracer` · `DNS` · `DHCP` · `Network Services` · `Client Configuration`

## Overview
This lab focused on **standing up two core network services from scratch** in Cisco Packet Tracer: a DNS server resolving a custom domain, and a DHCP server handing out addresses to clients automatically. A single switch (`2960-24TT`) ties together a DNS server, a DHCP server, an "any access" client, and a dedicated DHCP client, and the goal was to get all four talking correctly without any static addressing on the client side.

The point of this one wasn't just clicking through server config pages — it was making sure I understood *why* each setting exists: why a DNS record needs to point to the right IP before a domain will resolve, and why a DHCP scope has to be sized and excluded correctly before a client can actually pull a lease.

## Objective
Configure a DNS server to resolve `cisco.com`, configure a DHCP server with a scope large enough for the network, and confirm both services work end-to-end: a browser test that resolves the domain through DNS, and a client that successfully obtains an IP lease through DHCP.

## Environment
- **Switch:** `2960-24TT` (`Switch0`), four active ports connecting the DNS server, DHCP server, and two end devices
- **Devices:**
  | Device | Port | Role |
  |--------|------|------|
  | Server-PT DNS | Fa0/2 | DNS server, domain `cisco.com` → `192.168.8.2` |
  | Server-PT DHCP | Fa0/4 | DHCP server at `192.168.8.3` |
  | Laptop-PT (any access client) | Fa0/3 | Client used to test browser/DNS resolution |
  | Laptop-PT DHCP Client | Fa0/1 | Client configured for DHCP, used to confirm lease assignment |
- **Lab platform:** Cisco Packet Tracer, Realtime mode

## Tools I Used

| Tool | What It Does | Why I Used It |
|------|--------------|----------------|
| **Cisco Packet Tracer** | Network simulation | Built the topology and configured the DNS/DHCP services directly on the simulated servers |
| **Packet Tracer's Web Browser (client desktop app)** | Simulated HTTP client | Used to confirm `cisco.com` actually resolves and loads through the DNS server |
| **Packet Tracer's IP Configuration (client desktop app)** | DHCP client interface | Used to confirm a client set to "DHCP" actually receives a lease from the server instead of sitting unaddressed |

## What I Did

### Setting Up DNS
1. Opened the DNS server's config page and added an A record mapping the domain `cisco.com` to `192.168.8.2`, the server's own address.
2. Made sure the DNS service was toggled on — a record with the service off silently does nothing, which is an easy thing to overlook the first time through.
3. Verified the record saved correctly by checking it back in the server's DNS table.

### Setting Up DHCP
1. Opened the DHCP server's config page and set its own static IP to `192.168.8.3`, since a DHCP server needs a fixed address of its own before it can hand out addresses to anyone else.
2. Configured the pool with a default gateway and DNS server address so clients would get more than just an IP — they'd also know how to route off-network and where to send DNS queries.
3. Set the scope to allow up to **30 IP addresses**, sized to comfortably cover the clients on this network with room to grow, and excluded the addresses already in use by the DNS and DHCP servers themselves so the pool wouldn't try to hand those out to a client.
4. Enabled the DHCP service.

### Testing DNS Resolution
1. On the "any access" client, opened the web browser and typed `cisco.com` instead of an IP address.
2. Confirmed the page loaded, which meant the client's DNS query was reaching the DNS server, resolving `cisco.com` to `192.168.8.2`, and successfully connecting — proof the DNS record and the client's DNS settings were both correct.

### Testing DHCP
1. On the DHCP client laptop, opened IP Configuration and switched addressing from static to DHCP.
2. Confirmed the client picked up an IP address, subnet mask, default gateway, and DNS server automatically from the pool, with no manual entry.
3. Used this as confirmation that the scope, exclusions, and gateway/DNS options on the server side were all configured correctly — a client that gets a full, correct configuration automatically is the real test of a DHCP setup, not just whether the service is "on."

## What's in This Repo

```
dns-dhcp-lab/
├── README.md                     # This file
└── screenshots/
    ├── 01-topology-overview.png   # Full topology with device roles
    ├── 02-dns-server-config.png   # DNS record for cisco.com
    ├── 03-dhcp-server-config.png  # DHCP pool and scope settings
    ├── 04-browser-test.png        # Successful cisco.com resolution
    └── 05-dhcp-client-lease.png   # Client with DHCP-assigned IP
```

## Skills I Picked Up
- **DNS and DHCP depend on each other more than it first appears.** A DHCP scope that doesn't hand out the right DNS server address will give a client an IP but leave it unable to resolve anything, even if the DNS server itself is configured perfectly.
- **A service being configured isn't the same as a service being enabled.** Both the DNS record and the DHCP scope needed their services explicitly turned on before either would do anything, which is an easy step to miss.
- **Sizing a DHCP scope isn't just "big enough for today."** Scoping for 30 addresses on a small network means the next handful of devices can join without anyone having to come back and re-touch the pool.
- **Testing by symptom, not by config page.** Confirming success meant actually resolving `cisco.com` in a browser and watching a client pull a real lease — not just assuming a saved config would work.

## How This Applies in the Real World
DNS and DHCP are two of the most common "invisible until broken" services on any network. Almost nobody thinks about them until a client can't get an address or a site won't resolve, and by then it's usually one of a small number of causes: a service that's configured but not enabled, a scope that's too small or missing an exclusion, or a DNS record that's stale or pointed at the wrong host. Being able to walk through both from scratch, and verify each one from the client side instead of just trusting the server config, is exactly the kind of troubleshooting instinct that matters on a real help desk or network team.

## Where I'm Coming From
I'm making the jump into cybersecurity from a background in **healthcare**. It's a different field on paper, but a lot of the muscle memory carries over: following procedures carefully, protecting sensitive information, and staying calm and methodical when something isn't configured the way it's supposed to be. I'm currently studying for **CompTIA Security+** and building labs like this one to get real hands-on reps in with the underlying network services that security work sits on top of.

## What I Want to Learn Next
- Adding a second DHCP scope on a different subnet and practicing DHCP relay/IP helper so a single server can serve multiple networks
- Setting up conditional or split DNS, where internal and external clients resolve the same domain differently
- Practicing DNS troubleshooting from the client side — reading `ipconfig /all`-style output to figure out whether a failure is DNS, DHCP, or routing
- Layering security onto this setup: DHCP snooping, restricting who can query the DNS server, and thinking about how a rogue DHCP server would be detected

## Limitations & What I'd Do Differently in Production
- **This lab used a flat, single-switch topology with no router**, so all traffic stayed on one broadcast domain. A production network would typically have DNS and DHCP serving multiple VLANs or subnets, which changes how scopes and relay are configured.
- **The DHCP scope was sized for this lab's device count, not for real growth planning.** In production, scope sizing should account for expected growth and be documented, not just set to "comfortably more than what's plugged in today."
- **No redundancy was built in.** A single DNS or DHCP server is a single point of failure; a real deployment would typically use secondary/failover DHCP and multiple DNS servers.

## References
- [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer)
- [CompTIA Security+ (SY0-701) Exam Objectives](https://www.comptia.org/certifications/security)
