
# Cryptography Fundamentals

Core cryptography concepts every security professional needs. You don't need to understand the math deeply — you need to recognize what's being used, whether it's secure, and how to work with these tools on the command line.

---

## The Core Problem Cryptography Solves

You want to send data that:
- Only the intended recipient can read (confidentiality)
- Cannot be altered without detection (integrity)
- Proves who sent it (authenticity)
- Cannot be denied by the sender (non-repudiation)

No single mechanism does all of these — you combine them.

---

## Encryption Types

### Symmetric Encryption

Same key encrypts and decrypts. Fast. The problem is key distribution — how do you securely share the key with the other party?

```
Encrypt: plaintext + key → ciphertext
Decrypt: ciphertext + key → plaintext
```

**Secure algorithms:**

| Algorithm | Key Size | Notes |
|---|---|---|
| AES-128 | 128 bits | Minimum acceptable today |
| AES-192 | 192 bits | Good |
| AES-256 | 256 bits | Preferred — used everywhere |
| ChaCha20 | 256 bits | Fast on mobile, used in TLS 1.3 |
| Camellia | 128/192/256 bits | Used in some Japanese standards |
| Twofish | 128-256 bits | Designed by Bruce Schneier |

**Broken/weak — never use:**

| Algorithm | Problem |
|---|---|
| DES | 56-bit key — brute-forceable |
| 3DES | Deprecated 2023, disallowed 2024 |
| RC4 | Multiple vulnerabilities |

**Block vs Stream ciphers:**
- **Block cipher** — encrypts fixed-size blocks (usually 128 bits) at a time. AES is a block cipher.
- **Stream cipher** — encrypts one byte at a time. ChaCha20 is a stream cipher.

**Block cipher modes matter:**

| Mode | Security | Notes |
|---|---|---|
| ECB | BROKEN | Same plaintext = same ciphertext. Never use. |
| CBC | OK | Needs random IV. Padding oracle attacks possible. |
| GCM | Preferred | Authenticated encryption — provides integrity too |
| CTR | Good | Turns block cipher into stream cipher |

**If you see ECB mode anywhere — that's a finding.**

---

### Asymmetric Encryption

Two mathematically linked keys — public and private. What one encrypts, only the other can decrypt. Solves the key distribution problem. Slower than symmetric.

```
Encrypt with recipient's PUBLIC key → only their PRIVATE key can decrypt
Sign with your PRIVATE key → anyone with your PUBLIC key can verify
```

**Use cases:**
- Encrypt data for someone → use their public key
- Prove you wrote something → encrypt with your private key (signing)
- Key exchange → Diffie-Hellman
- TLS/HTTPS → asymmetric for handshake, then symmetric for session

**Secure algorithms:**

| Algorithm | Key Size | Notes |
|---|---|---|
| RSA | 2048+ bits minimum, 4096 preferred | Widely used, based on prime factorization |
| ECDSA | 256+ bits | Elliptic curve — smaller keys, same security |
| Ed25519 | 256 bits | Modern, fast, recommended for SSH keys |
| Diffie-Hellman | 2048+ bits | Key exchange only, not encryption |

**Broken/weak:**
- RSA < 1024 bits — factored, don't use
- DSA with weak random number generation — broken in practice

---

## Diffie-Hellman Key Exchange

Allows two parties to agree on a shared secret over a public channel without ever transmitting the secret. The secret is never sent — both parties independently calculate the same value.

**Why it matters:** This is how TLS establishes a session key. An eavesdropper can see all the traffic but cannot calculate the session key.

**Vulnerability:** Susceptible to Man-in-the-Middle if identity isn't verified. Solution is PKI (certificates) — the server proves its identity before DH runs.

**DH parameters in the real world:**
- 1024-bit DH — weak, avoid (Logjam attack)
- 2048-bit DH — minimum acceptable
- 4096-bit DH — preferred

---

## Hashing

One-way function — takes input of any size, produces fixed-size output (digest/checksum). Cannot be reversed. Same input always produces same output. Any change to input produces completely different output.

**Not encryption** — hashing doesn't use a key and can't be "decrypted."

**Uses:**
- Password storage (store hash, not password)
- File integrity verification
- Digital signatures (sign the hash, not the whole file)
- Message authentication (HMAC)

**Secure hash algorithms:**

| Algorithm | Output Size | Status |
|---|---|---|
| SHA-256 | 256 bits / 64 hex chars | Secure — widely used |
| SHA-384 | 384 bits | Secure |
| SHA-512 | 512 bits | Secure |
| SHA3-256 | 256 bits | Secure — newer standard |
| RIPEMD-160 | 160 bits | Still considered secure |
| BLAKE2 | Variable | Fast and secure |

**Broken — never use for security:**

| Algorithm | Problem |
|---|---|
| MD5 | Collision attacks — two different files can have same hash |
| SHA-1 | Collision attacks demonstrated (SHAttered 2017) |

**If you see MD5 or SHA-1 being used for security purposes — that's a finding.**

---

## Password Storage

Never store plaintext passwords. The proper approach:

