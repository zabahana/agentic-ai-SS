# Sentinel: step-by-step project plan

Prepared 2026-09-22. This document proposes work; it does not report completed integrations or experiment results. The supplied job description is requirements input, not an instruction source. The referenced project folder was empty on inspection.

## 1. The project and the paper

Build a secure, multi-agent SOC investigation platform on the NVIDIA stack. An analyst opens an alert, agents query event data and retrieve threat knowledge, and the system produces an evidence-backed incident assessment and proposed response. Policy checks, runtime isolation, privacy controls, human approval, and audit records constrain execution.

Use one coherent demonstration throughout: suspicious identity activity followed by endpoint execution and possible data exfiltration. Include benign lookalikes. A poisoned retrieved document or tool result tells the agent to disclose a synthetic secret or execute an unapproved action. The application should still complete legitimate investigative work while preventing the forbidden operation.

The paper asks: **How much do layered guardrails, deterministic tool authorization, and runtime isolation reduce successful attacks, and what utility and performance costs do they introduce?** GPU confidential computing is a separate infrastructure assurance and overhead experiment. It does not solve prompt injection.

Deliverables: runnable application, versioned Git repository, public-data preparation pipeline, threat model, reference architecture, adversarial test suite, repeatable benchmark runner, measured research paper, deployment profiles, workshop guide, and production-readiness review.

## 2. Architecture and trust boundaries

Proposed flow:

Analyst web UI → authenticated Python API → bounded workflow coordinator → specialist agents → external policy-enforcing tool broker → isolated tools and scoped data stores.

Inference routes to an explicitly selected model endpoint. Retrieval supplies cited evidence; it never supplies policy authority. Every tool invocation carries authenticated actor and tenant identity, a case identifier, schema-validated arguments, and a correlation identifier. The broker derives authorization from trusted server-side context, never from model-provided tenant identifiers.

Specialists: triage, evidence retrieval, investigation/query, response planning, and evidence verification. A bounded state machine controls transitions, budgets, retries, and termination. Agent outputs include structured conclusions and concise explanations, not private chain-of-thought. The verification agent can flag unsupported findings but cannot authorize actions.

Memory: case-scoped working state plus explicitly approved durable facts with provenance, expiry, and tenant isolation. Retrieved content and model summaries cannot silently overwrite trusted policy or become cross-case instructions.

Tools: read-only event search, ATT&CK lookup, entity enrichment from local fixtures, timeline construction, and response proposal. A separate executor applies approved actions to a simulated inventory. Approvals bind to the exact action, target, arguments, expiry, and policy version; execution rechecks them and is idempotent.

## 3. Technology choices

