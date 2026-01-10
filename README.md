# MCC Specification

This repository contains the official specifications for **MCC (MyCloudConfig) protocols**.

The goal of MCC specifications is to define **minimal, stable, transport-level contracts**
that can be implemented independently by servers, SDKs, and client applications.

Specifications are written in a protocol-first manner and intentionally avoid coupling
to specific implementations, CLIs, or deployment environments.

## Specifications

### Object Fetch Protocol

- **File:** `object-fetch-v1.md`
- **Description:**  
  Defines a minimal HTTP-based protocol for retrieving encrypted opaque objects
  by identifier and optional version.

### Cryptography Profiles

- **Directory:** `crypto/`
- **Description:**  
  Cryptography profiles define how object payloads are encrypted and decrypted.
  Profiles are referenced by protocol headers and are versioned independently
  from transport protocols.

Currently defined profiles:

- `mcc-crypto-v1.md` — MVP symmetric encryption profile

## Design Principles

- Minimal surface area
- Streaming-friendly by default
- Server is blind to plaintext data
- Cryptography separated from transport
- Deterministic and strict grammars
- Backward compatibility via versioning

## Versioning

- Transport protocols are versioned in the URL path (e.g. `/mcc/v1/`)
- Cryptography profiles are versioned independently
- Breaking changes require a new major version
