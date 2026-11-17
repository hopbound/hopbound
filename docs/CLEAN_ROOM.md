# Clean-room record

HopBound is an independent project started by Lei Zhou in November 2026,
outside any employment.

## Rules this repository follows

1. **Written from first principles.** No source code, specification text,
   schema, figures or data are copied from any earlier project, including
   the author's earlier work carried out during previous employment.
2. **Prior work is cited, not reused.** Earlier public work may be referred
   to as prior art, with a citation.
3. **No employer or customer material.** No confidential information,
   customer names, customer data or details derived from customer
   engagements, anonymised or not.
4. **Own copyright from the first commit.** All content is
   Copyright 2026 Lei Zhou unless a file states otherwise, and is licensed
   as described in `LICENSE` and `LICENSE-docs`.
5. **Dated, signed-off history.** Every commit is made with
   `git commit -s`. The commit history, together with the design log below,
   records when and by whom each part was created.

## Design log

| Date | Entry |
|---|---|
| 2026-11-16 | Project started. Name chosen (HopBound). Licences chosen: Apache-2.0 for code, CC BY 4.0 for documentation, specifications and datasets. Initial README, principles, specification outline and Campaign 01 outline written. Verdict names fixed as `configure / isolate / limit` ("isolate" follows the industry convention of spatial and temporal isolation). |
| 2026-11-16 | README and specification restructured: concept and a concrete scenario first, then the method; two organising axes added — the loop (profile, attribute, control, verify) and four deployment models (bare-metal RTOS, virtualised, on-chip AMP, off-chip AMP). |
| 2026-11-16 | Specification v0.2: core (concept, model, loop, deployment models, outputs) separated from informative annexes A–E (counters and time, observability by layer, clock alignment, deterministic data flow, containment). Added: per-hop visibility (inside, platform, boundary), hop anatomy, composition hypothesis, graded-interference sensitivity, prevention vs containment, verification of containment under load. Annex content written by the author from his own research. |
| 2026-11-16 | Pre-publication review: wording aligned between README and specification (attribution method by visibility); section on how HopBound relates to existing tools added; repository layout updated. |
| 2026-11-16 | Campaign 01 re-scoped to an end-to-end use case: a robot arm under a learned policy with an AI-driven safety stop and a control-side watchdog (file renamed to `campaign-01-arm-safety-stop.md`). Campaign 00 added: bring-up of the full pipeline and instruments in simulation, producing no measurements. Deployment model D0 (single OS, partitioned cores) added; workload placement declared separately from the deployment model. Drafted with AI assistance; reviewed and committed by the author. |
