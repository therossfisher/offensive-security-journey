# Cryptography Fundamentals

**Context:** Core concepts for security work — understanding encryption, hashing, key exchange, and digital signatures. Relevant for CTFs, pentesting, and defensive security.

---

## The Three Concepts — Don't Confuse Them

| Concept | Reversible? | Key Required? | Purpose |
|---|---|---|---|
| **Hashing** | No | No | Integrity verification, password storage |
| **Encoding** | Yes | No | Data format conversion (Base64, ASCII) |
| **Encryption** | Yes | Yes | Confidentiality |

**Hashing** — one way, can't reverse it, fixed size output  
**Encoding** — just format conversion, no security, anyone can decode  
**Encryption** — protects confidentiality, requires key to reverse

---

## Hashing

### What It Does
Takes input of any size → produces fixed size output (the hash/digest). Same input always produces same output. Change one bit of input → completely different output.

### Why It Matters
- Password storage — servers never store your actual password, only the hash
- File integrity — verify a downloaded file matches the original
- Finding duplicates — same hash = same file

### Common Hash Algorithms

| Algorithm | Output Size | Status |
|---|---|---|
| MD5 | 128 bits / 16 bytes | Broken — don't use for security |
| SHA1 | 160 bits / 20 bytes | Broken — don't use for security |
| SHA256 | 256 bits / 32 bytes | Secure |
| SHA512 | 512 bits / 64 bytes | Secure |
| bcrypt | 184 bits | Secure — designed for passwords |
| Argon2 | Variable | Secure — current best practice for passwords |

### Hash Collisions
When two different inputs produce the same hash. Mathematically unavoidable (pigeonhole effect) but good algorithms make collisions computationally impractical to engineer. MD5 and SHA1 have known collision attacks — don't trust them.

### Linux Password Hashes
Stored in `/etc/shadow` (root only). Format: `$prefix$options$salt$hash`

| Prefix | Algorithm |
|---|---|
| `$y$` | yescrypt — current default on modern Linux |
| `$2b$`, `$2a$` | bcrypt |
| `$6$` | SHA512crypt |
| `$1$` | MD5crypt — old, insecure |

### Salting
A random value added to the password before hashing. Stored alongside the hash. Prevents rainbow table attacks — even identical passwords produce different hashes.

```
password + random_salt → hash(password+salt) → store hash + salt
```

### Identifying Hashes on Kali
```bash
hashid HASH
hash-identifier
```

Or use context — web app database = likely MD5, Linux shadow = likely bcrypt or SHA512

---

## Hash Cracking

### Tools
- **hashcat** — GPU accelerated, fastest
- **John the Ripper** — CPU based, works in VMs
- **Online:** crackstation.net, hashes.com (rainbow tables — only works without salt)

### Hashcat Syntax
```bash
hashcat -m <mode> -a <attack_mode> hashfile wordlist
```

**Attack modes:**
- `-a 0` — dictionary attack (straight)
- `-a 3` — brute force

**Common hash modes:**

| Mode | Hash Type |
|---|---|
| 0 | MD5 |
| 100 | SHA1 |
| 1400 | SHA2-256 |
| 1700 | SHA2-512 |
| 1800 | SHA512crypt (`$6$`) — Linux shadow |
| 3200 | bcrypt (`$2a$`) |
| 1000 | NTLM (Windows) |
| 1750 | HMAC-SHA512 (key = $pass) |
| 2410 | Cisco-ASA MD5 |

**Find any hash mode:**
```bash
hashcat --example-hashes | grep -i "ALGORITHM_NAME" -B 2
```

**Example commands:**
```bash
# MD5
hashcat -a 0 -m 0 hash.txt /usr/share/wordlists/rockyou.txt --force

# bcrypt
hashcat -a 0 -m 3200 hash.txt /usr/share/wordlists/rockyou.txt --force

# SHA512crypt (Linux shadow)
hashcat -a 0 -m 1800 hash.txt /usr/share/wordlists/rockyou.txt --force
```

**Note on VMs:** Hashcat runs slowly in VMs — no GPU access. For speed run on host OS. John the Ripper works fine in VMs (CPU based).

