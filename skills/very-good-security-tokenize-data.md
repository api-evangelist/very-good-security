---
name: Tokenize sensitive data in a VGS Vault
description: Store sensitive values (cards, bank accounts, PII) as format-preserving aliases and reveal them when authorized.
api: openapi/very-good-security-vault-openapi-original.yml
operations: [createAliases, revealAlias, revealMultipleAliases]
generated: '2026-07-21'
method: generated
---

# Tokenize sensitive data in a VGS Vault

## Setup

- Base URL: `https://api.sandbox.verygoodvault.com` for testing, `https://api.live.verygoodvault.com` (or `https://api.live-eu-1.verygoodvault.com`) for production.
- Auth: HTTP Basic with vault access credentials from the dashboard (Settings > Access credentials), or an OAuth2 client-credentials token from `https://auth.verygoodsecurity.com/auth/realms/vgs/protocol/openid-connect/token` with the `aliases:write` / `aliases:read` scopes AND the vault assigned to the credential (missing vault assignment = 401).
- Sandbox aliases are prefixed `tok_sandbox_`, live aliases `tok_live_`.

## Steps

1. **Store values** — `createAliases` (`POST /aliases`) with `data[]` of `{value, classifiers[], format, storage}`. Max **20 values per request**; payloads up to 32MB. Choose `format` (e.g. `UUID`) and `storage` (`PERSISTENT` or `VOLATILE`). The response returns each value with its new alias.
2. **Reveal one value** — `revealAlias` (`GET /aliases/{alias}`). NOTE: reveal endpoints are **disabled by default**; VGS support must enable them for your vault.
3. **Reveal in bulk** — `revealMultipleAliases` (`GET /aliases?q=alias1,alias2`) with a comma-separated alias list (also disabled by default).

## Rules

- The API is **not idempotent** — re-posting the same value stores it again; to attach a secondary alias to existing data, reference the existing alias in `data[].alias` instead of resending the raw value.
- Rate limit: 3,000 requests/minute per vault; watch `x-ratelimit-remaining`, back off on 429.
- Errors arrive as `errors[]: {status, title, detail, href}` (not RFC 9457). 400 with "Too many values" means you exceeded the 20-value batch cap.
- Never log revealed values; keep raw data out of your systems — that is the point of the vault.