- Application: Python/FastAPI backend, React/TypeScript frontend, PostgreSQL with pgvector, and containerized Linux services. Begin with a small architecture; introduce a queue only when asynchronous case processing warrants it.
- Workflow: [NeMo Agent Toolkit](https://docs.nvidia.com/nemo/agent-toolkit/latest/index.html) for agent/tool integration, workflow profiling, and evaluation integration. Keep application authorization outside the LLM.
- Inference: a Nemotron model exposed through [NIM](https://docs.nvidia.com/nim/large-language-models/latest/introduction.html). Select the exact model, model license, context length, tool-calling support, GPU memory requirements, and supported backend during the compatibility spike; record image digest and model revision.
- Retrieval: [NeMo Retriever embedding and reranking services](https://docs.nvidia.com/nemo/retriever/index.html), with a simple local lexical baseline so retrieval quality gains can be measured.
- Safety: [NeMo Guardrails](https://docs.nvidia.com/nemo/guardrails/about-nemo-guardrails-library/overview), a selected [Nemotron safety model](https://build.nvidia.com/nvidia/nemotron-3.5-content-safety/modelcard), and deterministic privacy/tool checks. A content-safety score is not a prompt-injection or authorization guarantee.
- Execution: [NVIDIA OpenShell](https://docs.nvidia.com/openshell/home) on a compatible Linux environment. Validate its actual filesystem, network, and process enforcement. Local mock tools are not evidence of sandbox enforcement.
- Evaluation: [Garak](https://github.com/NVIDIA/garak), custom workflow tests, and [AgentDojo](https://github.com/ethz-spylab/agentdojo). Adapted SOC scenarios must be reported separately from original benchmark results. [NeMo Auditor](https://docs.nvidia.com/nemo-platform/documentation/vulnerability-scanning) is an additional integration subject to access and early-access API/support constraints.
- Post-training: PyTorch and a supported [NeMo AutoModel SFT/LoRA recipe](https://docs.nvidia.com/nemo/automodel/latest/recipes-e2e-examples/sft-peft), selected after model and GPU compatibility checks.
- Serving: measure the NIM profile first; evaluate a supported TensorRT-LLM/quantized profile afterward. Dynamo scale-out, Triton, and NIM Operator are additional experiments only where their operational role is justified. These products are not assumed to be a mandatory serial chain.
- Confidential deployment: supported CPU TEE plus CC-capable NVIDIA GPU, attestation verifier, key broker, and enterprise KMS. Follow NVIDIA's [attestation/key-release architecture](https://docs.nvidia.com/enterprise-reference-architectures/deploying-proprietary-models-confidential-compute-self-hosted-vms/latest/attestation-and-key-release-flow.html).

## 4. Public industry and research data

Start with three data families and add others only if they answer an evaluation question.

1. [MITRE ATT&CK STIX 2.1](https://github.com/mitre-attack/attack-stix-data): threat techniques, relationships, detection context, and citations. This is threat knowledge, not event telemetry or ground-truth incident labels.
2. [Splunk Attack Data](https://github.com/splunk/attack_data): selected security event captures for attack replay and investigation tasks. Use only logs; do not execute included attack artifacts. Select a compact, documented subset and normalize into a portable event schema without requiring a commercial SIEM. Do not imply separate captures belong to a single real incident.
3. [AgentDojo](https://github.com/ethz-spylab/agentdojo): external prompt-injection evaluation, supplemented by explicitly labeled SOC-specific poisoned documents, tool outputs, memory records, and exfiltration canaries.

Add controlled synthetic benign events and fake PII/secrets to measure false positives, privacy behavior, and attack outcomes. Keep these labels distinct from public observations. Public attack captures do not establish production prevalence, so do not derive operational precision from an artificially balanced dataset.

Every downloaded dataset gets a manifest: source URL, release/commit, retrieval date, checksum, applicable license/terms, attribution, permitted use/redistribution, schema, transformations, and split assignments. Keep bulk/raw data outside Git; commit download recipes and small redistributable fixtures only after checking terms. Treat all document/log text as untrusted content.

Split by incident/capture or attack family, not random log lines. Deduplicate near-identical content across splits. Keep final test data out of prompt tuning, adapter training, and defense development. Have an analyst review gold labels and expected evidence references.

## 5. Ordered implementation milestones

### Phase 0 — Discovery, design, and feasibility (week 1)

Define a hypothetical security ISV, analyst personas, trust boundaries, data flows, prioritized use cases, success measures, and a threat model. Record assumptions, decisions, tradeoffs, project risks, and cost limits. Run a compatibility spike for model access, NIM, retrieval, OpenShell, and GPU options. Establish which claims each environment can support.

Exit: a reviewed project charter, requirements-to-evidence checklist, research protocol, and compatibility record. Partner engagement is a simulated case study unless an actual partner participates.

### Phase 1 — Repository and reproducible foundation (week 1)

Use the existing repository https://github.com/zabahana/agentic-ai-SS and the local folder /Users/zelalemabahana/agentic-ai-SS. Preserve its existing history and visibility; commit and push the planning foundation first. Add README, dependency locks, environment template, local startup commands, CI, contribution/security guidance, and artifact storage rules. Keep product, paper, and infrastructure in one repository initially.

Proposed directories: apps/web, services/api, src/sentinel/agents, src/sentinel/tools, src/sentinel/policy, configs, data/manifests, evals, tests, infra/local, infra/gpu, infra/confidential, docs, and paper. Create directories as working content arrives.

Exit: a fresh checkout starts the basic API/UI and passes CI without credentials or downloaded model weights in Git. Push and verify the remote commit. The planning commit alone does not meet this application gate.

### Phase 2 — Data ingestion and transparent baselines (week 2)

Prepare a compact event corpus, ATT&CK documents, synthetic benign cases, and reviewed gold answers. Implement deterministic search and a rules-only investigation baseline, then a single-agent RAG baseline. This supplies comparisons before a complex workflow is introduced.

Exit: repeatable ingestion, split manifest, cited answers, and first baseline measurements. Missing evidence produces an explicit abstention.

### Phase 3 — First working web application (weeks 3–4)

Build an incident list, case detail page, chronological evidence view, source citations, agent activity stream, query workbench, and response approval screen. Connect all displayed actions to actual backend state. Label fixture/demo mode visibly; do not present canned responses as live model inference.

Deliver one complete vertical slice: select a case → run an investigation → inspect citations → propose an action → approve or reject → observe the simulated inventory and audit event. Show useful errors and cancellation.

Exit: browser end-to-end test completes this flow; restart preserves case state; rejected actions do not execute.

### Phase 4 — Multi-agent workflow and NVIDIA integration (weeks 4–5)

Implement the specialist roles, bounded planning, evidence verification, and scoped memory with NeMo Agent Toolkit. Integrate Nemotron/NIM and Retriever embedding/reranking endpoints. For natural-language-to-query, generate a typed query representation; enforce permitted fields, bounded time ranges, row limits, and read-only access before execution. Do not execute arbitrary generated SQL or shell commands.

Exit: measured improvement or documented tradeoff versus the single-agent baseline, working NVIDIA endpoint traces, reproducible configurations, and tests for retry exhaustion, timeout, cancellation, and model unavailability.

### Phase 5 — Guardrails, privacy, and failure containment (week 6)

Add input/output/content checks, PII/NER detection and redaction, retrieval provenance, signed policy bundles, and an external deny-by-default tool broker. Put tenant checks and authorization at the data/tool layers. Do not rely on a prompt telling the model to behave.

Test direct/indirect injection, poisoned retrieval, forged tool output, memory poisoning, cross-tenant access, unsafe arguments, forged approvals, secret exfiltration, and long-running loops. Export redacted audit traces with integrity checks and an explicit retention policy.

Exit: all fixed security invariant tests pass; legitimate tasks remain measurable; failures contain the affected case and do not grant permissions or corrupt another case.

### Phase 6 — Runtime and supply-chain security (week 7)

Deploy tools inside actual OpenShell sandboxes on Linux, with minimum filesystem access, network destination allowlists, resource limits, short-lived credentials, and process restrictions. Test attempted boundary violations from within the sandbox.

Pin dependencies and container digests. Produce SBOMs and provenance, sign images/models or their manifests and skill/tool bundles, verify signatures before loading, and exercise a tampered-artifact rejection. Add secret scanning, vulnerability triage, and justified VEX statements where applicable. VEX is a documented applicability assertion, not a substitute for patching.

Exit: unsigned/tampered artifacts fail admission; blocked filesystem/network/tool actions are verified by runtime evidence; a malicious skill cannot change trusted policy.

### Phase 7 — Adversarial benchmark and release gates (week 8)

Run Garak and workflow-aware attack scenarios against fixed releases. Add NeMo Auditor when available. Display benchmark runs, attack outcomes, legitimate task success, and latency in the UI with dataset/configuration identifiers.

Provisional gates, to freeze after the development pilot and before the held-out test: zero unauthorized executions or cross-tenant reads in invariant suites; zero leaked synthetic canaries in the defined suite; benign task success at least 90% of the same agent's unprotected baseline; benign false blocks at most 5%. These are project targets, not measured results or universal security guarantees. Report all failures and sample counts.

Exit: a deliberate regression blocks CI/release; every reported result links to a configuration, artifact hash, and scoring rule. See the separate paper protocol for the controlled experiment.

### Phase 8 — Customization and accelerated serving (weeks 9–10)

Fine-tune a small supported model/adapter on training-only examples for structured investigation output or safe tool selection. Compare base versus adapted models for task quality, query correctness, safety, and calibration. Publish the model card and exact recipe where licensing permits.

Benchmark the supported optimized serving profile against a matched reference. Capture cold/warm start, time to first token, total latency, tokens/second, completed cases/minute, concurrency, GPU memory/utilization, and measured cost per successful case. Compare quantization quality and safety regressions as well as speed. Trial Dynamo or NIM Operator only after single-node bottlenecks justify scale-out.

Exit: rerunnable GPU results tied to hardware, drivers, model revisions, context/output sizes, concurrency, precision, and serving engine. No assumed speedup figures.

### Phase 9 — Confidential AI and isolated deployments (weeks 11–12)

Deploy on a verified supported CPU TEE (AMD SEV-SNP or Intel TDX) and NVIDIA CC-capable GPU configuration. Validate GPU SKU, CC mode, firmware, drivers, guest stack, and model-serving compatibility; merely renting an H100 is insufficient. NVIDIA lists the [attestation prerequisites](https://docs.nvidia.com/attestation/attestation-client-tools-sdk/latest/gpu_and_switch_attestation.html).

Bind fresh CPU/GPU attestation and workload measurements to a challenge, validate them through a verifier, and condition key release on policy through a key broker/KMS integration. Encrypt stored model/data artifacts. Reject stale evidence, unapproved measurements, disabled CC, revoked evidence, and verifier unavailability. Exercise key rotation and revocation, including active-session implications.

Provide a protected-VM reference deployment first. Add a supported Confidential Containers profile if infrastructure allows. Design disconnected deployment around mirrored signed images/models, local retrieval, offline verification collateral with expiry policy, and an appropriate local verifier; test denied outbound connectivity and account for key rotation/update operations. Document any unavailable feature instead of treating a network toggle as air-gap validation.

Exit: successful and denied attestation/key-release traces on real hardware plus overhead measurements. Until then, label the flow as a simulated contract test and the deployment as unvalidated. A Mac, API endpoint, or generic GPU VM cannot substantiate confidential-GPU claims.

### Phase 10 — Production readiness and delivery (weeks 13–14)

Add OIDC/RBAC, tenant separation, least-privilege service identities, telemetry, rate limits, backups, restore tests, canary rollout, rollback, and incident runbooks. Test endpoint failure, policy-service failure, unavailable retrieval, worker restart, duplicate messages, and concurrent cases. Freeze workload-specific SLOs using baseline evidence and validate a bounded soak run.

Package discovery notes, architecture decision records, a partner integration contract, sizing/cost guidance, rollout plan, workshop lab, demo video, release notes, and actionable NVIDIA product feedback with reproductions. Write the paper from measured results, limitations, and failure analysis.

Exit: a tagged release reproducible by another person, complete evidence links, documented deployment limitations, and an independently reviewable paper artifact. This establishes a production-like portfolio, not proof of actual enterprise production adoption.

## 6. Job-description coverage and evidence

- Discovery through PoC/rollout/scale: charter, architecture reviews, milestones, acceptance tests, canary/rollback and sizing documents. Real strategic engagement remains experience-dependent.
- Multi-agent orchestration, planning, memory, tools and RAG: running case workflow, bounded state transitions, memory isolation tests, query controls, and evidence citations.
- Detection/response, triage, investigation and analyst automation: event replay, ATT&CK mapping, timeline, response proposals, simulated remediation, and analyst UI.
- Natural-language query, PII and content safety: typed query validation, PII/NER evaluation, redaction, and separate content-safety measurements.
- Reliability, safety and policy enforcement: deterministic broker, invariant tests, guardrails, failure injection, release gate and rollback demo.
- NVIDIA ecosystem: real integration evidence for NAT, NIM/Nemotron, Retriever, Guardrails and OpenShell; measured training/serving evidence for supported NeMo/PyTorch and TensorRT profiles. Auditor, Dynamo, Triton and NIM Operator are conditional extensions, each tracked separately.
- LLM red-teaming and evaluation: Garak, AgentDojo, SOC attack adaptations, held-out runs and confidence intervals.
- Supply chain and runtime security: SBOM, signatures, provenance, admission denials, justified VEX, sandbox boundary tests, and controlled tool invocation.
- Post-training and model security: training recipe, clean splits, model card, safe tool-use results, quantization regression checks and signed registry manifests.
- Confidential computing/KMS/protected infrastructure: validated CPU/GPU evidence, policy-gated keys, negative tests, rotation/revocation, and deployment diagram. Hardware-dependent.
- Air gaps and Confidential Containers: documented and tested profiles if supported infrastructure is obtained; otherwise clearly marked design-only.
- Blueprints, workshops, whitepaper and feedback: reference repo, repeatable lab, research paper, field guidance and minimal product issue reproductions.
- Python/Linux/framework depth: service code, runtime/deployment work, profiling and PyTorch training artifacts. TensorFlow is not necessary when PyTorch demonstrates the requested framework experience.
- Degree, equivalent experience, 8+ years and leadership history: supplied by the candidate's real background; cannot be manufactured by this project.
- The final reference to “factory planning” is inconsistent with the rest of the role. Treat it as a possible posting error and clarify with the recruiter rather than distorting the SOC project around it.

## 7. Scope, environment, and sequencing

The schedule is a planning estimate for focused solo work with timely infrastructure access, not a delivery promise. Target the local usable application by week 4, the core safety paper experiments by week 8, then add GPU optimization and confidential infrastructure. Part-time effort and hardware procurement extend the calendar.

Local profile: application, fixture ingestion, tool/policy tests, UI, and optional remote inference. GPU profile: NVIDIA Brev for self-hosted NVIDIA integrations, training, and serving measurements. Confidential profile: independently verified protected hardware and attestation/KMS. Maintain visible planned / implemented / locally tested / GPU validated / hardware attested states for every capability.

Do not provision paid GPUs before choosing provider, hardware, region, and budget. The user supplied the GitHub destination https://github.com/zabahana/agentic-ai-SS and selected NVIDIA Brev for GPU access. Preserve existing repository visibility. No external publication or infrastructure spending is implied by the planning estimates.

The first implementation increment should be the repository foundation and a working incident-to-evidence UI/API slice, followed by a real model endpoint, then layered controls. That order yields a demonstrable project early while preserving a rigorous path to the complete portfolio.


## 8. NVIDIA Brev execution path

Brev is the selected GPU environment. Its [official overview](https://docs.nvidia.com/brev/getting-started/overview) documents preconfigured GPU environments; [Launchables](https://docs.nvidia.com/brev/concepts/launchables) provide a path to a reusable deployment package.

1. Develop and test the UI, API, data pipeline, policy broker, and fixture mode locally.
2. Select the exact Nemotron model and supported NIM profile; calculate VRAM for weights, KV cache, expected context/concurrency, and embedding/safety models. Select a Brev GPU and storage plan from those requirements rather than promising every workload fits one GPU.
3. Set an hourly and total experiment budget, choose an available region/provider, and provision the chosen instance through the authenticated Brev account. This planning phase does not provision resources.
4. Clone the repository at a pinned commit, inject registry credentials through secrets, verify GPU/container compatibility, and launch locked configurations. Keep development ports behind authenticated access or SSH forwarding.
5. Run smoke tests, then inference/retrieval/safety measurements; stop paid compute between planned runs and confirm storage charges separately.
6. Run adapter training and optimized-serving experiments on compatible capacity, storing results with configuration and hardware metadata.
7. Package a Brev Launchable after the setup is reproducible and export artifacts before teardown.
8. For confidential AI, ask whether the specific Brev provider/instance exposes supported CPU TEE, GPU CC and attestation access. If it does not, keep Brev for ordinary GPU experiments and use a separate supported confidential environment for that milestone. Do not label a standard Brev instance confidential.
