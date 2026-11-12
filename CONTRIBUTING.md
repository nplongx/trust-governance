# Contributing

Preserve fail-closed trust behavior. Changes must retain exact domain/scope binding, canonical serialization, immutable configuration fingerprints, contiguous high-water versions, terminal revocation semantics, signature validity windows, maker/checker separation, and one-use nonce replay protection.

Never add private-key custody, implicit default trust roots, automatic replay on ambiguous verification, or workflow execution authority to this crate. Keep the Governance receipt envelope clearly marked draft until a separate owner ratifies its cross-service contract.

Tests must use generated test keys and temporary or in-memory databases. Do not use production trust roots, databases, signers, or service endpoints.

Run `cargo test --locked`, `cargo fmt -- --check`, `cargo clippy --all-targets --locked -- -D warnings`, and rustdoc with warnings denied.
