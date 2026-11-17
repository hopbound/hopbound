# Annex D — Deterministic data flow (informative)

Some delays in a chain are not caused by contention but by **when data is
read and published**. Making data flow deterministic is a control option for
the *data wait* part of a hop (spec §4.3).

- **Fixed communication windows** (logical execution time, originating in the
  time-triggered languages of the early 2000s and later adopted in automotive
  software architectures): a task reads its inputs at the start of a fixed
  window and publishes its outputs at the end. Scheduling inside the window is
  ordinary preemptive scheduling, so the pattern can be implemented on most
  real-time operating systems.
  - Gives: data flow that does not depend on execution jitter or preemption
    order.
  - Does not give: deterministic execution. Every task must still finish
    inside its window, so worst-case timing evidence is still needed.
  - Costs: buffers, copies, and added latency (outputs wait for the window
    end).
- **Other routes:** time-triggered or cyclic schedules, partition schedules,
  synchronous languages, reactor-based coordination languages.
- **Observation for HopBound:** event-driven publish-subscribe middleware
  usually gives no ordering guarantee by default. Where a chain mixes fixed
  windows and event-triggered hops, the composition pattern (spec §1.8)
  differs per hop and must be identified, not assumed.
