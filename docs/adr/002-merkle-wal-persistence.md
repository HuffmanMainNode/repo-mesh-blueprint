# ADR-002: Merkle WAL Persistence

- Status: Accepted
- Date: 2026-11-21
- Decision owners: namespace authority

## Context

Long-running automation can be interrupted between side effects and status reporting. A simple last-run marker cannot distinguish durable completion from a partially written operation.

## Decision

Use append-only JSONL records linked by SHA-256 predecessor hashes. Record `PENDING` before work and `COMMITTED` only after byte-level verification. Flush and fsync each record. Recovery validates the complete chain before interpreting state.

## Consequences

The log provides deterministic tamper detection and a replay boundary with minimal runtime dependencies. It does not provide authenticity, confidentiality, or protection against an attacker who can rewrite the entire log. Backups and external signatures remain deployment responsibilities.

## Alternatives considered

- Mutable status file: rejected because updates can be lost or overwritten.
- Database-only state: deferred because it adds a service/runtime dependency.
- Hash chain without fsync: rejected because acknowledged records may not survive power loss.
