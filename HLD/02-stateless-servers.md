# Stateless Servers

**Problem it solves:** If every server does both application logic *and* stores data, you get several compounding problems as you scale.

## Problems with app+data on the same server
1. **Can't scale independently** — app servers need CPU/RAM/network; DB servers need CPU/RAM/lots of fast disk. Coupling them forces every server to be over-provisioned for both.
2. **Downtime during deploys** — data on server S is only reachable through the app code on server S, so deploying new app code makes that server's data briefly unavailable. At high deploy frequency (e.g. ~2000 deploys/day at Microsoft-scale), some server is *always* mid-deploy.
3. **Cognitive/ownership overload** — app developers end up needing deep DB internals knowledge just to reason about sharding.

## The fix: separate app layer from data layer
- **State = data.** Stateless = server holds no data; Stateful = server holds data.
- App servers become **stateless** — they don't store anything, so any app server can serve any request.
- The **database gets its own Load Balancer**, separate from the app LB.
- **App LB → Round Robin is fine** (since any stateless app server can handle any request — no sharding constraint here).
- **DB LB → Consistent Hashing** (data *is* sharded, so routing must respect that).

## One-liner
Stateless app tier + Round Robin in front, stateful DB tier + Consistent Hashing in front — this is the standard pattern that lets each layer scale and deploy independently.

## Connects to
This is the architecture that makes the Round Robin vs Consistent Hashing choice make sense together instead of picking one for everything — see `03-load-balancing/02-routing-algorithms.md`.
