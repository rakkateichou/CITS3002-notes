# Network Layer (Part 2) - Addressing, Routing & IP

## 1. Forwarding vs. Routing
*   **Forwarding (The "Doer"):** Moves packets from a router's input port to the correct output port. Happens at extremely short timescales (nanoseconds) and is implemented in **hardware**.
*   **Routing (The "Thinker"):** The background algorithms that determine the end-to-end paths packets take. Happens at longer timescales (seconds) and is implemented in **software**.

## 2. IP Addressing & CIDR
An IP address (IPv4) is a 32-bit identifier for host/router interfaces.
*   *Note:* Switches and Hubs operate at Layer 2/Layer 1, so they do **not** have IP addresses!
*   **Classful Addressing (The Old Way):** Divided IPs into rigid classes (Class A = /8, Class B = /16, Class C = /24). This caused rapid address exhaustion because Class B was too large for most companies, but Class C was too small.
*   **CIDR (Classless Inter-Domain Routing - The New Way):** Allows subnet masks of *any* arbitrary length.
    *   Format: `a.b.c.d/x` (where `x` is the number of bits in the subnet portion).
    *   **Subnet part:** The high-order (leftmost) bits.
    *   **Host part:** The low-order (rightmost) bits.

### The Subnetting Math Rules (Guaranteed Exam Topic)
If you are given `200.23.16.130/23`:
1.  **Subnet Mask:** The first 23 bits are `1`s, the remaining 9 bits are `0`s. 
    `11111111.11111111.11111110.00000000` = `255.255.254.0`
2.  **Network Address:** Perform a Bitwise AND between the IP and the Subnet Mask. (This zeroes out the host bits).
3.  **Total IPs:** If there are $h$ host bits (e.g., $32 - 23 = 9$), Total IPs = **$2^h$** (e.g., $2^9 = 512$).
4.  **Usable (Valid) Hosts:** Always **$2^h - 2$**. You must subtract 2 because:
    *   All `0`s in the host part = **Network Address** (identifies the subnet itself).
    *   All `1`s in the host part = **Broadcast Address** (sends to everyone in the subnet).

## 3. Longest Prefix Matching
Routers use aggregate forwarding tables to save space. When a packet arrives, the router compares the destination IP to the table.
*   **The Rule:** If an IP matches multiple subnet ranges in the table, the router forwards it to the interface with the **longest prefix** (the highest `/x` number), because it is the most specific match.

## 4. DHCP & NAT
### DHCP (Dynamic Host Configuration Protocol)
Dynamically assigns IP addresses to hosts when they join a network. 
*   **DORA Process:** **D**iscover, **O**ffer, **R**equest, **A**CK.
*   DHCP provides *more* than just an IP. It also provides:
    1.  The Subnet Mask.
    2.  The First-hop Router (Default Gateway).
    3.  The DNS Server address.

### NAT (Network Address Translation)
Solves IP exhaustion by hiding an entire local network behind **one single public IP address**.
*   **How it works:** 
    *   *Outgoing:* Router replaces the private source IP and Port with its public NAT IP and a newly generated Port.
    *   *Incoming:* Router checks its NAT Table, matches the destination Port, and replaces the public IP/Port back to the private IP/Port.
*   **The Controversy:** NAT violates the "end-to-end" principle by interfering with IP addresses at the network edge. It also breaks peer-to-peer apps (like Skype or gaming servers) because external hosts cannot initiate a connection to a hidden private IP (NAT Traversal problem).

## 5. IP Datagrams & Fragmentation
The IP Header is typically **20 bytes** (plus a 20-byte TCP header = 40 bytes of total overhead).
*   **MTU (Maximum Transfer Unit):** The largest possible frame a specific link can handle (e.g., Ethernet MTU is usually 1500 bytes).
*   **Fragmentation:** If a router receives an IP datagram larger than the next link's MTU, it must chop the data into smaller fragments. 
*   *Crucial Rule:* Fragments are **only reassembled at the final destination**, not by the routers in between.

### Fragmentation Math Rules:
If MTU = 1500 bytes, the **maximum data payload** per fragment is $1500 - 20 (\text{IP Header}) = 1480 \text{ bytes}$.
*   **MF (More Fragments) Flag:** `1` if there are more fragments coming. `0` if it is the absolute last fragment.
*   **Offset:** Tells the receiver where this fragment belongs. Calculated as: `(Total Data Bytes sent so far) / 8`.

## 6. IPv6 & Tunneling
Created to solve IP exhaustion (128-bit addresses). 
*   **Key changes:** Fixed 40-byte header, and **fragmentation is NOT allowed** by routers (must be done by the end host).
*   **Tunneling:** How to pass IPv6 packets through older IPv4 networks? The IPv6 packet is encapsulated *inside* the payload of an IPv4 packet. 

