**Total Marks: 20** | **Format: Short Answer & Math**

---

## The Questions

### Question 1: ALOHA Probability & Throughput (5 marks)
Consider a shared communication channel using the ALOHA protocol. The frame time ($T$) is **5 ms**. There are **20 terminals** on the network. Each terminal generates frames at a mean rate of **2 frames per second**.

**a)** Calculate the total load ($G$) per frame time. *(2 marks)*
**b)** Using the Poisson distribution shortcuts, write the formula/value for the probability that **exactly 0 frames** (perfect silence) are generated during one frame time. *(1 mark)*
**c)** Write the formula/value for the probability that **exactly 1 frame** is generated during one frame time. *(1 mark)*
**d)** If this network is using **Slotted ALOHA**, what is the exact formula for the Throughput ($S$) given your load $G$? *(1 mark)*

### Question 2: Hamming Code Error Correction (3 marks)
A receiver gets a 7-bit Hamming code and performs its parity checks. 
* Parity check **P1** (checks bits 1, 3, 5, 7) results in a **PASS** (0).
* Parity check **P2** (checks bits 2, 3, 6, 7) results in a **PASS** (0).
* Parity check **P4** (checks bits 4, 5, 6, 7) results in a **FAIL** (1).

**a)** Which exact bit position contains the error? *(1 mark)*
**b)** Is the corrupted bit a **Data** bit or a **Parity** bit? Explain how you know. *(2 marks)*

### Question 3: Wireless Networks & CSMA/CA (4 marks)
In a wired Ethernet network, stations use **CSMA/CD** (Collision Detection). However, IEEE 802.11 Wireless LANs use **CSMA/CA** (Collision Avoidance).

**a)** Explain the physical reason why a wireless station cannot use Collision Detection (CSMA/CD) while it is transmitting. *(2 marks)*
**b)** Explain how the **NAV (Network Allocation Vector)** works in CSMA/CA to help avoid collisions. *(2 marks)*

### Question 4: Flow Control (Stop-and-Wait) (4 marks)
A satellite link uses the Stop-and-Wait protocol. 
* The time it takes to push the frame onto the wire ($T_{trans}$) is **1 ms**.
* The one-way propagation delay to the satellite ($T_{prop}$) is **10 ms**.

**a)** Calculate the channel utilization (efficiency) of this connection. *(2 marks)*
**b)** The sender upgrades to a **Go-Back-N** sliding window protocol to fix this terrible efficiency. The sender transmits frames 0, 1, 2, 3, and 4. Frame 1 is lost in space, but frames 2, 3, and 4 arrive safely. Exactly what will the Go-Back-N receiver do with frames 2, 3, and 4? *(2 marks)*

### Question 5: Distance Vector Routing (4 marks)
Router A is using Distance Vector Routing. It knows the cost to reach its two neighbors:
* Cost to reach Router B = **3**
* Cost to reach Router C = **6**

Router A receives routing table updates from both neighbors regarding destination **Router F**:
* Router B says: "My cost to reach F is **5**."
* Router C says: "My cost to reach F is **1**."

**a)** Based on the Distance Vector algorithm, which router (B or C) will Router A choose as its **next hop** to reach F? *(2 marks)*
**b)** What will be the **total cost** written into Router A's routing table for destination F? *(2 marks)*

---
<br><br><br><br><br>
*(Scroll down for solutions)*
<br><br><br><br><br>

---

## Solutions & Explanations

### Question 1: ALOHA Probability & Throughput
**a) Total Load ($G$) = 0.2 frames per frame time.**
*   *Step 1:* Find the total frame generation rate ($\lambda$). 20 terminals $\times$ 2 frames/sec = 40 frames/sec total.
*   *Step 2:* Convert frame time ($T$) to seconds. 5 ms = 0.005 seconds.
*   *Step 3:* Multiply $\lambda \times T$ to find $G$. $40 \times 0.005 = 0.2$.
*   *(Dimensional check: $\frac{\text{frames}}{\text{sec}} \times \frac{\text{sec}}{\text{frame time}} = \frac{\text{frames}}{\text{frame time}}$)*

**b) Probability of 0 frames = $e^{-0.2}$**
*   *Explanation:* Using the Poisson distribution ($Pr[k] = \frac{G^k e^{-G}}{k!}$) where $k=0$.

**c) Probability of 1 frame = $0.2 \times e^{-0.2}$**
*   *Explanation:* Using the Poisson distribution where $k=1$.

**d) Slotted ALOHA Throughput ($S$) = $0.2 \times e^{-0.2}$**
*   *Explanation:* Throughput ($S$) for Slotted ALOHA is $S = G \times e^{-G}$. It is mathematically identical to the probability of exactly 1 frame being generated in a single slot.

### Question 2: Hamming Code Error Correction
**a) Position 4.**
*   *Explanation:* You find the erroneous bit by summing the position numbers of the failed parity checks. Since only P4 failed, the sum is just 4.

**b) It is a Parity bit.**
*   *Explanation:* In a Hamming code, parity bits sit at positions that are powers of 2 (1, 2, 4, 8...). Because the error occurred exactly at position 4, we know the parity bit itself was corrupted during transmission, not the data bits (which sit at non-power-of-2 positions like 3, 5, 6, 7).

### Question 3: Wireless Networks & CSMA/CA
**a) Physical limitation:**
*   A wireless station cannot transmit and listen at the exact same time. The physical signal it transmits is exponentially stronger than any incoming signal from another station, effectively "deafening" the hardware to incoming collisions while it is speaking. 

**b) NAV (Network Allocation Vector):**
*   The NAV acts as a virtual countdown timer. When stations exchange RTS (Request to Send) and CTS (Clear to Send) frames, those frames include a duration field. Any other station that overhears the RTS or CTS reads that duration, updates its NAV timer, and goes into a silent state until the timer expires, successfully avoiding a collision.

### Question 4: Flow Control (Stop-and-Wait)
**a) Channel Utilization ($U$) = 0.047 (or ~4.7%)**
*   *Formula:* $U = \frac{T_{trans}}{T_{trans} + 2 \times T_{prop}}$
*   *Calculation:* $U = \frac{1}{1 + (2 \times 10)} = \frac{1}{21} \approx 0.047$

**b) The receiver will discard them.**
*   *Explanation:* Go-Back-N has a receiving window size of exactly 1. It only accepts frames in strict sequential order. Because it is waiting for Frame 1, it will drop/discard Frames 2, 3, and 4 and refuse to acknowledge them until Frame 1 arrives successfully.

### Question 5: Distance Vector Routing
**a) Next Hop = Router C.**
*   *Explanation:* Calculate the total path costs. 
    *   Path via B = (Cost A $\rightarrow$ B) + (B's cost to F) = 3 + 5 = 8.
    *   Path via C = (Cost A $\rightarrow$ C) + (C's cost to F) = 6 + 1 = 7.
    *   Router A chooses the path with the minimum cost (Path via C).

**b) Total Cost = 7.**
*   *Explanation:* This is the minimum total cost calculated in Part A.