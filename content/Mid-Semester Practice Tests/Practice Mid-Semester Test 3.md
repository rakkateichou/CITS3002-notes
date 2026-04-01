**Total Marks: 40** | **Format: Short Answer & Math**

---

## The Questions

### Question 1: Latency & Physical Layer (6 marks)
Host A is sending a **1500-Byte** packet to Host B over a single wired link. 
* The link bandwidth is **100 Mbps** ($10^8$ bps).
* The link distance is **200 km** ($200,000$ meters).
* The signal propagation speed is **$2 \times 10^8$ m/s**.

**a)** Calculate the Transmission Delay ($T_{trans}$) in milliseconds (ms). *(2 marks)*
**b)** Calculate the Propagation Delay ($T_{prop}$) in milliseconds (ms). *(2 marks)*
**c)** In theory, a 100 Mbps link has a bandwidth of 100 million bits per second. Give two reasons why the *actual throughput* achieved by Host A will be lower than this nominal bandwidth. *(2 marks)*

### Question 2: Error Control: CRC & Hamming (8 marks)
**a) CRC (4 marks):** A sender wants to transmit the data `11011` using a CRC divisor of `1011`. 
Calculate the exact final bit pattern that the sender will transmit. *(Show your modulo-2 division).*

**b) Hamming Bounds (2 marks):** A system wants to use a Hamming code to send a payload of exactly **11 data bits** ($m = 11$). Using the Hamming inequality formula, calculate the *minimum* number of redundant parity bits ($r$) required to correct single-bit errors.

**c) Hamming Theory (2 marks):** In a Hamming code, how does the receiver mathematically determine the exact position of a flipped bit? 

### Question 3: Flow Control & Sliding Windows (6 marks)
A 1 Mbps ($1,000,000$ bps) link has a one-way propagation delay of **20 ms** (0.02 seconds). The sender uses the **Stop-and-Wait** protocol with a frame size of **5000 bits**.

**a)** Calculate the channel utilization (efficiency) of this link. *(3 marks)*
**b)** To fix this efficiency, the sender switches to a **Selective Repeat** sliding window protocol. Frames 0, 1, 2, 3, 4, and 5 are transmitted. Frame 2 is completely lost in transit. Exactly how will the Selective Repeat receiver handle the arrival of Frames 3, 4, and 5? *(3 marks)*

### Question 4: Medium Access Control: ALOHA & Ethernet (6 marks)
**a) ALOHA (3 marks):** A shared channel uses **Pure ALOHA**. The network load ($G$) is currently **1.5** attempts per packet time. Write the formula/value for the total **Throughput ($S$)** of this channel. *(You may leave 'e' in your answer).*

**b) Ethernet Efficiency (3 marks):** Consider a classic Ethernet network (CSMA/CD). 
* The frame transmission time ($P$) is **100 $\mu s$**. 
* The one-way propagation delay ($\tau$) is **10 $\mu s$**. 
* The probability of exactly one station successfully acquiring the channel ($A$) is **0.1**. 
Calculate the channel efficiency ($\eta$) as a percentage. 

### Question 5: Wireless Networks (6 marks)
In an IEEE 802.11 wireless network, the probability of a single bit breaking is $p = 10^{-5}$. A station uses the RTS/CTS mechanism. 
* $L_{RTS} = 200$ bits
* $L_{CTS} = 200$ bits
* $L_{DATA} = 8000$ bits
* $L_{ACK} = 200$ bits

**a) Wireless Probability (3 marks):** Write the formula/value for the probability that the entire transmission exchange completes successfully without any bit errors in one attempt.
**b) Wireless Theory (3 marks):** Draw or describe the **Exposed Terminal Problem**. Explain why it results in wasted bandwidth. 

### Question 6: Network Layer Routing (8 marks)
**a) Distance Vector Routing (4 marks):** 
Router A connects to Neighbors B and C. 
* Cost from A to B = **2**
* Cost from A to C = **5**
Neighbor B advertises that its cost to reach destination Router Z is **6**. 
Neighbor C advertises that its cost to reach destination Router Z is **2**. 
What is Router A's new **Next Hop** and **Total Cost** to reach Z?

**b) Dijkstra's Algorithm (4 marks):** 
Imagine a graph where Node A is the source. 
* Link A $\rightarrow$ B costs 3. 
* Link A $\rightarrow$ C costs 6.
After Step 1 of Dijkstra's algorithm, Node A is marked as **Permanent**. Node B has a tentative label of 3, and Node C has a tentative label of 6. 
*Exactly what happens in Step 2 of the algorithm?* (Which node becomes the new working node, and what status does it receive?)


---
<br><br><br><br><br><br><br><br>
*(Scroll down for solutions)*
<br><br><br><br><br><br><br><br>

---

## ✅ Solutions & Explanations

### Question 1: Latency & Physical Layer
**a) $T_{trans}$ = 0.12 ms**
*   *Step 1:* Convert Bytes to bits. $L = 1500 \times 8 = 12,000 \text{ bits}$.
*   *Step 2:* $T_{trans} = \frac{L}{R} = \frac{12,000}{10^8} = 0.00012 \text{ seconds}$.
*   *Step 3:* Multiply by 1000 to get ms. $0.00012 \times 1000 = \mathbf{0.12 \text{ ms}}$.

