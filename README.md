# HopBound

**Per-hop timing evidence for systems where AI and real-time control share a platform.**

---

## The problem, in one scene

A delivery robot moves down a corridor at 1.5 m/s. Its camera produces a
frame every 33 ms. A neural network looks for people in each frame, and when
it finds one, the motion controller must start braking. The design says the
whole chain — frame to brake command — takes at most 100 ms, which means the
robot travels at most 15 cm before it reacts.

On the bench, the chain takes about 40 ms. The requirement is met with room
to spare.

Then the robot does real work. A mapping job runs at the same time and pulls
heavily on memory. A second model shares the accelerator. A log upload
streams over the network. Most frames still take about 40 ms. But one frame
in several thousand takes 180 ms — and in that frame the robot travels 27 cm
before it reacts.

*(These numbers are illustrative, chosen to show the shape of the problem.
HopBound has not measured them.)*

Nothing failed. Every component did what it was designed to do. The deadline
was missed by the **sum** of small delays, at boundaries no single team owns:
the camera's DMA waited behind the mapping job's memory traffic; the
detection job queued behind the other model on the accelerator; the result
waited for a scheduler on another core. The average said "fine". The tail
said otherwise, and the tail is the frame that matters.

This happens wherever learned, variable-time computation feeds deterministic
control on shared hardware: a robot arm, a vehicle's driver-monitoring chain,
an inspection line, a drone's obstacle avoidance.

## The theme: from chain of trust to chain of timing

Platform security learned long ago that trust has to be established link by
link — each stage verifies the next, and the chain is only as strong as its
weakest link. Timing needs the same discipline. A deadline belongs to a whole
chain of hops, and it is met only if every hop keeps a stated promise, under
the load the system will really carry.

HopBound's purpose is to make those promises explicit and to back each one
with evidence:

1. **Where does the time go?** A per-hop breakdown that adds up to the
   measured end-to-end time, with the unexplained remainder shown.
2. **Why does it go there under load?** Each late outlier traced to the
   shared resource or software path that caused it — even when a component is
   a black box.
3. **What will bound it, and did that work?** Each cause ends in a control
   decision, and the decision is verified by measuring again.

The result is a **timing contract** for each hop, and the evidence behind it.

## One method, five ways of deploying

The same loop applies however the system is built; what changes is which
interference channels exist, what can be observed, and which controls are
available.

```
                 Profile  ──►  Attribute  ──►  Control  ──►  Verify
                    │              │              │             │
 Single OS,         ●              ●              ●             ●
   partitioned cores
   (e.g. Linux with real-time preemption, critical path on isolated cores)
 Bare-metal RTOS    ●              ●              ●             ●
 Virtualised        ●              ●              ●             ●
 On-chip AMP        ●              ●              ●             ●
   (e.g. Cortex-A + Cortex-R/M sharing memory and interconnect)
 Off-chip AMP       ●              ●              ●             ●
   (discrete processor or safety island over a network, PCIe or another link)
```

Models combine — a hypervisor on the application cores with real-time cores
outside it is virtualised plus on-chip AMP. Where each AI workload runs (CPU,
on-chip accelerator, discrete accelerator) is declared separately from the
deployment model, and results with different placements are never pooled.

- **Profile:** measure a victim path against declared, realistic
  competing workloads ("aggressors"), hop by hop.
- **Attribute:** find what caused each late outlier — software path,
  hardware resource, or both. Traces show the mechanism where a component
  can be seen inside; comparing loaded and unloaded runs establishes the
  cause; counters corroborate.
- **Control:** choose the measure the cause calls for: **configure**
  (placement, priority, routing), **isolate** (spatial or temporal isolation
  mechanisms), or accept a **limit** and design around the measured bound.
- **Verify:** measure again under the same and harder loads; check that the
  fix holds, and that it did not move the problem to another hop or starve
  another workload.

One more property decides **how** each hop is attributed: its
**visibility** — whether you can observe its own execution (*inside*), only
the platform resources around it (*platform*), or only when work enters and
leaves it (*boundary*). One chain usually mixes all three: an open driver, a
middleware layer, a closed accelerator runtime. HopBound declares the
visibility of every hop and chooses the method to match — tracing where the
inside is visible, controlled comparison where it is not.

