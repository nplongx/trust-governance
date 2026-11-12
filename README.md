# Runalith Trust

Linux-oriented trust configuration, Ed25519 public-key verification, signed approval policy, reviewer lifecycle, maker/checker evidence verification, trust high-water ledger, and Governance receipt primitives extracted from Runalith.

This package is a trust-domain library, not a standalone identity provider, authorization server, key-management system, workflow engine, or production Governance deployment.

## Included capabilities

- Strict administrator-managed trust configurations for five separate AGG domains and immutable Ed25519 public-root fingerprints.
- Fail-closed Ed25519 public-key parsing and raw-signature verification through the host OpenSSL CLI.
- Strict signed-envelope, domain-separated-signature, reviewer-attestation, approval-policy, and evidence-verification primitives.
- Maker/checker quorum verification with distinct reviewer identities, bounded validity, policy high-water checks, and one-use nonce consumption inside a caller-owned SQLite transaction.
- Append-only trust configuration history, monotonic version checks, root rotation/revocation lifecycle, and reviewer identity lifecycle contracts.
- Trust snapshot references, Governance receipt binding, durable nonce-port contracts, and an in-process Governance service boundary with host-injected authentication, signer, verifier, and transactional backend ports.

## Authority boundaries

The host application remains responsible for:

1. Authenticated administrator identity and authorization to provision trust roots or reviewer identities.
2. Custody and use of private signing keys. This crate accepts public PEM roots and signer/verifier ports; it does not provision or protect production private keys.
3. Authentication of service-to-service Governance requests and binding the authenticated principal to the request producer/audience.
4. Supplying authoritative trust configuration observers, nonce ledgers, approval backends, and transactional storage. `GovernanceDecisionBackend` implementations must make evidence verification, nonce consumption, policy high-water checks, receipt signing, and decision persistence atomic where specified by the contract.
5. Protecting SQLite files, backups, filesystem permissions, process identity, clock synchronization, and operational key rotation.
6. Deciding how verified approval/evidence maps to a workflow action. Verification does not itself schedule work, dispatch a provider, settle a lease, or execute a command.

A structurally valid envelope, a matching hash, or a locally verified Ed25519 signature is not proof that the signer is authorized, that trust configuration is current, or that an action is approved. Callers must enforce the relevant trust high-water, domain/scope, issuer/audience, identity, expiry, nonce, policy, and subject bindings.

## Draft contract warning

`governance_receipt.rs` contains a draft cross-service Governance receipt signature envelope. Its source documentation explicitly says this is not a ratified production contract. Owner ratification and independent protocol/security review are required before relying on that draft across services. Treat the draft builders/verifiers as experimental interoperability primitives, not as proof of a production deployment contract.

## Cryptographic implementation and platform

The Ed25519 verifier first validates the PEM envelope, strict Base64 body, exact Ed25519 SPKI prefix and DER length locally, then invokes `/usr/bin/openssl` directly without a shell. It compares OpenSSL's parsed DER with the supplied DER, validates executable metadata (root ownership and no group/other write permission), clears the child environment, uses private temporary files, bounds public-key and message sizes, and classifies operational failures separately from invalid signatures. Local PEM prevalidation prevents malformed input from being misclassified merely because OpenSSL versions emit different parser diagnostics.

Requires Linux/Unix semantics and an available trusted OpenSSL binary at `/usr/bin/openssl`. The exact path and ownership check are intentional. The package does not bundle OpenSSL or silently switch to another executable.

## Storage

`TrustLedger` and approval decision helpers use SQLite and caller-owned transactions. Append-only database triggers are part of the integrity model and are behaviorally probed. Do not point this library at an existing production database without a separately reviewed migration and backup plan. Existing unsupported or unversioned schemas may be rejected rather than migrated automatically.

## Dependencies

The only Runalith crate dependency is [`protocol-contracts`](https://github.com/nplongx/protocol-contracts), pinned to immutable Git revision `6dc2e11bed7fa74bb5e508933eed34e17afd2cac`.

This Git dependency is not available from crates.io. The package currently targets Cargo Git dependencies; crates.io packaging requires `protocol-contracts` to be published or vendored.

## Build and test

Requires Rust 1.85 or newer and trusted OpenSSL at `/usr/bin/openssl`.

```sh
cargo test --locked
cargo fmt -- --check
cargo clippy --all-targets --locked -- -D warnings
RUSTDOCFLAGS='-D warnings' cargo doc --locked --no-deps
```

GitHub Actions runs the suite on Rust 1.85 and stable on Ubuntu 24.04.

The tests use generated signing keys, in-memory/temporary SQLite databases, and deterministic protocol fixtures. They do not contact external identity providers or deploy a Governance service.

## Provenance

Extracted from `nplongx/runalith` at source commit `29c24cfa2b6ead5704d26cf2746bc384f2938b80`, from `rust/crates/runalith-trust`. The `runalith-protocol` workspace dependency is replaced with the immutable public `protocol-contracts` repository. This extraction also adds strict local PEM/Base64/SPKI prevalidation before OpenSSL invocation so malformed-key classification does not depend on OpenSSL diagnostic wording. Other trust-domain logic is retained. This extraction does not constitute a security audit or production approval.

## License

MIT. See [LICENSE](LICENSE).
