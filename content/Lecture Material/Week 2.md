# Physical Layer & Network Measurements

## 1. Basic Network Measurements
Network performance is strongly influenced by bandwidth and latency.
* **Bandwidth:** The *nominal (peak)* number of bits per second (bps) that can be transmitted through a network connection.
* **Throughput:** The *actual (achieved)* data rate in practice. It is usually lower than bandwidth due to congestion, packet loss, and protocol overhead.
* **Prefixes in Networking:** 
  * 1 Kbps ≈ 1,000 bits per second
  * 1 Mbps ≈ 1,000,000 bits per second
  * 1 Gbps ≈ 1,000,000,000 bits per second

## 2. Latency (Delay)
Latency is the total time required for a whole message to be delivered to its destination.
> **Total Latency = Transmission Delay + Propagation Delay + Queueing Delay**

### The Delay Formulas (Crucial for Exam Math)
1. **Transmission Delay ($T_{trans}$):** The time it takes to push all the bits of a packet onto the wire.
   * **Formula:** `L / R`
   * `L` = Packet length in **bits** *(Trap: If given in Bytes, multiply by 8!)*
   * `R` = Link bandwidth in **bps**
2. **Propagation Delay ($T_{prop}$):** The time it takes for a single bit to travel from the start of the wire to the destination.
   * **Formula:** `d / s`
   * `d` = Distance of the physical link (in meters)
   * `s` = Propagation speed in the medium (e.g., $2 \times 10^8$ m/s)
3. **Queueing Delay ($T_{queue}$):** Time spent waiting at output links/routers. Depends on network congestion.

## 3. Physical Layer Functions
The Physical Layer handles how raw "data" (0s and 1s) travels from one end to the other.
* **Encoding:** Converting digital data into a **digital signal**. Less complex, less expensive. Used primarily for wired mediums (copper cable, fiber optic).
* **Modulation:** Converting digital data into an **analog signal**. Used primarily for wireless mediums (Wi-Fi, cellular).

### Digital Signaling & Clocking
* **Clocking (Synchronization):** The receiver must know exactly when a bit starts and ends to interpret it correctly. Without timing, bits are misread.
* **NRZ-L (Non-Return-to-Zero Level):** Voltage is constant during a bit interval. Two different voltages represent 0 and 1. (No transitions).
* **NRZI (Non-Return-to-Zero Invert on ones):** A *differential encoding* technique. Data is encoded by the presence or absence of a signal transition. 
  * Transition (low-to-high or high-to-low) = Binary `1`
  * No transition = Binary `0`
  * *Advantage:* Detecting a transition is more reliable in the presence of noise than comparing a flat voltage to a threshold.

## 4. Errors & Modulo-2 Math (XOR)
Errors usually occur in bursts due to thermal noise, impulse noise, or crosstalk.
* **Detection:** You only know the data is wrong (e.g., CRC).
* **Correction:** You know exactly what the data *should* have been and can fix it without asking for a retransmission (e.g., Hamming Code). This is called *Forward Error Correction*.

**Modulo-2 Arithmetic (XOR):**
Used heavily in error detection/correction. Rule: *Same = 0, Different = 1.*
* `0 ⊕ 0 = 0`
* `1 ⊕ 1 = 0`
* `1 ⊕ 0 = 1`
* `0 ⊕ 1 = 1`

## 5. CRC (Cyclic Redundancy Check)
A systematic error detection method. 
1. **Padding:** Count the bits in the agreed-upon Divisor. Subtract 1. Append that many `0`s to the end of your Data.
2. **Division:** Perform Modulo-2 division (XOR) on the padded data using the Divisor.
3. **Syndrome (Remainder):** The remainder of this division replaces the padded `0`s. This is transmitted with the data.
4. **Receiver Check:** The receiver divides the whole received string by the same Divisor. If the remainder is `0`, there is no error. If it's not `0`, the frame is corrupted.

## 6. Hamming Code (Forward Error Correction)
Hamming codes consist of Data bits ($m$) and Redundant/Parity bits ($r$).
* **Hamming Distance Rules:**
  * To *detect* $n$ errors, you need a distance of $n+1$.
  * To *correct* $n$ errors, you need a distance of $2n+1$.
* **Calculating Redundant Bits ($r$):** Must satisfy the inequality: `(m + r + 1) ≤ 2^r`
* **Finding the Error:**
  * Parity bits sit at positions that are powers of 2 (1, 2, 4, 8...).
  * If a parity check fails at the receiver, note its position.
  * *Magic Rule:* Add the position numbers of all the *failed* parity bits together. The sum equals the exact position of the erroneous bit. Flip that bit to correct the error.

---

# Sample Exam Questions & Solutions

### Question 1: Delay Calculation (Math)
Host A sends a packet of 64 bytes to Host B over a single link. The bandwidth is 1000 bps. The length of the link is 10 km (10,000 m), and the propagation speed is $2 \times 10^8$ m/s. What is the total end-to-end delay in milliseconds?

**Solution:**
1. **Convert Bytes to Bits:** $L = 64 \text{ Bytes} \times 8 = 512 \text{ bits}$.
2. **Transmission Delay ($T_{trans}$):** $L / R = 512 / 1000 = 0.512 \text{ seconds} = 512 \text{ ms}$.
3. **Propagation Delay ($T_{prop}$):** $d / s = 10,000 / (2 \times 10^8) = 0.00005 \text{ seconds} = 0.05 \text{ ms}$.
4. **Total Delay:** $512 \text{ ms} + 0.05 \text{ ms} = \mathbf{512.05 \text{ ms}}$.

### Question 2: CRC Error Detection (Math)
A sender wishes to transmit the data `1101` using a CRC divisor of `101`. What is the final transmitted bit pattern?

**Solution:**
1. **Padding:** The divisor (`101`) has 3 bits. Append $3-1 = 2$ zeros to the data. Padded data = `110100`.
2. **Division:**
   ```text
      110100
      101      <-- XOR
      ---
      0111     <-- Bring down the 1
       101     <-- XOR
       ---
       0100    <-- Bring down the 0
        101    <-- XOR
        ---
        001    <-- Remainder is 01
   ```
3. **Final Pattern:** Replace the padded zeros with the remainder. Transmitted pattern = **`110101`**.

### Question 3: Physical Layer (Theory)
Explain the difference between NRZ-L and NRZI encoding. Why is NRZI considered more reliable in noisy environments?

**Solution:**
In **NRZ-L** (Non-Return-to-Zero Level), binary values are represented by constant voltage levels (e.g., positive for 1, negative for 0). In **NRZI** (Invert on ones), data is encoded using a differential approach: a *transition* in voltage represents a binary 1, and *no transition* represents a 0. NRZI is more reliable because detecting a change (transition) in a signal is mathematically easier and less prone to disruption by background noise than reading a static absolute voltage level against a threshold.

### Question 4: Hamming Code Error Correction (Math)
A receiver gets a 7-bit Hamming code. After performing parity checks, it finds that parity bit P1 (position 1) and parity bit P4 (position 4) have failed, but parity bit P2 (position 2) is correct. 
a) Which bit position contains the error?
b) If the bit at that position is currently a `0`, what should the receiver do?

**Solution:**
a) To find the erroneous bit, sum the position numbers of the failed parity bits: $1 + 4 = \mathbf{5}$. The error is in bit position 5.
b) The receiver should perform forward error correction by **flipping the bit** at position 5 from a `0` to a **`1`**.