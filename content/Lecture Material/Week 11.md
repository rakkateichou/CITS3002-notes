# Application Layer (Part 2) - Network Security

## 1. The Goals of Network Security
When Alice and Bob want to communicate securely, they need to protect themselves against intruders (Trudy) who might intercept, delete, or insert messages. 
*   **Confidentiality:** Only the sender and intended receiver should understand the message contents (Encryption).
*   **Authentication:** The sender and receiver need to confirm each other's identity.
*   **Message Integrity:** Ensuring the message was not altered in transit without detection.

## 2. The Evolution of Authentication Protocols
How does Bob prove to Alice that he is actually Bob?
*   **ap1.0:** Alice says "I am Alice." $\rightarrow$ *Fail:* Trudy can just say "I am Alice."
*   **ap2.0:** Alice sends an IP packet with her Source IP address. $\rightarrow$ *Fail:* Trudy can spoof (fake) the IP address.
*   **ap3.0:** Alice sends a secret password. $\rightarrow$ *Fail:* **Playback/Replay Attack**. Trudy records the packet and plays it back to Bob later.
*   **ap3.1:** Alice sends an *encrypted* password. $\rightarrow$ *Fail:* Playback attack still works! Trudy doesn't need to decrypt it; she just records the encrypted packet and plays it back.
*   **ap4.0:** Use a **Nonce** ($R$), a number used only *once in a lifetime*. Bob sends Alice $R$. Alice encrypts $R$ with a shared symmetric key and sends it back. 
    *   *Drawback:* Requires a shared symmetric key to be established securely beforehand.
*   **ap5.0:** Use a Nonce and **Public Key Cryptography**. Alice encrypts $R$ with her *Private Key*. Bob decrypts it with Alice's *Public Key*. 
    *   *Fatal Flaw:* **Man-in-the-Middle (MITM) Attack**.

### The Man-in-the-Middle Attack (MITM)
In `ap5.0`, Bob asks Alice for her public key. Trudy intercepts the request, and sends Bob **Trudy's public key**, pretending it is Alice's. 
Now, when Bob sends a message encrypted with what he *thinks* is Alice's public key, Trudy can intercept it, decrypt it, read it, re-encrypt it with the *real* Alice's public key, and send it to Alice. Neither Bob nor Alice knows Trudy is sitting in the middle reading everything!

## 3. Digital Signatures & Message Digests
*   **Digital Signature:** A cryptographic technique analogous to a handwritten signature. It provides **verifiability** and **non-repudiation** (Alice can take Bob to court to prove he sent the message).
    *   *How it works:* Bob encrypts the message using his **Private Key**. Anyone can decrypt it using his **Public Key**, proving that only Bob could have sent it.
*   **Message Digest (Hash):** Encrypting large messages with public keys is computationally expensive/slow. Instead, we use a Hash Function ($H$) to create a fixed-size "fingerprint" of the message.
    *   *Properties:* Many-to-1, fixed size, computationally infeasible to reverse-engineer the original message from the hash.
    *   *Algorithms:* MD5 (128-bit), SHA-1 (160-bit).
    *   *(Note: The Internet Checksum is a hash, but it's terrible for security because it's too easy to find a different message that results in the exact same checksum).*

### The "Signed Message Digest" Workflow
1. Bob creates a message $m$.
2. Bob hashes the message: $H(m)$.
3. Bob digitally signs the hash using his private key: $K_B^-(H(m))$.
4. Bob sends the plaintext $m$ and the encrypted signature together.
5. Alice receives both. She hashes the plaintext $m$ herself. She decrypts Bob's signature using Bob's public key. If the two hashes match, the message is authentic and unaltered!

## 4. Certification Authorities (CAs)
How do we solve the Man-in-the-Middle attack? How does Bob *know* the public key actually belongs to Alice and not Trudy?
*   **Certification Authority (CA):** A trusted third party that binds a public key to a specific entity.
*   Alice registers her public key with the CA and proves her real-life identity.
*   The CA creates a **Certificate** containing Alice's public key, and digitally signs it using the **CA's Private Key**.
*   When Bob wants to talk to Alice, he gets her certificate. He uses the CA's widely known public key to verify the certificate. Since he trusts the CA, he now trusts that Alice's public key is genuine.

## 5. Secure E-mail (PGP - Pretty Good Privacy)
Combines all the above concepts to send secure emails. 
To provide **secrecy, sender authentication, and message integrity**, Alice uses exactly **three keys**: her private key, Bob's public key, and a newly generated, one-time symmetric session key.

*   **Integrity/Authentication:** Alice hashes the email and signs the hash with her **Private Key**. She attaches this to the email.
*   **Secrecy (Efficiency):** Public key crypto is too slow for large files. Alice generates a random **Symmetric Key ($K_S$)** and encrypts the whole email (plus signature) with it.
*   **Key Exchange:** Alice encrypts the Symmetric Key ($K_S$) using **Bob's Public Key**. 
*   She sends the encrypted email and the encrypted Symmetric Key to Bob.

