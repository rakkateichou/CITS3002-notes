# Application Layer & Network Security

## 1. Web and HTTP (Hypertext Transfer Protocol)
HTTP is the foundation of the World Wide Web. It uses a **Client/Server** model and relies on **TCP** for transport.
*   **Stateless:** HTTP is inherently stateless. The server maintains absolutely *no information* about past client requests. If you ask for the same file twice, the server sends it twice without remembering you.
*   **HTTP Request Message:** Uses ASCII text. Contains a request line (`GET`, `POST`, `HEAD`, `PUT`, `DELETE`), header lines, and an optional body.
*   **HTTP Response Message:** Contains a status line (e.g., `HTTP/1.1 200 OK`), header lines, and the requested data.
    *   *Common Codes:* `200 OK` (Success), `301 Moved Permanently`, `400 Bad Request`, `404 Not Found`.

### Persistent vs. Non-Persistent HTTP
How does HTTP handle a webpage with a base HTML file and 10 images?
*   **Non-Persistent HTTP:** Opens a brand new TCP connection for *every single object*. 
    *   Requires **2 RTTs** per object (1 RTT to open TCP, 1 RTT to request/receive file) + Transmission Time.
*   **Persistent HTTP:** Server leaves the TCP connection open after sending the first response. 
    *   Requires only **1 RTT** for the initial TCP connection, then just 1 RTT per subsequent object.

## 2. Keeping State: Cookies
Because HTTP is stateless, websites use **Cookies** to remember users (for shopping carts, logins, recommendations).
*   **How it works:** 
    1. You visit a site. The server generates a unique ID and creates a backend database entry for you.
    2. The server sends back an HTTP response with a `Set-Cookie: 1678` header.
    3. Your browser saves this cookie file.
    4. On your next visit, your browser includes a `Cookie: 1678` header in its HTTP request. The server reads it, checks its database, and remembers who you are.

## 3. Web Caching (Proxy Servers)
A Web Cache acts as both a client and a server. It is usually installed by an ISP or a University.
*   **Goal:** Satisfy a client request *without* involving the original origin server.
*   **How it works:** You request a webpage. The request goes to the Proxy Server. If the proxy has a saved copy of the page, it serves it to you immediately. If not, the proxy downloads it from the origin server, saves a copy for itself, and then passes it to you.
*   **Why?** Drastically reduces response time for clients and reduces traffic on the institution's external network link.

## 4. Other Application Layer Protocols
*   **FTP (File Transfer Protocol):** Transfers files between hosts. It uses **Out-of-Band Control**. 
    *   It opens a TCP connection on **Port 21** just for control commands (login, browse directory).
    *   When a file is requested, it opens a *second* temporary TCP connection on **Port 20** to send the actual data.
    *   **Stateful:** The FTP server remembers your current directory and authentication status.
*   **SMTP (Simple Mail Transfer Protocol):** Used to send emails between mail servers. Uses **TCP on Port 25**.
    *   Has 3 phases: Handshaking, Transfer, Closure.

## 5. DNS (Domain Name System)
Translates human-readable hostnames (e.g., `www.google.com`) into IP addresses (e.g., `142.250.190.36`).
*   **Structure:** It is a **Distributed, Hierarchical Database**. It is *not* centralized (to avoid a single point of failure and massive traffic delays).
*   **The Hierarchy:**
    1.  **Root DNS Servers:** Over 400 worldwide. They direct queries to the correct TLD.
    2.  **TLD (Top-Level Domain) Servers:** Responsible for `.com`, `.edu`, `.net`, `.au`, etc.
    3.  **Authoritative Servers:** The organization's own DNS server, holding the actual specific IP mappings for its hosts.
*   **Local DNS Server:** Your ISP or University's default server. It acts as a proxy, doing the legwork of traversing the hierarchy for you and caching the results.

## 6. Network Security & Cryptography
*   **The Goals:** Confidentiality (encryption), Authentication (proving who you are), Message Integrity (proving the message wasn't tampered with).
*   **Symmetric Key (e.g., AES, DES):** Both Alice and Bob use the *exact same* secret key to encrypt and decrypt. Fast, but dangerous because they must safely share the key first.
*   **Asymmetric Key (Public Key, e.g., RSA):** Everyone has two keys. A **Public Key** (known to the whole world) and a **Private Key** (kept totally secret).
    *   If Alice wants to send a secret to Bob, she encrypts it using **Bob's Public Key**.
    *   Only **Bob's Private Key** can decrypt it.

### RSA Math (Guaranteed Exam Question)
1.  Choose two prime numbers: $p$ and $q$.
2.  Compute the modulus: $n = p \times q$.
3.  Compute $z = (p-1) \times (q-1)$.
4.  Choose $e$ (Public exponent) that has no common factors with $z$.
5.  Choose $d$ (Private exponent) such that $(e \times d) \mod z = 1$.
    *   **Public Key:** $(n, e)$
    *   **Private Key:** $(n, d)$
6.  **To Encrypt a message ($m$):** $c = m^e \mod n$
7.  **To Decrypt ciphertext ($c$):** $m = c^d \mod n$

---

# Sample Exam Questions & Solutions (Lab 10)

### Question 1: DNS Resolution (Iterative)
Explain how the hierarchical DNS system resolves the web address `https://www.handbooks.uwa.edu.au` using an iterative DNS query process, assuming the host first sends the query to a local DNS server. 

**Solution:**
1. The host sends a DNS query to its **Local DNS Server**.
2. If the Local DNS server does not have the IP cached, it queries a **Root DNS Server**. The Root server replies with the IP address of the **TLD DNS Server** responsible for `.au` (or `.edu.au`).
3. The Local DNS server then queries the **TLD DNS Server**. The TLD server replies with the IP address of the **Authoritative DNS Server** specifically managed by `uwa.edu.au`.
4. Finally, the Local DNS server queries the **Authoritative DNS Server**, which returns the exact IP address mapping for the specific host `www.handbooks`.
5. The Local DNS server passes this IP back to the requesting host and caches it for future use.

### Question 2: DNS & Web Performance
A university uses a local DNS server inside the campus network. As the number of students increases, many users repeatedly access the same popular website.
a) What solution could the university implement to improve DNS performance?
b) Which solution could the university implement to reduce network traffic and improve response time for frequently accessed web content?

