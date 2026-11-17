# HopBound Specification — v0.2 outline

**Status:** Outline. Nothing here is normative yet; the core (§0–§8) becomes
normative only after a campaign has exercised it. The annexes are informative
and evolve with tools and platforms. Revisions follow evidence, not opinion.
**Author:** Lei Zhou · **Licence:** CC BY 4.0

Every number in a future version will be marked either *binding* (part of
conformance) or *default* (may be changed with a declaration). An unmarked
number is a drafting error. Statements not yet exercised by a campaign are
marked *hypothesis* or *candidate*.

---

## 0. Concept

### 0.1 The situation HopBound is for

Learned perception and planning now run on the same platforms as
deterministic control. Their execution time varies with input and load; the
control that depends on them has a deadline. Between the two sit many
boundaries — device to driver, driver to accelerator, accelerator to CPU,
partition to partition, chip to chip — and at each one, work can wait for a
resource someone else is using.

A deadline in such a system belongs to a **chain**. The chain is met only if
every hop keeps its share of the budget under the load the system really
carries. Averages hide the failures that matter: a chain can be fast almost
always and late exactly when the platform is busiest.

### 0.2 Who the evidence is for

The same evidence serves two kinds of user:

- **Engineers optimising a deployment** — they need the chain to meet its
  deadline at the lowest cost in hardware, isolation overhead and lost
  throughput for other workloads.
- **Teams building an assurance argument** — whatever protects a deadline
  (isolation, a deadline monitor, a fallback path) rests on timing
  assumptions: that the protection is independent of what it protects, that
  it detects and reacts in time, that a fallback can actually run when it is
  needed. On a shared platform those assumptions must be tested **while the
  platform is under load**, because that is when they are needed.

HopBound produces the evidence; it does not make the argument (§7.3).

### 0.3 What HopBound asks of each hop

1. **What does it take, and what does it promise?** — its needs (CPU time,
   memory bandwidth, accelerator time, link bandwidth, buffers) and its
   timing promise (bound, period, jitter) under declared conditions.
2. **What breaks the promise?** — the interference that pushes it late, and
   where that interference comes from.
3. **What keeps the promise?** — the control that bounds it, shown to work.

The answers form a **timing contract** per hop. Contracts compose into a
chain budget; a chain whose hop contracts do not compose has no defensible
deadline.

### 0.4 Two axes and one property per hop

**Axis 1 — the loop.** The method, the same everywhere:

```
Profile ──► Attribute ──► Control ──► Verify
   ▲                                     │
   └──────────── next iteration ─────────┘
```

**Axis 2 — the deployment model.** What the method is applied to:

| Deployment model | Example |
|---|---|
| D1 Bare-metal RTOS | Control loop on a microcontroller or a dedicated core |
| D2 Virtualised | Real-time guest and rich-OS guest under a hypervisor on one SoC |
| D3 On-chip AMP | Application cores and real-time cores (e.g. Cortex-A with Cortex-R/M) sharing memory, interconnect and peripherals |
| D4 Off-chip AMP | A discrete processor, accelerator or safety island connected over PCIe or another high-speed link |

The loop does not change across deployment models. What changes is which
interference channels exist, what can be observed, how clocks relate, and
which controls are available (§6).

**Visibility — a property of each hop.** How much of a hop can be observed:
its own execution, the platform around it, or only its boundaries (§1.6).
Visibility selects the attribution method (§3). One chain usually mixes
visibility levels.

---

## 1. Model

### 1.1 Path (chain)
The sequence of processing steps from a stimulus to its effect — for example
from a camera frame becoming available to a control element acting on the
result. A path has a declared deadline.

### 1.2 Hop
One step of a path that runs in one execution context: a device transfer, a
CPU stage, an accelerator job, a hand-off between partitions, cores or chips,
a consumer applying the result.

### 1.3 Boundary
The point where one hop hands work or data to the next. Each boundary has a
timestamp point, a clock domain and a transport (shared memory, message, DMA,
link).

### 1.4 Victim and aggressor
The victim is the path whose timing is being protected. An aggressor is any
other workload that shares a resource with it.

### 1.5 Contract
For each hop: needs and promise under declared conditions (§0.3), with the
sensitivity of the promise to each shared resource (§3.4). A path's budget is
met only if its hops' contracts compose (§1.8).

### 1.6 Visibility
The level of observation available into a hop. Declared for every hop, per
platform; it may differ between platforms and may improve over time.

| Visibility | What can be observed | Also known as |
|---|---|---|
| **Inside** | The component's own execution: source or symbols, probes, traces | White box |
| **Platform** | The resources around it — scheduler, memory system, interconnect, device events — and its boundaries | Grey box |
| **Boundary** | Only when work enters and leaves the hop | Black box |

### 1.7 Hop anatomy
A hop's time divides into five parts. Each points to a different class of
control (§4.3).