**b) $T_{prop}$ = 1 ms**
*   *Step 1:* $T_{prop} = \frac{d}{s} = \frac{200,000}{2 \times 10^8} = 0.001 \text{ seconds}$.
*   *Step 2:* Multiply by 1000 to get ms. $0.001 \times 1000 = \mathbf{1 \text{ ms}}$.

**c)** Actual throughput is lower due to: 1) **Protocol Overhead** (MAC headers, IP headers, CRC take up bits without being actual user data), 2) **Congestion/Queueing delays**, or 3) **Errors/Collisions** forcing retransmissions.

### Question 2: Error Control: CRC & Hamming
**a) Transmitted Pattern: `11011001`**
*   *Step 1: Padding.* Divisor `1011` is 4 bits. Append 3 zeros. Padded data = `11011000`.
*   *Step 2: Division.*
```text
   11011000
 ^ 1011       (XOR first 4)
   ----
   01101      (Bring down 1)
 ^  1011      (XOR)
    ----
    01100     (Bring down 0)
 ^   1011     (XOR)
     ----
     01110    (Bring down 0)
 ^    1011    (XOR)
      ----
      01010   (Bring down 0)
 ^     1011   (XOR)
       ----
       0001   <-- REMAINDER
```
*   *Step 3: Replace padding.* `11011` + `001` = **`11011001`**.

**b) $r = 4$ parity bits.**
*   *Formula:* $(m + r + 1) \le 2^r$. Here, $m=11$. 
*   Substitute: $(11 + r + 1) \le 2^r \rightarrow (12 + r) \le 2^r$.
*   If $r=3$: $15 \le 8$ (False).
*   If $r=4$: $16 \le 16$ (True). 

**c)** The receiver calculates the parity for each group. If a group fails the check, it records the power-of-2 position number of that parity bit. The receiver then **adds together the position numbers of all the failed parity bits** (the syndrome). The resulting sum is the exact position of the corrupted bit.

### Question 3: Flow Control & Sliding Windows
**a) Channel Utilization ($U$) = 0.111 (or ~11.1%)**
*   *Step 1:* $T_{trans} = 5000 / 1,000,000 = 0.005 \text{ seconds (5 ms)}$.
*   *Step 2:* Formula $U = \frac{T_{trans}}{T_{trans} + 2 \times T_{prop}}$
*   *Step 3:* $U = \frac{5}{5 + (2 \times 20)} = \frac{5}{5 + 40} = \frac{5}{45} = 1/9 \approx \mathbf{0.111}$.

**b)** A Selective Repeat receiver has a window size greater than 1. Because Frame 2 is missing, the receiver will **buffer (store)** Frames 3, 4, and 5 in its memory and send a Negative Acknowledgment (NAK) specifically for Frame 2, waiting for the sender to retransmit just that single missing frame.

### Question 4: Medium Access Control
**a) Pure ALOHA Throughput ($S$) = $1.5 \times e^{-3}$**
*   *Formula:* Pure ALOHA throughput is $S = G e^{-2G}$. 
*   Substitute $G=1.5$: $S = 1.5 e^{-(2 \times 1.5)} = \mathbf{1.5 e^{-3}}$.

**b) Ethernet Channel Efficiency ($\eta$) = 33.3%**
*   *Formula:* $\eta = \frac{P}{P + (2\tau / A)}$
*   *Substitute:* $\eta = \frac{100}{100 + (2 \times 10 / 0.1)}$
*   $\eta = \frac{100}{100 + (20 / 0.1)}$
*   $\eta = \frac{100}{100 + 200} = \frac{100}{300} = 1/3 \approx \mathbf{33.3\%}$.

### Question 5: Wireless Networks
**a) Probability = $(1 - 10^{-5})^{8600}$**
*   *Explanation:* Total bits = $200 + 200 + 8000 + 200 = 8600$. The probability of all bits surviving is $(1 - p)^n$. 

**b) Exposed Terminal Problem:**
*   *Scenario:* Station B is transmitting to Station A. Station C overhears B's transmission and wants to transmit to Station D. 
*   *Wasted Bandwidth:* Because C hears B talking, it assumes the channel is busy and defers its transmission to D. However, C transmitting to D would not have caused a collision at A (because C is out of A's range). C is unnecessarily staying silent, which wastes available network bandwidth.

### Question 6: Network Layer Routing
**a) Next Hop = Router C | Total Cost = 7**
*   *Via B:* Cost to B (2) + B's cost to Z (6) = 8.
*   *Via C:* Cost to C (5) + C's cost to Z (2) = 7. 
*   Router A chooses the minimum cost (7) via C.

**b) Dijkstra Step 2:**
*   In Step 2, the algorithm looks at all unvisited nodes with "Tentative" labels (Node B: 3, Node C: 6). It selects the node with the **lowest** tentative distance. 
*   **Node B** is selected as the new working node and its label is upgraded from Tentative to **Permanent**.