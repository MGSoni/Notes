# HLD 5-Step Framework — Master Template

> **Purpose:** This is not a solved example. This is a *question bank* — the same questions apply to every HLD problem (feed, URL shortener, rate limiter, chat app, parking lot, etc.). Solve any new problem by walking through these questions in order, answering from scratch each time.
>
> **How to use this file:** Copy the "Question Checklist" section into a new note for each problem you attempt, then fill in your own answers. Keep this master file untouched as your reference.

---

## Step 1: Problem Statement (~2-3 mins)

**Goal:** Narrow an infinitely large problem into something you can actually design in 30 minutes.

### Question Checklist
1. **Who uses this and why?** — What is the one core user action? (e.g., "consume content," "shorten a link," "book a ride")
2. **What's explicitly NOT part of this?** — Name 2-3 things you're deliberately cutting. This signals scoping skill, not ignorance.
3. **Is there a real product to anchor to?** — Naming one (e.g., "like Twitter's feed") gives you free intuition about scale and edge cases later.

### Output format
One sentence: *"Design a system where [user] can [core action], optimized for [1-2 key qualities]."*

### Amendments / notes
- _(space for you to add refinements as you practice more problems)_

---

## Step 2: Functional Requirements (~3-5 mins)

**Goal:** List *features*, not architecture. Keep it short — 3-5 bullets max.

### The one trick: "What are the verbs?"
1. List the **nouns** in the problem (e.g., User, Post, Follow, Feed).
2. For each noun, ask: what can a user **do** to it? (Create / Read / Update / Delete / Follow / Search / Notify)
3. Convert each verb+noun pair into one bullet.

### Output format
- Create ___
- View/Read ___
- Update/Delete ___ (if relevant)
- Any special action unique to this system (e.g., "follow," "shorten," "book")

### Amendments / notes
- _(space for you to add refinements)_

---

## Step 3: Non-Functional Requirements (~3-5 mins)

**Goal:** Define system *qualities*. This step is the hinge — your Step 5 design should be a direct consequence of these answers.

### Fixed checklist (ask every single time, same 4 questions)
1. **Read-heavy or write-heavy?** → ratio hints at caching vs. write-optimization later.
2. **Does stale data matter (consistency)?** → strict consistency vs. eventual consistency.
3. **Does this need to feel instant (latency)?** → determines if precomputation/caching is required.
4. **Availability vs. Consistency, if forced to choose (CAP)?** → most consumer-facing read systems pick availability.

### Optional extra questions (use if relevant to the problem)
- Durability — can we ever afford to lose data? (e.g., payments = never; feed post = rarely matters)
- Security/privacy — any auth, encryption, or access-control angle?

### Output format
A short table:

| Question | Answer | Why it matters |
|---|---|---|
| Read/write ratio | ... | ... |
| Consistency | ... | ... |
| Latency | ... | ... |
| CAP choice | ... | ... |

### Amendments / notes
- _(space for you to add refinements)_

---

## Step 4: Scale Estimation (~5 mins)

**Goal:** Convert vague scale ("lots of users") into concrete numbers you can defend. This is mechanical multiplication, not creativity.

### The 4-question formula (always in this order)
1. **Total users → Daily Active Users (DAU)?** — Just pick a round number appropriate to scale (1M small, 100M-1B big). Consistency matters more than "correctness."
2. **How many times does one user do the main action per day?** — e.g., "views feed 5x/day," "shortens 1 link/week."
   → Multiply: DAU × actions/day = **total actions/day**
3. **Convert per-day → per-second** — Divide by ~100,000 (rounded seconds in a day) to get **average requests/sec**. Then multiply by 3-5x to estimate **peak requests/sec** (traffic is never uniform — always mention this).
4. **Repeat steps 1-3 for the write path** (the less-frequent action, e.g., "create post," "create short URL").

### Always finish with
**Read:Write ratio** — this single number is your justification for every caching/async decision in Step 5.

### Optional: Storage estimation
`(DAU) × (actions/day) × (avg size per item) = storage/day` → extrapolate to storage/year if asked.

### Output format
```
DAU: ___
Reads/day: ___ → Reads/sec avg: ___ → peak (x5): ___
Writes/day: ___ → Writes/sec avg: ___
Read:Write ratio: ___
Storage/day (optional): ___
```

### Amendments / notes
- _(space for you to add refinements)_

---

## Step 5: System Design (~15-20 mins)

**Goal:** Convert each NFR from Step 3 into ONE architecture decision at a time. Never invent from scratch — always trace back to "which NFR forced this?"

### The decision table (fill this in for every problem)

| NFR from Step 3 | Small question it raises | Design decision |
|---|---|---|
| Read-heavy | "How do I make reads fast without hitting DB every time?" | Add a **cache** in front of DB |
| Low latency required | "How do I avoid computing results live every request?" | **Precompute** ahead of time, store the result |
| Eventual consistency OK | "Can slow work happen off the main request path?" | Do it **asynchronously** via a **queue** |
| High availability priority | "What if one server/DB node dies?" | **Replicate** data across nodes/regions |
| Huge scale (from Step 4 numbers) | "Can one DB/server handle this alone?" | **Shard** data across multiple DBs (usually by user_id or similar key) |
| Extreme skew (e.g., a "celebrity" or "hot key" problem) | "What breaks the standard approach at the extreme?" | **Hybrid approach** — special-case the outlier instead of redesigning everything |

### The core toolbox (memorize this short list — it covers most HLD answers)
| Small question | Known tool/answer |
|---|---|
| Make reads fast | Cache (Redis / Memcached) |
| Don't block the user during slow background work | Message queue (Kafka / SQS) |
| One DB can't handle the load | Sharding (split by a key, e.g., user_id) |
| Handle extreme skew / hot keys | Hybrid push/pull, or dedicated handling for the outlier case |
| Don't lose data if a server dies | Replication (multiple copies of data) |
| Serve users across geographies fast | CDN / edge caching |
| Coordinate multiple services reliably | API Gateway / Load Balancer in front |

### Output format (in order)
1. **APIs** — 2-4 endpoints max (e.g., `POST /resource`, `GET /resource?params`)
2. **Data model** — core tables/entities and their key fields
3. **High-level flow** — walk through one read request and one write request, end to end
4. **Design decisions table** (above) — explicitly link each choice back to a Step 3 NFR
5. **Bottleneck / extreme case** — name the one thing that breaks the naive design at scale, and how you'd fix it (this is usually the most impressive part of an answer)

### Amendments / notes
- _(space for you to add refinements)_

---

## Meta-notes: how to practice this

- **Don't memorize solutions.** Memorize the *questions* in each step — the questions are identical across problems; only the answers change.
- After solving a new problem, come back and add a note in that problem's "Amendments" section about what felt hard or what new tool/pattern you learned — over time this file becomes your own reference, not just mine.
- Good practice problems to run this framework against next: URL Shortener → Rate Limiter (you've already implemented this one for real!) → Chat App → Parking Lot → Ride Sharing.
- Time-box yourself once comfortable: Steps 1-3 ≈ 10 min, Step 4 ≈ 5 min, Step 5 ≈ 15-20 min. Total ≈ 30 min, matching real interview pacing.
