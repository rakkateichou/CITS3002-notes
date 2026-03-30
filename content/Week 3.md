# Data Link Layer & Flow Control

## 1. Introduction to the Data Link Layer (Layer 2)
The Data Link Layer sits between the Physical Layer (Layer 1) and the Network Layer (Layer 3). Its main job is to provide reliable transmission across a single wire or wireless link.
* **Key Services:**
  1. **Framing:** Bundling and unbundling groups of raw bits into structured containers called *frames*.
  2. **Flow Control:** Controlling the sending rate to ensure the receiver is **not overwhelmed** (flooded) by a fast sender.
  3. **Error Control:** Detecting and correcting "higher-level" transmission errors (e.g., lost frames, out-of-order frames).
* **MAC (Media Access Control):** The protocol responsible for how devices access the shared communication medium. 
* **MAC Address:** A unique link-layer identifier used to address physical network interfaces.

## 2. Framing & Encapsulation
* **Encapsulation:** The process of wrapping a Network Layer *packet* with **header** and **trailer** information to create a Data Link *frame*.
* **Standard Frame Structure:**
  * **Flag:** Indicates the start and end of the frame.
  * **Header:** Source/Destination MAC addresses and control info.
  * **Payload (Data):** The actual packet from the Network Layer.
  * **Trailer:** Error detection/correction code (e.g., CRC checksum).

### Ethernet Frame Specifics (IEEE 802.3)
* **Preamble & Start-of-Frame Delimiter:** 8 bytes total (used for physical clock synchronization).
* **Payload Size:** Variable, strictly between **42 bytes and 1500 bytes**. (There is no length field; the transceiver just grabs the whole frame).
* **Interframe Gap:** A required silent period of at least **96 bits** where no data is transmitted, allowing the receiver hardware to reset.

## 3. Levels of Data Link Complexity
1. **Unacknowledged Connectionless (Simplex):** Sender sends frames without waiting for ACKs. No attempt to recover lost frames. (Used by modern Ethernet; leaves error resolution to higher layers like TCP).
2. **Acknowledged Connectionless (Half-Duplex):** Each frame is individually acknowledged. Lost frames are retransmitted after a timeout.
3. **Acknowledged Connection-Oriented (Full-Duplex):** Frames are numbered and guaranteed to be received exactly once and in the correct order.

## 4. Stop-and-Wait Protocol
The simplest flow control protocol. The sender transmits **one frame** and then *stops and waits* for an Acknowledgment (ACK) before sending the next one.
* **Dealing with Corruption:** Uses a checksum (CRC). If the receiver detects a bad checksum, it drops the frame or sends a Negative ACK (NACK). 
* **Dealing with Frame Loss:** Uses **Timers**. If a data frame or its ACK is lost, the sender will be left waiting forever. To fix this, the sender starts a timer. If the timer expires (Timeout) before an ACK arrives, the sender **retransmits** the frame.
* *Programming Note:* Moving from noiseless to noisy channels requires changing from iterative programming to **event-driven** programming to handle timeouts.

## 5. Improving Efficiency
Stop-and-wait is highly inefficient, especially over long distances (like a satellite link with a 270ms propagation delay) because the sender is mostly idle.
* **Piggybacking:** Instead of sending small, separate ACK frames (which waste bandwidth and cause extra CPU interrupts), the receiver waits until it has its own outgoing Data frame and "hitches" or *piggybacks* the ACK onto that outgoing frame's header.

## 6. Sliding Window Protocols (Pipelining)
To fix the idle time of Stop-and-Wait, Sliding Window protocols allow the sender to transmit **multiple frames** (a window) before needing an acknowledgment.

### Go-Back-N (GBN)
* **Receiver Window Size = 1.**
* The receiver only accepts frames in perfect, sequential order.
* If a frame is lost (e.g., Frame 3), it simply **discards** all subsequent frames (4, 5, 6) even if they arrive perfectly. 
* When the sender's timer expires, it goes back and **retransmits N frames** (3, 4, 5, 6...). 
* *Con:* Wastes bandwidth on noisy channels.

