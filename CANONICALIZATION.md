Canonicalization Rules (Execution Envelope v1)
Status

Normative
Applies to: Execution Envelope v1
Version: 1

1. Purpose

The Execution Envelope MUST produce a deterministic exec_hash.

Any implementation computing the hash MUST produce identical output given identical envelope data.

This document defines canonicalization rules required prior to hashing.

2. Hash Derivation Overview
exec_hash = SHA256(
  domain_separator || canonicalized_envelope_bytes
)


Where:

- domain_separator = ASCII string "EXEC:ENV:v1"
- canonicalized_envelope_bytes = UTF-8 encoded canonical JSON

The domain separator MUST be prepended prior to hashing.

3. Canonical JSON Requirements

The envelope MUST be canonicalized according to the following rules:

- 3.1 Encoding
- UTF-8 encoding MUST be used.
- No BOM (Byte Order Mark) permitted.

3.2 Key Ordering

All object keys MUST be sorted lexicographically by:

- Unicode code point order
- Strict ascending order

Nested objects MUST apply the same ordering rules recursively.

3.3 Whitespace

Canonical JSON MUST:

- Contain no trailing whitespace
- Contain no indentation
- Contain no newline characters
- Contain no spaces after commas or colons

Example:

Valid:

{"a":1,"b":2}


Invalid:

{
  "a": 1,
  "b": 2
}

3.4 Null Handling

Fields defined as nullable in the schema MUST:

- Be explicitly included
- Contain null if not populated
- Fields MUST NOT be omitted.

Omitting nullable fields results in a different hash and MUST be rejected.

3.5 Numbers

The following numeric encoding rules apply:

All integer fields MUST be encoded as base-10 integers

- No leading zeros allowed
- No scientific notation
- No floating point values allowed in envelope-binding fields

Amounts MUST be string-encoded integers as defined in the schema.

Example:

Valid:

"amount": "1000000000000000000"


Invalid:

"amount": 1e18
"amount": 1.0

3.6 Boolean Values

Boolean values MUST be lowercase:

- true
- false

3.7 Deterministic Field Inclusion

All required fields defined in JSON_SCHEMA.json MUST be present.

No additional properties are permitted except within metadata.

If present, metadata MUST NOT influence exec_hash unless explicitly declared by implementation.

4. Metadata Exclusion Rule

By default:

The metadata object MUST be excluded from the canonicalized envelope used for exec_hash.

Rationale:

Metadata is informational and SHOULD NOT affect authorization binding.

If an implementation chooses to bind metadata, it MUST document that deviation.

5. Domain Separator

The domain separator prevents cross-protocol replay.

Current domain separator:

"EXEC:ENV:v1"

Future versions MUST change the domain string.

6. Reference Canonicalization Approach

Implementations SHOULD use:

- RFC 8785 (JSON Canonicalization Scheme), or
- Equivalent deterministic canonicalization logic meeting the requirements above.

Implementations MUST verify:

- Byte-level equality across independent implementations
- Stable hash reproduction across platforms

7. Security Implications

Improper canonicalization may result in:

- Authorization mismatch
- Replay vector expansion
- Cross-implementation incompatibility

Hash collision surface increase

Canonicalization MUST be deterministic and strictly enforced.
