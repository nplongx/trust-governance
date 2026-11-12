# Security policy and limitations

This project implements security-sensitive trust, signature-verification, approval, and evidence primitives. It has not undergone an independent security audit. Do not rely on it as the sole authority protecting production actions or sensitive data.

Report suspected vulnerabilities privately to the repository maintainer before publishing exploit details. Do not include private signing keys, production trust databases, credentials, or other secrets in reports.

## Trust assumptions

- The kernel, filesystem, `/usr/bin/openssl`, SQLite runtime, process identity, and host clock are trusted.
- Administrator and reviewer identity is authenticated by the host before invoking lifecycle services.
- Production private-key custody remains with an external signing port; no private-key implementation is supplied here.
- The host supplies authoritative trust-observation and nonce-ledger implementations. Missing or unavailable authority must remain unavailable, not be treated as approval.
- Approval decision backends preserve transaction boundaries and append-only ledger semantics.

## Known limitations

- The Governance cross-service receipt envelope in `governance_receipt.rs` is explicitly draft/unratified in source documentation.
- The fixed five trust domains and exact scopes are Runalith AGG-specific.
- The OpenSSL verifier requires `/usr/bin/openssl` to be root-owned and not group/other-writable.
- A local signature verification result does not establish authorization, current trust, reviewer authority, remote side-effect completion, or workflow settlement.
- SQLite schema and operational migration/backup are host responsibilities.