## Principles

- **Contracts at every boundary.** A deadline that exists only for the whole
  path cannot be enforced anywhere.
- **Evidence that does not depend on the system being judged.** A closed or
  unfamiliar component is characterised by its effect on its surroundings —
  with and without it — not by what it reports about itself.
- **A cause needs a control run.** Traces and counters show what happened
  during a late outlier; a cause is accepted only if it separates the loaded
  case from a matched unloaded case.
- **Close the ledger.** A breakdown that does not add up to the whole is
  hiding something; the residual is always reported.
- **Decide before the data.** Deadlines, thresholds and pass criteria are
  declared before a run, never fitted afterwards.
- **Every result ends in a verdict**, or in a stated reason why there is none.

---

## Status — read this before citing anything here

HopBound starts in November 2026. **It has no measurements yet.**

| Tier | What exists |
|---|---|
| Measured | Nothing yet |
| Written | Concept, principles, specification outline (`spec/`) |
| Planned | Campaign 00: the full pipeline brought up in simulation — no measurements. Campaign 01: a robot arm under a learned policy, with an AI-driven safety stop that must meet a deadline (`campaigns/`) |

Disagreement with the method is welcome now, before data exists to defend it.

## What HopBound produces — and what it does not

HopBound produces **characterisation data**: measured, attributed timing
behaviour of a declared configuration. That data can be an input to a safety
argument made and assessed by others. It is **not** qualified safety evidence
under ISO 26262, IEC 61508, ISO PAS 8800 or any other standard; the tools are
not qualified, and the repository makes no such claim.

## Repository layout

```
spec/        Concept, model, and the rules for each stage of the loop
spec/annexes Mechanisms: counters, observability, clock alignment, data flow, containment
campaigns/   Measurement campaign plans, one per platform and question
docs/        Background, provenance and clean-room record
```

## How HopBound relates to existing work

HopBound is meant to sit on top of tools communities already use, not to
replace them:

- **Tracing and chain-latency tools** — for example `ros2_tracing` and
  CARET for ROS 2, and trace viewers such as Perfetto and Trace Compass —
  show where time goes along a chain. HopBound uses such traces and adds
  controlled interference, attribution to shared resources, control verdicts
  and per-hop contracts.
- **Deterministic-execution approaches** — logical execution time,
  time-triggered scheduling, reactor-based coordination such as Lingua
  Franca — fix *when* data moves. HopBound treats them as controls to select
  and verify (spec Annex D).
- **Isolation mechanisms** — memory-bandwidth regulation, cache partitioning,
  Arm MPAM, hypervisor partitioning, IOMMU domains — are what HopBound's
  `isolate` verdicts choose between and then verify.
- **Multicore interference practice in avionics** (CAST-32A, AMC 20-193)
  established the idea of identifying interference channels per platform;
  HopBound applies the same idea, openly, to AI and real-time control on
  shared platforms.

## Relation to earlier work

The author's earlier, separate work on real-time benchmarking is prior art for
this project and is cited, not reused:

> Zhou, L. (2026). *Is Linux RT-PREEMPT Ready for Automotive Safety-Critical
> Workloads? A Systematic Benchmark Evaluation.* Embedded World Conference
> 2026. https://doi.org/10.5281/zenodo.20967588

HopBound is written from first principles in a new repository and contains no
code or text from earlier projects. See `docs/CLEAN_ROOM.md`.

The argument behind HopBound is set out in the position paper *From chain of
trust to chain of timing* (2026):
https://zlhk100.github.io/library/chain-of-timing.html

## Contributing

Most useful now: challenges to the method, nominations of representative
pipelines, and measurements from platforms the author does not have. See
`CONTRIBUTING.md`. All commits carry a `Signed-off-by` line (DCO).

## Licence

- Source code: Apache License 2.0 — `LICENSE`
- Documentation, specifications and datasets: CC BY 4.0 — `LICENSE-docs`

## Maintainer

Lei Zhou — https://zlhk100.github.io · ORCID 0009-0006-6999-0729
