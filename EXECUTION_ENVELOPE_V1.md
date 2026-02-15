Execution Envelope Specification v1
Status

Draft
Version: 1
Domain Separator: TGS:EXEC:ENV:v1

1. Purpose

The Execution Envelope defines a canonical, hash-bound structure for authorizing irreversible actions in environments where execution and settlement may be atomic.

The envelope ensures that:

- Authorization binds to exact execution parameters
- Replay is prevented deterministically
- Time validity is enforced
- Hard constraints are validated before execution
- Parameter mutation invalidates authorization

This specification defines structure and derivation rules only.

2. Terminology

- Envelope: Canonical JSON object containing all execution-bound fields.
- exec_hash: SHA256 digest of the canonicalized envelope with domain separation.
- Nonce: Monotonic or single-use replay prevention value.
- TTL: Expiry timestamp after which execution MUST revert.
- Hard Constraint: Deterministic invariant that MUST hold prior to execution.
- Rail: Underlying settlement mechanism (EVM, Bank, Internal, etc).

3. Envelope Structure

An Execution Envelope MUST be a JSON object with the following top-level fields:

{
  "version": 1,
  "intent_id": string,
  "execution_id": string,
  "tenant_id": string,
  "action_type": string,

  "rail": { ... },
  "actor": { ... },
  "target": { ... },
  "constraints": { ... },
  "policy_binding": { ... },

  "metadata": { ... }
}


All fields MUST be included unless explicitly marked optional.

4. Field Definitions:

4.1 version (required)

- Integer. MUST equal 1.

4.2 intent_id (required)

- Unique identifier for the logical intent.

4.3 execution_id (required)

- Unique identifier for this specific execution attempt.

4.4 tenant_id (required)

- Identifier for the organizational boundary under which execution is authorized.

4.5 action_type (required)

Examples:

- TRANSFER
- MINT
- REDEEM
- ADMIN_CHANGE
- AUTHN
- AUTHZ

Implementations MAY define additional action types.

5. Rail Descriptor
"rail": {
  "type": string,
  "network": string,
  "chain_id": integer | null,
  "contract": string | null
}

Rules:

- type MUST define execution environment (e.g., EVM, BANK_RAIL, INTERNAL).
- chain_id MUST be included for EVM rails.
- contract MUST be included when execution binds to a specific contract.
- Null values MUST be explicit if unused.

6. Actor Descriptor
"actor": {
  "caller": string,
  "role": string | null
}


Defines the initiating subject bound to execution.

7. Target Descriptor
"target": {
  "to": string,
  "asset": string,
  "asset_class": integer | null,
  "amount": string,
  "decimals": integer
}

Rules:

- amount MUST be string-encoded to prevent float ambiguity.
- asset_class MAY define partitioned exposure buckets.

All parameters are execution-binding.

8. Constraints
"constraints": {
  "ttl": integer,
  "nonce": integer,
  "replay_scope": string
}

8.1 ttl

Unix timestamp (seconds).
Execution MUST revert if current_time > ttl.

8.2 nonce

Replay-prevention value.
Implementation MUST enforce single-use semantics.

8.3 replay_scope

Defines replay boundary. Examples:

- caller
- tenant
- global

9. Policy Binding
"policy_binding": {
  "policy_id": string,
  "policy_version": integer,
  "policy_hash": string
}


This binds execution to a specific policy state at authorization time.

If policy state changes, new authorization MUST be required.

10. Metadata
"metadata": {
  "created_at": integer
}


Non-binding informational fields.

Metadata MUST NOT affect exec_hash semantics.

11. Canonicalization

Prior to hashing, the envelope MUST:

- Use stable key ordering
- Include null values explicitly
- Use UTF-8 encoding
- Contain no extraneous whitespace
- Represent all numeric values deterministically

See CANONICALIZATION.md for exact rules.

12. exec_hash Derivation
exec_hash = SHA256(
  domain: "TGS:EXEC:ENV:v1",
  canonicalized_envelope
)

Requirements:

- Domain separator MUST be included.
- Any mutation of any field MUST change the resulting hash.
- Authorization MUST include the resulting exec_hash.
- Execution MUST verify equality prior to settlement.

13. Hard Constraint Enforcement

Implementations MUST validate:

- TTL not expired
- Nonce unused
- Exposure ceilings not exceeded
- Sufficient balance (if applicable)
- Policy binding matches current policy state

Failure of any constraint MUST result in deterministic rejection.

14. Security Considerations

This envelope mitigates:

- Replay attacks
- Stale authorization execution
- Parameter mutation
- Cross-rail execution drift
- Settlement race conditions
- Exposure overflow

See THREAT_MODEL.md.

15. Scope and Limitations

This specification:

- Does not define risk scoring
- Does not define orchestration logic
- Does not define settlement engines
- Does not define governance upgrade paths

It defines execution-binding semantics only.
