# Week 6: Network Layer Routing & Algorithms

## 1. Network Layer Basics (Layer 3)
While the Data Link Layer handles transmission across a *single* link, the **Network Layer** routes packets end-to-end across multiple different networks (Internetworking).
*   **Key Function:** Routing packets from the source to the destination via multiple hops.
*   **Store-and-Forward Packet Switching:** A router receives a packet, stores it until it is fully received and its checksum is verified, and then forwards it to the next router.

### The Two Main Router Processes
1.  **Routing:** The "background math." Constructing and updating the routing tables.
2.  **Forwarding:** The "active job." Looking at an incoming packet, checking the routing table, and moving the packet to the correct outgoing link.

## 2. Connectionless vs. Connection-Oriented Services
A major historical debate in network design is whether Layer 3 should guarantee delivery.
*   **Connectionless (Datagram Networks):** 
    *   No setup phase. Each packet is routed independently.
    *   Delivery is **not guaranteed** and packets may arrive out of order.
    *   *Analogy:* The Postal System.
    *   *Real World:* **IP (Internet Protocol)** uses this. It provides "best-effort" delivery, leaving error recovery and ordering to Layer 4 (TCP).
*   **Connection-Oriented (Virtual Circuit Networks):** 
    *   A path is established *before* transmission begins.
    *   Data follows the same predefined route, ensuring reliable, in-order delivery.
    *   *Analogy:* The telephone system.

## 3. Distance Vector Routing (DVR)
A *dynamic/adaptive* routing algorithm where routers collaboratively determine the shortest path.
*   **How it works:** Each router maintains a routing table storing the shortest known distance to every destination. Routers periodically **exchange their entire tables** with their immediate neighbors.
*   **The Math:** To find the shortest path to a destination, a router calculates: `Cost to reach Neighbor + Neighbor's advertised cost to Destination`. It picks the minimum result.

### The Count-to-Infinity Problem (The "Bad News" Flaw)
*   DVR reacts quickly to good news (a new, faster link) but extremely slowly to bad news (a broken link).
*   **The Flaw:** When Router X tells Router Y about a path to a destination, Router Y has no way of knowing if it is *already part* of that path. 
*   **The Result:** If a link breaks, routers may blindly bounce outdated routing information back and forth, slowly counting up to infinity while falsely believing a valid path still exists. 

## 4. Link State Routing (LSR)
Developed to replace DVR (used in ARPANET until 1979) because it solves the Count-to-Infinity problem and reacts much faster to topology changes.
*   **How it works:** 
    1.  Discover neighbors (send HELLO packets).
    2.  Measure link costs (often inversely proportional to link bandwidth).
    3.  Create a **Link State Packet (LSP)** containing this local knowledge.
    4.  **Flood** the network with this packet so *every* router receives it.
    5.  Compute the shortest path using **Dijkstra's Algorithm**.
*   **The Result:** Every single router builds an identical, complete map of the entire network.

### Controlling the Flood (LSP Management)
Flooding can cause infinite loops if not managed. To prevent this, LSPs use:
*   **Sequence Numbers:** Increased for every new packet sent. If a router receives a duplicate sequence number, it discards it. If it receives a lower one, it rejects it as obsolete.
*   **Age/TTL (Time to Live):** A 32-bit sequence number could wrap around, or a crashed router might restart at 0. To fix this, an "Age" field is added and decremented once per second. When Age = 0, the packet is discarded.

## 5. Dijkstra’s Algorithm (Shortest Path)
The math engine behind Link State Routing. It computes the absolute shortest path from a single source node to all other nodes.
*   **Tentative vs. Permanent:** Nodes are labeled with their distance from the source.
    *   *Tentative:* The current best guess. Can be updated if a faster path is found.
    *   *Permanent:* Mathematically locked in. Represents the absolute shortest possible path and will never be changed.

**The Algorithm Steps:**
1.  Mark the starting node as **Permanent** (distance 0). All others are Tentative ($\infty$).
2.  Look at all neighbors of the current node. Update their tentative distances: `Current Node Distance + Link Cost`.
3.  Look at the entire graph. Find the Tentative node with the **smallest label**. Make it **Permanent**. This is the new working node.
4.  Repeat until all nodes are Permanent.

