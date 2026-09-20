# MCC Object Fetch Protocol
## Version 1

## 1. Overview

This document specifies the **MCC Object Fetch Protocol v1**, a minimal HTTP-based protocol for retrieving encrypted opaque objects from a storage service.

The protocol defines:
- a single HTTP GET operation,
- a fixed request path format,
- mandatory response headers,
- an opaque encrypted payload body,
- strict value grammars.

The server does not interpret object contents and does not participate in cryptographic operations.

## 2. Request

### 2.1 Method

```
GET
```

### 2.2 Request URI

```
/mcc/v1/object/{object-id}[?version=N]
```

- `object-id` — required
- `version` — optional
  - if omitted, the latest version MUST be returned

No request headers are required.

## 3. Response

### 3.1 Success Response

#### HTTP Status

```
200 OK
```

#### Response Headers (mandatory)

```
MCC-Protocol: mcc-protocol-v1
MCC-Object-Version: <uint>
MCC-Object-Checksum: sha256:<hex>
MCC-Crypto-Profile: mcc-crypto-v1
Content-Type: application/octet-stream
```

#### Response Body

```
<opaque encrypted payload blob>
```

The response body contains the complete encrypted object payload.
Its internal structure is opaque to the protocol and defined by the referenced crypto profile.

### 3.2 Error Responses

The server MUST respond with an appropriate HTTP status code and MAY include an empty body.

| Status | Meaning |
|-------:|---------|
| 400 | Invalid request |
| 404 | Object or version not found |
| 500 | Internal server error |

No MCC-specific headers are required for error responses.

## 4. Cryptography

### 4.1 Crypto Profile

The `MCC-Crypto-Profile` response header identifies the cryptographic profile used to produce the encrypted payload.

For this version:

```
mcc-crypto-v1
```

The exact cryptographic algorithms and payload layout are defined in the corresponding crypto profile specification.

All cryptographic material required for decryption (including salt and nonce) MUST be included in the encrypted payload.

---

## 5. Integrity Verification

The `MCC-Object-Checksum` header provides an integrity checksum of the encrypted payload.

The checksum:
- MUST be calculated over the complete encrypted payload blob
- MUST be verified by the client before attempting decryption

## 6. Exact Grammar

### 6.1 Object ID

```
object-id = 8-4-4-4-12 hexadecimal digits
```

Constraints:
- lowercase only
- no braces
- UUID version is not negotiated

Example:

```
550e8400-e29b-41d4-a716-446655440000
```

---

### 6.2 Version (Query Parameter)

```
version = 1*DIGIT
```

Constraints:
- decimal integer
- unsigned
- no leading zeros

### 6.3 MCC-Protocol Header

```
MCC-Protocol = "mcc-protocol-v1"
```

- exact match
- case-sensitive

### 6.4 MCC-Crypto-Profile Header

```
MCC-Crypto-Profile = "mcc-crypto-v1"
```

### 6.5 MCC-Object-Checksum Header

```
MCC-Object-Checksum = algo ":" hash
algo                 = "sha256"
hash                 = 64 lowercase hexadecimal characters
```

Formal requirement:

The `MCC-Object-Checksum` header MUST be of the form
`sha256:<64 lowercase hex characters>` and MUST be calculated over the encrypted payload.

## 7. Security Considerations

- The server is blind to plaintext data.
- All cryptographic parameters are client-defined and profile-scoped.
- Object confidentiality relies on the secrecy of encryption keys.
- Integrity is ensured through checksum verification prior to decryption.
- Access control is out of scope for this protocol.

## 8. Extensibility

Future versions MAY introduce:
- additional crypto profiles,
- new object operations,
- new protocol versions under `/mcc/v2/`.

## 9. Summary

- One operation
- One path
- No JSON
- Streaming-friendly
- Cryptography cleanly separated
- Fully deterministic grammar
