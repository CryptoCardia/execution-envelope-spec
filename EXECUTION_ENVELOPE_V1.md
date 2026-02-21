Execution Envelope Specification v1

Status: Draft
Version: 1
Domain Separator: CryptoCardia.ExecutionEnvelope.v1

1. Purpose

The Execution Envelope defines a canonical, hash-bound structure for authorizing irreversible actions in environments where execution and settlement may be atomic.

The envelope ensures that:

- Authorization binds to exact execution parameters
- Replay is prevented deterministically
- Time validity is enforced
- Hard constraints are validated before execution
- Parameter mutation invalidates authorization
- Policy state is cryptographically bound at authorization time

This specification defines structure, canonicalization, and derivation rules only.

2. Terminology

Envelope: Canonical JSON object containing all execution-bound fields.

Binding Payload (Binding View): Subset of the Envelope used to compute exec_hash.

exec_hash: SHA-256 digest of the canonicalized Binding Payload with domain separation.

Nonce: Single-use replay prevention value.

TTL: Expiry timestamp after which execution MUST revert.

Hard Constraint: Deterministic invariant that MUST hold prior to execution.

Rail: Underlying settlement mechanism (EVM, BANK_RAIL, INTERNAL, etc).

2.1 Binding Payload (Binding View)

The Binding Payload is defined as:

The Execution Envelope with the entire metadata field removed.

All other fields defined by the v1 schema are binding unless explicitly declared non-binding in a future version.

Non-Binding Fields (v1)

The following fields MUST NOT affect exec_hash:

metadata

Implementations MUST compute exec_hash over the Binding Payload only.

3. Envelope Structure

An Execution Envelope MUST be a JSON object with the following top-level fields:

{
  "version": 1,
  "intent_id": "string",
  "execution_id": "string",
  "tenant_id": "string",
  "action_type": "string",

  "rail": { ... },
  "actor": { ... },
  "target": { ... },
  "constraints": { ... },
  "policy_binding": { ... },

  "metadata": { ... }
}

All fields MUST be present unless explicitly marked optional.

Unknown fields MUST result in rejection.

4. Field Definitions
4.1 version (required)

Integer.

MUST equal 1.

4.2 intent_id (required)

Unique identifier for the logical intent.

String.

MUST be stable across retries for the same logical action.

4.3 execution_id (required)

Unique identifier for this execution attempt.

MUST differ across retries.

4.4 tenant_id (required)

Organizational boundary under which execution is authorized.

Replay scope may depend on this field.

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
  "type": "string",
  "network": "string",
  "chain_id": "integer|null",
  "contract": "string|null"
}

Rules:

- type MUST define execution environment.
- chain_id MUST be included for EVM rails.
- contract MUST be included when execution binds to a specific contract.
- Null values MUST be explicit if unused.

6. Actor Descriptor
"actor": {
  "caller": "string",
  "role": "string|null"
}

Defines initiating subject bound to execution.

7. Target Descriptor
"target": {
  "to": "string",
  "asset": "string",
  "asset_class": "integer|null",
  "amount": "string",
  "decimals": "integer"
}

Rules:

- amount MUST be string-encoded.
- JSON numeric types MUST NOT be used for amount.
- decimals MUST be integer.
- All parameters are binding.

8. Constraints
"constraints": {
  "ttl": "integer",
  "nonce": "integer",
  "replay_scope": "string"
}

8.1 TTL

UNIX timestamp (seconds, UTC).

Execution MUST reject if:

current_time > ttl + allowed_clock_skew

Default allowed_clock_skew = 0.

8.2 Nonce

Replay-prevention value.

Nonce MUST be unique within replay scope.

8.3 Replay Scope

Allowed values:

- caller
- tenant
- global

Replay uniqueness MUST be enforced within the selected scope.

8.4 Atomic Nonce Consumption (Normative)

Nonce verification MUST be atomically coupled to execution authorization.

Implementations MUST ensure:

- Nonce is reserved before irreversible execution.
- Reservation is persisted in a write preventing concurrent reuse.
- Nonce MUST NOT be reusable even if execution fails.
- Failure to enforce atomic consumption voids replay guarantees.

9. Policy Binding
"policy_binding": {
  "policy_id": "string",
  "policy_version": "integer",
  "policy_hash": "string"
}

Execution MUST bind to the exact policy state active at authorization time.

9.1 policy_hash Derivation (Normative)

policy_hash MUST be derived from the following canonical snapshot:

{
  "policy_namespace": "cryptocardia.policy.v1",
  "policy_version": "integer",
  "policy_effective_at": "integer",
  "policy_doc": { ... }
}

Canonicalization

Policy snapshot MUST be canonicalized using RFC 8785 (JCS).

Domain Separation

ASCII string:

CryptoCardia.PolicyHash.v1
Hash Computation
policy_hash = SHA256(
  ASCII(domain_sep) ||
  0x00 ||
  UTF8(JCS(policy_snapshot))
)

Encoded as lowercase hex with 0x prefix.

Drift Rule

During verification, policy snapshot MUST be recomputed.

If mismatch:

Execution MUST reject.

10. Metadata

"metadata": {
  "created_at": "integer"
}

Metadata is informational only.

Metadata MUST NOT affect exec_hash.

11. Canonicalization (Normative)

Execution Envelope v1 implementations MUST use:

RFC 8785 — JSON Canonicalization Scheme (JCS)

Additional constraints:

- No duplicate keys
- No extraneous fields
- UTF-8 encoding
- Explicit null values
- JSON numeric types MUST NOT be used for amounts
- Failure to canonicalize deterministically MUST result in rejection.

12. exec_hash Derivation (Normative)

Domain Separator:

CryptoCardia.ExecutionEnvelope.v1

Hash Input Byte Layout

The exact byte sequence MUST be:

ASCII(domain_separator)
0x00
UTF8(JCS(binding_payload))
Hash Function
exec_hash = SHA256(domain_sep || 0x00 || UTF8(JCS(binding_payload)))

Encoding:

- Lowercase hexadecimal
- 64 characters
- 0x prefix

Requirements:

- Any mutation of any binding field MUST change exec_hash.
- Authorization MUST include resulting exec_hash.
- Execution MUST verify equality prior to settlement.

13. Verification Algorithm (Normative)

Given:

- Envelope = E
- Authorization Object = A

The verifier MUST execute:

Step 0: Schema Validation

- Parse JSON
- Validate schema
- Reject unknown fields
- Reject missing fields

Step 1: Time Validation

- Reject if TTL expired.

Step 2: Replay Protection

- Verify nonce unused within replay scope.
- Atomically consume nonce.
- Reject if replay detected.

Step 3: Compute exec_hash

- Remove metadata
- Canonicalize Binding Payload
- Compute SHA-256 with domain separation

Step 4: Verify Authorization Object

- Signature valid
- AO subject matches expected
- AO tenant matches envelope
- AO audience valid
- AO not expired
- AO references computed exec_hash
- Reject if mismatch.

Step 5: Verify Policy Binding

- Recompute policy snapshot
- Derive policy_hash
- Compare
- Reject on mismatch.

Step 6: Enforce Hard Constraints

Validate:

- Exposure ceilings
- Balance checks
- Destination restrictions
- Velocity limits
- Rail-specific invariants
- Reject on violation.

Step 7: Permit Execution

If all checks pass, execution MAY proceed.

14. Security Considerations

This envelope mitigates:

- Replay attacks
- Stale authorization execution
- Parameter mutation
- Policy drift execution
- Cross-rail execution drift
- Settlement race conditions
- Exposure overflow
- Partial replay under concurrency

See THREAT_MODEL.md.

15. Scope and Limitations

This specification:

- Does not define risk scoring
- Does not define orchestration logic
- Does not define settlement engines
- Does not define governance upgrade paths
- Does not define Authorization Object structure

It defines deterministic execution-binding semantics only.
