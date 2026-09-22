# Sentinel — Secure Agentic SOC on NVIDIA

A planned research and reference implementation for evidence-grounded security investigation, controlled tool use, adversarial evaluation, and confidential AI deployment.

**Status: planning foundation only.** Application, integrations, benchmarks, GPU validation, and paper results are not implemented yet.

Start with [the step-by-step project plan](docs/PROJECT_PLAN.md), including scope, architecture, datasets, acceptance criteria, job-description coverage, and delivery milestones. The [paper protocol](paper/PROTOCOL.md) defines the research before experiments begin.

Target application: a web console where an analyst selects an incident, observes agent actions and cited evidence, reviews a proposed response, and inspects safety decisions. Remediation initially changes only a simulated asset inventory.

Planned stack: Python/FastAPI, React/TypeScript, PostgreSQL/pgvector, NVIDIA NeMo Agent Toolkit, Nemotron through NIM, NeMo Retriever, NeMo Guardrails, OpenShell, and an evaluation harness. Additional GPU optimization and confidential-computing profiles come after the local vertical slice.

The project is independent and does not imply NVIDIA or dataset-publisher endorsement.

Repository: https://github.com/zabahana/agentic-ai-SS

Selected GPU environment: NVIDIA Brev. Confidential GPU support remains an instance-specific validation requirement.
