# ADR-001: Hub-and-Spoke Isolation

- Status: Accepted
- Date: 2026-11-21
- Decision owners: namespace authority

## Context

A monolithic repository couples unrelated release cycles and failure domains. Unlinked fork sprawl creates ambiguous ownership, drifting copies, and dependencies that cannot be reconstructed reliably.

## Decision

Adopt a namespace topology with one authority, one archive, and N satellites. Satellites remain independently versioned and declare pinned contract dependencies. Authority governs policy and release; archive preserves checkpoints and provenance. Neither role silently subsumes the other.

## Consequences

The topology limits blast radius and permits independent release cadence. It adds explicit contract maintenance, graph verification, and role stewardship. Cross-namespace reuse requires a trust relationship and pinned references.

## Alternatives considered

- Monolith: rejected because it concentrates change and recovery risk.
- Unmanaged repository collection: rejected because relationships and ownership are implicit.
- Direct peer mesh: deferred because N-squared trust and coordination costs grow rapidly.
