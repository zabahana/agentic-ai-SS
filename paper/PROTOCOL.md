# Research protocol — draft before experiments

Working title: **Layered Defenses for Tool-Using Security Agents on NVIDIA Infrastructure: Utility, Containment, and Performance**.

Status: proposed methodology only; no results have been collected. Novelty is not established. Begin with related work on agent prompt injection, least-privilege execution, RAG security, tool authorization, and confidential GPU inference before claiming a research contribution.

## Questions

1. Do model/content guardrails plus an external authorization broker and runtime isolation reduce successful attacks while retaining legitimate SOC task utility?
2. Which defense layer stops which failure modes, and what happens when individual layers are removed?
3. How do retrieval reranking and domain adaptation affect accuracy and safety?
4. What are the measured latency/throughput costs of defenses, optimized serving, and separately, confidential execution?

## Study design

First establish a rules/search baseline and a single-agent RAG baseline. They measure architectural usefulness; do not use different agent architectures to infer a defense layer's causal effect.

For the primary security comparison, keep the same multi-agent workflow, model revision, tasks, retrieval snapshots, tool schemas, and generation settings across:

- A: no application guardrails/broker restrictions, within an outer test harness that still confines effects to synthetic assets and test sinks.
- B: model/content guardrails only.
- C: B plus deterministic tool authorization and bound approvals.
- D: C plus runtime isolation and network/filesystem enforcement.

Run paired tasks and attack variants across configurations. Add selected leave-one-layer-out ablations to reveal interactions. No test enables live destructive actions or sends actual secrets to uncontrolled endpoints. The external harness is shared by all arms and is documented so it does not masquerade as the evaluated defense.

Run content safety, privacy extraction/redaction, and permission enforcement as separate outcomes. For GPU confidential computing, compare matched supported hardware/workloads with and without CC as an independent deployment study; do not attribute prompt-injection resistance to CC.

## Data and sampling

Pilot with approximately 20 reviewed cases to validate labels and scorers. Target a final set of at least 100 legitimate tasks and 300 attack-task pairs spanning at least six attack families, subject to feasibility and a pilot-informed power/precision analysis. Freeze counts and gates before examining held-out results. Report repetitions and correlated variants; do not inflate independent sample size by counting retries or variants of one case as unrelated examples.

Use incident/capture/family splits with near-duplicate checks. Keep training, development, and final test sets separate. Preserve original public benchmark definitions; report SOC adaptations as a new task set. Record all exclusions and missing/failed runs.

## Scoring and metrics

- Task success: rubric-scored correct conclusion, required evidence, and permissible action; report completed/attempted tasks.
- Evidence grounding: citation precision, supported-claim rate, and retrieval recall against reviewed references.
- Query utility: executable valid query rate and semantic result correctness, plus forbidden-query denials.
- Attack success: observed attacker objective reached at the output, memory, tool, filesystem, or controlled network sink; attempted requests alone are not successful side effects. Report both attempts and realized harm per category.
- Privacy: synthetic canary disclosure, PII entity precision/recall, redaction misses, and over-redaction.
- False blocking: legitimate actions blocked/legitimate attempted actions, with task-level utility reported separately.
- Performance: p50/p95 case latency, time to first token, throughput, GPU memory/utilization, and cost per successful case at stated concurrency. Separate queueing and inference where possible.
- Assurance: attestation acceptance/rejection, replay detection, and key-release correctness for the confidential profile.

Prefer deterministic checks for authorization, tool effects, and exact canaries. Use blinded human review for ambiguous investigation quality, with a reviewed subset scored by a second reviewer and disagreements resolved explicitly. If model judges supplement scoring, pin their version and prompt and calibrate against human labels.

Report per-family rates and aggregate rates with denominators; use paired bootstrap confidence intervals clustered by task/incident when variants are correlated. Never present zero observed failures as proof that failure probability is zero. Publish null and negative findings.

## Reproducibility and integrity

Record Git SHA, dataset hashes, task split, seed, prompt/policy hash, model revision, serving image/backend, adapter hash, hardware/driver, precision, temperature, token limits, timeouts, and concurrency. Retain sanitized result JSONL and scripts that regenerate figures. Store bulky private artifacts outside Git and link checksums/manifests.

The planned paper sections are abstract, introduction, related work, threat model, system design, datasets/methods, results, ablations, deployment/performance study, limitations, and conclusion. Write the results section only after running experiments. Public-data simulations and one-person review limit enterprise generalization; disclose them. Publication venue and novelty claims come after literature review and results.