## 6. SSL / TLS (Secure Sockets Layer)
An enhanced, widely deployed version of TCP that provides confidentiality, integrity, and authentication (the "S" in HTTPS).
*   **Handshake:** Sender and receiver use certificates/private keys to authenticate and exchange a shared secret.
*   **Negotiation:** Client and server agree on which cipher suite to use (e.g., AES for symmetric, RSA for public key).
*   **Key Derivation:** They use the shared secret to generate the symmetric keys used for the actual fast data transfer.

---
## 🚨 FINAL EXAM STRATEGY ALERT (From Slide 32)
*   **Format:** 2 hours. NO programming questions.
*   **Distribution:** **75% Calculation-based**, **25% Short-answer/Design**.
*   **Materials:** UWA-approved calculator + **TWO double-sided handwritten A4 cheat sheets**.
*   **Advice:** The lab materials and mid-semester exams are **EXTREMELY IMPORTANT**. Put every math formula (Delay, CRC, Hamming, ALOHA, Ethernet Efficiency, TCP RTT/Timeout, Subnetting, RSA) on your cheat sheet, along with a step-by-step example of Dijkstra and DVR!

---

# Sample Exam Questions & Solutions

### Question 1: Authentication Protocol Flaws (Theory)
In the development of authentication protocols, Alice attempts to authenticate herself to Bob by sending an encrypted password over the network (Protocol `ap3.1`). Explain why encrypting the password does not make the protocol secure against an eavesdropper (Trudy). Mention the specific type of attack Trudy would use.

**Solution:**
Encrypting the password does not make the protocol secure because it is still vulnerable to a **Playback (or Replay) Attack**. Trudy does not need to decrypt or understand the password. She simply acts as a packet sniffer, records the encrypted packet as it travels over the network, and later sends that exact same encrypted packet to Bob. Bob will decrypt it, see the correct password, and falsely authenticate Trudy as Alice.

### Question 2: Secure Email & PGP (Design/Logic)
Alice wants to send a highly confidential, 1-Gigabyte video file to Bob. She has Bob's Public Key. Instead of simply encrypting the 1GB file with Bob's Public Key, her email software (PGP) generates a brand-new Symmetric Key, encrypts the file with the Symmetric Key, and then encrypts the Symmetric Key with Bob's Public Key.
Explain the primary technical reason why the PGP protocol uses this two-step process instead of just using Bob's Public Key for everything.

**Solution:**
The primary reason is **computational efficiency**. Asymmetric (Public Key) cryptography requires complex mathematical operations (like modular exponentiation with massive prime numbers) which are incredibly CPU-intensive and slow. It is computationally prohibitive to encrypt a massive 1GB file using a Public Key. Symmetric Key cryptography (like AES) is exponentially faster. Therefore, PGP uses the fast Symmetric key for the heavy lifting (encrypting the bulk data), and only uses the slow Public Key cryptography to securely transmit the tiny symmetric key itself. 

### Question 3: Digital Signatures vs. Encryption (Theory)
Assume Alice wants to send a digitally signed message to Bob to provide *Non-repudiation* and *Message Integrity*. 
a) Which key does Alice use to sign the message digest (hash)?
b) Which key does Bob use to verify the signature?
c) Does a digital signature provide *Confidentiality* (secrecy) for the message payload? Explain why or why not.

**Solution:**
**a)** Alice uses **her own Private Key** to encrypt/sign the message digest.
**b)** Bob uses **Alice's Public Key** to decrypt the signature and verify the digest.
**c)** **No**, a digital signature does not provide confidentiality. A digital signature is just a cryptographic hash appended to the message; the message payload itself is sent in plain text. Anyone eavesdropping on the network can read the message, even if they cannot alter it or forge the signature. (To get confidentiality, the entire message would need to be encrypted *after* being signed).

### Question 4: Certification Authorities (Theory)
Explain the specific attack that a Certification Authority (CA) is designed to prevent, and briefly describe how a CA achieves this.

**Solution:**
A CA is designed to prevent the **Man-in-the-Middle (MITM) attack**. In a MITM attack, an intruder (Trudy) intercepts a request for a public key and substitutes her own public key, tricking the sender into encrypting data so Trudy can read it. 
A CA prevents this by acting as a trusted third party that physically verifies a person's identity and then digitally signs a Certificate binding that person's identity to their actual public key. When Bob receives Alice's public key, it is wrapped in this Certificate. Bob uses the CA's widely known, trusted public key to verify the CA's signature on the certificate, mathematically proving that the public key inside genuinely belongs to Alice and has not been tampered with or substituted by Trudy.