**Solution:**
**a)** **DNS Caching.** The local DNS server can cache recent name-to-IP address translations. When a second student requests the same popular website, the local DNS server can provide the IP address immediately from its cache without having to query the Root, TLD, or Authoritative servers again.
**b)** **Web Caching (Proxy Server).** The university can install a Proxy Server on campus. It stores local copies of frequently requested web objects (like images and HTML files). When a student requests a page, the Proxy serves it directly from the campus network, drastically reducing external internet traffic and speeding up response times.

### Question 3: HTTP State
A university develops an online learning platform. After authentication, students expect the website to remember their login session and personalized settings while they browse different pages. Which mechanism could the university use to maintain user session information between different HTTP requests? Explain how this mechanism works.

**Solution:**
The university should use **Cookies**, because HTTP is an inherently "stateless" protocol. 
**How it works:** When the student successfully logs in, the web server generates a unique session ID and stores it in its backend database. The server sends an HTTP response back to the student containing a `Set-Cookie: [ID]` header. The student's web browser stores this cookie. On every subsequent click or page request, the browser automatically includes a `Cookie: [ID]` header. The server reads this ID, looks up the session in its database, and "remembers" that the student is already logged in.

### Question 4: Stateful vs. Stateless Protocols
Classify the following protocols as either stateless or stateful. Briefly justify your answer based on whether the protocol keeps information about previous communication states: HTTP, TCP, UDP, FTP.

**Solution:**
*   **HTTP: Stateless.** A web server treats every incoming request as an entirely new, independent event and retains no memory of previous client requests.
*   **TCP: Stateful.** It establishes a connection via a 3-way handshake and tracks the continuous state of the connection using Sequence numbers, Acknowledgment numbers, and window sizes until the connection is officially closed.
*   **UDP: Stateless.** It fires individual datagrams without establishing a connection, tracking sequence numbers, or confirming delivery.
*   **FTP: Stateful.** It maintains an ongoing control connection (Port 21) throughout the session, remembering the user's authentication status and their current remote directory while data transfers occur on separate ports.

### Question 5: RSA Encryption Math (60% of Exam is Math!)
By considering the values of $p=17$ and $q=11$, use the RSA algorithm to answer these questions:
a) What are the values of $n$ and $z$ (used for RSA encryption)?
b) By considering $e=7$ and $d=23$, encrypt the binary value `00101101`.
c) Decrypt your answer for part b.

**Solution:**
**a) Find $n$ and $z$:**
*   $n = p \times q \implies 17 \times 11 = \mathbf{187}$.
*   $z = (p-1)(q-1) \implies (16) \times (10) = \mathbf{160}$.

**b) Encrypt the binary value `00101101`:**
*   *Step 1:* Convert binary to decimal. $32 + 8 + 4 + 1 = \mathbf{45}$. So, our message $m = 45$.
*   *Step 2:* Apply the encryption formula: $c = m^e \mod n$.
*   $c = 45^7 \mod 187$.
*   *Math trick (Modular Exponentiation):* Break $45^7$ down so your calculator doesn't overflow!
    *   $45^1 = 45$
    *   $45^2 = 2025 \equiv 155 \pmod{187}$
    *   $45^4 = 155^2 = 24025 \equiv 89 \pmod{187}$
    *   $45^7 = 45^4 \times 45^2 \times 45^1 = 89 \times 155 \times 45$
    *   $89 \times 155 = 13795 \equiv 144 \pmod{187}$
    *   $144 \times 45 = 6480 \equiv \mathbf{122} \pmod{187}$
*   **The encrypted ciphertext ($c$) is 122.**

**c) Decrypt the ciphertext:**
*   *Formula:* $m = c^d \mod n$.
*   $m = 122^{23} \mod 187$.
*   *Math trick:* Break down $122^{23}$.
    *   $122^1 = 122$
    *   $122^2 = 14884 \equiv 111 \pmod{187}$
    *   $122^4 = 111^2 = 12321 \equiv 166 \pmod{187}$
    *   $122^8 = 166^2 = 27556 \equiv 67 \pmod{187}$
    *   $122^{16} = 67^2 = 4489 \equiv 1 \pmod{187}$
    *   $122^{23} = 122^{16} \times 122^4 \times 122^2 \times 122^1$
    *   $122^{23} = 1 \times 166 \times 111 \times 122$
    *   $166 \times 111 = 18426 \equiv 100 \pmod{187}$
    *   $100 \times 122 = 12200 \equiv \mathbf{45} \pmod{187}$
*   **The decrypted message ($m$) is 45**, which converts perfectly back to the binary `00101101`.