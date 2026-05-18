# CITS3002: Mid-Semester Test (2026) - Explanations & Study Guide

This note breaks down the official 2026 Mid-Semester Test. Under each question, you will find the official solution followed by a **"How to understand this"** section that explains the exact steps and logic used to get that answer.

---

## Question 1: CRC Error Detection (10%)
**Question:** Consider a CRC error detection scheme with the divisor `1001`. When the sender wants to transmit the data `11001111`, determine the bit sequence received at the receiver by assuming that no transmission error occurs.

**Official Solution:**
```text
      11010101
 1001|11001111000
      1001
      ----
       01011
       1001
       ----
       001011
         1001
         ----
         001010
           1001
           ----
           001100
             1001
             ----
             0101 
```
Transmitted data is **`11001111101`** (appending remainder to data).

**How to understand this:**
1. **Padding:** The divisor `1001` is 4 bits long. You must append $4-1 = 3$ zeros to the original data. `11001111` becomes `11001111000`.
2. **XOR Division:** You perform modulo-2 division. Whenever the leading bit of your current chunk is a `1`, you XOR it with `1001`. If the leading bit is `0`, you XOR with `0000`.
3. **The Result:** After dropping down every bit, your final remainder is `101`. 
4. **Final Step:** Replace the 3 padded zeros with your 3-bit remainder `101` to get the final answer.

---

## Question 2: Hamming Distance & Correction (10%)
**Question:** Consider a system with three valid codewords: CW1 = `11001010`, CW2 = `11000100`, and CW3 = `00111010`.
a) Find the Hamming distance of the system.
b) How many errors can be detected and corrected?

**Official Solution:**
**a)** 
*   CW1 ⊕ CW2 = `00001110` -> Distance = 3
*   CW1 ⊕ CW3 = `11110000` -> Distance = 4
*   CW2 ⊕ CW3 = `11111110` -> Distance = 7
*   The Hamming distance of the system is min(3,4,7) = **3**.

**b)** 
*   To detect $n$ errors: $n+1 = 3 \implies n = \mathbf{2}$.
*   To correct $n$ errors: $2n+1 = 3 \implies n = \mathbf{1}$.

**How to understand this:**
*   **Hamming Distance of two words:** The number of bits that are different between them. You find this by doing an XOR (`⊕`) between the two words and simply counting how many `1`s are in the result.
*   **System Hamming Distance:** The *smallest* distance between *any* pair of valid codewords. Here, the smallest is 3.
*   **The Formulas:** Just plug the system distance (3) into the standard rules you memorized: $n+1$ for detection, $2n+1$ for correction.

---

## Question 3: Pure ALOHA Math (20%)
**Question:** In Pure ALOHA, frame time is $t$, and mean number of frames generated is $d$ per frame time.
a) What is the throughput when probability of no collision is $p$?
b) What is the vulnerable period? Explain.
c) What is the throughput using the Poisson formula $Pr[k] = \frac{d^k e^{-d}}{k!}$?

**Official Solution:**
**a)** Throughput = $d \times p$
**b)** Vulnerable period is **$2t$**. A frame sent at $t_0$ will collide with any transmission occurring within the interval from $t_0 - t$ to $t_0 + t$.
**c)** The probability that 0 frames are generated during the vulnerable period (which is twice the frame time, so $2d$) is given by $Pr[0] = e^{-2d}$. Therefore, throughput is **$d \times e^{-2d}$**.

**How to understand this:**
This question is trying to trick you by changing the variable letters! 
*   $d$ is just the Load ($G$).
*   $t$ is just the frame time ($T$).
*   **Part A:** Throughput ($S$) is always `Load × Probability of Success`. So, $S = d \times p$.
*   **Part B:** Pure ALOHA vulnerable period is always twice the frame time ($2t$), because a frame can collide with someone who started sending *just before* them, or *just after* them.
*   **Part C:** It asks you to derive the Pure ALOHA throughput formula ($S = G e^{-2G}$). Because the vulnerable period is double ($2t$), the expected load in that period is double ($2d$). The chance of zero collisions is $e^{-2d}$. Multiply that by the load $d$ to get $d e^{-2d}$.

---

## Question 4: Slotted vs. Pure ALOHA (10%)
**Question:** What is the main difference between Slotted and Pure Aloha that leads to better performance?

