Canonicalization Rules (Execution Envelope v1)

Status: Normative
Applies to: Execution Envelope v1
Version: 1

1. Purpose

The Execution Envelope MUST produce a deterministic exec_hash.

Any compliant implementation computing exec_hash from identical envelope data MUST produce identical output at the byte level.

This document defines canonicalization requirements that MUST be applied prior to hashing.

2. Hash Derivation Overview

The exec_hash is computed as:

exec_hash = SHA256(
  ASCII(domain_separator) ||
  0x00 ||
  UTF8(JCS(binding_payload))
)

Where:

- domain_separator = ASCII string
- "CryptoCardia.ExecutionEnvelope.v1"
- 0x00 = single NUL byte
- binding_payload = Execution Envelope with metadata removed
- JCS() = RFC 8785 JSON Canonicalization Scheme
- UTF8() = UTF-8 encoded canonical JSON bytes

The domain separator MUST be prepended prior to hashing.

3. Binding Payload Requirement

Canonicalization MUST be performed on the Binding Payload, not the full Envelope.

For v1:

- Binding Payload = Envelope with metadata field removed.
- All other fields are binding and MUST be included.

If metadata is included in canonicalization, the resulting hash is invalid under this specification.

4. Canonical JSON Requirements (Normative)

Execution Envelope v1 implementations MUST use:

- RFC 8785 — JSON Canonicalization Scheme (JCS)

No alternative canonicalization algorithms are permitted.

4.1 Encoding

- UTF-8 encoding MUST be used.
- No BOM (Byte Order Mark) permitted.
- The exact UTF-8 byte sequence produced by JCS MUST be hashed.

4.2 Object Key Ordering

- RFC 8785 defines lexicographic sorting by Unicode code point.
- Implementations MUST NOT implement custom ordering logic.
- Nested objects MUST be canonicalized recursively.

4.3 Whitespace

RFC 8785 canonical JSON:

- Contains no indentation
- Contains no newline characters
- Contains no spaces after commas or colons
- Contains no trailing whitespace

Example (valid canonical form):

{"a":1,"b":2}

Example (invalid):

{ "a": 1, "b": 2 }

Whitespace differences MUST produce rejection.

4.4 Null Handling

Fields defined as nullable in the schema MUST:

- Be explicitly included
- Contain null when unpopulated
- Nullable fields MUST NOT be omitted.
- Omission results in a different canonical representation and MUST cause rejection.

4.5 Numeric Rules

The following numeric encoding rules apply:

Envelope Binding Fields

JSON numeric types MUST NOT be used for:

- amount
- identifiers
- nonces
- policy hashes

These MUST be string-encoded if specified as strings in the schema.

Integer Fields

Integer fields defined as JSON numbers (e.g., ttl, nonce, policy_version) MUST:

- Be base-10 integers
- Contain no leading zeros (unless value is 0)
- Not use scientific notation
- Not use floating point

Valid:

"amount": "1000000000000000000"
"ttl": 1700000000

Invalid:

"amount": 1e18
"amount": 1.0
"ttl": 01

4.6 Boolean Values

Boolean values MUST be lowercase:

- true
- false

4.7 Deterministic Field Inclusion

- All required fields defined in JSON_SCHEMA.json MUST be present.
- No additional properties are permitted except within metadata.
- If unknown fields are detected outside metadata, verification MUST fail.

5. Metadata Exclusion Rule

The metadata object MUST be excluded from the Binding Payload.

Rationale:

- Metadata is informational.
- Metadata MUST NOT affect authorization binding.
- If an implementation chooses to bind metadata, it is non-compliant with Execution Envelope v1.

6. Domain Separator

The domain separator prevents cross-protocol replay.

Current domain separator:

- CryptoCardia.ExecutionEnvelope.v1

Future versions MUST change this string.

- Domain separation MUST precede canonical JSON with a single 0x00 byte delimiter.

7. Required Hash Input Byte Layout

The exact byte sequence hashed MUST be:

ASCII("CryptoCardia.ExecutionEnvelope.v1")
0x00
UTF8(JCS(binding_payload))

Any deviation from this layout results in non-compliance.

8. Reference Implementation Guidance

Implementations MUST:

- Use a compliant RFC 8785 library
- Verify byte-level equality across independent implementations
- Confirm stable reproduction across platforms

Implementations MUST NOT:

- Perform manual JSON string concatenation
- Rely on default language-specific JSON serializers without JCS compliance
- Normalize or coerce numeric fields beyond schema definitions

9. Security Implications

Improper canonicalization may result in:

- Authorization mismatch
- Replay vector expansion
- Cross-implementation incompatibility
- Silent hash divergence
- Constraint bypass
- Increased collision surface

Canonicalization MUST be deterministic and strictly enforced.

Failure to enforce canonicalization invalidates all security guarantees of the Execution Envelope.