### Selective Repeat (SR)
* **Receiver Window Size > 1.**
* The receiver is smart. It **buffers** out-of-order frames.
* If Frame 3 is lost but 4 and 5 arrive, it buffers 4 and 5 and sends a NAK for 3.
* The sender **only retransmits the specific lost frame** (Frame 3).
* *Pro:* Much more efficient on noisy channels.

---

# Sample Exam Questions & Solutions

### Question 1: Flow Control Logic (Theory)
A sender is transmitting a sequence of numbered frames using a sliding window protocol. Frame 5 is corrupted during transmission, but Frames 6, 7, and 8 arrive flawlessly. 
a) Explain how a **Go-Back-N** receiver will react to the arrival of Frames 6, 7, and 8.
b) Explain how a **Selective Repeat** receiver will react to the arrival of Frames 6, 7, and 8.

**Solution:**
a) A **Go-Back-N** receiver has a receiving window size of 1, meaning it only accepts frames in strict sequence. Because Frame 5 is missing, it will **discard** Frames 6, 7, and 8 and refuse to acknowledge them. The sender will eventually timeout and retransmit Frames 5, 6, 7, and 8.
b) A **Selective Repeat** receiver has a window size greater than 1. It will **buffer** Frames 6, 7, and 8, keep them in memory, and request a retransmission specifically for Frame 5 (often by sending a NAK for Frame 5). Once Frame 5 successfully arrives, it will process 5, 6, 7, and 8 together.

### Question 2: Stop-and-Wait Efficiency (Math)
A satellite link has a bandwidth of 2 Mbps (2,000,000 bps) and a one-way propagation delay of 250 ms (0.25 seconds). The sender uses the Stop-and-Wait protocol to send frames that are 10,000 bits in length. Calculate the channel utilization (efficiency) of this link. 

**Solution:**
1. **Calculate Transmission Delay ($T_{trans}$):**
   $T_{trans} = L / R = 10,000 \text{ bits} / 2,000,000 \text{ bps} = 0.005 \text{ seconds (or 5 ms)}$
2. **Calculate Propagation Delay ($T_{prop}$):**
   $T_{prop} = 250 \text{ ms} = 0.25 \text{ seconds}$
3. **Calculate Utilization ($U$):**
   The formula for Stop-and-Wait efficiency is $U = \frac{T_{trans}}{T_{trans} + 2 \times T_{prop}}$
   $U = \frac{0.005}{0.005 + 2(0.25)}$
   $U = \frac{0.005}{0.005 + 0.50}$
   $U = \frac{0.005}{0.505} \approx \mathbf{0.0099}$
   **Answer:** The channel utilization is approximately **0.99%**.

### Question 3: Piggybacking (Theory)
In the context of the Data Link layer, what is "piggybacking" and what two specific overheads does it reduce?

**Solution:**
**Piggybacking** is a technique where a receiver temporarily delays sending an acknowledgment (ACK) for a received frame until it has its own data frame to send back to the sender. The ACK is then embedded (piggybacked) into the header of that outgoing data frame. 
This reduces two main overheads:
1. It saves **network bandwidth** by reducing the total number of independent DL_ACK frames traveling on the network.
2. It reduces the **load on the operating system** by decreasing the number of hardware interrupts that the CPU must process for incoming tiny ACK frames.

### Question 4: Ethernet Encapsulation
An Ethernet frame payload has a strict minimum size requirement of 42 bytes. Briefly explain the purpose of the 96-bit "Interframe gap" that follows an Ethernet transmission.

**Solution:**
The 96-bit **Interframe gap** is a mandatory silent period on the medium during which no data is transmitted. Its purpose is to provide a brief recovery window that allows the receiving hardware's transceivers to reset and prepare for the next incoming frame.