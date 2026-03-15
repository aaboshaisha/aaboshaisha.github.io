---
layout: post
title: "The Life of an Internet Packet"
date: 2026-03-15
---

# The life of a packet

What happens when one types `www.wikipedia.org` in the browser?

First, `www.wikipedia.org` has to be translated to an IP address. This is done via the DNS system ("phone book" of the internet — translating human-readable names like `www.wikipedia.org` into IP addresses).

Before DNS, there was a central text file called `HOSTS.TXT` that mapped hostnames to IP addresses. As the network grew, this system didn't scale well, so the distributed Domain Name System was introduced.

DNS today runs on servers that you query over the network. It could be a dedicated DNS server run by your ISP or a public one like Google's (`8.8.8.8`) or Cloudflare's (`1.1.1.1`).

Who constructs the DNS query?

> The browser asks the operating system's DNS resolver for the address. The resolver then constructs the DNS query. DNS is an application layer protocol.

So, now the resolver will construct a DNS query (some bytes) to send over the wires.

```
+----------------------------+
|        DNS Header          |
|  ID: 0x1234  (query ID)    |
|  Flags: standard query     |
+----------------------------+
|        Question            |
|  Name:  www.wikipedia.org  |
|  Type:  A (IPv4 address)   |
|  Class: IN (internet)      |
+----------------------------+
```

Now, this data needs to find its way to the host machine. How?

We know that IP is responsible for finding the way over the network but once we're at the host machine, it will be running hundreds of programs. How does the OS know which program should receive the incoming packet?

This is where Transport protocols like UDP or TCP come in. They include `src` and `dst` ports in their headers so the OS knows which application the packet belongs to.

Who is responsible for prepending UDP or TCP headers?

> The OS kernel.

This is done via the socket interface.

*(the **application** makes calls to the socket API, and the **OS kernel** does the actual header construction. The socket is the interface between the two.)*

Now that we took care of this, the next step is to find how to get to our destination. This is where IP comes in. Again, the OS prepends the necessary IP headers, which includes:

* **Source IP**: your machine's IP address
* **Destination IP**: `8.8.8.8` (Google's DNS server)

So now the packet looks something like:

```
+----------------------------+
|        IP Header           |
|  Src IP:  192.168.1.5      |
|  Dst IP:  8.8.8.8          |
+----------------------------+
|        UDP Header          |
|  Src Port: (random)        |
|  Dst Port: 53 (DNS)        |
+----------------------------+
|        DNS Query           |
|  www.wikipedia.org         |
+----------------------------+
```

Where do we go from here?

Suppose I'm doing this from my own computer at home. I get the internet at home via my ISP through a router. The computer is configured to send DNS queries to a specific DNS resolver (often provided by the ISP or a public DNS service).

So the logical next step is to get the packet from my computer to the router (the **default gateway**).

So, how does my computer know how to talk to / find the router?

Each machine with a network connection has a network card / network interface controller (NIC) and this will have a unique address called a MAC address (media access control address). MAC addresses are 6 bytes long and look like `aa:bb:cc:dd:ee:ff`. They are designed to be globally unique identifiers assigned to network interfaces.

At this stage, the network card will prepend the Ethernet header with things like `src-mac` and `dst-mac`.

But how does my computer know the router's MAC address?

My computer knows the router's IP (it's the default gateway), but Ethernet needs a MAC address. So the computer:

1. Broadcasts to the local network:
   "Who has IP `192.168.1.1`? Tell me your MAC"

2. Router replies:
   "That's me! My MAC is `aa:bb:cc:dd:ee:ff`"

3. Computer now can fill the Ethernet header.

This process is called **ARP (Address Resolution Protocol)**.

So now the packet looks like:

```
+----------------------------+
|      Ethernet Header       |
|  Src MAC: your NIC         |
|  Dst MAC: router's MAC     |
+----------------------------+
|        IP Header           |
|  Src IP:  192.168.1.5      |
|  Dst IP:  8.8.8.8          |
+----------------------------+
|        UDP Header          |
|  Dst Port: 53 (DNS)        |
+----------------------------+
|        DNS Query           |
+----------------------------+
```

Now our packet is at the router. The router removes the Ethernet frame and processes the IP packet.

The router:

1. Removes the Ethernet header (it has done its job)
2. Looks at the **destination IP** (`8.8.8.8`)
3. Consults its **routing table** to find the next hop
4. Wraps the packet in a **new link-layer frame** (often Ethernet) with the next hop's MAC address

This repeats at every router along the way — the IP packet mostly stays the same (except fields like TTL), but the link-layer header gets replaced at each hop.

When the packet finally arrives at the DNS server, the reverse "unpacking" process takes place. The DNS server processes the query and sends back a DNS response with Wikipedia's IP address.

---

# Side note: Why we need IPs and MAC addresses

Reasons are historical and related to the **layered design** of network protocols.

Local networks (LANs) like Ethernet used MAC addresses to identify devices and deliver frames within the same physical network. Network switches use these MAC addresses to forward frames to the correct device.

When the internet was designed, the goal was to connect many different kinds of networks together. The Internet Protocol (IP) was designed as a **network-layer protocol** that could operate above many different link-layer technologies (Ethernet, Wi-Fi, etc.).

So Ethernet and MAC addresses continue to operate at the **link layer**, while IP operates at the **network layer**, allowing packets to move across many different networks.

---

# Side note: public and private IP addresses

Remember IPv4. It has 4 bytes so a maximum of about **4.3 billion addresses**. There are many more devices than this.

How can each device get its own IP?

Think of this: you get the internet at your home via your ISP through a router. To the outside world, all the machines connected to the internet at your home appear to share the same **public IP address**.

Each device on the local network instead has a **private IP address** (like `192.168.x.x`).

The router uses **NAT (Network Address Translation)** to translate between private internal addresses and the single public IP. It usually also uses **port translation**, allowing many internal connections to share the same public address.

---

MAC addresses are **flat** — they don't contain information about where a device is in the network.

IP addresses, on the other hand, are **hierarchical** and work with routing prefixes (like `192.168.1.0/24`) so routers can make scalable routing decisions.

So the reason we use private IPs locally (rather than MACs) is that **IP routing logic works the same way** whether you're on a local network or the global internet — it's a consistent system all the way through.

Using MACs for large-scale routing would not scale well across many networks.

---

Within your home network, Ethernet + MAC addresses handle the **final delivery** of frames.

But your computer still needs IP addresses to know:

* **which device is the gateway** (router) to the internet
* how to send packets to external networks
* how applications communicate across networks

Your computer learns the router's **IP address** (the default gateway), and ARP then translates that IP → MAC for the Ethernet step.

Private IPs also allow the router to **manage the local network**, assign addresses via DHCP, and track connections.

So the short answer:

**MAC addresses handle local delivery, while IP addresses handle logical routing across networks. They work together rather than replacing each other.**
