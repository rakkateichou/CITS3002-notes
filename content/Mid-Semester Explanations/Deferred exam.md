# CITS3002: Deferred Mid-Semester Test (2026) - Explanations & Study Guide

This note breaks down the official 2026 *Deferred* Mid-Semester Test. This version of the test features some notoriously tricky edge-cases (especially Question 3). Under each question, you will find the official solution followed by a **"How to understand this"** section that explains the exact steps and logic.

---

## Question 1: CRC Error Checking at Receiver (10%)
**Question:** Consider a CRC error detection scheme with a divisor of `1010`. If the received data is `10110011001`, determine whether any error has occurred during transmission.

**Official Solution:**
```text
      1010101
 1010|10110011001
      1010
      ----
      0001001
         1010
         ----
         001110
           1010
           ----
           01000
            1010
            ----
            00101
```
Since the remainder is not `000`, the receiver detects an error.

**How to understand this:**
Unlike the sender, the **receiver does NOT add padding**. It simply takes the exact string it received from the wire and divides it by the divisor using XOR. 
*   If the remainder at the very end is zero, the data is flawless.
*   If the remainder is anything other than zero (in this case, it's `101`), bits were flipped during transmission and the receiver flags an error.

---

## Question 2: Hamming Code Construction (20%)
**Question:** Given the data bits `11011011` (8 bits):
a) How many redundant (parity) bits are required to detect and correct a single-bit error?
b) Using even parity, construct the Hamming codeword.

**Official Solution:**
**a)** $(8 + r + 1) \le 2^r$ implies that $r$ must be greater or equal to **4**.
**b)** Total bits = $8 + 4 = 12$. Parity positions: 1, 2, 4, 8. Data positions: 3, 5, 6, 7, 9, 10, 11, 12.
*   P1 = B3 ⊕ B5 ⊕ B7 ⊕ B9 ⊕ B11 = `1`
*   P2 = B3 ⊕ B6 ⊕ B7 ⊕ B10 ⊕ B11 = `1`
*   P4 = B5 ⊕ B6 ⊕ B7 ⊕ B12 = `1`
*   P8 = B9 ⊕ B10 ⊕ B11 ⊕ B12 = `1`
Final result: **`110111011111`**

**How to understand this:**
*   **Part a:** Just plug $m=8$ into the inequality $(m + r + 1) \le 2^r$ and test numbers. If $r=3$, $12 \le 8$ (False). If $r=4$, $13 \le 16$ (True).
*   **Part b:** Map out a 12-bit table. Put your data bits `1 1 0 1 1 0 1 1` into the non-power-of-2 slots (12, 11, 10, 9, 7, 6, 5, 3). Then, calculate the parity bits by XOR-ing the specific data bits they are responsible for. (Remember: XOR just counts the `1`s. If there is an odd number of `1`s, the parity bit is `1`. If even, it's `0`).

---

## Question 3: Network Delay (The "First Bit" Trap) (10%)
**Question:** Hosts A and B are connected via two links (L1 and L2) with a Router in between. Node A sends a 16-byte packet to Node B. **How long does it take for node B to receive the *first bit* of the packet?**
*   L1 = 1000 bps, 40 m.
*   L2 = 2000 bps, 10 m.
*   Propagation speed = $5 \times 10^8$ m/s.

**Official Solution:**
*   Trans delay A $\rightarrow$ R1 (whole packet) = $(16 \times 8) / 1000 = 0.128 \text{ s}$
*   Prop delay A $\rightarrow$ R1 = $40 / (5 \times 10^8) = 0.00000008 \text{ s}$
*   Trans delay R1 $\rightarrow$ B (**for the first bit**) = $1 / 2000 = 0.0005 \text{ s}$
*   Prop delay R1 $\rightarrow$ B = $10 / (5 \times 10^8) = 0.00000002 \text{ s}$
*   Total Delay = $0.128 + 0.00000008 + 0.0005 + 0.00000002 =$ **`0.1285001 s`**

**How to understand this:**
**MASSIVE EXAM TRAP!**  Read the question carefully: "receive the **first bit**". 
Because the router uses **Store-and-Forward**, it must wait to receive the *entire* 128-bit packet from A before it can do anything. So Link 1 has a full transmission delay.
However, once the router starts pushing the packet onto Link 2, the question stops the clock the microsecond the *first bit* hits B. Therefore, the transmission delay on Link 2 is calculated for **only 1 bit** ($1 / 2000$), not the whole packet!

---

## Question 4: Go-Back-N vs Selective Repeat (10%)
**Question:** Explain the key difference between Go-Back-N and Selective Repeat protocols that results in improved performance for one of them.

**Official Solution:**
In **Go-Back-N**, when a packet is lost or corrupted, the sender retransmits that packet **and all subsequent packets**, even if some were received correctly (because the receiver window size is 1). In contrast, **Selective Repeat** retransmits **only the specific lost or corrupted packets** while buffering correctly received ones at the receiver (receiver window size > 1). This drastically improves performance on noisy channels.

**How to understand this:**
This is the textbook definition. Focus on two keywords for full marks: **Window size** (1 vs >1) and **Buffering** (discarding vs saving out-of-order frames).

---

## Question 5: Ethernet Efficiency Variables (15%)
**Question:** CSMA/CD Ethernet. 40 stations. Probability of success in a slot ($A$) is $0.4$.
a) Find the mean number of contention slots per contention interval.
b) Does this mean number affect efficiency? Explain.
c) How does the duration of a contention slot affect efficiency?

