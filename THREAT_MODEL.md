Threat Model: Execution Envelope v1

Status: Normative
Applies to: Execution Envelope v1

1. Scope

This document analyzes structural threats to deterministic execution binding in atomic or near-atomic settlement environments.

Execution Envelope v1 provides cryptographic guarantees around:

- Execution parameter immutability
- Replay resistance
- Time validity enforcement
- Policy state binding
- Deterministic canonicalization
- Cross-rail isolation

This model evaluates threats mitigated by the Envelope specification itself, independent of orchestration or settlement engine design.

2. Assumptions

The model assumes:

- Adversaries may control network transport.
- Adversaries may replay or mutate payloads.
- Adversaries may attempt cross-rail misuse.
- Concurrent execution attempts may occur.
- Policy state may evolve between authorization and execution.
- Canonicalization libraries may vary across implementations.
- Settlement may be atomic or near-atomic (T+0 environments).

The model does NOT assume:

- Compromised signing keys (analyzed separately).
- Governance capture.
- Oracle correctness.
- Liquidity sufficiency.

2. STRIDE Classification
Threat Category  |	       STRIDE Class      |	Mitigated by Envelope  |	Requires External Controls

Replay attack	              Tampering	               Yes	                     No
Parameter mutation	        Tampering	               Yes	                     No
Cross-rail replay	          Spoofing	               Yes	                     No
Stale authorization	        Elevation	               Yes	                   Partial
Exposure overflow	          Tampering	               Yes	                     No
Settlement race condition 	DoS/Tamper	           Partial	                   Yes
Policy state drift	        Elevation	               Yes	                   Partial
Canonicalization mismatch	  Tampering	               Yes	                     No
Key compromise	            Spoofing	               No	                       Yes
Governance takeover	        Elevation              	 No	                       Yes

3. Threat Severity Matrix

Severity evaluated across:

Impact: Low / Medium / High / Systemic
Likelihood: Low / Medium / High

Threat         |	      Impact        |     Likelihood      |	Residual Risk

Replay	                High	                High	               Low
Parameter mutation	    High	               Medium	               Low
Stale execution   	    High	               Medium	               Low
Exposure overflow     	Systemic	           Medium	               Low
Settlement race	        High	               Medium	               Medium
Canonicalization drift	Medium	             Medium	               Low
Policy drift	          High	               Medium	               Low
Key compromise	        Systemic	           Low	                 High

Execution Envelope significantly reduces structural systemic exposure but does not mitigate key compromise or governance capture.

5. Detailed Attack Flows

5.1 Replay Attack

Scenario:

Attacker intercepts a valid Envelope and resubmits it.

Without Execution Envelope:

- Transaction may execute multiple times.

Duplicate settlement possible.

With Execution Envelope v1:

- Nonce uniqueness enforced within replay scope.
- Atomic nonce consumption prevents race replay.
- Deterministic rejection with EE_NONCE_REPLAY.

Residual Risk:

- None, provided nonce consumption is atomic.

5.2 Parameter Mutation Attack

Scenario:

- Destination, amount, rail, or constraints modified in transit.

Without Envelope:

- Signature may bind only partial fields.

Mutation may succeed.

With Envelope:

- All binding fields included in Binding Payload.
- Canonicalization via RFC 8785.
- exec_hash mismatch invalidates Authorization Object.
- Deterministic rejection (EE_HASH_MISMATCH).

Residual Risk:

- None, if canonicalization rules are strictly enforced.

5.3 Stale Authorization Attack

Scenario:

- Valid authorization reused after policy exposure caps are reduced.

Mitigation:

- TTL expiration.
- policy_hash cryptographically binds authorization to policy snapshot.
- Recomputed policy snapshot MUST match at execution time.
- Rejection Code: EE_POLICY_DRIFT

Residual Risk:

- Dependent on accurate policy snapshot derivation.

5.4 Cross-Rail Replay

Scenario:

- Envelope authorized for one rail reused on another (e.g., EVM → BANK_RAIL).

Mitigation:

- Rail descriptor included in Binding Payload.
- Domain separator isolation.
- exec_hash differs across rail contexts.

Residual Risk:

- None, if strict enforcement applied.

5.5 Exposure Overflow Race Condition

Scenario:

- Two concurrent transfers consume remaining exposure allowance.

Risk:

- Cap exceeded if state mutation non-atomic.

Mitigation Required

Envelope ensures deterministic binding but requires:

- Atomic state mutation
- Locking before execution
- Optimistic concurrency with rejection
- On-chain revert semantics (if applicable)

Residual Risk:

- Medium if implementation fails atomic state enforcement.

5.6 Canonicalization Drift

Scenario:

- Two implementations derive different canonical JSON representations.

Risk:

- False rejection
- Hash divergence
- Ambiguous binding

Mitigation:

- Mandatory RFC 8785 compliance
- Deterministic test vectors
- Byte-level equality verification

Residual Risk:

- Low if JCS strictly enforced.

5.7 Policy State Drift

Scenario:

- Policy updated between authorization and execution.

Mitigation:

- policy_hash derived from canonical policy snapshot.
- Recomputed at execution time.
- Drift produces EE_POLICY_DRIFT.

Residual Risk:

Dependent on correctness of policy snapshot retrieval.

6. Settlement-Specific Risks

As settlement approaches T+0:

- Monitoring no longer mitigates execution risk.
- Liquidity buffers shrink.
- Partial state resolution increases systemic fragility.

Execution Envelope addresses:

- Parameter immutability
- Exposure determinism
- Replay resistance
- Policy binding

Execution Envelope does NOT address:

- Liquidity insolvency
- Oracle manipulation
- Network partition events
- Cross-domain bridge failure

These require additional control layers.

6. Control Coverage Summary

Control              |          Type	         | Provided by Envelope

Deterministic                  binding	                 Yes
Single-use                  authorization	               Yes
Time-bound                     validity	                 Yes
Exposure                   cap enforcement	             Yes
Policy                      state locking	               Yes
Liquidity                     assurance	                 No
Key                            custody	                 No
Governance                    protection	               No

7. Residual Risk Summary

Execution Envelope reduces:

• Structural replay risk
• Parameter tampering
• Stale execution
• Hard-cap bypass
• Cross-environment drift
• Policy drift execution
• Canonicalization ambiguity

Execution Envelope does not mitigate:

• Key compromise
• Collusion
• Governance abuse
• Oracle risk
• Insolvency
• External settlement failure

These require additional control layers.

9. Conclusion

Execution Envelope v1 provides deterministic, cryptographically bound pre-execution authorization suitable for atomic settlement environments.

It shifts risk resolution upstream:

- From monitoring
- To deterministic authorization binding.

It does not replace custody, governance, liquidity, or economic controls.

It enforces structural invariants at execution time.
