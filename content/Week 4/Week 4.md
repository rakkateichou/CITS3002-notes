# Local Area Networks (LANs) & Medium Access Control

## 1. LAN Basics & Topologies
A Local Area Network (LAN) covers a short physical distance and connects multiple devices for sharing data at high rates.
* **Bus:** All stations share a single communication channel. Only one station can transmit at a time.
* **Ring:** Consists of repeaters in a closed loop. Links are unidirectional. A frame circulates around the ring, the destination copies it, and the source removes it.
* **Star (Most Popular):** All stations connect to a central node. 
  * If the central node is a **Hub**, it acts logically like a Bus (broadcasts to everyone).
  * If the central node is a **Switch**, it acts as a frame-switching device (directs traffic only to the destination).

## 2. Hubs vs. Switches (Crucial Exam Comparison)
* **Hub (Layer 1 - Physical):** 
  * Acts as a repeater. When one station transmits, the hub broadcasts the signal to **all** stations.
  * **Collision Domain:** All stations are in the **same** collision domain. If two stations transmit at the same time, a collision occurs.
  * **Capacity:** Bandwidth is shared. As more stations are added, each gets a decreasing share of the fixed capacity.
* **Switch (Layer 2 - Data Link):**
  * Buffers incoming frames and retransmits them *only* on the specific outgoing link to the destination.
  * **Collision Domain:** Each port is its own **independent collision domain**. 
  * **Capacity:** If the cable is full-duplex, collisions are impossible. Stations can send frames simultaneously without worrying about other ports.

## 3. Channel Allocation Strategies
When multiple users share a single channel (like a satellite or a shared wire), we must decide who gets to talk.
* **Static Allocation:** 
  * **FDMA (Frequency Division):** Channel divided into frequency bands.
  * **TDMA (Time Division):** Channel divided into time slots.
  * **CDMA (Code Division):** Transmitters send simultaneously using different mathematical codes.
  * *Disadvantage:* Rigid. If a user has nothing to send, their frequency/time slot is completely wasted.
* **Dynamic/Contention-Based Allocation:**
  * Stations transmit when they have data. No advance coordination.
  * *Disadvantage:* If frames overlap in time, a **collision** occurs, and both frames are lost, diminishing throughput.

## 4. ALOHA Protocols
The earliest contention-based protocols.
* **Pure ALOHA:**
  * **Rule:** Users transmit *whenever they want*. If there is a collision, wait a random time and retransmit.
  * **Vulnerable Period:** $2t$ (A frame can collide with a frame sent just before it or just after it).
  * **Throughput ($S$):** $S = G e^{-2G}$ (Maximum throughput is $\approx 18.4\%$).
* **Slotted ALOHA:**
  * **Rule:** Time is divided into discrete slots. Stations must wait for the **beginning of the next slot** to transmit.
  * **Vulnerable Period:** $t$ (This halves the vulnerable period because frames can only collide if they start in the exact same slot).
  * **Throughput ($S$):** $S = G e^{-G}$ (Maximum throughput is $\approx 36.8\%$).

> **Math Note (Poisson Distribution):** The probability of exactly $k$ frames being generated when the average load is $G$ is: $Pr[k] = \frac{G^k e^{-G}}{k!}$

## 5. CSMA (Carrier Sense Multiple Access)
"Listen before you talk." Stations detect what others are doing and adapt.
* **1-persistent CSMA:** Listen to the channel. If idle, transmit immediately (probability of 1). If busy, wait until it becomes idle and then immediately transmit. *(Trap: If two stations are waiting for a busy channel to clear, they will 100% collide when it does).*
* **Non-persistent CSMA:** Listen to the channel. If idle, transmit. If busy, **wait a random period of time** and check again. *(Reduces collisions, but increases delay).*
* **p-persistent CSMA:** If idle, transmit with probability $p$. Defer to the next slot with probability $q = (1 - p)$.

## 6. Ethernet & CSMA/CD
Classic Ethernet uses **1-persistent CSMA/CD** (Collision Detection).
* Stations listen, send if idle, and monitor the channel *while* sending.
* If a collision is detected, they abort the transmission, send a short "jam signal", and wait a random interval before retransmitting.

