**Total Marks: 22** | **Format: Short Answer & Math**

---

## The Questions

### Question 1: Layers and Addressing (4 marks)
In Lab 1, you inspected your network configuration and found both an IP address and a MAC address. 
**a)** Explain the primary difference between how an IP address and a MAC address are assigned to a device. *(2 marks)*
**b)** State exactly which OSI layer each of these two addresses operates on. *(2 marks)*

### Question 2: Network Delay (4 marks)
Host A sends a **128-byte** packet to Host B over a single link. 
* The link bandwidth is **2 Mbps** ($2,000,000$ bps). 
* The propagation speed of the link is **$2 \times 10^8$ m/s**. 
* The distance between the hosts is **50 km** ($50,000$ meters). 

Calculate the **total end-to-end delay** in milliseconds (ms). *(Show your working for $T_{trans}$ and $T_{prop}$).*

### Question 3: Error Detection / CRC (4 marks)
A sender wants to transmit the data `1011` using CRC for error detection. The agreed-upon divisor is `110`. 
Calculate the exact **final bit pattern** that the sender will transmit over the link. *(Show your modulo-2 division steps).*

### Question 4: Stop-and-Wait Efficiency (3 marks)
A network uses the Stop-and-Wait protocol. 
* The frame transmission time ($T_{trans}$) is **2 ms**.
* The one-way propagation delay ($T_{prop}$) is **24 ms**. 

Calculate the **channel utilization (efficiency)** of this connection. *(Express your answer as a percentage or a decimal).*

### Question 5: Medium Access Control (3 marks)
Two stations on a classic Ethernet network collide for the **second** time. They both apply the Binary Exponential Backoff (BEB) algorithm. 
What is the exact probability that they will pick the same random wait time and **collide again** on their third attempt? 

### Question 6: Distance Vector Routing (4 marks)
Router X is using Distance Vector Routing. It knows the cost to reach its two immediate neighbors:
* Cost to reach Router Y = **4**
* Cost to reach Router Z = **2**

Router X receives routing table updates from both neighbors regarding a faraway destination, **Router D**:
* Router Y says: "My cost to reach D is **3**."
* Router Z says: "My cost to reach D is **8**."

**a)** Calculate the total cost for Router X to reach Router D via Router Y, and the total cost via Router Z. *(2 marks)*
**b)** Based on the algorithm, which router (Y or Z) will Router X choose as its **next hop**, and what will be the **final cost** stored in its routing table? *(2 marks)*

---
<br><br><br><br><br>
*(Scroll down for solutions)*
<br><br><br><br><br>

---

## Solutions & Explanations

### Question 1: Layers and Addressing
**a)** A **MAC address** is a physical address permanently assigned to the device's Network Interface Card (NIC) by the manufacturer during assembly. An **IP address** is a logical address that is assigned dynamically when a device connects to a network, meaning it changes depending on the device's location in the network topology.
**b)** 
*   MAC Address: **Data Link Layer** (Layer 2)
*   IP Address: **Network Layer** (Layer 3)

### Question 2: Network Delay
**Total Delay = 0.762 ms.**
*   *Step 1: Convert Bytes to Bits!* $L = 128 \text{ Bytes} \times 8 = \mathbf{1024 \text{ bits}}$.
*   *Step 2: Transmission Delay ($T_{trans}$):* 
    $T_{trans} = \frac{L}{R} = \frac{1024}{2,000,000} = 0.000512 \text{ seconds}$.
*   *Step 3: Propagation Delay ($T_{prop}$):* 
    $T_{prop} = \frac{d}{s} = \frac{50,000}{2 \times 10^8} = 0.00025 \text{ seconds}$.
*   *Step 4: Total Latency in seconds:* 
    $0.000512 + 0.00025 = 0.000762 \text{ seconds}$.
*   *Step 5: Convert to milliseconds:* 
    $0.000762 \times 1000 = \mathbf{0.762 \text{ ms}}$.

### Question 3: Error Detection / CRC
**Final Transmitted Pattern: `101110`**
*   *Step 1: Padding.* Divisor `110` has 3 bits. Append 2 zeros to the data. Padded Data = `101100`.
*   *Step 2: Modulo-2 Division (XOR).*
```text
   101100
 ^ 110      (XOR first 3 bits)
   ---
   0111     (Bring down the 1)
 ^  110     (XOR)
    ---
    0010    (Bring down the 0)
 ^   000    (Leading bit is 0, so XOR with 000)
     ---
     0100   (Bring down final 0)
 ^    110   (XOR)
      ---
       10   <-- REMAINDER
```
*   *Step 3: Replace padding.* Original data `1011` + remainder `10` = **`101110`**.

### Question 4: Stop-and-Wait Efficiency
**Utilization ($U$) = 0.04 (or 4%)**
*   *Formula:* $U = \frac{T_{trans}}{T_{trans} + 2 \times T_{prop}}$
*   *Calculation:* $U = \frac{2}{2 + (2 \times 24)} = \frac{2}{2 + 48} = \frac{2}{50} = \mathbf{0.04}$

### Question 5: Medium Access Control (BEB)
**Probability = 0.25 (or 25% or 1/4).**
*   *Explanation:* After the second collision ($i = 2$), the contention window size is $2^i - 1$, which means the stations choose a random wait time slot from the range $[0, 3]$. 
*   There are 4 possible choices (0, 1, 2, or 3) for Station A, and 4 for Station B, meaning $4 \times 4 = 16$ possible combinations. 
*   They will only collide if they pick the exact same number: (0,0), (1,1), (2,2), or (3,3). 
*   4 collision scenarios out of 16 total scenarios = $4/16 = 1/4 = 0.25$. *(Shortcut formula: $\frac{1}{2^i}$)*.

### Question 6: Distance Vector Routing
**a) Total costs:**
*   Path via Y = (Cost to Y) + (Y's cost to D) = $4 + 3 = \mathbf{7}$.
*   Path via Z = (Cost to Z) + (Z's cost to D) = $2 + 8 = \mathbf{10}$.

**b) Final Routing Table Entry:**
*   **Next Hop:** Router Y
*   **Final Cost:** 7
*   *(Explanation: The router selects the path with the minimum total cost).*