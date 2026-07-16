# Load Balancing — Fundamentals

**Problem it solves:** A single server can't handle all traffic, and end users shouldn't need to know/care which of many backend servers handles them.

## Core idea
The **Load Balancer (LB)** sits between DNS and the app servers. DNS resolves the domain to the LB's IP, not to individual servers.

Two jobs of the LB:
1. Give a **unified view** of the backend to the client.
2. **Distribute load** evenly across servers.

### How the LB tracks live servers
- **Heartbeat** (push): each server pings `/heartbeat` on the LB periodically (~10s); LB assumes dead if it misses several beats.
- **Healthcheck** (pull): LB pings each server's `/healthcheck`; times out → assumed dead.
- New servers **register** themselves (`/register`) or the LB is configured with an IP range to probe.

### Is the LB itself a bottleneck / SPoF?
- A single LB can handle ~100K+ req/sec (vs. an app server's ~100–1000 req/sec), so it's rarely a bottleneck — until Google-scale (10M req/sec).
- It **is** a single point of failure if only one LB exists.
- **Fix:** multiple LBs, with **DNS acting as a load balancer in front of the LBs** — DNS returns multiple IPs (one per LB); client/GeoDNS picks one. Do NOT add "another LB in front of LBs" — that just moves the SPoF up one level.
- Less common alternative: active/passive (hot/cold) LB failover.

## Trade-off table
| Approach | Bottleneck risk | SPoF risk | Fix |
|---|---|---|---|
| Single LB | Low (100K+ req/s capacity) | Yes | Multiple LBs |
| Multiple LBs, no coordination | No | No | Need DNS or GeoDNS to distribute across them |
| LB-in-front-of-LBs | — | Just moved the SPoF up | Don't do this |

## Numbers worth remembering
- App server: ~100–1000 req/sec.
- LB: ~100,000+ req/sec.
- Google-scale traffic: ~10M req/sec (needs many LBs, not one).

## Connects to
Once requests reach the LB, it must decide **which server** to forward to — that's the Routing Algorithm (see `03-load-balancing/02-routing-algorithms.md`), which is tightly coupled to how data is **sharded** (`04-databases/01-sharding.md`).
