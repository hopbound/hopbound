# Annex B — Observability by layer (informative)

A platform's coverage statement (spec §3.7) lists, layer by layer, which
instruments exist and what they can see. Tool names below are examples only.

| Layer | Typical instrument classes | Common gaps |
|---|---|---|
| Application and AI runtime | Framework and runtime profilers; vendor SDK profilers | Often unavailable or not usable in production builds; the usual source of boundary-only visibility |
| Middleware | Middleware tracing (e.g. LTTng-based tracing for ROS 2) | Callback and transport events only; no resource view |
| User space | Dynamic user probes, static user tracepoints | Probe cost per hit; needs symbols or offsets |
| OS kernel | Tracepoints, dynamic kernel probes, scheduler statistics, pressure metrics | Kernel configuration decides what is available |
| Hypervisor | Hypervisor trace and exit accounting, guest steal time | Often host-side only |
| CPU cores | Performance counters, statistical sampling, trace units | Per-core events; definitions vary by core type |
| Memory system | Uncore counters (interconnect, memory controller, IOMMU), resource monitoring (e.g. MPAM monitors where supported) | Kernel support varies and lags hardware |
| Accelerators | Driver scheduler events (queued vs running), per-process usage statistics, vendor profilers | Driver-dependent; start of execution often not observable |
| Real-time cores / RTOS | RTOS tracing (e.g. CTF-based), cycle counters | Separate clock domain; buffer extraction |
| Links | Link and DMA counters where exposed; boundary timestamps | Often only boundary timestamps |
| Joining layers | Common clock plus a trace viewer that merges sources | Alignment across domains (Annex C) |

Event sources and front-ends are different things: a dynamic probe is a
source; it can be consumed by several front-ends. A coverage statement names
both.