### Binary Exponential Backoff (BEB) Algorithm
How Ethernet decides the random waiting time after a collision:
1. After $i$ collisions, a station chooses a random number of slots to wait from the range $[0, 2^i - 1]$.
   * 1st collision: Choose from $\{0, 1\}$
   * 2nd collision: Choose from $\{0, 1, 2, 3\}$
   * 3rd collision: Choose from $\{0, 1, 2, 3, 4, 5, 6, 7\}$
2. **Freeze:** After 10 collisions, the interval is frozen at a maximum of 1023 slots.
3. **Abort:** After 16 collisions, the hardware gives up and reports a failure.

## 7. Ethernet Performance (Efficiency Math)
The efficiency of a CSMA/CD network depends on the Transmission Delay ($P$), the Propagation Delay ($\tau$), and the probability that exactly one station successfully acquires the channel ($A$).
* **Formula:** $\eta = \frac{P}{P + (2\tau / A)}$

---

# Sample Exam Questions & Solutions

### Question 1: ALOHA Probability (Math)
Consider a shared channel using the ALOHA protocol. The frame time is 10 ms. There are 10 terminals, and each terminal generates frames at a mean rate of 1 frame per second. 
a) What is the total load ($G$) per frame time?
b) What is the probability that exactly 1 frame is generated during one frame time?

**Solution:**
a) Total rate ($\lambda$) = $10 \text{ terminals} \times 1 \text{ frame/sec} = 10 \text{ frames/sec}$. 
The frame time ($T$) is $10 \text{ ms} = 0.01 \text{ seconds}$. 
Load ($G$) = $\lambda \times T = 10 \times 0.01 = \mathbf{0.1 \text{ frames per frame time}}$.
b) Use the Poisson distribution formula for $k=1$: $Pr[1] = \frac{G^1 e^{-G}}{1!}$
$Pr[1] = 0.1 \times e^{-0.1} \approx 0.1 \times 0.9048 = \mathbf{0.09048}$ (or 9.05%).

### Question 2: Binary Exponential Backoff (Math)
Two nodes on an Ethernet network attempt to transmit at the same time and collide. They both apply the Binary Exponential Backoff algorithm. They happen to pick the same random number and collide a second time. 
What is the probability that they will collide on their **third** attempt?

**Solution:**
After the 2nd collision, the collision counter $i = 2$.
The range of slots they can pick from is $[0, 2^2 - 1] =[0, 3]$.
Both stations independently pick a random number from the set $\{0, 1, 2, 3\}$.
* Total possible combinations = $4 \times 4 = 16$.
* They will collide if they pick the exact same number: (0,0), (1,1), (2,2), or (3,3). There are 4 collision combinations.
* Probability = $4 / 16 = 1/4 = \mathbf{0.25}$ (or 25%).

### Question 3: CSMA Protocols (Theory)
Explain the operational difference between 1-persistent CSMA and non-persistent CSMA when a station senses that the channel is currently busy. 

**Solution:**
In **1-persistent CSMA**, if a station senses a busy channel, it continues to listen continuously and transmits immediately (with a probability of 1) the exact moment the channel becomes idle. 
In **non-persistent CSMA**, if a station senses a busy channel, it does not keep listening. Instead, it waits for a completely random period of time before checking the channel again. This results in better channel utilization (fewer collisions) but introduces longer delays.

### Question 4: Ethernet Channel Efficiency (Math)
Consider a classic Ethernet network. Suppose the frame transmission time ($P$) is 200 $\mu s$ and the one-way propagation delay ($\tau$) is 25 $\mu s$. The probability that a station successfully acquires the channel in a contention slot ($A$) is calculated to be 0.0005. 
Calculate the channel efficiency ($\eta$).

**Solution:**
The formula for Ethernet channel efficiency is $\eta = \frac{P}{P + (2\tau / A)}$.
Substitute the values:
$\eta = \frac{200}{200 + (2 \times 25 / 0.0005)}$
$\eta = \frac{200}{200 + (50 / 0.0005)}$
$\eta = \frac{200}{200 + 100,000}$
$\eta = \frac{200}{100,200} \approx \mathbf{0.00199}$
The channel efficiency is approximately **0.199%**.