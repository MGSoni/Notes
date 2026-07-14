# Data Sharding

**Problem it solves:** Once you have multiple servers, you must decide *where* each piece of data lives, in a way that's repeatable (so you can find it again).

## Core idea — types of partitioning
| Type | What it splits | Why |
|---|---|---|
| **Vertical partitioning (normalization)** | Columns, same server | Removes DB anomalies/redundancy (classic normalization) |
| **Horizontal partitioning** | Rows, same server | Better indexing, multi-tenancy isolation, handling hot rows (e.g. Twitter influencers separated from normal users) |
| **Vertical partitioning across servers** | Tables → different servers | Microservice separation of concerns (e.g. auth service vs profile service) |
| **Sharding = Horizontal partitioning across servers** | Rows → different servers | Data no longer fits on one server; also boosts read/write throughput |

## Sharding key
- Every shard is based on one **sharding key** (can be composite, e.g. `(city, gender)`, or a single column like `user_id`).
- You cannot have multiple independent sharding keys for the same data.

## The critical insight: Sharding ⇔ Routing
**Sharding is not a separate algorithm — it's a side effect of routing.**
- Whatever logic decides "which request → which server" *automatically* decides "which data → which server," because a server only ever receives (and therefore only ever stores) the data for the users routed to it.
- Therefore sharding logic and routing logic **must be identical** — if they diverge, requests will hit servers that don't have the data.

## Trade-off / risk to remember
If all of a user's data lives on exactly one server (because of the sharding key), that server is a **SPoF for that user**. Fixed later via **replication** — not by changing the sharding scheme.

## Connects to
Because sharding = routing, the real design problem is choosing a good **routing algorithm** — see `03-load-balancing/02-routing-algorithms.md`.
