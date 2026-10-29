# Managing Blocks

This page explains how to query and lift a Block.

## Query and Unblock

| Action | How |
|---|---|
| Query | `sentry.Manager.Block.IsBlock(ip)`; returns `false` on a Redis error |
| Unblock | No API; `redis-cli DEL block:{ip}` |
| Let a blocked IP through | Add it to the Allow list; Allow takes precedence over Block |

For how block duration and the block record are computed, see [Temporary Blocking](/blocking).
