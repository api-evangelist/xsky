---
name: Create an object-storage user and bucket
description: Provision an XEOS (S3-compatible) object-storage user and bucket through the XMS controller API.
api: openapi/xsky-xms-swagger-original.yaml
operations: [Login, CreateObjectStorageUser, ListObjectStorageUsers, CreateBucket, ListBuckets]
generated: '2026-07-21'
method: generated
---

# Create an object-storage user and bucket

XEOS object storage is S3-compatible (see
`../conformance/xsky-conformance.yml`); users and buckets are administered
through the XMS controller API, then consumed by S3 clients.

## Steps

1. **Authenticate** — `Login` (`POST /auth/tokens:login`); send the token as
   `Xms-Auth-Token`.
2. **Check for an existing user** — `ListObjectStorageUsers`
   (`GET /os-users/`) to avoid duplicates.
3. **Create the user** — `CreateObjectStorageUser`
   (`POST /os-users/`). This is the principal that will own
   buckets and S3 credentials.
4. **Create the bucket** — `CreateBucket` (`POST /os-buckets/`)
   owned by that user.
5. **Verify** — `ListBuckets` (`GET /os-buckets/`) and confirm the
   new bucket appears with the expected owner and quota.

## Rules

- Creation calls may return `202 Accepted` — poll the resource before handing
  credentials to applications (`../conventions/xsky-conventions.yml`).
- `409 Conflict` on create usually means the name is taken — list first, and
  never retry a create blindly (no idempotency-key mechanism).
- Error semantics: `../errors/xsky-problem-types.yml`.
