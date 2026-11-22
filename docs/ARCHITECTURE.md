# Architecture Specification

## 1. Normative language

MUST, MUST NOT, SHOULD, and MAY express requirements.

## 2. Recursive tiers 0–4

- **Tier 0, artifact:** immutable artifact with digest and provenance.
- **Tier 1, solo node:** repository containing node metadata, summon contract, and local WAL.
- **Tier 2, namespace:** authority, archive, and N satellites under one namespace policy.
- **Tier 3, federation:** namespaces joined through explicit trust roots and pinned inter-namespace contracts.
- **Tier 4, federation of federations:** independently governed federations exchange authenticated manifests through declared gateways. No global root authority is implied.

## 3. Topology

Each namespace designates one logical authority, one logical archive, and zero or more satellites. Replicas require explicit consistency and failover policy. Authority and archive SHOULD have separate credentials and storage failure domains. Satellites MUST declare parent authority and pinned contract.

## 4. Six planes

1. **contract-plane:** schemas, API contracts, compatibility declarations, dependency pins.
2. **ingress-plane:** accepted inputs, authentication, validation, rate controls.
3. **compute-plane:** deterministic execution units, workers, runtime adapters.
4. **provenance-plane:** source references, digests, WAL, build evidence, attestations.
5. **control-plane:** policy, authorization, orchestration, approvals, lifecycle.
6. **tooling-plane:** CLIs, test harnesses, operator utilities, agent adapters.

## 5. Invariants

- Identity is stable and distinct from repository name or URL.
- Dependencies are pinned and graph-verified before execution.
- State transitions are recorded before and after side effects.
- Recovery is deterministic and idempotent where possible.
- Trust is explicit, scoped, revocable, and authenticated.
- Publication is a projection, never reverse synchronization.

## 6. WAL durability

Each entry contains sequence, timestamp, step, status, artifact, note, prev_hash, and entry_hash. Hash input is canonical UTF-8 JSON with sorted keys and compact separators, excluding entry_hash. The first prev_hash is 64 zeroes.

Writers MUST serialize concurrent writes, append one complete line, flush, and fsync. Readers MUST reject malformed complete lines, sequence gaps, predecessor mismatch, and digest mismatch. An unterminated final fragment may be truncated to the last complete record under an exclusive lock. A complete final object may be normalized only after hash verification.

PENDING precedes side effects; COMMITTED follows verified output; FAILED records terminal failure. Recovery tooling reports a deterministic plan and MUST NOT execute unresolved commands automatically. WAL integrity is tamper evidence only, not identity proof.

## 7. Summon

summon.lock is executable reconstruction configuration. Implementations MUST provide dry-run by default, bounded execution, explicit approval before execution, and argv-based invocation without implicit shell interpretation.

## 8. Diode promotion

The public projection MUST exclude records not explicitly marked visibility=public and all records with private=true. It MUST emit only allowlisted fields, normalize serialization and ordering, report redaction count, compute a deterministic SHA-256 Merkle root, and require human review before publication. A diode is not a network isolation device.

## 9. Trust and keys

TRUSTED_KEYS.json is operator-maintained. Key rotation, revocation, identity proofing, and signature algorithms require documented deployment policy. Presence in a file is not proof of key possession.

## 10. Compatibility

Schema changes MUST be versioned. Breaking changes require a new major version and an ADR. Nodes MUST reject unsupported major versions.
