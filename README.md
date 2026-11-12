# Trust Governance

A standalone trust and governance library with Ed25519 public-key verification, signed approval policy, reviewer lifecycle, maker/checker evidence verification, trust high-water ledger, and signed decision receipts.

This package is a trust-domain library, not an identity provider, authorization server, key-management system, workflow engine, or deployed governance authority.

## Install

```toml
[dependencies]
trust-governance = { git = "https://github.com/nplongx/trust-governance", tag = "v0.2.1" }
```

All parsing/canonicalization dependencies required for this crate are included under `vendor/`.

## Terminology

See [TERMINOLOGY.md](TERMINOLOGY.md) for shared identifier, lifecycle, evidence, time-unit, and compatibility conventions.

## Included capabilities

- Strict administrator-managed trust configurations for five separate AGG domains and immutable Ed25519 public-root fingerprints.
- Fail-closed Ed25519 public-key parsing and raw-signature verification through the host OpenSSL CLI.
- Strict signed-envelope, domain-separated-signature, reviewer-attestation, approval-policy, and evidence-verification primitives.
- Maker/checker quorum verification with distinct reviewer identities, bounded validity, policy high-water checks, and one-use nonce consumption inside a caller-owned SQLite transaction.
- Append-only trust configuration history, monotonic version checks, root rotation/revocation lifecycle, and reviewer identity lifecycle contracts.
- Trust snapshot references, Governance receipt binding, durable nonce-port contracts, and an in-process governance service boundary with host-injected authentication, signer, verifier, and transactional backend ports.

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

Strict parsing and canonicalization source required by this package is included as an internally vendored snapshot under `vendor/`, with its license and source revision. No sibling-package or Git dependency is required.

Crates.io publication is disabled because the package intentionally resolves these components from local vendored paths. Consume it as a Git dependency.

## Build and test

Requires Rust 1.85 or newer and trusted OpenSSL at `/usr/bin/openssl`.

```sh
cargo test --locked
cargo fmt -- --check
cargo clippy --all-targets --locked -- -D warnings
RUSTDOCFLAGS='-D warnings' cargo doc --locked --no-deps
```

GitHub Actions runs the suite on Rust 1.85 and stable on Ubuntu 24.04.

The current approval profile requires exactly five independently administered trust domains and exact signing scopes. It is a closed compatibility profile, not a generic API for arbitrary trust-domain registration.

The tests use generated signing keys, in-memory/temporary SQLite databases, and deterministic protocol fixtures. They do not contact external identity providers or deploy a governance service.

## Provenance

Created from immutable source snapshot `29c24cfa2b6ead5704d26cf2746bc384f2938b80`. Parsing and canonicalization source required for compilation is vendored locally. This package has not undergone an independent security audit and is not production approval.

## License

MIT. See [LICENSE](LICENSE).


## Legacy storage and signature identifiers

Version 1 retains exact signature-domain prefixes, SQLite table/trigger names, and savepoint identifiers already used by the approval and attestation profiles. These values are part of signed or persisted compatibility contracts, not package identity or repository dependencies. Renaming them in place would invalidate signatures or make existing ledger state invisible. A replacement requires a new protocol/storage version and an explicit migration.