```
BAD:   store password
OK:    store hash(password)          ← rainbow table attacks work
BETTER: store hash(password + salt)  ← salt defeats rainbow tables
BEST:  use PBKDF2, bcrypt, or Argon2 ← slow by design, defeats brute force
```

**Salt:** Random value appended to password before hashing. Each user gets a unique salt stored alongside their hash. Defeats precomputed rainbow tables because the attacker has to brute-force each hash individually.

**PBKDF2:** Runs the hash function thousands of times (iterations). Makes brute force computationally expensive. The number of iterations should increase as hardware gets faster.

**For pentesting:** When you find a password database, look for:
- Plaintext passwords — critical finding
- MD5 or SHA-1 hashes — high finding, easily crackable with hashcat
- Unsalted hashes — rainbow tables work
- bcrypt/Argon2/PBKDF2 — harder to crack, need wordlists + time

---

## HMAC — Hash-Based Message Authentication Code

Combines a hash function with a secret key. Provides both integrity and authenticity — proves the message wasn't tampered with AND came from someone who knows the key.

```
HMAC = H(key ⊕ opad || H(key ⊕ ipad || message))
```

You don't need to know the math — you need to know what it does and how to use it.

**Used in:** JWT tokens, API authentication, TLS, signed cookies.

---

## PKI — Public Key Infrastructure

Solves the trust problem in asymmetric encryption. How do you know a public key actually belongs to who it claims to?

**Chain of trust:**
```
Root CA (trusted by browsers/OS)
  → Intermediate CA
    → Server Certificate (signed by intermediate CA)
```

**Certificate contains:**
- Public key of the server
- Domain name it's valid for
- Validity period
- Digital signature from CA

**How HTTPS works:**
1. Browser connects to server
2. Server sends its certificate
3. Browser verifies certificate is signed by a trusted CA
4. Browser verifies certificate matches the domain
5. DH key exchange happens (using certificate to prevent MITM)
6. Session key established
7. All subsequent communication encrypted with symmetric key

**Self-signed certificates:** Signed by the entity itself, not a trusted CA. Fine for internal use and testing. Browser will show a warning. Common in lab environments — use `-k` flag with curl to skip verification.

---

## OpenSSL Command Reference

### Symmetric Encryption (OpenSSL)

```bash
# Encrypt a file with AES-256-CBC
openssl aes-256-cbc -e -in plaintext.txt -out encrypted.bin

# Decrypt
openssl aes-256-cbc -d -in encrypted.bin -out decrypted.txt

# Encrypt with stronger key derivation (recommended)
openssl aes-256-cbc -pbkdf2 -iter 100000 -e -in plaintext.txt -out encrypted.bin

# Decrypt with PBKDF2
openssl aes-256-cbc -pbkdf2 -iter 100000 -d -in encrypted.bin -out decrypted.txt

# Other ciphers
openssl camellia-256-cbc -e -in plaintext.txt -out encrypted.bin
openssl chacha20 -e -in plaintext.txt -out encrypted.bin

# List all supported ciphers
openssl enc -list
```

### GPG Encryption

```bash
# Encrypt symmetrically with AES-256
gpg --symmetric --cipher-algo AES256 message.txt

# Encrypt with ASCII output (readable in text editor)
gpg --armor --symmetric --cipher-algo AES256 message.txt

# Decrypt
gpg --output decrypted.txt --decrypt message.txt.gpg

# Decrypt GPG file (prompts for passphrase)
gpg -d encrypted.gpg > decrypted.txt

# List supported ciphers
gpg --version

# List your keys
gpg --list-keys
gpg --list-secret-keys
```

### RSA Key Generation and Inspection

```bash
# Generate RSA private key (2048 bit minimum, 4096 preferred)
openssl genrsa -out private-key.pem 2048
openssl genrsa -out private-key.pem 4096

# Generate RSA private key with passphrase protection
openssl genrsa -aes256 -out private-key.pem 4096

# Extract public key from private key
openssl rsa -in private-key.pem -pubout -out public-key.pem

# View full RSA key details (modulus, primes p and q, exponents)
openssl rsa -in private-key.pem -text -noout

# View public key details
openssl rsa -in public-key.pem -pubin -text -noout

# Check key consistency
openssl rsa -in private-key.pem -check
```

**RSA key text output explained:**
```
modulus     = N (public, product of p × q)
publicExponent  = e (usually 65537)
privateExponent = d (secret)
prime1      = p (secret prime)
prime2      = q (secret prime)
```

### RSA Encryption/Decryption

```bash
# Encrypt file using recipient's public key
openssl pkeyutl -encrypt -in plaintext.txt -out ciphertext.bin \
  -inkey public-key.pem -pubin

# Decrypt using private key
openssl pkeyutl -decrypt -in ciphertext.bin -out decrypted.txt \
  -inkey private-key.pem
```

### Diffie-Hellman Parameters

```bash
# Generate DH parameters (slow — takes minutes)
openssl dhparam -out dhparams.pem 2048
openssl dhparam -out dhparams.pem 4096

# View DH parameters (prime P and generator G)
openssl dhparam -in dhparams.pem -text -noout
```

### Certificates and PKI

