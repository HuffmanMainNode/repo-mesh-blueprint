# Architecture Specification

## 1. Normative language

The terms MUST, MUST NOT, SHOULD, and MAY express requirements.

## 2. Recursive tiers 0–4

- **Tier 0, artifact:** one immutable artifact with digest and provenance.
- **Tier 1, solo node:** one repository containing node metadata, summon contract, and local WAL.
- **Tier 2, namespace:** one authority, one archive, and N satellites under one namespace policy.
- **Tier 3, federation:** M namespaces joined through explicit trust roots and pinned inter-namespace contracts.
- **Tier 4, federation of federations:** independently governed federations exchange signed or otherwise authenticated manifests through declared gateways. Tier 4 does not imply a global root authority.

Each tier composes lower-tier units. Recursive composition MUST preserve identity, provenance, isolation, and recovery boundaries.

## 3. M x (1 + 1 + N) topology

For each namespace `m`, designate exactly one logical authority, one logical archive, and zero or more satellites. High availability may replicate a role, but replicas MUST share an explicit consistency and failover policy. Authority and archive SHOULD have separate credentials and storage failure domains. A satellite MUST declare its parent authority and pinned contract.

## 4. Six architectural planes

1. **contract-plane:** schemas, API contracts, compatibility declarations, and dependency pins.
2. **ingress-plane:** accepted inputs, authentication boundaries, validation, and rate controls.
3. **compute-plane:** deterministic execution units, workers, and runtime adapters.
4. **provenance-plane:** source references, digests, WAL, build evidence, and release attestations.
5. **control-plane:** policy, authorization, orchestration, approvals, and lifecycle state.
6. **tooling-plane:** CLIs, test harnesses, operator utilities, and agent adapters.

A repository may implement multiple planes, but its node metadata MUST name one primary plane. Gap analysis SHOULD identify secondary contributions separately.

## 5. Six architectural invariants

- Identity is stable and distinct from repository name or URL.
- Dependencies are pinned and graph-verified before execution.
- State transitions are recorded before and after side effects.
- Recovery is deterministic and idempotent where possible.
- Trust is explicit, scoped, and revocable.
- Publication is a projection, never a reverse synchronization path.

## 6. WAL format and durability

Each JSONL entry contains `sequence`, `timestamp`, `step`, `status`, `artifact`, `note`, `prev_hash`, and `entry_hash`. Hash input is the canonical UTF-8 JSON encoding of all fields except `entry_hash`, with sorted keys and compact separators. The first `prev_hash` is 64 zeroes. The writer MUST append one complete line, flush, and `fsync`. A consumer MUST reject malformed lines, sequence gaps, predecessor mismatch, and digest mismatch.

A `PENDING` entry precedes side effects. `COMMITTED` follows verified output. `FAILED` records a terminal failure. A later `COMMITTED` for the same step resolves a previous pending attempt. WAL integrity is tamper-evidence only; protect it with filesystem permissions and external signatures where adversarial tampering is in scope.

## 7. Summon and reconstruction

`summon.lock` is an executable reconstruction contract. It contains ordered bootstrap commands, environment defaults, and a health check. Operators MUST review commands before summon. Implementations SHOULD support a dry-run mode and bounded execution. This blueprint does not mandate a container runtime.

## 8. Diode promotion

The private catalog is the source of truth. The public projection MUST:

1. Exclude every record with `private` set to boolean true.
2. Emit only allowlisted public fields.
3. Normalize JSON serialization and line ordering.
4. Record the number of redacted records.
5. Compute a deterministic SHA-256 Merkle root over exported canonical record bytes.
6. Never copy secrets, internal notes, local paths, credentials, or private repository URLs.

The diode is an export process, not a network isolation device. A real security boundary also requires separate credentials, review, and publication controls.

## 9. Trust and keys

`TRUSTED_KEYS.json` is an operator-maintained trust store. Key rotation, revocation, identity proofing, and signature algorithms are deployment policy and MUST be documented by each federation. Presence in the file alone is not proof of key possession.

## 10. Compatibility and evolution

Schema changes MUST be versioned. Breaking changes require a new major contract version and an ADR. Nodes MUST reject unsupported major versions rather than guessing.
