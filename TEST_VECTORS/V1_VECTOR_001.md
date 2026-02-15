# Execution Envelope v1 Test Vector 001

## Domain Separator
EXEC:ENV:v1

## Hash Input Construction
hash_input = ASCII("EXEC:ENV:v1") || UTF8(canonical_json)

## Canonical JSON
See: V1_CANONICAL_001.json

## Expected exec_hash (SHA-256, hex)

41a95b883c3034a20749ba6195086d155aa3fe672ce0e4260f352accf09fe4bf

