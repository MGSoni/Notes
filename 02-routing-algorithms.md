# Routing Algorithms (Round Robin → Consistent Hashing)

**Problem it solves:** Given N servers, which one should handle a given request/user's data? This algorithm runs inside the LB.

## What makes a routing algorithm "good"
1. **Fast** — cheap to compute.
2. **Equal distribution** — no server gets 2-3x more load.
3. **Elastic** — servers can be freely added/removed (they crash; you also scale out).
4. **Minimal data movement** on add/remove — only affected users should move.
5. **Deterministic without coordination** — multiple LBs must independently agree on the same server for the same user, without talking to each other (constant cross-LB sync is too expensive).

## Trade-off table — this is the highest-yield thing to memorize

| Algorithm | Mechanism | Fast? | Equal dist? | Add/remove servers? | Data movement on change | No-sync needed? |
|---|---|---|---|---|---|---|
| **Round Robin** (`user_id % N`) | Modulo on live server count | ✅ | ✅ | ✅ | ❌ Massive — N changes, so nearly all mappings change | ✅ |
| **Bucketing** (`user_id / bucket_size`) | Fixed ranges per server | ✅ | ✅ | ❌ Can't even add new users past the cap | ❌ Massive reshuffle | ✅ |
| **Mapping Table** (hashmap of user→server) | Explicit lookup table | ✅ | ✅ | ✅ | ✅ Minimal (only affected users) | ❌ Table must be kept in sync across all LBs — hard (CAP theorem territory) |
| **Consistent Hashing** | Hash ring | ✅ | ✅ (improves with more virtual nodes) | ✅ | ✅ Minimal | ✅ — same hash functions baked into every LB's code |

**One-liner:** Round Robin and Bucketing are simple but reshuffle everything on any change. Mapping Tables fix that but require expensive cross-LB synchronization. Consistent Hashing gets all the benefits with none of the drawbacks — it's the answer whenever the interviewer asks "how do you route/shard with minimal disruption."

## Consistent Hashing — mechanism
1. Use `k` hash functions (typically k=32 or 64) to place each **server** at k points on a hash ring (output space e.g. 0…2^64-1).
2. Use 1 hash function to place each **user/request** on the same ring.
3. Route the request to the **first server clockwise** from the user's position (binary search on the sorted ring → O(log(N·k))).

**Why it satisfies all 5 properties:**
- Fast: binary search on a sorted ring (~30 iterations even at Google scale ≈ microseconds).
- Equal distribution: more virtual nodes per server (higher k) → more even spread.
- Elastic: adding/removing a server only touches the ring positions near it.
- Minimal movement: when a server dies, only *its* users get redirected to their next-clockwise server; everyone else is untouched. When a server is added, it takes a small slice from *every* other server (not all from one).
- No sync needed: every LB runs the *same* hash functions and knows the *same* live-server list (via heartbeat/healthcheck) → they independently compute identical rings → automatically agree, with zero cross-LB communication.

**Why "cascading failure" is avoided:** if one server dies, its load spreads evenly across *all* remaining servers rather than dumping onto one neighbor (which could then also overload and die — a chain reaction).

## Common confusion to watch for
- Hash collisions are theoretically possible but negligible in practice (astronomically low probability) and don't break the algorithm even if they occur.
- Consistent hashing solves *routing*; it does **not** by itself solve data replication/durability — that's a separate concern (see sharding card: "SPoF per user" problem is fixed by replication, not by the routing algorithm).

## Numbers worth remembering
- Binary search over N·k entries: `log2(N*k)`. Example: Google, N=10M servers, k=64 → ~30 iterations ≈ 3 microseconds.
- Mapping table cost: ~12 bytes/user (8B user_id + 4B server_id). At 5B users ≈ 60GB — manageable but must be replicated/synced across every LB, which is the actual bottleneck (not the size).
