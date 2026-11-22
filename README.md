# Federated Repository Mesh Blueprint

Version: 1.0.1 development line  
Status: Architectural standard and starter contracts

## Purpose

This repository defines a domain-agnostic federated repository topology for systems that remain independently operable while sharing explicit contracts, provenance, and recovery semantics. It is a standard and set of starter templates, not a hosted control plane.

## Core topology: M x (1 + 1 + N)

For each namespace, designate one authority, one archive, and N independently versioned satellites. Authority and archive are separate logical roles and should occupy separate failure domains. Satellites depend on pinned contracts rather than mutable branch heads.

## Operating tiers

| Tier | Topology | Intended use |
|---|---|---|
| 1 | Solo node | One repository with local WAL and summon contract |
| 2 | Authority + satellites | Bounded product family with pinned dependencies |
| 3 | Federated namespaces | Multiple administrative domains with explicit trust roots |

Tier 3 starter files include node metadata, summon contract, dependency lock, namespace declarations, and trusted-key placeholders.

## Persistence and recovery

The WAL is an append-only SHA-256 hash-chained JSONL journal. Writers serialize concurrent appends, flush, and fsync complete records. Readers reject complete malformed records, sequence gaps, predecessor mismatch, and digest mismatch. An interrupted unterminated final fragment may be truncated to the last complete record under an exclusive lock. Hash chains are tamper-evident, not signatures.

Recovery tooling reports unresolved work deterministically. It MUST NOT replay arbitrary commands automatically. An operator or orchestrator decides whether a retry is safe and idempotent.

## Summon

A node is reconstructed from `summon.lock`. Commands are executable code and must be inspected. Implementations should default to dry-run, use bounded execution, and require explicit approval before execution.

## Dual projection

The Private Workshop is the authoritative inventory. A one-way export emits a sanitized Public Showcase Ledger. Only records explicitly marked public may be considered. Internal fields are removed through an allowlist, and human review is required before publication. A Merkle root provides deterministic integrity, not confidentiality or signer identity.

## Repository map

- `AGENT_HANDSHAKE.md`: machine-readable operating contract and agent safety boundaries.
- `FAILURE_LEDGER.md`: failure patterns and corrective controls.
- `docs/ARCHITECTURE.md`: normative architecture.
- `schemas/`: JSON Schema contracts.
- `templates/`: Tier 1–3 starter layouts.

## Conformance

Implementations preserve namespace isolation, validate schema and dependency versions, pin dependency identities, reject missing or cyclic DAG edges, record state transitions around side effects, and keep private-to-public projection one-way. Authorization is never inferred from repository visibility or a checksum.

## License

Apache-2.0.
