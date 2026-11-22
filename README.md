# Federated Repository Mesh Blueprint

Version: 1.0.0  
Status: Initial architectural standard

## Purpose

This repository specifies a domain-agnostic, federated repository topology for systems that must remain independently operable while sharing explicit contracts, provenance, and recovery semantics. It is a standard and set of starter templates, not a hosted control plane.

## Core topology: M x (1 + 1 + N)

For each namespace `m` in a federation of `M` namespaces, the baseline contains:

- **1 authority node**: owns namespace policy, trust roots, and release decisions.
- **1 archival node**: records immutable checkpoints, manifests, and recoverable state.
- **N satellite nodes**: independently versioned implementations or specialist repositories.

The authority and archive are separate failure domains. Satellites depend on pinned contracts rather than mutable branch heads. A namespace may be operated by one account or several; account count does not change the topology. Cross-namespace edges require explicit trust and version pins.

## Operating tiers

| Tier | Topology | Intended use |
|---|---|---|
| 1 | Solo node | One repository with local WAL and summon contract |
| 2 | Authority + satellites | A bounded product family with pinned dependencies |
| 3 | Federated namespaces | Multiple accounts or administrative domains with trusted keys |

The architecture document defines recursive tiers 0–4 and six planes.

## Persistence and recovery

Runtime state is reconstructed from declared configuration, pinned dependencies, and an append-only SHA-256 hash-chained JSONL WAL. A `PENDING` record is durable before an operation begins. A `COMMITTED` record is written only after artifact bytes are verified. Recovery validates the complete chain and replays only unresolved work. The WAL is tamper-evident, not a digital signature or proof of author identity.

## `summon` contract

A node is reconstructed on demand from `summon.lock`. Environment defaults are applied without overwriting an existing environment. Bootstrap commands run in declared order. A health check is mandatory and determines the final exit status. Commands are executable code: inspect a lockfile before invoking it.

## Dual projection

The Private Workshop is the authoritative working inventory. A one-way cryptographic diode emits a sanitized Public Showcase Ledger. Promotion is deny-by-default: records explicitly marked private are excluded, internal fields are removed, and only an allowlisted public projection is serialized. The public ledger cannot be used to reconstruct private source records. A Merkle root provides deterministic integrity checking for the exported lines; it does not establish confidentiality or signer identity.

## Repository map

- `AGENT_HANDSHAKE.md`: machine-readable operating contract and agent safety boundaries.
- `FAILURE_LEDGER.md`: blameless failure patterns and corrective controls.
- `docs/ARCHITECTURE.md`: normative architecture.
- `schemas/`: JSON Schema contracts.
- `templates/`: Tier 1–3 starter layouts.

## Conformance baseline

A conforming implementation must preserve namespace isolation, verify WAL chains before recovery, pin dependency identities, reject missing or cyclic DAG edges, and keep private-to-public projection one-way. It must not infer authorization from repository visibility or from a successful checksum.

## License

Apache-2.0.
