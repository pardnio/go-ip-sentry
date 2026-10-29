# Managing Allow and Deny Lists

This page explains the manual steps needed to remove an Allow or Deny entry and how runtime additions behave across instances.

## Removing

There is no removal API. Because `Check` queries Redis first, editing the file alone has no effect; do all of the following:

1. Delete the Redis key: `redis-cli DEL allow:203.0.113.10`
2. Remove the entry from the list file
3. Restart every instance to clear the memory caches

## Multi-Instance Boundaries

| Case | Behavior |
|---|---|
| Instance A calls `Add` | Redis applies to every instance at once; only A's memory cache and list file have the entry |
| Several instances share one list file | Each `Add` overwrites the whole file from its own cache, erasing entries other instances added |
| Restart after Redis is flushed | Only entries written to the list file come back |

When consistency across instances matters, treat Redis as the source of truth and the list file as a single-instance backup.

For how the lists load, are queried, and behave across instances, see [Allow and Deny Lists](/access-lists).
