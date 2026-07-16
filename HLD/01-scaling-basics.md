# Scaling Basics

**Problem it solves:** A single server (CPU/RAM/disk/network) cannot handle growth in data volume or request rate. HLD = study of problems that appear *at scale* and their solutions.

## Core idea
- **Scale = (1) amount of data + (2) requests/sec.**
- Two knobs to increase capacity:

| | Vertical Scaling (scale up) | Horizontal Scaling (scale out) |
|---|---|---|
| What | Replace server with a more powerful one | Add more machines alongside existing ones |
| Ease | Easy — just spend money | Hard — needs distributed-systems thinking |
| Ceiling | Limited by current hardware tech (~370 cores, 12TB RAM, 2PB disk as of 2025) | Practically unlimited |
| Cost curve | Gets very expensive near the ceiling | Cheaper at scale (economy of scale on bulk hardware) |
| Use when | Early stage, simple, cost still reasonable | Once vertical scaling is maxed out or too costly |

**In practice: use both.** Scale vertically while it's cheap/simple; horizontally once you hit a wall or cost curve gets bad. Compute-heavy apps (video processing, LLM serving) need both simultaneously — powerful individual nodes *and* many of them.

## Why horizontal scaling is hard
Once you have multiple machines, you must solve: which server holds which data (sharding), which server should a request go to (routing), what if a server dies (failover/replication), how do LBs agree without constant sync (determinism). This is essentially the rest of the HLD curriculum.

## Trade-off to remember
Vertical = easy but bounded. Horizontal = unbounded but operationally painful. Almost every HLD problem downstream (load balancing, sharding, consistent hashing, replication) exists *because* horizontal scaling was chosen.

## Numbers worth remembering
- 50 PB does not fit in RAM or on a single HDD → must be distributed (Map-Reduce style).
- Typical server (2025): 1–16 cores, 1–64 GB RAM, 500GB–8TB disk.
- Max single-server config (2025): ~370 cores, 12TB RAM, 2PB disk, ~5 crore/year cost.
