---
name: Provision a block volume and export it
description: Pick a pool, create an XSKY block volume, expose it over an access path, and snapshot it.
api: openapi/xsky-xms-swagger-original.yaml
operations: [Login, ListPools, CreateBlockVolume, GetBlockVolume, CreateAccessPath, CreateBlockSnapshot]
generated: '2026-07-21'
method: generated
---

# Provision a block volume and export it

## Steps

1. **Authenticate** — `Login` (`POST /auth/tokens:login`); send the returned
   token as `Xms-Auth-Token`.
2. **Choose a pool** — `ListPools` (`GET /pools/`) with `limit`/`offset`
   pagination; filter with `q`/`sort` if the cluster has many pools.
3. **Create the volume** — `CreateBlockVolume` (`POST /block-volumes/`).
   Provisioning is asynchronous: a `202 Accepted` means the job was queued.
4. **Poll until ready** — `GetBlockVolume`
   (`GET /block-volumes/{block_volume_id}`) until the volume reaches its
   active status before attaching clients.
5. **Export it** — `CreateAccessPath` (`POST /access-paths/`) to publish the
   volume to client hosts (e.g. iSCSI).
6. **Protect it** — `CreateBlockSnapshot` (`POST /block-snapshots/`) for a
   point-in-time copy before handing the volume to workloads.

## Rules

- Long-running changes return `202` and complete asynchronously — always poll
  the resource rather than assuming completion
  (`../conventions/xsky-conventions.yml`).
- `409 Conflict` means the resource state prevents the operation (e.g. volume
  still provisioning) — back off and re-poll; see
  `../errors/xsky-problem-types.yml`.
- There is no idempotency-key header: before retrying a failed
  `CreateBlockVolume`, list volumes (`ListBlockVolumes`) to check whether the
  first attempt landed.