## 6. Routing in the Internet
Because the internet is too massive for every router to maintain a map of the whole world, we use **Hierarchical Routing**.
*   The internet is divided into **Autonomous Systems (ASes)** (e.g., specific ISPs or Universities).
*   **Intradomain Routing (IGP):** Routing *within* an AS. Routers know the full local topology. (Common protocol: **OSPF**, which uses Link State Routing).
*   **Interdomain Routing (EGP):** Routing *between* different ASes. Networks are treated as separate regions to reduce memory and bandwidth overhead. (Common protocol: **BGP**).

---

# Sample Exam Questions & Solutions

### Question 1: Routing Theory
Explain the fundamental difference in how network topology information is shared in Distance Vector Routing (DVR) compared to Link State Routing (LSR).

**Solution:**
In **Distance Vector Routing (DVR)**, a router only shares its information with its **immediate neighbors**, but the information it shares is its **entire routing table** (its calculated costs to reach every destination in the network). 
In **Link State Routing (LSR)**, a router shares its information with **every router in the entire network** (via flooding), but the information it shares is only the state of its **own immediate links** (its local neighbors and costs).

### Question 2: The Count-to-Infinity Problem
Describe the "Count-to-Infinity" problem in Distance Vector Routing and explain the fundamental flaw in the DVR algorithm that causes it. 

**Solution:**
The Count-to-Infinity problem occurs when a link fails or a router goes down (bad news). Instead of immediately removing the route, neighboring routers may continuously bounce outdated routing information back and forth, slowly incrementing the cost of the path to infinity. 
The fundamental flaw causing this is that when Router X tells Router Y it has a path to a destination, **Router Y cannot determine if it is already part of that path**. This allows routing loops to form, where X relies on Y, and Y relies on X, neither realizing the actual destination is dead.

### Question 3: Link State Packet Flooding
In Link State Routing, what is the purpose of including an "Age" (or Time-to-Live) field in the Link State Packets (LSPs)? Give two specific problems this field prevents.

**Solution:**
The "Age" field acts as a countdown timer that is decremented once per second; when it hits zero, the packet is discarded. 
This prevents two major problems:
1.  **Indefinite Circulation:** It stops orphaned or duplicated packets from looping around the network forever.
2.  **Sequence Number Corruption/Wraparound:** If a bit error corrupts a sequence number (e.g., jumping from 4 to 65,000), or if a crashed router resets its sequence to 0, the Age field ensures that those corrupted or confusing packets will eventually expire and be purged from the network's memory.

### Question 4: Datagram vs. Virtual Circuit
The Network Layer can be implemented using either a Datagram Network or a Virtual Circuit Network. 
a) State which of these two approaches is used by the Internet Protocol (IP). 
b) State one advantage and one disadvantage of the Datagram approach compared to a Virtual Circuit.

**Solution:**
**a)** The Internet Protocol (IP) uses a **Datagram Network** approach (Connectionless service).
**b)** 
*   *Advantage:* No connection setup phase is required, meaning packets can be sent immediately, and the network can easily route around sudden link failures since each packet is evaluated independently.
*   *Disadvantage:* There is no guarantee of delivery, reliability, or that the packets will arrive in the correct order. 

### Question 5: Dijkstra's Algorithm (Math/Logic)
Consider a network where Node A is the source. The costs of the links are:
*   Cost(A, B) = 5
*   Cost(A, C) = 2
*   Cost(B, C) = 1
*   Cost(B, D) = 4
*   Cost(C, D) = 6

**a)** In Step 1, Node A is marked Permanent. What are the Tentative labels assigned to Node B and Node C?
**b)** In Step 2, which node becomes the new working (Permanent) node, and why?
**c)** After Step 2 is completed (including updating neighbors), what is the new Tentative label for Node B, and why did it change?

**Solution:**
**a)** Node B = 5. Node C = 2.
**b)** **Node C** becomes the new working (Permanent) node because it has the smallest Tentative label (2) in the entire graph.
**c)** The new Tentative label for Node B becomes **3**. This changes because the algorithm discovers a "shortcut" through Node C. The original direct path (A $\rightarrow$ B) cost 5. The new path (A $\rightarrow$ C $\rightarrow$ B) costs $2 + 1 = 3$. Since $3 < 5$, the label for B is updated.