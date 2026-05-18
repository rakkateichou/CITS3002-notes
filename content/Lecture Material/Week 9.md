# Transport Layer (Part 2) - TCP & Applications

## 1. Demultiplexing Review
How the receiver directs an incoming transport-layer segment to the correct application socket.
*   **UDP (Connectionless):** Uses a **2-tuple** `(Destination IP, Destination Port)`. 
    *   If two different clients send a segment to the same Destination IP/Port, they are directed into the **exact same socket**.
*   **TCP (Connection-Oriented):** Uses a **4-tuple** `(Source IP, Source Port, Destination IP, Destination Port)`.
    *   A server (e.g., a web server on Port 80) spawns a **new, dedicated socket** for every unique client connection.

## 2. TCP: Transmission Control Protocol
Provides reliable, in-order, full-duplex byte-stream delivery.
*   **Segment Structure:**
    *   *Sequence Number:* The byte-stream number of the **first byte** of data in this segment.
    *   *Acknowledgment Number:* The sequence number of the **next byte the receiver expects**. (Cumulative ACKs).
    *   *Receive Window (`rwnd`):* Used for Flow Control.
*   **Connection Management (Handshake):**
    *   *Open:* 3-Way Handshake (`SYN` $\rightarrow$ `SYNACK` $\rightarrow$ `ACK`).
    *   *Close:* Both sides send a `FIN` and reply with an `ACK`.

## 3. TCP Reliability Mechanisms
TCP creates reliable data transfer on top of IP's unreliable service using pipelining, cumulative ACKs, and a single retransmission timer.
*   **Timeout (Lost ACK/Packet):** If the timer expires, TCP retransmits the oldest unacknowledged segment and restarts the timer.
*   **Fast Retransmit:** Timeouts can be long. If a sender receives **3 duplicate ACKs** for the same data, it assumes the packet immediately following that ACK was lost and retransmits it immediately *before* the timer expires.

## 4. TCP Flow Control
Prevents the sender from overwhelming the **receiver's buffer**.
*   The receiver tells the sender how much free space it has left by putting a value in the **`rwnd` (Receive Window)** field of the TCP header.
*   The sender guarantees it will never have more unacknowledged data in flight than the `rwnd` value.

## 5. TCP Congestion Control
Prevents the sender from overwhelming the **entire network** (routers dropping packets due to full queues). 
*   **AIMD (Additive Increase Multiplicative Decrease):** The core philosophy.
    *   *Increase:* Add 1 MSS (Maximum Segment Size) to the Congestion Window (`cwnd`) every RTT (probing for bandwidth).
    *   *Decrease:* Cut `cwnd` in half when loss occurs.
*   **Slow Start Phase:** When a connection begins, `cwnd` starts at 1 MSS. It **doubles** every RTT (exponential growth) until it hits the `ssthresh` (Slow Start Threshold).
*   **Congestion Avoidance (CA) Phase:** Once `ssthresh` is reached, `cwnd` grows linearly (adds 1 MSS per RTT).
*   **Reacting to Loss:**
    *   *Loss by Timeout (Severe congestion):* `ssthresh` is set to `cwnd / 2`. `cwnd` drops all the way back to **1 MSS**, and Slow Start begins again.
    *   *Loss by 3 Duplicate ACKs (Mild congestion - TCP Reno):* `ssthresh` is set to `cwnd / 2`. `cwnd` is cut in half (`cwnd / 2 + 3`) and continues growing linearly (Fast Recovery).

## 6. HTTP & Web Applications (Layer 7)
HTTP (Hypertext Transfer Protocol) is a client/server, **stateless** protocol (the server keeps no memory of past requests).
*   **Response Time Math:**
    *   It always takes **1 RTT** to initiate the TCP connection.
    *   It takes **1 RTT** to send the HTTP request and wait for the first byte of the file to return.
    *   It takes **Transmission Time** to actually push the file data.
    *   *Base Response Time:* **$2 \times RTT + \text{Transmission Time}$**.

### Persistent vs. Non-Persistent HTTP
If a base HTML file references 10 images:
*   **Non-Persistent HTTP:** Opens a brand new TCP connection for *every single object*. 
    *   Time per object = $2 \times RTT + \text{Trans Time}$.
    *   *Total Time:* $11 \times (2 \times RTT + \text{Trans Time})$.
