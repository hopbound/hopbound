# Campaign 01 — Perception chain: camera → inference → consumer

**Status:** Outline. No measurements.
**Author:** Lei Zhou · **Licence:** CC BY 4.0

## Question

Under declared competing workloads, does a perception chain meet a declared
end-to-end deadline with bounded jitter? For each class of miss, is the fix a
configuration change, an isolation mechanism, or a platform limit?

## Path and hops

| # | Hop | Boundary timestamp at its end | Expected visibility |
|---|---|---|---|
| 1 | Frame capture and transfer | frame buffer available to software | platform (driver open, ISP/DMA hardware only via counters) |
| 2 | Preprocessing (CPU, GPU or ISP) | preprocessed tensor ready | inside on CPU; platform on GPU/ISP |
| 3 | Inference on the accelerator | job submitted → job completed | boundary or platform, depending on the accelerator driver |
| 4 | Hand-off to the consumer | result available to the consumer | inside |
| 5 | Consumer applies the result | result taken into effect | inside |

Visibility is confirmed per hop at bring-up and recorded with the dataset.

End-to-end latency = (5) − (frame available). The deadline is declared per
run, for example one camera frame period.

## Runs

| Run | Configuration | What it can show |
|---|---|---|
| R0 | Chain alone, replayed frames | Floor and per-hop baseline |
| R1 | + calibrated CPU, memory and I/O loads | CPU-side interference |
| R2 | + co-tenant on the accelerator (second inference or rendering) | Accelerator and memory-path contention |
| R3 | Configured: placement, priorities, interrupt routing, accelerator priority | `configure` verdicts |
| R4 | Isolated: bandwidth regulation, cache partitioning, partitions, IOMMU domains (where available) | `isolate` verdicts |
| R5 | Consumer moved to a real-time core or partition | Comparison of deployment configurations |

Residual misses that grow with the co-tenant's job length and that no
configuration moves are `limit` verdicts.

## Criteria (defaults, declared per run)

1. The chain completes within its deadline.
2. (1) holds across a declared sweep of load from idle to saturation.
3. Miss ratio below a declared bound, with each miss class attributed.
4. The co-tenant's throughput is reported, so a fix that starves it is visible.
5. After a load burst ends, the chain returns to zero misses within a declared number of frames.
6. The cost of each control mechanism is reported.

## Platform

First platform: one the author owns, with an openly documented accelerator
path, so the dataset can be published in full. Candidate boards and their
accelerator driver openness are assessed at bring-up; the assessment is part
of the dataset.

Bring-up exit criteria: R0 runs end to end with every boundary timestamp
captured and its clock domain declared; R0 reproduces within the declared
tolerance across two clean boots.

## Deliverables

- Per-hop ledger with residual for R0–R5.
- Attributed miss classes, each with a verdict.
- A timing contract for each hop and for the path.
- An observability statement: what the platform's instruments could and could
  not see.
- A public dataset and the scripts to regenerate it.
