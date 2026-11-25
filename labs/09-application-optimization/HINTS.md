# Lab 09 — Progressive hints

Hints are released one at a time, from broad prompts to more specific direction. Record which hint was needed and what it helped you notice. Level 2–4 hints are prepared when those exercises are activated; they are intentionally not included here.

## Level 1 — Guided implementation

### Hint 1 — Start with a hypothesis

Before changing configuration, name the user-visible symptom and the metric that would confirm it. Is the concern response time, compute cost, throttling, or freshness? A measurement that cannot change your decision is probably not useful.

### Hint 2 — Keep the experiment comparable

Write a run sheet before the first timing run. Hold the code revision, dataset, request sequence, client location, concurrency, and warm-up policy constant. Repeat runs so you can see whether the difference is larger than normal variation.

### Hint 3 — Separate the clocks

The caller's end-to-end time includes more than Lambda execution. Compare client-observed latency with Lambda duration and note where network/API overhead, cold starts, or throttles may contribute. Do not treat one CloudWatch average as a p95.

### Hint 4 — Cost and memory move together

Lambda's memory setting also affects available CPU. For each measured configuration, use the configured memory and billed duration to estimate compute cost at the observed invocation count. A shorter duration alone does not prove lower cost.

### Hint 5 — A warm cache is local state

An in-memory cache belongs to one Lambda execution environment. Repeated requests can land on different environments, and deployments/cold starts discard local entries. Treat hit rate and freshness as workload-dependent observations, not a shared-cache guarantee.

### Hint 6 — Make staleness visible

Use a synthetic record revision that you can change deliberately. Compare the returned revision with the stored revision after a cache fill, then observe the configured freshness bound. Explain whether this behavior meets the scenario's requirement.

### Hint 7 — Read a result table, not a highlight

Put the baseline and candidates side by side: request count, concurrency, p50/p95, Lambda duration, memory, errors/throttles, cache hit/miss, and estimated cost. State the conditions under which your conclusion would no longer hold.

## Later exercise levels

Level 2, Level 3, and Level 4 workload-specific hints are issued only when each level begins. This file does not disclose their scenario, diagnosis, or optimization solution.