| Part | Meaning |
|---|---|
| **Data wait** | Ready to start, but its input is not yet there (phase, sampling, release time) |
| **Ready, not running** | Input present, waiting for a CPU, vCPU or execution slot |
| **Running, slowed** | Executing, but slower than in isolation because shared resources are contended |
| **Device wait** | Waiting on an accelerator, DMA engine or I/O device — queued or executing |
| **Transfer** | Moving data or a result to the next hop |

Not every part is observable at every visibility level; parts that cannot be
separated are reported together and the reason stated.

### 1.8 Composition *(hypothesis)*
How a delay in one hop reaches the end of the chain depends on how hops are
triggered and buffered. Candidate patterns, to be confirmed by campaigns:

| Pattern | Typical cause | Effect downstream |
|---|---|---|
| **Additive** | Event-triggered hops | Delay passes through unchanged |
| **Quantised** | Periodic or fixed-window hops | Small delays absorbed until a sample point is missed; then the result is one period older |
| **Amplifying** | Queues, drops, back-pressure | Delay grows as work piles up |
| **Absorbing** | Slack in the schedule | Delay disappears until the slack runs out |

A pattern is identified by injecting a controlled delay into one hop and
observing downstream.

---

## 2. Profile

### 2.1 Measurement
- **Endpoints first.** Timestamp points for every boundary are defined before
  any configuration is measured, at the same semantic point in every
  configuration compared.
- **One timeline.** Every item (frame, request) carries a correlation
  identifier across hops. Every interval names the clock it was read from.
- **Clock alignment with a stated error.** When boundaries are timestamped in
  different clock domains, the mapping between them and its error bound are
  measured and reported. An interval whose alignment error is not small
  compared with the effect being studied is not reported as a result
  (methods: Annex C).
- **Ledger.** Per-hop times are measured per item and must add up to the
  measured end-to-end time; the residual is reported.
- **Input control.** Inputs whose content changes processing time are
  declared: a fixed replayed set for scored runs, live input only as a
  separately labelled run.
- **Instrument cost.** Scored runs carry no extra instrumentation;
  attribution runs report their own overhead.
- **Statistics follow the claim.** A worst-case claim, an overhead claim and
  an improvement claim each use their own statistic and sample unit, stated
  in advance.
- **Reproducibility.** A configuration is re-run from clean boots; the
  tolerance is declared.

### 2.2 Load
Aggressors are declared, calibrated and mapped to the real interference they
represent: CPU, cache and memory pressure; device and DMA traffic; a
co-tenant on the same accelerator; activity in another partition or on
another chip; link traffic. A real workload is paired with a calibrated
synthetic equivalent where possible, so results can be compared across
platforms.

---

## 3. Attribute

Two questions per hop: **where** did the time go, and **why**. Timestamps
measure where; mechanisms and controlled interference explain why.

### 3.1 Declare visibility first
The visibility of each hop (§1.6) is declared before attribution, because it
decides the method and what a result can claim.

### 3.2 Inside visibility
Trace the late execution path directly — passively (tracepoints, sampling,
scheduler events) or intrusively (probes, added instrumentation), with the
instrument's cost reported. Tracing explains the **mechanism inside**. It
does not by itself show the **external cause**: an event present in late
outliers is a cause only if it is absent, or much rarer, when the suspected
aggressor is removed (§3.3).

### 3.3 Differential comparison — all visibility levels
The same path is measured with and without a given aggressor or component,
everything else fixed. A suspected cause must explain the difference and must
not appear equally when the aggressor is absent. For **boundary**
visibility this is the primary method; for **platform** visibility it
settles cause where counters only show correlation; for **inside**
visibility it confirms the external cause behind a traced mechanism.

### 3.4 Graded interference — all visibility levels
An aggressor is raised in steps and the hop's latency is measured at each
step. The result is a **sensitivity curve** — how much of a shared resource a
neighbour can take before the hop breaks its promise. Sensitivity curves feed
the contract (§1.5) and size the controls (§4).

### 3.5 Platform observation corroborates
Counters and platform traces (cores, caches, memory controller, interconnect,
IOMMU, links, accelerators where exposed) support a cause; on their own they
show correlation, not cause. Counters count events, not time (Annex A).

### 3.6 Where the cause lies
A cause is attributed to the layer where the time accrues — a software path
(kernel, hypervisor, middleware, runtime, driver) or a hardware resource
(core, cache, memory, interconnect, IOMMU, link, accelerator) — and to both
when one triggers the other.

### 3.7 Coverage
Each platform states what its instruments can and cannot observe, layer by
layer (Annex B). What cannot be observed is attributed differentially or
declared unattributed — never guessed.

---

## 4. Control

### 4.1 Verdicts
Every attributed class of late outliers ends in exactly one verdict, with the
evidence and the measure taken.

| Verdict | Meaning |
|---|---|
| `configure` | A configuration change removes it: placement, priority, interrupt routing, queue or accelerator priority, phase alignment |
| `isolate` | A named spatial or temporal isolation mechanism bounds it — spatial: partitions, IOMMU/SMMU domains, firewalls; temporal: bandwidth regulation, cache partitioning, time partitioning |
| `limit` | Nothing available removes it; the design must work within the measured bound — by budgeting for it, moving the critical work elsewhere, or containing its consequences (§4.2) |

