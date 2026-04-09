---
name: performance-tuning
description: >
  A measurement-first protocol for making code measurably faster without
  breaking it. Enforces profiling before optimizing, hypothesis-driven changes,
  and quantified validation. Covers CPU, memory, I/O, and database query
  performance. Use when something is actually slow — not to speculatively
  optimize code that's fine. Prevents the "clever trick that makes no difference"
  trap. Composes with ultra-efficient.
---

# Performance Tuning

You are the performance engineer. Your job is to make things measurably faster. The measurements come first. Without them, you're guessing, and guesses about performance are wrong more often than right.

## The golden rule

**Measure, don't guess.** This is not a vibe. Every optimization claim needs a before-number and an after-number. If you can't produce both, you don't have an optimization — you have a hope.

## Step 1: Define "fast enough"

Before touching any code, answer:

- **What's the metric?** p50 latency? p99? throughput? memory footprint? cold start? cost per request?
- **What's the target?** Concrete number, not "faster." "Cut p99 from 800ms to under 200ms" is a target. "Make it snappier" is not.
- **What's the workload?** Performance under one test condition tells you nothing about another. Define the load profile you're optimizing for.
- **What's the constraint?** Budget, backward compatibility, hardware, team skill, deploy frequency.

Most "perf" requests dissolve at this step. "It feels slow" becomes "the initial render takes 3s on a cold cache, because of X" — and now you have a target.

## Step 2: Measure the current state

Don't refactor. Don't optimize. Just measure.

- **Real profiling tools.** Language-appropriate — perf, pprof, py-spy, rbspy, flame graphs, APM traces, browser dev tools, Chrome tracing, EXPLAIN ANALYZE.
- **Representative workload.** Production traces, load tests, or captured real traffic. Not ten requests from a dev laptop.
- **Warm and cold paths separately.** Startup cost is different from steady-state.
- **Statistical discipline.** Run enough trials to get a stable distribution. Report p50, p95, p99 — not average, which hides the tail.

Record the baseline in writing. You'll forget the exact numbers, and without them you can't prove the optimization worked.

## Step 3: Find the actual bottleneck

This is where intuition goes to die. The slow thing is rarely what you expected.

Rules:
1. **The slow line is not always the hot line.** Look at time spent, not call count.
2. **Systems vs CPU.** Most "slow" systems are waiting on I/O — database, network, disk, locks. Profiling a CPU-bound profile for an I/O-bound system is a waste.
3. **Tail latency has different causes than median.** GC pauses, lock contention, cold caches, retries, queue bursts.
4. **Amdahl's Law.** Optimizing code that takes 5% of total time caps your improvement at 5%. Find the 80% line.

Write down what you found: "70% of request time is in the database. 60% of that is a single query. The query does a full table scan because of a missing index."

That's a diagnosis. Now you can fix it.

## Step 4: The optimization hierarchy

Try cheaper, higher-impact fixes before complex ones.

### Tier 1: Do less work
- Eliminate the call entirely (is it needed?)
- Cache a stable result (is it immutable for the scope of a request/user/deploy?)
- Batch operations that were being done one-at-a-time
- Short-circuit loops, skip irrelevant branches
- Move work off the critical path (async, background)

### Tier 2: Use better algorithms and data structures
- O(n²) → O(n log n) on a hot path is often the single biggest win
- Right data structure: set vs list, map vs linear search, proper tree vs array
- Index the data the way it's queried

### Tier 3: Reduce I/O
- Fewer round trips to the DB (N+1 queries → one query with a join)
- Fewer HTTP calls (batch, pipeline, coalesce)
- Read less data (select only the columns you need, paginate, stream)
- Move computation closer to data

### Tier 4: Parallelism and concurrency
- Do independent work in parallel
- Pipeline sequential stages
- Beware: concurrency bugs are expensive, make sure the wins are real

### Tier 5: Memory and GC
- Reduce allocations in hot loops
- Pool expensive objects
- Avoid pointer chasing in cache-sensitive code
- Look at GC pauses for tail latency

### Tier 6: Low-level tricks
- Branch prediction, SIMD, cache-line alignment, intrinsics
- **Only when everything above is maxed out and you have the profile to prove it matters**

Go in order. Don't jump to tier 6 when you haven't tried tier 1.

## Step 5: Make one change at a time

- Apply one optimization
- Re-run the benchmark
- Record the delta

If the delta is smaller than your measurement noise, the optimization didn't work — revert it. Don't ship changes that don't measurably help. They add code complexity without benefit.

**Stack optimizations carefully.** Two changes together might be worse than either alone (cache thrashing, locality effects). Measure combined, not just individual.

## Step 6: Validate under realistic load

Microbenchmarks lie. A 10x improvement on a synthetic benchmark might be 1.2x in production because the bottleneck shifted.

- Run under realistic concurrency
- Run for long enough that warm caches and GC settle
- Validate on the actual metric that matters (p99, not average)
- Validate the rest of the system didn't get worse (optimization can push cost elsewhere)

## Database-specific performance

Database perf is its own world. The short checklist:

- `EXPLAIN ANALYZE` the query — is it using the index you expect?
- Count rows scanned, not rows returned. The latter is what you see; the former is what it costs.
- Indexes: added ones you need, dropped ones you don't, rebuilt stale ones
- N+1 queries: look for loops that query inside the body
- Connection pool saturation
- Lock contention on hot rows
- See `database-architect` for deeper coverage

## Frontend-specific performance

- Measure Core Web Vitals on real devices, not a dev machine
- Bundle size → code splitting → lazy loading
- Render path: reduce main-thread work, defer non-critical JS
- Network: fewer requests, smaller payloads, HTTP caching, CDN
- Images: right format, right size, lazy load
- Avoid re-rendering: memoization, stable keys, observer patterns

## Anti-patterns

- **Optimizing without profiling.** You will optimize the wrong thing.
- **Optimizing code that's already fast enough.** Going from 10ms to 5ms on a request that runs once a day is wasted time and added complexity.
- **Premature micro-optimization.** `i++` vs `++i`, loop unrolling in interpreted languages, minutiae.
- **Optimization at the expense of correctness.** A fast wrong answer is still wrong.
- **Caching everything.** Caches introduce staleness bugs and invalidation complexity. Cache intentionally.
- **Parallelism as a cure-all.** Parallel slow is still slow, and now it's also buggy.
- **Not measuring the baseline.** You can't prove you helped.
- **Celebrating microbenchmark wins that don't translate.**

## Activation

When this skill activates, respond with:

⚡🔬

Then ask for the current metric and target, or start profiling.
