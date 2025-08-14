# Encryption 101
---

## 1. The Concept of Encryption

* **Encryption** is the process of transforming readable data (**plaintext**) into unreadable data (**ciphertext**) so that only authorized parties can read it.
* It’s like putting your message in a locked box — only someone with the right key can open it.

---

## 2. Two Main Types of Encryption

### a) **Symmetric Encryption**

* The same **single key** is used to encrypt and decrypt.
* Example:

  * Key: `12345`
  * Encrypt message: "Hello" → "Xj#29@"
  * Decrypt: "Xj#29@" → "Hello" (using the same key)
* **Pros:** Fast, good for large data.
* **Cons:** Both parties must securely share the same key first — risky.

### b) **Asymmetric Encryption**

* Uses **two keys**: a **public key** and a **private key**.
* They are mathematically linked, but you **can’t figure out the private key from the public key**.
* Public key = lock
* Private key = key that opens the lock

---

## 3. How Public and Private Keys Work

Let’s use an example with you and me:

1. **I have a key pair:**

   * Public Key (everyone can have it)
   * Private Key (only I keep it secret)

2. **If you want to send me a secret message:**

   * You use **my public key** to encrypt it.
   * Only **my private key** can decrypt it.
   * Even you can’t decrypt it once encrypted with my public key.

3. **If I want to prove it’s really me (digital signature):**

   * I use **my private key** to sign a message.
   * Anyone can use **my public key** to verify the signature.
   * If it matches, they know the message really came from me.

---

## 4. In AWS IoT Core

* Each IoT device has its **own private key** (kept secret on the device) and a **public key** (shared via its certificate).
* When a device connects:

  1. AWS IoT sends a challenge (random data).
  2. The device signs it with its **private key**.
  3. AWS IoT verifies the signature using the **public key** in the device certificate.
  4. If valid → trust is established.

---

## 5. Real-World Analogy

* **Public key** = your home address (you can give it to anyone, so they can send you mail).
* **Private key** = your house key (only you can open your mailbox and read the letters).

---


## **so the public key is at the sender side and private key are at the receiver side right?**

Yes — in the **encryption** context, that’s correct:

* **Sender**: Uses the **receiver’s public key** to encrypt the message.
* **Receiver**: Uses their own **private key** to decrypt it.

This ensures:

1. Only the intended receiver can read the message (because only they have the private key).
2. The sender never needs the private key — so it stays secure.

But in **digital signatures** (proving authenticity), it’s the opposite:

* The sender signs data with their **private key**.
* The receiver verifies the signature using the sender’s **public key**.

## who generates the private key?

The **private key** is always generated on the device (or in a secure system) that will *own* it — **never by AWS** in the AWS IoT Core case.

Here’s how it works in different scenarios:

---

### 1. **When registering a device manually**

* The IoT device (or your development computer) runs a key generation command (e.g., using OpenSSL).
* This creates a **private key** (kept on the device) and a matching **public key**.
* You use the **public key** to create a certificate that AWS IoT Core will trust.

---

### 2. **When using AWS IoT Core to generate**

* AWS IoT Core can create a key pair for you **only once** during certificate creation.
* AWS sends you both:

  * **Private key file** (`*.key`) → You must securely install it on your device.
  * **Certificate file** (`*.crt`) → Contains the public key, signed by a trusted CA.
* AWS **never stores the private key** after that moment — if you lose it, you must create a new one.

---

### 3. **When provisioning at scale (JITR/JITP)**

* The device manufacturer’s factory process generates a private key on each device during production.
* The matching public key is signed by the manufacturer’s CA.
* That CA is registered with AWS IoT Core so AWS can trust all devices from that production batch.

---

**Key point for exam & security:**
The private key should **never leave the device** after it’s generated.
If it’s stolen, the attacker can impersonate that device in AWS IoT Core.

---
