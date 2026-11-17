# Annex E — Containment of low-visibility components (informative)

When a hop has only boundary visibility and its interference cannot be
removed, a design may **contain** it instead of explaining it: fence the
component, watch its boundaries with something simple and independent, and
keep a simpler fallback ready. HopBound does not design or assess such
arguments; it tests the timing assumptions they rest on (spec §0.2, §5.1).

## What can be watched at a component's boundaries

| Boundary | Question | Examples of mechanisms |
|---|---|---|
| Time | Is it alive and on time? | Heartbeat, deadline monitor, output freshness, program-flow checkpoints, independent watchdog |
| Value | Is the output believable? | Range and rate limits, physical consistency, diverse cross-check, safety-envelope check |
| Input / context | Is it operating inside the conditions it was validated for? | Operating-domain and out-of-distribution monitoring |
| Resources | Is it harming others? | CPU, memory-bandwidth and accelerator budgets with enforcement; IOMMU faults; thermal |

## Timing assumptions HopBound can test

On a shared platform, a containment design assumes that:
1. the monitor and fallback are not slowed by the component they watch
   (shared memory bandwidth, caches, interconnect, sometimes the accelerator);
2. detection plus reaction fits the time available **while the component is
   misbehaving** and the platform is most loaded;
3. the fallback can actually be scheduled when needed — in particular if it
   shares an accelerator that does not preempt running work;
4. monitor thresholds sit above the component's normal latency tail under
   realistic load, so they do not trip on ordinary jitter;
5. resource enforcement caps the component's impact, and the throttled
   component still meets its own deadline.

Each is measurable with the loop in the specification. If monitor and
fallback run on physically separate hardware, interference becomes mainly an
availability question; consolidation onto shared platforms makes it a
timing-assurance question.
