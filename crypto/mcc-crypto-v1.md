# MCC Crypto Profile v1

**Profile ID:** `mcc-crypto-v1`  

## 1. Overview

This document defines the **MCC Crypto Profile v1**, a symmetric encryption profile
used to encrypt and decrypt object payloads transmitted via MCC protocols.

The profile is designed for MVP usage and prioritizes:
- simplicity,
- deterministic behavior,
- wide library availability.

This profile is referenced by transport protocols via the `MCC-Crypto-Profile` header.

## 2. Cryptographic Algorithms

### 2.1 Key Derivation Function (KDF)

- **Algorithm:** PBKDF2-HMAC-SHA256
- **Purpose:** Derive a symmetric encryption key from a secret

### 2.2 Authenticated Encryption

- **Algorithm:** AES-256-GCM
- **Mode:** AEAD (Authenticated Encryption with Associated Data)

AES-GCM provides confidentiality and integrity for the encrypted payload.

## 3. Payload Structure

The encrypted payload is a binary blob with the following logical structure:

```
| salt (16) | nonce (12) | ciphertext | auth_tag (16) |
```

Where:
- `salt`: 16 bytes — random salt used for PBKDF2
- `nonce`: 12 bytes — unique AES-GCM nonce
- `ciphertext` — encrypted plaintext data
- `auth_tag`: 16 bytes — authentication tag produced by AES-GCM

The exact byte lengths are implementation-defined but MUST be consistent
between encryption and decryption.

The payload structure is opaque to the transport protocol and server.

## 4. Key Derivation

The encryption key MUST be derived as follows:

- Input secret: arbitrary byte sequence supplied by the caller.
- Salt: randomly generated per object version
- KDF: PBKDF2-HMAC-SHA256
- Output key length: 32 bytes
- Iterations: 100000

The salt MUST be included in the encrypted payload.
The salt MUST be generated using a cryptographically secure
random number generator.

## 5. Encryption and Decryption

### 5.1 Encryption

1. Generate a random salt
2. Derive the encryption key using PBKDF2-HMAC-SHA256
3. Generate a unique nonce
4. Encrypt plaintext using AES-256-GCM
5. Output payload: `salt | nonce | ciphertext | auth_tag`

The nonce MUST be generated using a cryptographically secure
random number generator.
The nonce MUST NOT be reused with the same derived AES key.

### 5.2 Decryption

1. Parse and validate payload structure.
2. Extract salt.
3. Derive the AES-256 key using PBKDF2.
4. Extract nonce and ciphertext+auth_tag.
5. Decrypt and authenticate using AES-256-GCM.
6. Reject the payload if authentication fails.

## 6. Integrity and Authentication

AES-256-GCM provides built-in authentication of the ciphertext.

The MCC transport protocol MAY provide an external SHA-256
checksum of the complete encrypted payload.

The checksum is intended for early detection of payload corruption
before cryptographic decryption.

The checksum MUST NOT be considered a cryptographic authentication
mechanism.

## 7. Security Considerations

- Secrets used for key derivation MUST be protected
- Salts and nonces MUST be generated using a cryptographically secure RNG
- Nonces MUST NOT be reused with the same key
- This profile does not define key management or rotation

## 8. Extensibility

This profile is intended for MVP usage.

Future profiles MAY introduce:
- alternative KDFs
- different AEAD algorithms

Such changes MUST be introduced as new crypto profiles.

## 9. Summary

- Symmetric encryption profile
- PBKDF2-HMAC-SHA256 for key derivation
- AES-256-GCM for authenticated encryption
- The payload contains all non-secret parameters required for decryption
- Server remains blind to plaintext data
