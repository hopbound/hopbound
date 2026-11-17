# Annex A — Counters and time (informative)

Hardware counters count **events**, not durations. Using them to explain
latency requires knowing what kind of quantity each one is.

| Kind of measurement | What it gives | Relation to time |
|---|---|---|
| Event count | How often something happened | None on its own. Events overlap, so count × average latency is not the time lost. |
| Cycle- or slot-qualified count | Cycles (or issue slots) spent while a condition held | Time lost per category, e.g. top-down stall analysis on modern cores |
| Occupancy accumulator | Outstanding requests summed per cycle | Average latency, via Little's law |
| Sampled latency | Latency of individual sampled operations | A distribution, including the slow tail (statistical profiling extensions on some architectures) |
| Timestamps | Start and end of an activity | Real durations |
| Utilisation against peak | Share of theoretical maximum | Saturation only |

How counters are used in HopBound:
- **Timestamps measure; counters explain.** A counter supports a cause found by
  differential comparison (spec §3.3); it does not establish it alone.
- **Sensitivity fit.** Latency change against counter change under graded
  interference (spec §3.4) gives a usable relationship for one platform.
- **Pessimistic bound.** Count × worst-case per-event penalty gives a safe but
  pessimistic upper bound, valid only if the per-event worst case is itself
  bounded.
- **Budgets.** Counters can drive enforcement, for example memory-bandwidth
  regulation that throttles a core after a budget of accesses.

Caveats:
- Event definitions differ between core types and vendors.
- Multiplexing more events than hardware counters gives scaled estimates.
- Frequency scaling changes the conversion from cycles to time.
- Detailed accelerator profilers often replay work in several passes; they
  are laboratory tools and cannot be used live under interference.
- All of these are averages or samples, not worst cases.
