# Secure File Transfer System (AES + HMAC)

A **.NET client-server application** that demonstrates secure file transmission over raw TCP using AES-256 encryption and HMAC-SHA256 integrity verification — without relying on TLS or other higher-level protocols.

---

## Features

- **AES-256 Encryption** — file contents are encrypted before transmission; intercepted traffic reveals nothing
- **HMAC-SHA256 Integrity** — the server verifies a cryptographic tag before decrypting, rejecting any tampered or corrupted data
- **Structured Packet Protocol** — length-prefixed fields allow reliable parsing of variable-length data
- **Reliable TCP Reads** — a custom `ReadExact()` helper prevents silent data loss from partial stream reads
- **Modular Crypto Library** — reusable encryption and integrity primitives, cleanly separated from application logic

---

## Project Structure

```
SecureFileTransfer/
│
├── Crypto/
│   ├── Encryption/
│   │   └── AesEncryption.cs       # AES-256 encrypt/decrypt
│   └── Integrity/
│       └── HmacService.cs         # HMAC-SHA256 generation & verification
│
├── FileClient/
│   └── Program.cs                 # Reads, encrypts, and sends files
│
└── FileServer/
    └── Program.cs                 # Receives, verifies, decrypts, and saves files
```

---

## Security Design

### Confidentiality — AES-256

```
File  →  AES Encrypt  →  Ciphertext
```

Files are encrypted before leaving the client. Intercepted network traffic cannot reveal file contents.

### Integrity — HMAC-SHA256

```
Ciphertext  →  HMAC  →  Integrity Tag
```

The server recomputes the HMAC on receipt and compares it to the tag sent by the client. If they don't match, the file is rejected. Integrity is always verified **before decryption** to prevent processing tampered ciphertext.

---

## Network Protocol

The client sends a single structured packet:

```
[4 bytes]  FileNameLength
[n bytes]  FileName

[4 bytes]  IVLength
[n bytes]  Initialization Vector

[4 bytes]  CipherLength
[n bytes]  Encrypted File Data

[4 bytes]  HmacLength
[n bytes]  HMAC-SHA256
```

---

## Threat Model

The system protects against an attacker who can:

- Observe all network traffic
- Modify or inject packets in transit
- Attempt to corrupt transmitted files

**Assumption:** the attacker does not have access to the shared cryptographic keys.

---

## Getting Started

### Prerequisites

- [.NET SDK](https://dotnet.microsoft.com/download) (6.0 or later)

### 1. Start the server

```bash
dotnet run --project FileServer
```

### 2. Run the client

```bash
dotnet run --project FileClient
```

When prompted, enter the full path to the file you want to send:

```
Enter full file path to send:
C:\Users\user\Documents\report.pdf
```

The decrypted file will be saved on the server as:

```
received_report.pdf
```

---

## Design Decisions

| Decision | Rationale |
|---|---|
| Modular `Crypto` library | Keeps security logic separate from application code; easier to test and reuse |
| HMAC verified before decryption | Prevents the server from processing potentially malicious ciphertext |
| `ReadExact()` TCP helper | Guarantees full buffers are read; avoids silent protocol failures from partial reads |
| Length-prefixed packet fields | Allows reliable parsing of variable-length data without delimiters |

---

## Possible Future Improvements

- Diffie-Hellman key exchange (eliminate the shared-secret assumption)
- Mutual authentication
- Replay attack protection (nonces / timestamps)
- TLS integration as an optional transport layer
- Chunked streaming for large files

---