**Official Solution:**
The main difference lies in the **timing of frame transmission**. In Pure ALOHA, a sender can transmit at any time. In Slotted ALOHA, time is divided into equal slots, and transmission is only allowed at the **beginning of a slot**. As a result, the **vulnerable period** is $2t$ in Pure ALOHA and reduced to $t$ in Slotted ALOHA, which leads to higher throughput.

**How to understand this:**
Exactly what we covered in the notes! Mentioning the **"vulnerable period halves from 2t to t"** is the magic phrase that gets you full marks here.

---

## Question 5: Ethernet Efficiency Math (20%)
**Question:** Ethernet with 3 stations. Transmission probability in a contention slot is 0.5. Contention slot duration is 40 ms. Mean frame transmission time is 20 ms. Determine channel efficiency.

**Official Solution:**
*   $k = 3$, $p = 0.5$, slot duration = $40 \text{ ms}$, transmission time $d = 20 \text{ ms}$.
*   Prob station acquires channel ($A$) = $k \times p \times (1-p)^{k-1} = 3(0.5)(1-0.5)^2 = \mathbf{0.375}$.
*   Mean slots per contention interval = $1 / A = 1 / 0.375 = \mathbf{2.66}$.
*   Channel Efficiency = $\frac{d}{d + (slot\_duration \times \text{mean\_slots})} = \frac{20}{20 + (40 \times 2.66)} = \mathbf{0.15}$.

**How to understand this:**
This is the $\eta = \frac{P}{P + 2\tau/A}$ formula, just written differently.
1.  **Find A (Probability exactly one station talks):** The formula is $A = k \cdot p \cdot (1-p)^{k-1}$. (3 stations $\times$ 50% chance to talk $\times$ 50% chance the other two stay quiet). 
2.  **Find the Wasted Time:** How many slots does it take to get a success? $1 / 0.375 = 2.66$ slots. Multiply that by the length of a slot (40 ms). Total wasted time = $106.6 \text{ ms}$.
3.  **Efficiency:** $\frac{\text{Useful Time (20)}}{\text{Useful Time (20)} + \text{Wasted Time (106.6)}} = \mathbf{0.15}$ (or 15%).

---

## Question 6: Dijkstra's Algorithm (20%)
**Question:** Graph provided. Find shortest paths from node B. What happens if (D,F) changes to 8?

**Official Solution:**
**a)**
*   **Step 1:** Start at **B**. Update neighbors: A(8), C(5), D(9).
*   **Step 2:** **C** is lowest (5). Make C permanent. C connects to E. Path to E via C is 5+15=20.
*   **Step 3:** **A** is lowest (8). Make A permanent. A connects to E. Path to E via A is 8+4=12. (12 is better than 20, so E updates to 12).
*   **Step 4:** **D** is lowest (9). Make D permanent. D connects to F. Path to F is 9+13=22.
*   **Step 5:** **E** is lowest (12). Make E permanent.
*   **Step 6:** **F** is lowest (22). Make F permanent.
*   *Shortest paths from B:* A(8), C(5), D(9), E(12), F(22).

**b)** No, it will not affect the shortest paths from B to any *other* node. Node F is an endpoint (leaf) in this shortest-path tree, so changing its link cost only changes the path *to* F, not to any other nodes.

**How to understand this:**
This tests your ability to trace Dijkstra properly. The key is in Step 3: Node C initially sets Node E to `20`. But when Node A becomes the working node, it finds a "shortcut" to E (`8 + 4 = 12`). Because `12 < 20`, you cross out the 20 and update it. For part B, it's testing your logical graph understanding: since nobody travels *through* F to get anywhere else, making F's road faster doesn't help anyone else.

---

## Question 7: Distance Vector Routing Theory (10%)
**Question:** Node H uses DVR. Node J sends its vector to H, but immediately after, J's links change (so the vector is outdated). 
a) What will H do? 
b) Will it impact H's routing table?

**Official Solution:**
**a)** Node H will treat the received distance vector as valid and use it to update its routing table. Node H has **no way of knowing** that the vector is outdated.
**b)** It can be either yes or no. If the outdated information from J leads H to compute a shorter path to a destination, H will update its table incorrectly. If the outdated costs do not result in a "better" path, the routing table will ignore it and remain unchanged.

**How to understand this:**
This is the ultimate limitation of Distance Vector Routing. It is a "blind trust" system. Routers only know what their neighbors tell them, and they take it as absolute truth. They have no global map to fact-check the information.