### Extract rockyou.txt if needed
```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

### Checking File Integrity
```bash
sha256sum filename
md5sum filename
sha1sum filename
```

---

## Symmetric vs Asymmetric Encryption

### Symmetric
- Same key encrypts and decrypts
- Fast
- Problem: how do you share the key securely?
- Examples: AES, DES

### Asymmetric (Public Key)
- Two keys: public key (share freely) and private key (never share)
- Public key encrypts, private key decrypts
- Slower than symmetric
- Used to securely exchange symmetric keys
- Examples: RSA, ECC

### How They Work Together
1. Asymmetric encryption used once to exchange a symmetric key securely
2. Symmetric encryption used for the rest of the conversation
3. Best of both worlds — security of asymmetric + speed of symmetric

---

## RSA

### Key Variables
| Variable | Meaning |
|---|---|
| p, q | Two large prime numbers (kept secret) |
| n | p × q (public modulus) |
| e | Public exponent (part of public key) |
| d | Private exponent (part of private key) |
| m | Plaintext message |
| c | Ciphertext |
| ϕ(n) | Euler's totient — (p-1)(q-1) |

### Key Formulas
```
n = p × q
ϕ(n) = (p-1)(q-1)
Public key = (n, e)
Private key = (n, d)
```

### Why RSA Is Secure
Multiplying two large primes is easy. Factoring their product back into the primes is computationally impractical with numbers large enough (300+ digits each). An attacker who knows n can't calculate ϕ(n) without knowing p and q.

### RSA CTF Tools
- **RsaCtfTool** — github.com/RsaCtfTool/RsaCtfTool
- **rsatool** — for generating and solving RSA parameters

---

## Diffie-Hellman Key Exchange

Allows two parties to establish a shared secret over an insecure channel without transmitting the secret itself.

### Process
1. Alice and Bob agree on public values: large prime `p` and generator `g`
2. Each picks a private key (a for Alice, b for Bob)
3. Each calculates their public key:
   - Alice: `A = g^a mod p`
   - Bob: `B = g^b mod p`
4. They exchange public keys
5. Each calculates the shared secret:
   - Alice: `B^a mod p`
   - Bob: `A^b mod p`
   - Both equal `g^(ab) mod p` — the shared secret

### Why It's Secure
An eavesdropper sees g, p, A, and B but cannot determine a or b (discrete logarithm problem). Without a or b they cannot calculate the shared secret.

### In Practice
Diffie-Hellman handles key agreement. RSA handles digital signatures and authentication. Combined in protocols like TLS/HTTPS.

---

## HMAC

**Hash-based Message Authentication Code** — combines a hash function with a secret key to verify both authenticity and integrity.

```
HMAC(K, M) = H((K⊕opad) || H((K⊕ipad) || M))
```

- **Integrity** — hash proves the message hasn't been modified
- **Authenticity** — secret key proves who created it

**Hashcat modes:**
- HMAC-SHA512 (key = $pass): mode **1750**
- HMAC-SHA256 (key = $pass): mode **1450**

---

## SSH Keys

### Key Types
| Algorithm | Notes |
|---|---|
| RSA | Default, works everywhere, longer keys |
| Ed25519 | Modern, shorter keys, same security |
| ECDSA | Elliptic curve variant |
| DSA | Old, deprecated |

### Generate a Key Pair
```bash
ssh-keygen -t ed25519
ssh-keygen -t rsa -b 4096
```

### Use a Specific Key
```bash
ssh -i /path/to/privatekey user@host
```

### Permissions Must Be Correct
```bash
chmod 600 ~/.ssh/id_rsa        # private key
chmod 644 ~/.ssh/id_rsa.pub    # public key
chmod 700 ~/.ssh/               # directory
```

### Authorized Keys
`~/.ssh/authorized_keys` — public keys that are allowed to SSH in. Adding your public key here gives you passwordless SSH access.

### CTF Use
Finding an SSH private key during enumeration is a direct path to authentication. Always check:
```bash
find / -name "id_rsa" 2>/dev/null
find / -name "*.pem" 2>/dev/null
find / -name "authorized_keys" 2>/dev/null
```

### Crack SSH Key Passphrase
```bash
ssh2john id_rsa > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

---

## Digital Signatures and Certificates

### Digital Signatures
Prove authenticity and integrity of a message or file.
1. Sign with private key
2. Verify with public key
3. Only the private key holder could have created it

### Certificates
Used in HTTPS to prove a server is who it claims to be. Chain of trust:
- Root CA (trusted by OS/browser by default)
- Intermediate CA
- Server certificate

### TLS Certificates
- **Let's Encrypt** — free TLS certificates for domains you own
- Certificates contain the server's public key signed by a CA

---

## GPG / PGP

**GPG (GnuPG)** — open source implementation of OpenPGP standard. Used for encrypting files and email, and digital signing.

```bash
# Generate key pair
gpg --full-gen-key

# Import a key
gpg --import backup.key

# Decrypt a file
gpg --decrypt confidential_message.gpg

# Encrypt a file for someone (using their public key)
gpg --encrypt --recipient email@address.com file.txt
```

### Crack GPG Passphrase
```bash
gpg2john encrypted.gpg > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

---

## Base64 Encoding

Not encryption — just a way to represent binary data as ASCII text. Commonly used to encode hashes, keys, and binary data for transmission.

```bash
# Encode
echo "text" | base64

# Decode
echo "dGV4dAo=" | base64 -d

# Decode a file
base64 -d encoded_file.txt
```

**Spot it:** Base64 strings end with `=` or `==` padding and use A-Z, a-z, 0-9, +, /

---

## Quick Reference — Hash Identification

| Format | Type |
|---|---|
| 32 hex chars | MD5 |
| 40 hex chars | SHA1 |
| 64 hex chars | SHA256 |
| 128 hex chars | SHA512 |
| `$2a$` or `$2b$` prefix | bcrypt |
| `$6$` prefix | SHA512crypt (Linux) |
| `$y$` prefix | yescrypt (Linux) |
| `$1$` prefix | MD5crypt (Linux) |
| No special chars, 32 hex | Could be MD5 or NTLM — check context |