*   **Persistent HTTP:** Opens ONE TCP connection (1 RTT). Sends the base HTML (1 RTT + Trans Time). Keeps the connection open, then sends all 10 images sequentially.
    *   Time for base file = $2 \times RTT + \text{Trans Time}$.
    *   Time for each subsequent file = $1 \times RTT + \text{Trans Time}$.

---

# Sample Exam Questions & Solutions

### Question 1: TCP Congestion Control (Math/Logic)
A TCP sender starts a new connection. The initial `ssthresh` is set to **8 MSS**. 
a) State the value of the Congestion Window (`cwnd`) at RTT 0, RTT 1, RTT 2, and RTT 3. 
b) At RTT 3, does TCP remain in Slow Start or switch to Congestion Avoidance? Why?
c) At RTT 8, `cwnd` reaches **12 MSS**. Suddenly, a **timeout** occurs. What are the new values of `ssthresh` and `cwnd` immediately following the timeout?

**Solution:**
**a)** During Slow Start, `cwnd` doubles every RTT. 
*   RTT 0: `cwnd` = **1 MSS**
*   RTT 1: `cwnd` = **2 MSS**
*   RTT 2: `cwnd` = **4 MSS**
*   RTT 3: `cwnd` = **8 MSS**

**b)** It switches to **Congestion Avoidance**. This happens because `cwnd` (8) has reached the `ssthresh` (8). In Congestion Avoidance, `cwnd` will only increase linearly (+1 MSS per RTT).

**c)** A timeout indicates severe congestion. 
*   `ssthresh` is set to half of the current `cwnd` $\rightarrow 12 / 2 =$ **`6 MSS`**.
*   `cwnd` drops all the way back to **`1 MSS`** (and Slow Start begins again).

### Question 2: HTTP Response Time Math
A user clicks a hyperlink. The requested web page consists of a base HTML file and **6 referenced images**. 
*   The RTT between client and server is **50 ms**. 
*   The transmission time for *each* file (HTML and images) is **20 ms**.
Calculate the total time elapsed from the click until all 7 objects are fully received, assuming the browser uses **Non-persistent HTTP**.

**Solution:**
In Non-persistent HTTP, *every single object* requires its own TCP connection setup.
The time to download one object is $2 \times RTT + \text{TransTime}$.
*   Time per object = $(2 \times 50) + 20 = 100 + 20 = \mathbf{120 \text{ ms}}$.
Since there are 7 total objects (1 HTML + 6 images):
*   Total Delay = $7 \times 120 \text{ ms} = \mathbf{840 \text{ ms}}$.

*(If the question asked for Persistent HTTP: The base file takes $2 \times 50 + 20 = 120 \text{ ms}$. The 6 images use the already-open connection, so they take $1 \times RTT + \text{TransTime}$ each: $6 \times (50 + 20) = 420 \text{ ms}$. Total = $120 + 420 = \mathbf{540 \text{ ms}}$.)*

### Question 3: Flow Control vs. Congestion Control (Theory)
Explain the difference between TCP Flow Control and TCP Congestion Control. Include what specific entity is being protected in each case.

**Solution:**
*   **Flow Control** protects the **receiver**. It prevents a fast sender from transmitting data so quickly that it overflows the receiver's memory buffer. The receiver controls this by advertising its remaining free buffer space in the `rwnd` (Receive Window) field of the TCP header.
*   **Congestion Control** protects the **entire network** (the routers and links between the sender and receiver). It prevents senders from injecting too much data into the network at once, which causes router buffers to overflow and drop packets. The sender controls this by maintaining its own internal `cwnd` (Congestion Window) and adjusting its sending rate based on packet loss.

### Question 4: TCP vs. UDP (Theory)
A developer is creating a live, interactive multiplayer video game. Which transport layer protocol (TCP or UDP) should they choose, and why? Give two reasons.

**Solution:**
They should choose **UDP**. 
1.  **Lower Delay (Timeliness):** Interactive games require real-time responsiveness. UDP has no connection setup handshake delay, and it does not use congestion control, meaning it will send data as fast as the application generates it.
2.  **Loss Tolerance:** In a live game, if a player's positional update packet is lost, it is better to simply wait for the next positional update a millisecond later. If TCP were used, the protocol would pause all incoming data, wait for a timeout, and force a retransmission of the outdated position, causing severe lag and "rubber-banding" in the game.