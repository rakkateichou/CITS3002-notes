# Week 8: Transport Layer - Part 1

## 1. Transport Layer vs. Network Layer
*   **Network Layer (Layer 3):** Provides **logical host-to-host** communication. It routes packets between different machines but offers a "best-effort," unreliable service.
*   **Transport Layer (Layer 4):** Provides **logical process-to-process** communication. It takes the data from the network layer and delivers it to the exact application (process) running on the host. It relies on the Network layer but can enhance it (e.g., by adding reliability).

## 2. Multiplexing & Demultiplexing
How does a computer know which app (e.g., Chrome vs. Spotify) gets which incoming packet?
*   **Multiplexing (Sender):** Gathering data chunks from different sockets, encapsulating each with a Transport Layer header to create segments, and passing them to the Network Layer.
*   **Demultiplexing (Receiver):** Delivering the received segments to the correct socket using the header information.

### Connectionless Demultiplexing (UDP)
*   A UDP socket is identified by a **2-tuple**: `(Destination IP, Destination Port)`.
*   If two UDP segments have different Source IPs or Source Ports, but the *same* Destination IP and Port, they will be directed to the **exact same socket**.

### Connection-Oriented Demultiplexing (TCP)
*   A TCP socket is identified by a **4-tuple**: `(Source IP, Source Port, Destination IP, Destination Port)`.
*   A web server uses this to keep clients separate. Even if three clients all send segments to the server's Port 80, the server will demultiplex them into **three entirely different sockets** based on their unique Source IPs/Ports.

## 3. UDP (User Datagram Protocol)
*   **Features:** "No frills", "Best effort". Segments may be lost or delivered out of order.
*   **Connectionless:** No handshaking/setup delay.
*   **Header:** Very small (8 bytes). No congestion control (can blast data as fast as desired).
*   **Use Cases:** Streaming multimedia (loss tolerant, rate sensitive), DNS.

### UDP Checksum Math (Exam Trap!)
Goal: Detect "flipped bits" in the transmitted segment.
1.  **Format Data:** Divide data into **16-bit words** (4 hexadecimal digits). 
    * *TRAP:* If your final hex group has fewer than 4 digits (e.g., `12`), you must **pad it with zeros ON THE RIGHT** (so `12` becomes `1200`).
2.  **Add Words:** Add the words together using binary addition.
3.  **Wraparound:** If there is a carry-out bit (a 17th bit), you must add it back to the rightmost bit of the result.
4.  **1's Complement:** Flip all the bits (0s become 1s, 1s become 0s) to get the final Checksum.

## 4. Reliable Data Transfer (RDT) Evolution
How do we build a reliable service on top of an unreliable channel?
*   **rdt 1.0 (Perfect Channel):** Assumes no bit errors and no packet loss. Sender sends, receiver receives.
*   **rdt 2.0 (Bit Errors):** Assumes packets can be corrupted. Introduces **Checksums**, **ACKs** (Acknowledged), and **NAKs** (Negative Acknowledgments). 
    *   *Fatal Flaw:* What if the ACK/NAK itself gets corrupted? The sender won't know what to do.
*   **rdt 2.1 (Fixes Corrupted ACKs):** Introduces **Sequence Numbers** (just 0 and 1). If an ACK is garbled, the sender just retransmits the packet. The receiver uses the Sequence Number to know if it's a new packet or a duplicate.
*   **rdt 2.2 (NAK-Free):** Instead of sending NAKs, the receiver just sends an ACK for the *last correctly received* packet, explicitly including its sequence number.
*   **rdt 3.0 (Packet Loss):** Assumes entire packets can vanish. Introduces **Timers**. The sender waits a "reasonable" time for an ACK. If the timer expires (Timeout), it retransmits. This is known as **Stop-and-Wait**.

## 5. Pipelined Protocols (Fixing rdt 3.0)
rdt 3.0 works, but its utilization is terrible. Pipelining allows the sender to have multiple "in-flight" (unacknowledged) packets.
*   **Go-Back-N:** Receiver window = 1. Only accepts in-order packets. If one fails, it drops everything after it. Sender uses a single timer for the oldest unacked packet and retransmits *everything* in the window on timeout.
*   **Selective Repeat:** Receiver window > 1. Buffers out-of-order packets. Sender uses an individual timer for *each* packet and only retransmits the specific lost packet.

## 6. TCP Overview & Segment Structure
*   **Features:** Connection-oriented (handshake), reliable, in-order byte stream, full-duplex. 
*   **Sequence Number:** The byte-stream number of the **first byte** in the segment's data. (e.g., If segment carries bytes 0-999, Seq = 0. The next segment will have Seq = 1000).
*   **Acknowledgment Number:** The sequence number of the **next byte the receiver expects**. (e.g., "I received up to byte 999, so my ACK = 1000").

