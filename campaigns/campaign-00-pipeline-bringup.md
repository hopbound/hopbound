# Campaign 00 — Pipeline bring-up in simulation

**Status:** Plan. **This campaign produces no measurements.**
**Author:** Lei Zhou · **Licence:** CC BY 4.0

## Purpose

Campaign 01 joins many software stacks into one system: robot middleware, a
learned policy, a perception model, a real-time control side and HopBound's
own instrumentation. Bringing them up for the first time on target hardware
mixes two kinds of problem — "it does not work yet" and "it is late" — and
makes both harder to solve.

Campaign 00 separates them. It brings up the complete Campaign 01 software
pipeline on a workstation, against a simulated arm and simulated cameras,
and builds HopBound's instrumentation there. Campaign 01 then starts on
hardware with a pipeline that already works and instruments that already
exist.

## The rule that keeps it honest

**Nothing timed in Campaign 00 is reported as a result.** Simulated time,
an emulated processor or a workstation CPU says nothing about the timing of
any target platform. Timestamps are captured here only to prove that the
instruments work and that the ledger closes.

## What runs where

| Piece | Runs as | Notes |
|---|---|---|
| Arm | A simulated model of the Campaign 01 arm, driven through the robot middleware's standard control interface | Same controller configuration as on hardware, with simulated hardware in place of the servo bus |
| Cameras | Simulated cameras in the same scene | Same image topics and formats as on hardware |
| Task policy | Learned policy (an action-chunking policy first, then a vision-language-action policy) running natively on the workstation | Trained on demonstrations recorded in simulation |
| Person detection | A small vision model watching the zone camera | Same node as on hardware; CPU inference |
| Control side | A real-time operating system running in its native simulator or under an emulator, with a micro-ROS client; the agent runs on the workstation | Exercises the off-chip shape (D4) of the link and the watchdog |
| Hypervisor layout (optional) | A type-1 hypervisor on an emulated Arm virtual machine, with a real-time guest and a rich guest | Learns the D2 domain layout; functional only |

## Stages

| # | Stage | Done when |
|---|---|---|
| S1 | Middleware and simulated arm | A scripted trajectory and teleoperation both move the simulated arm through the standard control interface |
| S2 | Task policy | Demonstrations recorded in simulation; the action-chunking policy completes the pick-and-place task closed-loop; then the vision-language-action policy does the same |
| S3 | Safety chain | A person appearing in the zone camera produces a stop decision, and the arm stops through the controller |
| S4 | Control side | The stop and a periodic "zone clear" heartbeat reach the control side over micro-ROS; a late heartbeat makes the control side stop the arm on its own |
| S5 | Instrumentation | Every hop of both chains records a boundary timestamp; each frame carries a correlation identifier through every hop; every interval names its clock; the per-hop ledger is computed and its residual reported |
| S6 | Evidence record (draft) | One run produces a complete evidence record in a draft format: configuration and build identity, inputs, loads, per-hop intervals, clock domains |
| S7 | First Arm step | The same stack, without the simulator, runs on a small Arm board the author already owns with a USB camera: the policy and the safety chain run, and the instruments record |
| S8 | Hypervisor layout (optional) | The real-time and rich guests boot, and a message crosses between them |

## Exit criteria (functional)

1. The simulated arm completes the task under the learned policy, repeatably.
2. The safety chain stops the arm every time a person appears, in a scripted
   sequence of at least 100 appearances.
3. The control-side watchdog stops the arm every time the heartbeat is
   withheld, and never while the heartbeat is on time.
4. Every hop of both chains is timestamped, every interval names its clock,
   and the ledger closes with a reported residual.
5. A full evidence record is produced by one command from a clean checkout.
6. Stage S7 runs the same instruments on Arm hardware.

## Deliverables

- A reproducible setup (container image or scripts) for the whole pipeline.
- HopBound instrumentation, version 0: boundary timestamps, correlation
  identifiers, clock naming, ledger computation.
- A draft evidence-record format (spec §8, open question 2).
- Bring-up notes, including what failed and why.
- A list of facts Campaign 01 must establish on hardware, for example the
  achievable servo-bus command rate and the camera's exposure and transfer
  time.

## Time box

Three weeks by default, run while Campaign 01 hardware is being procured.
An extension is declared with its reason.

## Not in scope

- Timing results of any kind.
- Choosing a platform on the basis of anything timed here.
