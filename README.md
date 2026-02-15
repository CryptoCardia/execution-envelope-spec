# execution-envelope-spec
A rail-agnostic specification for cryptographically binding execution parameters, exposure ceilings, and time constraints in real-time settlement environments.

Execution Envelope Specification (v1)
Deterministic Binding for Irreversible Actions

As settlement cycles compress toward real-time finality, traditional supervision models become structurally insufficient.

In T+1 environments, risk controls may exist downstream of execution.
In T+0 environments, execution and settlement become atomic.

When finality is instantaneous, exposure must be resolved before execution occurs.

The Execution Envelope defines a deterministic, rail-agnostic structure for binding:

- Authorization
- Exposure ceilings
- Execution parameters
- Time constraints
- Replay protection

into a single hash-bound unit of execution.

Core Principles:

Deterministic Enforcement
Execution must revert if any bound parameter deviates from the authorized state.

- Single-Use Authorization
- Nonce-bound and replay-resistant by construction.
- Time-Bound Validity
- Authorizations expire deterministically via TTL.
- Hard Constraint Modeling
- Exposure ceilings and invariant conditions must be enforced before execution.

Rail-Agnostic Structure
Applies to:

- EVM transactions
- Bank rails (ACH, RTP, FedNow)
- Tokenized asset issuance
- Administrative policy changes
- Authentication elevation

Execution Envelope Structure

An execution envelope is a canonical JSON object hashed with a domain separator.

exec_hash = SHA256(
  domain: "TGS:EXEC:ENV:v1",
  canonicalized_envelope
)

The resulting exec_hash must be included in the Authorization Object and verified before any rail execution.

Envelope High-Level Fields:

- intent_id
- execution_id
- tenant_id
- action_type
- rail descriptor
- actor
- target
- constraints (ttl, nonce, replay scope)
- policy binding metadata
- timestamp metadata

Full specification: see EXECUTION_ENVELOPE_V1.md

Why Monitoring Is Insufficient:

- Monitoring detects anomalies after execution.
- Execution envelopes prevent unauthorized or structurally invalid actions from executing in the first place.

As settlement latency approaches zero, enforcement must move upstream.

Threat Model Coverage

This specification mitigates:

- Replay attacks
- Stale authorization execution
- Parameter mutation
- Cross-rail replay
- Hard-cap bypass attempts
- Settlement race conditions

See THREAT_MODEL.md for detailed analysis.

Scope:

This repository defines a specification only.

It does not include:

- Production policy engines
- Multi-tenant orchestration logic
- Risk scoring implementations
- Rail-specific settlement engines

Intended Audience:

- Security architects
- Infrastructure engineers
- Digital asset custodians
- Tokenization platforms
- Risk and settlement system designers

Status

Specification draft v1
Open for architectural discussion and peer review.
