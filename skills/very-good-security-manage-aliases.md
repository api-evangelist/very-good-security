---
name: Manage alias lifecycle in a VGS Vault
description: Re-classify stored values and delete aliases when data must be retired.
api: openapi/very-good-security-vault-openapi-original.yml
operations: [updateAlias, deleteAlias, createAliases]
generated: '2026-07-21'
method: generated
---

# Manage alias lifecycle in a VGS Vault

## Setup

Same environments and auth as the tokenize skill: Basic auth vault access credentials or OAuth2 client-credentials (`aliases:write`/`aliases:delete` scopes, vault-bound). Sandbox host `https://api.sandbox.verygoodvault.com`.

## Steps

1. **Re-classify a value** — `updateAlias` (`PUT /aliases/{alias}`) with `data.classifiers[]` (e.g. `["credit-cards", "PII"]`). Returns 204 No Content. Classifiers drive routing/redaction policy downstream.
2. **Attach a secondary alias** — `createAliases` (`POST /aliases`) referencing the existing alias in `data[].alias` with a different `format`; both aliases then point at the same underlying value.
3. **Retire data** — `deleteAlias` (`DELETE /aliases/{alias}`). Returns 204. Deleting an alias removes that alias reference; use this for data-retention and right-to-erasure flows.

## Rules

- Deletion is permanent — verify the alias (and environment prefix `tok_sandbox_` vs `tok_live_`) before `deleteAlias`.
- All failures use the `errors[]` envelope; 401 usually means the credential is not assigned to the target vault.
- Respect the 3,000 req/min vault rate limit (`x-ratelimit-remaining` header, 429 on exceed).
