# Security Policy

Do not disclose credentials, private repository metadata, or exploit details in public issues. Use GitHub private vulnerability reporting when enabled; otherwise contact the repository owner through a verified private channel.

Security invariants:
- Visibility is not authorization to execute code.
- Private-to-public promotion is one-way, allowlisted, reviewed, and fail-closed.
- WAL hashes provide tamper evidence, not identity authentication or confidentiality.
- Dependencies must be pinned and verified before execution.
- Trust roots require out-of-band identity verification.
- Templates are examples, not production trust configurations.

Published release tags are historical snapshots and are not silently rewritten.