## 7. TCP Round Trip Time (RTT) & Timeout Math
How does TCP decide how long to set its timer? It uses an **Exponential Weighted Moving Average (EWMA)** to smooth out fluctuations.

**The Three Formulas:**
1.  **Estimated RTT:** 
    $EstRTT_{new} = (1 - \alpha) \times EstRTT_{old} + \alpha \times SampleRTT$
    *(Usually $\alpha = 0.125$)*
2.  **Deviation RTT (Safety Margin):** 
    *Trap: Use the NEWLY calculated EstRTT for this!*
    $DevRTT_{new} = (1 - \beta) \times DevRTT_{old} + \beta \times |SampleRTT - EstRTT_{new}|$
    *(Usually $\beta = 0.25$)*
3.  **Timeout Interval:**
    $Timeout = EstRTT_{new} + 4 \times DevRTT_{new}$

---

# Sample Exam Questions & Solutions

### Question 1: UDP Checksum Math
A UDP segment contains the hexadecimal value `C4A1 8B3F 7`. 
**a)** Divide the data into 16-bit words (4 hex digits each). 
**b)** Convert the final word to a 16-bit binary string.

**Solution:**
**a)** `C4A1`, `8B3F`, and `7000`. 
*(Explanation: The trap here is the final `7`. Because we are reading left-to-right and making 16-bit words, a trailing `7` must be padded with zeros **on the right** to fill out the 4 hex digits, becoming `7000`)*.
**b)** `7` in hex is `0111`. `0` in hex is `0000`. 
Therefore, `7000` in binary is: **`0111 0000 0000 0000`**.

### Question 2: TCP RTT Estimation Math
A TCP sender has a current $EstimatedRTT$ of **100 ms** and a $DevRTT$ of **5 ms**. 
It receives a new ACK, giving a measured $SampleRTT$ of **120 ms**. 
Given $\alpha = 0.125$ and $\beta = 0.25$, calculate the new $EstimatedRTT$, the new $DevRTT$, and the new $TimeoutInterval$.

**Solution:**
*   **Step 1: New Estimated RTT**
    $EstRTT = (1 - 0.125)(100) + 0.125(120)$
    $EstRTT = 0.875(100) + 0.125(120)$
    $EstRTT = 87.5 + 15 = \mathbf{102.5 \text{ ms}}$.
*   **Step 2: New Deviation RTT**
    *(Remember to use the 102.5 we just calculated!)*
    $DevRTT = (1 - 0.25)(5) + 0.25 \times |120 - 102.5|$
    $DevRTT = 0.75(5) + 0.25 \times (17.5)$
    $DevRTT = 3.75 + 4.375 = \mathbf{8.125 \text{ ms}}$.
*   **Step 3: New Timeout Interval**
    $Timeout = EstRTT + 4(DevRTT)$
    $Timeout = 102.5 + 4(8.125) = 102.5 + 32.5 = \mathbf{135 \text{ ms}}$.

### Question 3: Connectionless vs. Connection-Oriented Demultiplexing
A web server is running an Apache HTTP service on Port 80 (TCP) and a DNS service on Port 53 (UDP). 
Two different clients send a DNS request to the server's Port 53. Two other clients send an HTTP request to the server's Port 80.
How many *total sockets* will the server operating system use to handle these four incoming requests? Explain your reasoning. 

**Solution:**
The server will use **3 total sockets**. 
*   **UDP (DNS):** UDP uses a 2-tuple for demultiplexing (Dest IP, Dest Port). Because both DNS requests are going to the same Dest IP and Dest Port (53), they will both be directed into **1 single socket**.
*   **TCP (HTTP):** TCP uses a 4-tuple for demultiplexing (Source IP, Source Port, Dest IP, Dest Port). Because the two HTTP requests come from different clients (different Source IPs/Ports), the server will spawn a dedicated, unique socket for each one. That equals **2 sockets**.

### Question 4: RDT Evolution Theory
In the evolution of the Reliable Data Transfer (rdt) protocol, what specific network flaw was `rdt 2.1` trying to solve by introducing **Sequence Numbers** (0 and 1)?

**Solution:**
`rdt 2.1` introduced sequence numbers to solve the problem of **corrupted ACKs/NAKs**. In `rdt 2.0`, if a receiver's ACK was corrupted on the way back, the sender wouldn't know if the packet arrived safely or not, forcing it to blindly retransmit. The sequence number allows the receiver to look at the retransmitted packet and know if it is a brand-new packet (expected a 1, received a 1) or a duplicate packet (expected a 1, received a 0), allowing it to safely discard the duplicate.