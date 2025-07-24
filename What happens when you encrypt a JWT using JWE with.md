

# What happens when you encrypt a JWT using JWE with RSA + AES-256-GCM?

Imagine you want to send a secret message (JWT claims) securely to someone (the Gateway).
You want to make sure:

No one else can read the message

Only the Gateway can open (decrypt) it

Step 1: Generate a random secret key for encryption
Before encrypting your actual message, your system creates a random symmetric key (called Content Encryption Key (CEK)).

This key is just a bunch of random bytes, and it will be used to encrypt the actual JWT payload.

Step 2: Encrypt the JWT payload with the symmetric key
Now, the actual JWT payload (claims like username, roles, etc.) is encrypted using AES-256-GCM — this is the EncryptionMethod.A256GCM part.

Why AES-256-GCM?

It’s a fast and very secure way to encrypt data.

It both encrypts and ensures data integrity (detects tampering).

Step 3: Encrypt the symmetric key itself with the Gateway’s public RSA key
Because the symmetric key is secret, you don’t send it in the clear.

So, the symmetric key (CEK) is encrypted using the Gateway’s RSA public key (this is the JWEAlgorithm.RSA_OAEP_256 part).

This means only the Gateway (who has the private RSA key) can decrypt and get the symmetric key.

Step 4: Package everything into a compact JWE token
The final encrypted JWT (JWE) consists of multiple parts:

Header: Info about encryption algorithms used

Encrypted symmetric key: The CEK encrypted with RSA public key

Initialization Vector (IV): Random value used by AES-GCM

Ciphertext: The encrypted JWT payload (claims)

Authentication Tag: Used by AES-GCM to verify integrity

These parts are combined and separated by dots (.), just like normal JWTs.

Step 5: Sending the token to the Gateway
The Gateway receives this encrypted token.

It uses its private RSA key to decrypt the symmetric key (CEK).

Then it uses that symmetric key to decrypt the payload (claims).

Now, the Gateway can read the claims securely.

Summary:
WhatHow (Algorithm)Purpose
Encrypt the actual payloadAES-256-GCM (A256GCM)Encrypt claims efficiently
Encrypt the encryption keyRSA-OAEP-256 (RSA_OAEP_256)Securely share symmetric key

MAKE FORMAT

When you encrypt a JWT using JWE with **RSA + AES-256-GCM**, you implement a hybrid encryption process that ensures only the intended recipient (the Gateway) can read the claims, and no one else can access or tamper with the data in transit.

**Here’s exactly what happens step-by-step:**

### 1. **Generate a Random Symmetric Key (Content Encryption Key, CEK)**

- A random **CEK** is created by your system; this symmetric key will encrypt the JWT payload using AES-256-GCM[^1][^3].


### 2. **Encrypt the JWT Payload With AES-256-GCM**

- The actual claims (such as user info, roles, etc.) are encrypted using the **AES-256-GCM** algorithm. This mode is both secure and fast, providing data confidentiality and integrity (authenticated encryption)[^4][^7][^9].


### 3. **Encrypt the Symmetric Key With RSA-OAEP-256**

- Your generated CEK is encrypted using the recipient Gateway’s **public RSA key** and the **RSA-OAEP-256** algorithm. This protects the symmetric key during transit, ensuring only the Gateway (with the matching private RSA key) can decrypt it[^1][^3].


### 4. **Package as a Compact JWE Token**

The encrypted JWT (now a JWE) consists of five Base64Url-encoded parts, concatenated by dots (`.`)[^1][^3]:

- **Header**: Indicates algorithms used (e.g., `"alg":"RSA-OAEP-256"`, `"enc":"A256GCM"`)
- **Encrypted Key**: The CEK, encrypted using the Gateway’s RSA public key
- **Initialization Vector (IV)**: Random bytes used by AES-GCM for that encryption
- **Ciphertext**: The encrypted JWT payload
- **Authentication Tag**: Ensures data’s integrity, produced by AES-GCM

Example JWE Compact Serialization:

```
eyJhbGciOiJSU0EtT0FFUCIsImVuYyI6IkEyNTZHQ00ifQ.
<encrypted CEK>.
<IV>.
<ciphertext>.
<authentication tag>
```

The token remains unreadable and secure to anyone lacking the private key[^1][^3].

### 5. **Decryption by the Recipient (Gateway)**

- The Gateway receives the JWE and uses its **private RSA key** to decrypt the encrypted CEK[^1][^3].
- Using the decrypted CEK, the Gateway then decrypts the ciphertext (payload) using AES-256-GCM, also providing the IV and authentication tag and, crucially, the JWE header as *Additional Authenticated Data* (AAD)[^3].
- If authentication succeeds, the original claims (payload) are retrieved; if not, decryption fails, signaling tampering[^3][^7].


## **Key Points Table**

| Component | Algorithm | Purpose |
| :-- | :-- | :-- |
| Payload Encryption | AES-256-GCM (A256GCM) | Efficiently encrypts claims and provides integrity |
| CEK Encryption (Key Wrapping) | RSA-OAEP-256 | Allows secure delivery of CEK to recipient only |

### **Security Properties**

- **Confidentiality**: Only the Gateway can read the JWT claims (payload).
- **Integrity**: AES-GCM ensures modifications are detected. Any tampering leads to failed decryption.
- **Recipient Restriction**: Anyone without the corresponding private RSA key cannot recover the CEK or the data[^1][^3][^7][^9].


### **Limitations/Caveats**

- Key management is critical: security depends on the Gateway’s private RSA key being uncompromised[^1][^3].
- Decryption requires correct handling of header (AAD), IV, and authentication tag as per the JWE standard[^3].
- JWT/JWE implementations must be up-to-date and not vulnerable to known attacks or library flaws[^6][^10].

This hybrid encryption flow is the foundation of secure, recipient-specific JWT transmission[^1][^3][^7].


