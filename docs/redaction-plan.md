# Log Redaction Plan

## Goal

Ensure request logs never contain raw user identifiers before the next release ships.

## Approach

We redact sensitive fields at the **logging layer** so that every log emit point is covered by a single, centralized policy rather than scattered, ad-hoc masking.

### 1. Identify sensitive fields

User identifiers currently appearing in request logs:

- User IDs
- Email addresses
- Authorization/session tokens
- IP addresses

### 2. Centralized redaction middleware

Introduce a redaction filter applied to every structured request log event. The filter recognizes a known list of sensitive keys (`user_id`, `email`, `token`, `session`, `ip`) and replaces their values before serialization.

### 3. Masking scheme

- User IDs: hash to a stable pseudonym, preserving correlation without exposing the raw value.
- Emails: keep the domain, mask the local part (e.g. `he***@example.com`).
- Tokens: never logged; any value matching token patterns is replaced with `[REDACTED]`.
- IP addresses: truncate the last octet / mask the host portion.

### 4. Fail-safe default

If a field's sensitivity cannot be determined, redact by default. Any unclassified payload in a request log is dropped rather than emitted raw.

### 5. Verification

- Add a log-scanning test that asserts no configured identifier patterns appear in emitted request logs.
- Run the scanner against sample production traffic before the next release to confirm no raw identifiers remain.

## Rollout

1. Land the redaction filter behind a flag.
2. Enable in staging and compare pre/post logs for completeness of redaction.
3. Enable everywhere and remove the flag once verified.
