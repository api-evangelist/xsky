---
name: Authenticate to XMS and manage access tokens
description: Log in to an XSKY XMS controller, mint and validate long-lived access tokens, and rotate them safely.
api: openapi/xsky-xms-swagger-original.yaml
operations: [Login, CreateAccessToken, ValidateAccessToken, RegenerateAccessToken, Logout]
generated: '2026-07-21'
method: generated
---

# Authenticate to XMS and manage access tokens

All XMS requests are relative to `https://<xms-controller>/v1` and authenticated
with the `Xms-Auth-Token` header (a `token` query parameter also works, but the
header is preferred — see `../conventions/xsky-conventions.yml`).

## Steps

1. **Log in** — `Login` (`POST /auth/tokens:login`) with the XMS user's
   credentials. The response contains a session token; send it as
   `Xms-Auth-Token` on every subsequent call.
2. **Mint a long-lived token** for automation — `CreateAccessToken`
   (`POST /access-tokens/`). Use this for CI/agent workloads instead of
   session tokens.
3. **Validate before use** — `ValidateAccessToken` (`POST /access-tokens:validate`)
   to confirm a stored token is still good before starting a long workflow.
4. **Rotate** — `RegenerateAccessToken`
   (`POST /access-tokens/{access_token_id}:regenerate`) replaces the secret
   without recreating the resource; update your secret store atomically.
5. **Log out** — `Logout` (`POST /auth/tokens:logout`) when a session token is
   no longer needed.

## Rules

- There is no idempotency-key mechanism (`../conventions/xsky-conventions.yml`);
  do not blindly retry `POST` calls — check for the created resource first.
- `401` means the token is missing/expired: re-run `Login`. `403` means the
  user lacks permission — do not retry, escalate. See
  `../errors/xsky-problem-types.yml`.
- Auth profile detail: `../authentication/xsky-authentication.yml`.
