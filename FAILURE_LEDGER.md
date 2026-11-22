# Failure Ledger

This ledger records system conditions, contributing factors, detection, and prevention. Entries describe process and design failures, not individual fault.

## FL-001: Partial artifact write

- **Condition:** Process interruption leaves a truncated artifact or a temporary file.
- **Contributing factors:** Direct in-place writes; no durable precondition record; no post-write digest.
- **Detection:** Expected artifact digest differs or a `PENDING` WAL entry has no matching `COMMITTED` entry.
- **Control:** Write to a sibling temporary file, flush and atomically replace, verify bytes, then append `COMMITTED` with `fsync`.
- **Recovery:** Validate WAL; inspect the target; replay the idempotent step. Never mark a mismatched artifact committed.

## FL-002: Dependency graph cannot be reconstructed

- **Condition:** A satellite references a missing node or participates in a dependency cycle.
- **Contributing factors:** Mutable branch references; incomplete lockfile review; implicit dependency discovery.
- **Detection:** DFS reports a missing dependency or active-stack revisit.
- **Control:** Pin dependency identity and version; verify the full graph before summon or release.
- **Recovery:** Correct the lockfile and re-run verification before executing bootstrap commands.

## FL-003: Private data enters public projection

- **Condition:** An internal catalog record or field appears in a public ledger.
- **Contributing factors:** Allow-by-default export; schema drift; treating repository visibility as a data classification.
- **Detection:** Export tests search for private markers and forbidden keys; recompute the public Merkle root.
- **Control:** Deny by default, drop records with `private: true`, project only explicit public fields, and review the generated ledger.
- **Recovery:** Stop publication, revoke or replace the affected public artifact, assess exposure, and regenerate from the private source after correcting the allowlist.
