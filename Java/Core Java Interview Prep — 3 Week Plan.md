# Core Java Interview Prep — 3 Week Plan
**Target:** Senior SWE, product companies · **You:** 6+ yrs Java, rusty on internals · **Style:** rebuild pattern-level intuition, not rote syntax

---

## How to use this

Each topic below has:
- **The 20% that gets asked 80% of the time**
- **A memory hook** (one-line anchor so you don't blank under pressure)
- **Tie-back** to something you've actually shipped (IDFC/Verizon), so you can answer "have you used this" with a real story, not theory

Do 1 topic block/day. Weekends = mock interview + weak-spot review. Keep a `mistakes.md` like you do for DSA — write down every question you fumbled, in your own words, same night.

---

## Depth Guide

Given 6+ years of experience, here's where the notes alone are likely enough vs where you should budget extra time with an outside source (Baeldung / Oracle docs) before moving on:

- 🟢 **Quick refresh** — you've used this daily for years, it's rust not a gap. Read, say it out loud once, move on.
- 🟡 **Go deeper** — classic whiteboard/trick-question territory. If you stumble explaining it out loud, stop and pull up a real source before continuing.

---

## WEEK 1 — OOP + Collections (the foundation everyone tests first)

### Day 1 — OOP Pillars, done properly 🟢
- **Encapsulation**: hiding state, not just private fields — the *why* is controlling invariants.
- **Abstraction**: interface vs abstract class — interface = contract/capability ("can fly"), abstract class = partial identity + shared state ("is a Bird").
- **Inheritance**: prefer composition when relationship is "has-a" or behavior varies — classic trap question: "why favor composition over inheritance?" → tight coupling, fragile base class problem.
- **Polymorphism**: compile-time (overload) vs runtime (override) — know that overloading resolution happens at compile time based on static type, overriding at runtime based on actual object.
- **Memory hook:** *"Interface = what you can do, Abstract class = what you are."*
- **Your story:** Composite API platform — unifying NACH + UPI Intent behind one interface is a textbook abstraction example. Rehearse explaining it that way.

### Day 2 — equals/hashCode, immutability, String pool 🟢
- Contract: equal objects **must** have equal hashCodes; violating this silently breaks HashMap/HashSet.
- Why String is immutable: security (used as keys, class names), caching (String pool), thread-safety, hashcode caching.
- `String s = "a"` vs `new String("a")` — pool vs heap. `intern()`.
- Immutable class checklist: final class, final fields, no setters, defensive copy of mutable fields in constructor/getter.
- **Memory hook:** *"hashCode gets you to the bucket, equals confirms you're home."*

### Day 3 — Collections: List & Set internals 🟢
- ArrayList: backed by array, amortized O(1) add, resize = 1.5x (growth factor), O(n) insert/remove at head.
- LinkedList: O(1) insert/remove at ends, O(n) access — rarely the right answer in interviews unless they ask specifically about it.
- HashSet = HashMap under the hood (value is a dummy constant).
- TreeSet/TreeMap: Red-Black tree, O(log n), sorted iteration, needs Comparable/Comparator.
- LinkedHashMap: preserves insertion order — good for LRU cache (you can say "I'd reach for this" and it ties to your Redis caching work).
- **Memory hook:** *"Array = fast read, slow insert. Linked = slow read, fast insert at ends."*

### Day 4 — HashMap internals (this WILL be asked) 🟡
- Structure: array of buckets, each bucket = linked list (or tree if bucket size > 8, Java 8+).
- `hash()` spreads bits to reduce collisions; index = `hash & (capacity-1)`.
- Resize: when size > capacity * loadFactor (default 0.75), capacity doubles, everything rehashed.
- Treeification: bucket becomes a Red-Black tree above 8 entries (and table size ≥ 64) — turns worst-case O(n) into O(log n).
- **Common follow-up:** "what if hashCode is always the same?" → all entries in one bucket, degrades to O(n) (or O(log n) post-Java 8 treeification).
- **Memory hook:** *"HashMap = array of buckets; each bucket is a list until it gets crowded, then it becomes a tree."*
- **Your story:** you built a Redis-based caching layer at Verizon — great segue to explain in-memory vs distributed caching, and you can draw the HashMap analogy to Redis's own hash-slot sharding if asked about scaling caches.

### Day 5 — ConcurrentHashMap vs synchronized collections 🟡
- Old way: `Collections.synchronizedMap()` — locks the entire map on every operation.
- ConcurrentHashMap (Java 8+): no external locking needed for reads; writes use fine-grained locking (per-bin, via CAS + synchronized on bin head) instead of one global lock — far better under contention.
- No null keys/values allowed (unlike HashMap) — because null return is ambiguous with "key absent" in a concurrent context.
- **Memory hook:** *"HashMap for single-threaded, ConcurrentHashMap for shared state, never synchronizedMap unless you're maintaining legacy code."*

### Day 6 — Comparable vs Comparator + generics basics 🟢
- Comparable = natural ordering, one per class, `compareTo`.
- Comparator = external, multiple strategies, `compare`, lambda-friendly (`Comparator.comparing(...).thenComparing(...)`).
- Generics: type erasure — generics exist at compile time only, gone at runtime (why you can't do `new T()` or `T[] arr`).
- Bounded wildcards: `? extends T` (producer, read-only), `? super T` (consumer, write-only) — PECS rule: *Producer Extends, Consumer Super*.
- **Memory hook:** *"PECS — Producer Extends, Consumer Super."*

### Day 7 — Weekend: Mock + Review
- Do a self-timed 45-min mock: pick 5 questions from the week, answer out loud, then check against notes.
- Update `mistakes.md`.

---

## WEEK 2 — Concurrency (this is where senior candidates get separated from mid-level)

### Day 8 — Thread fundamentals + lifecycle 🟢
- States: NEW → RUNNABLE → (BLOCKED/WAITING/TIMED_WAITING) → TERMINATED.
- `Runnable` vs `Thread` (extend) vs `Callable` (returns value, throws checked exceptions).
- `start()` vs `run()` — classic gotcha: calling `run()` directly just executes on the current thread, no new thread created.
- **Memory hook:** *"start() spawns, run() just runs where you are."*

### Day 9 — synchronized, volatile, and the memory model 🟡
- `synchronized`: mutual exclusion + visibility (happens-before guarantee) — monitor lock per object.
- `volatile`: visibility only, no atomicity — good for flags/status, not counters.
- Classic gotcha: `volatile int counter++` is NOT thread-safe (read-modify-write isn't atomic even with volatile).
- Deadlock: 4 conditions (mutual exclusion, hold-and-wait, no preemption, circular wait) — prevention = consistent lock ordering.
- **Memory hook:** *"volatile = everyone sees the same value, synchronized = only one thread touches it at a time."*
- **Your story:** your retry orchestration + reconciliation workflows at IDFC First almost certainly deal with concurrent access to shared state — think through how you'd frame that with correct vocabulary (idempotency, race conditions, at-least-once vs exactly-once).

### Day 10 — Locks, Atomic classes 🟡
- `ReentrantLock` vs `synchronized`: explicit lock/unlock, tryLock (non-blocking), fairness policy, interruptible — but you must remember to unlock in `finally`.
- `ReadWriteLock`: multiple readers OR one writer — good when reads >> writes.
- `AtomicInteger`/`AtomicLong`: CAS (compare-and-swap) based, lock-free, faster than synchronized for simple counters.
- **Memory hook:** *"Atomic classes = lock-free counters via CAS; use them before reaching for synchronized on a simple counter."*

### Day 11 — Executor Framework & Thread Pools 🟡
- Why thread pools: creating threads is expensive; pools reuse them.
- `ExecutorService`, common pool types: `FixedThreadPool`, `CachedThreadPool`, `ScheduledThreadPool`, `SingleThreadExecutor` — know when each is wrong (CachedThreadPool can explode thread count under load — real production risk).
- Thread pool sizing: CPU-bound → ~number of cores; I/O-bound → higher (cores × (1 + wait/compute ratio)).
- `Future` (blocking `.get()`) vs `CompletableFuture` (non-blocking, composable via `.thenApply`, `.thenCombine`, exception handling via `.exceptionally`).
- **Memory hook:** *"CPU-bound: few threads. I/O-bound: more threads (they're mostly waiting, not computing)."*
- **Your story:** async processing + retry orchestration for 20,000+ daily transactions — this is your strongest concurrency story. Practice explaining thread pool choice and backpressure handling here.

### Day 12 — Producer-Consumer, BlockingQueue, wait/notify 🟢
- `wait()`/`notify()`/`notifyAll()` — must be called inside synchronized block, releases the lock while waiting (unlike `Thread.sleep`).
- `BlockingQueue` (ArrayBlockingQueue, LinkedBlockingQueue) — solves producer-consumer without manual wait/notify; this is what a real interviewer wants to hear you reach for.
- Kafka is essentially a distributed producer-consumer system — you already have the real-world analogy, use it.
- **Memory hook:** *"wait/notify is the manual transmission; BlockingQueue is automatic — use automatic unless asked to implement it from scratch."*

### Day 13 — ThreadLocal, CompletableFuture deep dive, common pitfalls 🟡
- `ThreadLocal`: per-thread variable copy — common use: DB connections, user context in web requests. Danger: memory leaks in thread pools if not cleared (`.remove()`).
- CompletableFuture chaining: `.thenApply` (sync transform) vs `.thenApplyAsync` (runs on different thread from pool) — know the distinction, it's a common trick question.
- **Memory hook:** *"ThreadLocal = each thread's own drawer; forget to clear it in a pooled thread and the next task inherits your mess."*

### Day 14 — Weekend: Mock + Review
- Full concurrency mock: explain thread pool sizing decision + walk through a deadlock scenario and how you'd fix it.
- Update `mistakes.md`.

---

## WEEK 3 — JVM Internals + Integration + Mock Interviews

### Day 15 — JVM Memory Model 🟡
- Heap (Young Gen: Eden + 2 Survivor spaces, Old Gen) vs Stack (per-thread, method frames, local vars) vs Metaspace (class metadata, replaced PermGen in Java 8).
- Object allocation: starts in Eden → survives GC → moves to Survivor → promoted to Old Gen after enough survived cycles.
- **Memory hook:** *"Young Gen = nursery, Old Gen = retirement home; objects that survive enough evictions get promoted."*

### Day 16 — Garbage Collection 🟡
- Mark-and-sweep basics; know GC types exist (Serial, Parallel, CMS, G1, ZGC) at a high level — G1 is default since Java 9, aims for low pause times via region-based collection.
- `finalize()` is deprecated — don't mention it as best practice.
- Memory leaks in Java despite GC: still happen via long-lived references (static collections, unclosed listeners, ThreadLocal misuse — ties back to Day 13).
- **Memory hook:** *"GC doesn't prevent leaks, it just prevents dangling pointers — a reference you forgot to drop is still a leak."*

### Day 17 — Exceptions, try-with-resources, Java 8+ features 🟢
- Checked vs unchecked — checked forces handling at compile time (IOException), unchecked = programming errors (NullPointerException).
- try-with-resources: auto-closes anything implementing `AutoCloseable`, closes in reverse order, suppressed exceptions if both try and close throw.
- Streams: intermediate (map, filter, lazy) vs terminal (collect, forEach, eager) operations; streams are single-use.
- Optional: use to signal "may be absent" in return types, not for every nullable field — avoid `Optional.get()` without checking.
- **Memory hook:** *"Streams are lazy until a terminal operation pulls the trigger."*

### Day 18 — Spring/Java integration questions (since your real experience is Spring Boot) 🟢
- Dependency Injection & IoC — why it matters (testability, loose coupling) — be ready to connect this to Java fundamentals (interfaces + polymorphism in practice).
- Bean scopes: singleton (default) vs prototype — and why singleton beans + mutable state = concurrency bugs (ties directly back to Week 2!).
- Be ready for: "how does Spring achieve thread safety for singleton beans handling concurrent requests?" → stateless bean design, thread-confined data via method-local variables, not instance fields.

### Day 19 — System design lite + behavioral bridge
- Rehearse 2-3 STAR stories using your resume metrics: the 60% DB call reduction (caching + concurrency), the 800ms→200ms latency story, the Composite API unification.
- For each, be ready to go one level deeper into the Java mechanics an interviewer might probe (e.g., "how did you decide on cache eviction policy" → ties back to Day 4/HashMap and Redis internals).

### Day 20 — Full Mock Interview Day
- Simulate a 60-min technical round: 2 core Java questions, 1 concurrency scenario, 1 "walk me through a project" deep dive.
- Time yourself. No notes during the mock — notes only in review after.

### Day 21 — Final Review
- Skim all `mistakes.md` entries from the 3 weeks.
- Re-say each "Memory hook" out loud from memory, no peeking — if any fail, that's your last-minute focus area.

---

## Quick-Reference Cheat Sheet (night-before scan)

| Topic | One-liner |
|---|---|
| Interface vs Abstract | What you can do vs what you are |
| equals/hashCode | hashCode → bucket, equals → confirms |
| ArrayList vs LinkedList | Fast read vs fast insert-at-ends |
| HashMap internals | Buckets → list → tree (>8 entries) |
| ConcurrentHashMap | Fine-grained locking, no global lock |
| PECS | Producer Extends, Consumer Super |
| start() vs run() | start spawns, run just runs here |
| volatile vs synchronized | Visibility only vs visibility + atomicity |
| Thread pool sizing | CPU-bound: few. I/O-bound: many |
| Young Gen vs Old Gen | Nursery vs retirement home |
| Streams | Lazy until terminal op |

---

*Suggested repo location: alongside your DSA and HLD notes, e.g. `interview-prep/core-java/`. Commit `mistakes.md` daily — same system that's worked for your DSA practice.*
