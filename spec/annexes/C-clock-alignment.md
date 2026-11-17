# Annex C — Clock alignment (informative)

A chain that crosses cores, partitions or chips is timestamped in more than
one clock domain. Spec §2.1 requires the mapping between domains and its
error to be measured and reported.

## Cases
1. **One counter, one domain.** Same counter read everywhere. Check that no
   layer applies an offset (some hypervisors give each guest a virtual
   counter offset).
2. **Shared hardware counter, different software clocks.** Read the hardware
   counter and the operating system's raw monotonic clock back to back, many
   times, and fit a line. Run tracers on a clock that is not adjusted by time
   synchronisation.
3. **No shared counter** (separate cores without a common counter, or separate
   chips). Exchange timestamped messages in both directions:
   offset ≈ ((t2 − t1) + (t3 − t4)) / 2, with error at most half the round
   trip. Keep the samples with the shortest round trip; fit offset and drift
   over time. Alternatives: a shared hardware event (e.g. a GPIO edge seen by
   both sides) or a network time protocol with hardware timestamping.

## Checks
- State the alignment error with every cross-domain interval.
- Check that no cause-and-effect pair appears in the wrong order after
  alignment.
- For long runs, re-fit drift periodically.

Whether two particular cores share a counter is platform-specific and must be
confirmed from the platform documentation and by measurement.
