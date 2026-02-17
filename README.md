# tekmor-enclave

TEE-agnostic enclave for trustless KYC attestation issuance.

Part of [Tekmor](https://github.com/tekmor-xyz/tekmor) — provable identity infrastructure.

## What this does

This enclave runs inside a hardware TEE (AWS Nitro or Intel SGX) and bridges KYC verification to on-chain attestations. The operator cannot see user data, tamper with verification logic, or forge attestations.

```
User documents ──► [Enclave] ──► KYC provider API (cert-pinned TLS)
                       │
                       ▼
                   Signed attestation ──► on-chain registry
```

## Security model

| Threat | Mitigation |
|---|---|
| Fake KYC provider | TLS certificate pinning (SPKI hash) inside enclave image |
| Signing key extraction | Key generated inside TEE, never exported. KMS attestation-based policies. |
| Modified enclave code | Deterministic build. PCR/MRENCLAVE values published on-chain. |
| Operator data access | Hardware-enforced isolation. Not even the host can read enclave memory. |

## Stack

- **Language:** Rust
- **TEE SDK:** [Fortanix EDP](https://edp.fortanix.com/) — single codebase, compiles to both Nitro and SGX
- **Build:** Deterministic (Nix / Docker → EIF/SGXS)
- **Signing:** Ed25519/Falcon re-signing for efficient on-chain verification

## Structure

```
src/
  main.rs           Entry point
  kyc/              KYC provider integrations
    mod.rs
    onfido.rs       Onfido sandbox integration
  attestation/      Attestation construction and signing
    mod.rs
    leaf.rs         Attestation leaf format
    signer.rs       Ed25519/Falcon signing
  tls/              Certificate pinning and TLS config
    mod.rs
    pins.rs         SPKI pin definitions
  chain/            On-chain transaction submission
    mod.rs
build/
  Dockerfile        Deterministic enclave build
  nix/              Nix build config
```

## Building

```bash
# Build for Nitro
cargo build --target x86_64-unknown-linux-fortanixvme

# Build for SGX
cargo build --target x86_64-fortanix-unknown-sgx

# Reproducible build (Docker)
docker build -t tekmor-enclave .
```

## Status

In development. See the [roadmap](https://github.com/tekmor-xyz/tekmor#roadmap).

## License

MIT