## 7. Routing in the Internet (AS, IGP, EGP)
The internet is divided into **Autonomous Systems (ASes)** (e.g., ISPs, universities).
*   **Intra-AS Routing (IGP - Interior Gateway Protocols):** Routing *within* a specific AS.
    *   **RIP (Routing Information Protocol):** Distance Vector. Metric = Hop count (Max 15). Advertises every 30s. If no ad for 180s, link is dead.
    *   **OSPF (Open Shortest Path First):** Link State. Uses Dijkstra's algorithm. Floods the entire AS.
*   **Inter-AS Routing (EGP - Exterior Gateway Protocols):** Routing *between* different ASes.
    *   **BGP (Border Gateway Protocol):** The de facto protocol of the internet.
    *   *Policy-based routing:* BGP doesn't just look at distance; it looks at business rules. For example, a customer network (Dual-homed to two ISPs) will **never** advertise a transit route between its two ISPs, because it gets no profit for carrying their traffic.

---

# Sample Exam Questions & Solutions

### Question 1: Subnetting Math
Given the IP address `200.23.16.130/23`:
a) How many total IP addresses are in this subnet?
b) How many *valid* IP addresses can be assigned to hosts?
c) Find the network (subnet) address.

**Solution:**
**a)** A `/23` mask leaves $32 - 23 = 9$ bits for hosts. Total IPs = $2^9 = \mathbf{512}$.
**b)** Valid hosts exclude the network and broadcast addresses. $512 - 2 = \mathbf{510}$.
**c)** The 23rd bit falls in the 3rd octet. 
* `16` in binary is `00010000`. `130` in binary is `10000010`.
* The subnet mask for /23 is `255.255.254.0` (`11111111.11111111.11111110.00000000`).
* Bitwise AND between IP and Mask zeroes out the host bits. 
* Network Address = **`200.23.16.0`**.

### Question 2: IP Fragmentation Math
A host sends a large IP datagram containing **6000 bytes of data** and a **20-byte IP header**. It travels over a link with an MTU of **1500 bytes**.
a) How many fragments are created?
b) For the *second* fragment, what is the Total Length (bytes), the Data Size (bytes), and the MF flag?
c) For the *last* fragment, what is the Total Length (bytes), Data Size (bytes), and the MF flag?

**Solution:**
*   *Step 1: Calculate Max Payload per fragment.* MTU (1500) - Header (20) = 1480 bytes of data per fragment.
*   *Step 2: Split the 6000 bytes of data.* $6000 = 1480 + 1480 + 1480 + 1480 + 80$.
**a)** **5 fragments** are created.
**b)** Second fragment: Data Size = **1480 bytes**. Total Length = $1480 + 20 = \mathbf{1500 \text{ bytes}}$. MF Flag = **1** (more fragments follow).
**c)** Last fragment: Data Size = **80 bytes**. Total Length = $80 + 20 = \mathbf{100 \text{ bytes}}$. MF Flag = **0** (no more fragments).

### Question 3: Longest Prefix Matching
A router uses longest prefix matching with the following table:
*   Interface 0: `10.1.0.0/16`
*   Interface 1: `10.1.2.0/24`
*   Interface 2: `10.1.2.128/25`
*   Interface 3: `Otherwise`
Which interface will a packet destined for `10.1.2.200` be forwarded to?

**Solution:**
First, check which ranges it fits into:
*   Matches `/16` (Starts with 10.1)
*   Matches `/24` (Starts with 10.1.2)
*   Matches `/25` (Starts with 10.1.2 and the last octet is between 128 and 255. 200 fits here!)
Because it matches all three, the router uses the **Longest Prefix** (the most specific match). `/25` is the longest prefix.
**Answer:** It will be forwarded to **Interface 2**.

### Question 4: Network Address Translation (NAT)
A home network uses a NAT router with the public IP `138.76.29.7`. An internal host (`10.0.0.1`) sends a TCP request from source port `3345` to an external web server (`128.119.40.186` on port `80`). 
a) What are the Source IP and Source Port on the packet *after* it leaves the NAT router?
b) When the web server replies, what Destination IP and Destination Port will it put on the packet?

**Solution:**
**a)** The NAT router replaces the internal private IP/Port with its own public IP and a new, unique port (e.g., 5001). 
Source IP: **`138.76.29.7`**. Source Port: **`5001`**. (The destination IP/Port remain the external server's).
**b)** The web server replies to the public-facing NAT router. 
Destination IP: **`138.76.29.7`**. Destination Port: **`5001`**. (When the NAT router receives this, it will look up 5001 in its table and forward it internally back to 10.0.0.1:3345).

### Question 5: BGP Routing Policy Theory
Autonomous System X is a customer network connected to two different provider ISPs (AS B and AS C). Explain why AS X will *not* advertise a route to AS B through AS C (e.g., path C-X-B). 

**Solution:**
BGP is a policy-based routing protocol driven by business agreements. AS X is a paying customer of both AS B and AS C. If AS X advertises a route between B and C, it would act as a transit network, carrying traffic between the two providers for free. Since AS X receives no profit for routing this transit traffic (and it would consume X's bandwidth), its routing policy will explicitly forbid advertising the C-X-B path. An ISP access network only routes traffic that originates from or is destined for its own network.