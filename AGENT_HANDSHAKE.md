# Agent Handshake Contract

Protocol: `repo-mesh-agent/1`  
Default posture: read-only, least privilege, explicit scope

## Routing envelope

Agents exchanging work across this mesh MUST carry the following fields in their task envelope:

```json
{
  "protocol": "repo-mesh-agent/1",
  "task_id": "stable-unique-id",
  "origin_namespace": "namespace-id",
  "target_node": "node-id",
  "requested_plane": "contract-plane",
  "operation": "inspect|plan|verify|propose|mutate",
  "scope": ["explicit/repository/path"],
  "inputs_sha256": "64-lowercase-hex",
  "expected_contract": "semver-or-digest",
  "expires_at": "RFC3339 timestamp",
  "approval_ref": null
}
```

## Routing rules

1. Route by namespace, node identity, plane, and pinned contract. Never route by display name alone.
2. Reject unknown protocol versions, expired tasks, malformed digests, and undeclared target nodes.
3. A task's scope is an upper bound. A downstream agent may narrow it but MUST NOT widen it.
4. `inspect`, `plan`, and `verify` are read-only. `propose` emits a patch or plan but does not apply it.
5. `mutate` requires a separately verifiable approval reference bound to the exact target, operation, and input digest.
6. Credentials, private file contents, and secrets MUST NOT be copied into task envelopes or public artifacts.
7. Agents MUST report evidence, uncertainty, and tool limitations separately. No synthetic execution claims.

## Agent package expectations

An agent integration SHOULD ship a small adapter package or executable entry point that implements envelope validation, capability declaration, deterministic output serialization, and evidence references. The adapter MUST use the host's standard library where feasible, declare optional capabilities explicitly, and never silently fall back to broader permissions.

## Result envelope

```json
{
  "protocol": "repo-mesh-agent/1",
  "task_id": "same-task-id",
  "node_id": "executor-node",
  "status": "completed|blocked|rejected|failed",
  "result_sha256": "64-lowercase-hex",
  "evidence": [],
  "limitations": [],
  "next_action": "none|review|approval-required"
}
```

## Stop conditions

Stop on scope mismatch, missing approval for mutation, failed integrity validation, unresolved dependency, trust-root mismatch, or an attempt to promote private data. Record the reason without assigning personal blame.
