# Agent Handshake Contract

Protocol: `mesh-archivist-agent/1`  
Default posture: read-only, least privilege, explicit scope

## Task envelope

```json
{
  "protocol": "mesh-archivist-agent/1",
  "task_id": "stable-unique-id",
  "origin_namespace": "namespace-id",
  "target_node": "node-id",
  "requested_plane": "contract-plane",
  "operation": "inspect|plan|verify|checkpoint|summon|crawl|promote|mutate",
  "scope": ["explicit/repository/path"],
  "inputs_sha256": "64-lowercase-hex",
  "expected_contract": "semver-or-digest",
  "expires_at": "RFC3339 timestamp with timezone",
  "approval_ref": null
}
```

## Authorization and routing

1. Route by namespace, node identity, plane, and pinned contract, never display name alone.
2. Reject unknown protocol versions, expired tasks, malformed digests, and undeclared targets.
3. Scope is an upper bound. Downstream agents may narrow it but MUST NOT widen it.
4. `inspect`, `plan`, and `verify` are read-only.
5. `checkpoint`, `summon`, `crawl`, `promote`, and `mutate` are side-effecting and require approval bound to target, operation, and input digest.
6. The receiving host MUST authenticate approval cryptographically. Matching fields or a non-empty object alone are not authentication.
7. Secrets and private contents MUST NOT enter task envelopes or public artifacts.
8. Agents report evidence, uncertainty, and tool limitations separately.

## Result envelope

```json
{
  "protocol": "mesh-archivist-agent/1",
  "task_id": "same-task-id",
  "node_id": "executor-node",
  "status": "completed|blocked|rejected|failed",
  "result_sha256": "64-lowercase-hex",
  "evidence": [],
  "limitations": [],
  "next_action": "none|review|approval-required"
}
```

Stop on scope mismatch, missing or unauthenticated approval, failed integrity, unresolved dependencies, trust-root mismatch, or attempted private-data promotion.