**Official Solution:**
**a)** Mean number of slots = $1 / A = 1 / 0.4 = \mathbf{2.5}$.
**b)** Channel efficiency is given by $\frac{P}{P + T/A}$. As the mean number of slots ($1/A$) increases, the total wasted time increases, which **reduces** channel efficiency.
**c)** Increasing $T$ (the duration of each slot) increases the time wasted per collision, which also **reduces** channel efficiency.

**How to understand this:**
This question tests your understanding of the formula $\eta = \frac{P}{P + \text{Wasted Time}}$. 
Wasted time is `(Number of tries) × (Length of a try)`.
*   If it takes more tries to get a success ($1/A$ goes up), efficiency drops.
*   If the awkward pause of a collision lasts longer ($T$ goes up), efficiency drops.

---

## Question 6: Distance Vector Routing (15%)
**Question:** Node C connects to A (cost 5), B (cost 7), D (cost 13). Node C receives vectors from A, B, and D. Construct the routing table for Node C. *(Vectors provided in prompt)*.

**Official Solution:**
*   **Destination A:** Via A = 5. Via B = 7+12=19. Via D = 13+11=24. $\rightarrow$ **Min: 5 via A.**
*   **Destination B:** Via A = 5+12=17. Via B = 7. Via D = 13+6=19. $\rightarrow$ **Min: 7 via B.**
*   **Destination D:** Via A = 5+11=16. Via B = 7+6=13. Via D = 13. $\rightarrow$ **Min: 13 via D (or B).**
*   **Destination E:** Via A = 5+3=8. Via B = 7+14=21. Via D = 13+8=21. $\rightarrow$ **Min: 8 via A.**

**How to understand this:**
For every destination, C asks itself: "What is my cost to my neighbor + my neighbor's advertised cost to the destination?" It calculates this for A, B, and D, and simply chooses the minimum resulting number. 

---

## Question 7: Dijkstra's Algorithm (20%)
**Question:** Graph provided: (A,B)=3, (A,C)=2, (B,C)=8, (B,E)=4, (B,D)=9, (D,E)=6. Find shortest paths from source node **C**.

**Official Solution:**
*   **Step 1:** Start at **C**. Update neighbors: A(2), B(8). Make **C** permanent.
*   **Step 2:** **A** is the minimum (2). Make A permanent. A connects to B. Path to B via A is 2+3=5. (5 is better than 8, so update B to 5).
*   **Step 3:** **B** is the minimum (5). Make B permanent. B connects to E and D. Path to E via B is 5+4=9. Path to D via B is 5+9=14.
*   **Step 4:** **E** is the minimum (9). Make E permanent. E connects to D. Path to D via E is 9+6=15. (15 is worse than 14, so keep 14).
*   **Step 5:** **D** is the minimum (14). Make D permanent.

**How to understand this:**
This tests the exact "Relaxation" rule of Dijkstra. In Step 1, Node C thinks the best way to get to B is the direct road (cost 8). But in Step 2, when Node A becomes permanent, we discover a "shortcut": going through A to get to B only costs 5! Because $5 < 8$, we cross out the 8 and update B's tentative distance to 5.