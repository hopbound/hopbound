# Campaign 01 — Robot arm with an AI safety stop

**Status:** Outline. No measurements.
**Author:** Lei Zhou · **Licence:** CC BY 4.0
**Depends on:** Campaign 00 (pipeline and instruments working in simulation)

## The use case

A low-cost open robot arm performs a pick-and-place task under a learned
policy, through standard robot middleware. A camera watches the work zone.
When a person enters the zone, the arm must stop within a declared deadline,
whatever the policy is doing at that moment.

Two chains share one platform:

| | Chain A — safety stop | Chain B — task policy |
|---|---|---|
| Role | Protected path (victim) | The main competing workload, and a second victim |
| Path | zone camera → person detection → stop decision → stop applied at the actuator bus | task cameras → learned policy → action chunk → controller update → actuator bus |
| Model | Small vision model (person detection) | Action-chunking policy first, then a vision-language-action policy |
| Promise | Stop applied within a declared deadline (default: 100 ms from the stimulus) | Controller runs at a declared rate with bounded jitter; each applied action's age is bounded |

The campaign asks whether learned computation on a shared platform delays a
safety reaction, what causes each late stop, which control bounds it — and
whether protecting chain A quietly starves chain B.

## Questions

1. Under declared competing workloads, does chain A meet its deadline across
   a sweep from idle to saturation?
2. For each class of late stop: is the fix a configuration change, an
   isolation mechanism, or a platform limit (`configure` / `isolate` /
   `limit`)?
3. What does each control cost chain B, in controller jitter and action age?
4. Does the control-side watchdog (below) detect and react in time while the
   platform is at its busiest?
5. How do the answers change when the control side moves between deployment
   models (D0, D2, D3, D4) with everything else fixed?

## Chain A — hops

| # | Hop | Boundary timestamp at its end | Expected visibility |
|---|---|---|---|
| A1 | Stimulus to frame available (exposure, readout, transfer) | frame buffer available to software | platform |
| A2 | Preprocessing | input tensor ready | inside on CPU; platform on GPU or ISP |
| A3 | Person detection | job completed | boundary or platform, depending on the accelerator driver |
| A4 | Stop decision | stop command issued | inside |
| A5 | Hand-off to the control side (thread, partition, core or chip, by deployment model) | stop received by the control side | inside or boundary, by deployment model |
| A6 | Stop applied | stop written to the actuator bus | inside |

## Chain B — hops

| # | Hop | Boundary timestamp at its end | Expected visibility |
|---|---|---|---|
| B1 | Frames available | frame buffers available to software | platform |
| B2 | Policy inference | action chunk ready | inside on CPU; boundary or platform on an accelerator |
| B3 | Action hand-off to the controller | action received by the controller | inside |
| B4 | Controller update | command written to the actuator bus | inside |

Chain B is reported as controller period jitter, missed controller periods,
and action age at application (time from frame capture to the command that
used it).

## Containment under test

The control side expects a "zone clear" heartbeat from the perception side
every declared period. If a heartbeat is later than its declared timeout, the
control side stops the arm on its own. The campaign measures, under every
load (spec Annex E):

1. whether the watchdog itself is slowed by the workloads it guards against;
2. whether detection plus stop fit the time available while the perception
   side is late *because* the platform is loaded;
3. whether the timeout sits above the heartbeat's normal tail, so it does not
   trip on ordinary jitter.

## Ground truth

- The stimulus is an image of a person switched on in the zone camera's view
  under control of a hardware line (a display or an LED panel). Its own
  switching delay is characterised first with a light sensor and subtracted.
- A logic analyser records the stimulus line and the actuator bus. That
  interval is the true stimulus-to-stop time, independent of any clock on the
  boards.
- The per-hop ledger of chain A must add up to the ground-truth interval; the
  residual is reported.
- **No person is placed within the arm's reach.** The arm has a physical
  power cut that the operator can reach at all times. The stop chain is a
  subject of measurement, not a safety function.

## Two axes for every run

**Deployment model of the control side** (spec §0.4):

| Model | Where the stop (A5–A6) and the watchdog run |
|---|---|
| D0 Single OS, partitioned cores | A real-time thread on an isolated core of the same operating system |
| D2 Virtualised | A real-time guest under a hypervisor |
| D3 On-chip AMP | A real-time or microcontroller core on the same chip |
| D4 Off-chip AMP | A separate microcontroller board over a network link |

**Workload placement** — where each model runs: CPU, on-chip accelerator,
discrete accelerator. Declared per run, never pooled.

Deployment models are compared on the same application chip, so that only
the control side's placement changes. Results from different chips are
reported per chip and not compared in absolute terms.

## Runs

| Run | Configuration | What it can show |
|---|---|---|
| R0 | Each chain alone | Floor and per-hop baseline |
| R1 | Both chains together, policy running the task | The realistic case |
| R2 | R1 + calibrated CPU, memory and I/O loads, swept in steps | Sensitivity curves for chain A's hops |
| R3 | R1 + a co-tenant on the accelerator | Accelerator and memory-path contention |
| R4 | Configured: placement, priorities, interrupt routing, accelerator priority | `configure` verdicts |
| R5 | Isolated: bandwidth regulation, cache partitioning, partitions, IOMMU domains, where available | `isolate` verdicts |
| R6 | Control side moved between deployment models, all else fixed | Deployment-model comparison |
| R7 | Perception deliberately delayed or stalled while loaded | Watchdog reaction under load |

## Criteria (defaults, declared per run)

1. Chain A applies the stop within its deadline in every trial of a scored
   run, across the declared load sweep.
2. Every class of late stop is attributed and ends in a verdict.
3. Chain B's controller jitter, missed periods and action age are reported
   for every run, so a fix that starves it is visible.
4. The watchdog stops the arm within its declared budget whenever the
   heartbeat is withheld, and never trips while the heartbeat is on time.
5. After a load burst ends, both chains return to their R1 behaviour within a
   declared time.
6. The cost of each control is reported.

## Platforms

- **First platform:** an application chip the author owns with an openly
  documented accelerator path, plus a separate microcontroller board, so D0
  and D4 (and D2 where a hypervisor is available) run on one application
  chip. Board choice and accelerator openness are assessed at bring-up and
  recorded with the dataset.
- **Second platform:** a chip with an on-chip real-time or microcontroller
  core, for D3.

Bring-up exit criteria: R0 runs end to end on hardware with every boundary
timestamp captured and its clock domain declared; the ledger closes against
the logic-analyser interval; R0 reproduces within the declared tolerance
across two clean boots.

## Deliverables

- Per-hop ledgers with residuals for both chains, R0–R7.
- Attributed classes of late stops, each with a verdict.
- Timing contracts for each hop of chain A and for the path.
- A deployment-model comparison on one application chip.
- The watchdog's measured reaction under load.
- An observability statement: what each platform's instruments could and
  could not see.
- A public dataset and the scripts to regenerate it.
- A short video of the arm, the stop, and the timing alongside.