A measurement without a verdict, or without a stated reason for having none,
is not a HopBound result.

### 4.2 Prevention and containment
- **Prevention** — `configure` and `isolate` reduce the interference itself.
- **Containment** — deadline monitors, output checks and fallback paths do
  not reduce interference; they bound its **consequences** when a limit
  remains. Containment is part of a `limit` verdict and has timing
  assumptions of its own, verified under load (§5.1).

### 4.3 The control follows the cause
The choice of control follows the attributed part of the hop, not habit:

| Hop part (§1.7) | Typical control classes |
|---|---|
| Data wait | Phase alignment, fixed communication windows (Annex D), event triggering |
| Ready, not running | Priorities, core or vCPU pinning, chain-aware scheduling, partitioning |
| Running, slowed | Memory-bandwidth regulation, cache partitioning, memory placement |
| Device wait | Accelerator priority or time-slicing, shorter jobs, accelerator partitioning |
| Transfer | Zero-copy or shared-memory transport, link quality-of-service, time-sensitive networking |

A scheduler change cannot fix a memory-bandwidth cause, and a bandwidth
regulator cannot fix a non-preemptible software path.

---

## 5. Verify

### 5.1 Rules
- **Decision rule before the data.** What counts as "fixed" (and as "no
  change") is declared before the verification runs.
- **Same load, then harder load.** The control is measured under the load
  that exposed the problem, then under a targeted stress aimed at the
  controlled resource.
- **Look at the whole chain.** A control can move interference to another
  hop; every hop's contract is re-checked, not only the one that was fixed.
- **Containment is verified under load.** A monitor or fallback is measured
  under the same interference as the path it protects: it must detect and
  react within its declared budget, and a fallback must be able to run,
  while the platform is busy (Annex E).
- **Report the cost.** Throughput lost by other workloads, and overhead added
  by the control, are reported beside the gain.

### 5.2 Outcome
Each control ends **verified**, **rejected**, or **no verdict** with the
reason (for example, the platform could not observe enough to decide).

---

## 6. What changes by deployment model

Initial view, to be revised by campaigns.

| | D1 Bare-metal RTOS | D2 Virtualised | D3 On-chip AMP | D4 Off-chip AMP |
|---|---|---|---|---|
| Typical interference channels | Software paths, cache, memory | Plus VM exits, address-translation stages, device emulation and backends | Shared memory, interconnect, coherence traffic, shared peripherals | Link bandwidth and latency, DMA, IOMMU; no shared cache |
| Typical visibility | Inside (own code) | Inside for guests; platform or boundary for the hypervisor | Varies by core type; real-time side often inside, accelerators often boundary | Mostly boundary for the remote side |
| Observable layers | Cycle counters, RTOS trace | Plus hypervisor traces and exit accounting | Per-core-type counters; interconnect monitors where present | Boundary timestamps, link and IOMMU counters where exposed |
| Clock relationship | One domain | One domain; guest time can be distorted by the hypervisor | Often two domains | Separate domains |
| Clock alignment | Not needed | Check per-guest counter offsets | Shared counter if the SoC provides one; otherwise message exchange | Message exchange or a shared hardware event |
| Typical controls | Priority, affinity, interrupt routing | Partitioning, vCPU pinning, backend placement | Bandwidth regulation, cache partitioning, firewalls, memory placement | Link quality-of-service, IOMMU domains, admission control, monitors and fallback |

---

## 7. Outputs

### 7.1 Timing contract
Per hop and for the path: needs, promise and sensitivity curves; floor and
tail separately; the conditions under which they hold; the visibility on
which each claim rests.

### 7.2 Evidence record
Configuration and build identity captured automatically; input set; loads;
statistics; clock alignment and its error; attributions; verdicts;
verification outcomes.

### 7.3 Limits statement
What was not observed, what was not tested, the residual — and the scope of
the claim. HopBound results are characterisation data. They can be inputs to
a safety or assurance argument made and assessed by others; they are not
qualified safety evidence under any standard.

---

## 8. Open questions

1. Do the composition patterns (§1.8) hold, and are they enough to compose
   hop contracts into a chain budget when hops overlap in time?
2. What is the minimum evidence record format, and should it reuse an
   existing trace format?
3. How should accelerator time be represented when only boundary visibility
   is available and the start of execution cannot be observed?
4. What alignment error is acceptable for a cross-domain interval, relative
   to the effect being studied?

---

## Annexes (informative)

| Annex | Topic |
|---|---|
| A | [Counters and time](annexes/A-counters-and-time.md) |
| B | [Observability by layer](annexes/B-observability-by-layer.md) |
| C | [Clock alignment](annexes/C-clock-alignment.md) |
| D | [Deterministic data flow](annexes/D-deterministic-data-flow.md) |
| E | [Containment of low-visibility components](annexes/E-containment.md) |