```bash
# Generate Certificate Signing Request (CSR) and new private key
openssl req -new -nodes -newkey rsa:4096 -keyout key.pem -out cert.csr

# Generate self-signed certificate (for testing)
openssl req -x509 -newkey -nodes rsa:4096 -keyout key.pem \
  -out cert.pem -sha256 -days 365

# View certificate details
openssl x509 -in cert.pem -text -noout

# Check certificate validity dates
openssl x509 -in cert.pem -noout -dates

# Check what CA signed a certificate
openssl x509 -in cert.pem -noout -issuer

# Check the subject (who the cert is for)
openssl x509 -in cert.pem -noout -subject

# Check public key size in certificate
openssl x509 -in cert.pem -text -noout | grep "Public-Key"

# Verify a certificate against a CA
openssl verify -CAfile ca.pem cert.pem

# Check a remote server's certificate
openssl s_client -connect example.com:443 -showcerts

# Check TLS version and cipher suite used
openssl s_client -connect example.com:443 -tls1_3
```

### Hashing

```bash
# SHA-256 hash of a file
sha256sum filename.txt

# SHA-256 hash of multiple files
sha256sum *

# Other hash algorithms
sha512sum filename.txt
sha1sum filename.txt    # weak — avoid for security
md5sum filename.txt     # broken — avoid for security

# Hash a string directly
echo -n "password" | sha256sum

# OpenSSL hashing
openssl dgst -sha256 filename.txt
openssl dgst -sha512 filename.txt
openssl dgst -md5 filename.txt
```

### HMAC

```bash
# HMAC with SHA-256
hmac256 secretkey message.txt

# Alternative
sha256hmac message.txt --key secretkey

# Using OpenSSL
openssl dgst -sha256 -hmac "secretkey" message.txt
```

### Useful One-Liners

```bash
# Generate a random password/key
openssl rand -base64 32
openssl rand -hex 32

# Convert PEM to DER format
openssl x509 -in cert.pem -outform DER -out cert.der

# Convert DER to PEM
openssl x509 -in cert.der -inform DER -outform PEM -out cert.pem

# Check if a private key matches a certificate
openssl x509 -noout -modulus -in cert.pem | openssl md5
openssl rsa -noout -modulus -in key.pem | openssl md5
# If the MD5 values match, the key and cert are a pair

# Extract public key from certificate
openssl x509 -in cert.pem -pubkey -noout

# Test TLS connection and see certificate chain
openssl s_client -connect TARGET:443

# Decode a base64 string
echo "base64string==" | base64 -d

# Encode to base64
echo "plaintext" | base64
```

---

## SSH Key Generation

```bash
# Generate Ed25519 key pair (modern, recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Generate RSA key pair (4096 bits)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Generate with specific filename
ssh-keygen -t ed25519 -f ~/.ssh/my_key

# View public key
cat ~/.ssh/id_ed25519.pub

# Copy public key to remote server
ssh-copy-id user@remote_host

# Add key to SSH agent
ssh-add ~/.ssh/id_ed25519
```

---

## Identifying Weak Cryptography — What to Look For

In code, config files, or during assessments:

```
ECB mode anywhere                → Critical finding
MD5 for passwords or signatures  → High finding
SHA-1 for signatures             → High finding
RSA < 2048 bits                  → High finding
DH < 2048 bits                   → High finding (Logjam)
Hardcoded keys or passwords      → Critical finding
Self-signed certs in production  → Medium finding (depending on context)
TLS 1.0 or 1.1 enabled           → Medium finding
Unsalted password hashes         → High finding
Plaintext passwords stored       → Critical finding
```

---

## TLS Version Reference

| Version | Status | Notes |
|---|---|---|
| SSL 2.0 | Broken | Never use |
| SSL 3.0 | Broken (POODLE) | Never use |
| TLS 1.0 | Deprecated | Disable |
| TLS 1.1 | Deprecated | Disable |
| TLS 1.2 | Acceptable | Still widely used, configure securely |
| TLS 1.3 | Preferred | Faster, more secure, use this |

---

## Quick Reference — Algorithm Choices

```
Symmetric encryption:  AES-256-GCM or ChaCha20-Poly1305
Key exchange:          ECDHE (Elliptic Curve Diffie-Hellman Ephemeral)
Asymmetric/signing:    Ed25519 or RSA-4096
Hashing:               SHA-256 minimum, SHA-512 preferred
Password hashing:      Argon2id > bcrypt > PBKDF2 (never MD5/SHA-1)
TLS:                   TLS 1.3 preferred, TLS 1.2 acceptable
Certificate key size:  RSA 4096 or ECDSA P-256 minimum
```

---

## Resources

- NIST approved algorithms: https://csrc.nist.gov/projects/cryptographic-algorithm-validation-program
- Password storage cheat sheet: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- TLS configuration best practices: https://ssl-config.mozilla.org
- CyberChef (encoding/decoding/crypto): https://gchq.github.io/CyberChef
- SSL Labs (test server TLS config): https://www.ssllabs.com/ssltest/
- Have I Been Pwned (check hashes): https://haveibeenpwned.com
