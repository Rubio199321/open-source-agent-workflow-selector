# Choosing an Open-Source Workflow or Agent Orchestrator


> **Decision:** Use this selector when choosing among event wiring, visual integration, script runners, scheduled DAGs, durable workflows, and AI-agent orchestration; skip it if connector count or screenshots are your only criteria.
>
> **For:** automation architects, AI platform teams, and operations engineers  ·  **Not for:** teams without a defined state, failure, approval, identity, or exit model

“Workflow automation” covers systems that do fundamentally different jobs: event-driven wiring, human-friendly visual integration, script execution, scheduled data DAGs, durable business processes, and conversations among AI agents. Comparing them by connector count or screenshot quality produces bad decisions.

This guide starts with six workload scenarios. For each one, specify the state, failure, approval, identity, portability, and recovery contract before looking at a project. The included [`decision-scenarios.yaml`](assets/decision-scenarios.yaml) is the main artifact; its states begin as `NOT_EVALUATED`.

No tool is declared a winner. Documentation establishes a declared mechanism, not local reliability, connector correctness, license suitability, or operational cost.

## Scenario A: personal event automation

**Example:** when an owned mailbox receives a labeled message, extract a non-sensitive field and add a task after user confirmation.

The important questions are connector availability, local deployment, credential storage, trigger deduplication, retries, rate limits, and a readable run history. [Activepieces installation documentation](https://www.activepieces.com/docs/install/overview/) and the [Node-RED user guide](https://nodered.org/docs/user-guide/) represent visual/event-oriented approaches. They do not promise equivalent connector semantics.

Reject a candidate if it cannot represent “already processed,” if retries can duplicate external effects, or if a low-code editor hides which credentials and data leave the instance.

## Scenario B: internal script and approval flow

**Example:** collect a request, run reviewed Python/TypeScript, show a generated diff, require approval, then call an internal API.

[Windmill documentation](https://www.windmill.dev/docs/intro) describes a script/workflow platform. Test how code is versioned, which identity executes each step, whether approval binds to exact parameters, and how secrets reach a worker. A convenient form does not replace authorization.

Reject candidates that require broad shared credentials, cannot separate preview from effect, or cannot replay with the same script and input identity.

## Scenario C: scheduled data pipeline

**Example:** ingest governed data, validate schema, compute a partition, and publish a table only when data-quality checks pass.

[Kestra workflow-component documentation](https://kestra.io/docs/workflow-components) and [Apache Airflow core concepts](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html) support data/workflow roles. Compare backfill, partition identity, scheduler semantics, code/config review, lineage, retry scope, and how a failed partial write is handled.

A data DAG is not automatically a durable business transaction. A green task after a partial external effect may be worse than a visible failure.

## Scenario D: long-running durable process

**Example:** wait days for payment or human evidence, survive worker restarts, handle duplicate events, and compensate an earlier effect.

[Temporal workflow-execution documentation](https://docs.temporal.io/workflow-execution) is a reference for durable execution concepts. [Prefect flow concepts](https://docs.prefect.io/v3/concepts/flows) document a Python-oriented workflow model. Do not infer identical replay, determinism, event history, or compensation semantics from the word “workflow.”

Reject a candidate when process state exists only in worker memory, external effects cannot be made idempotent, or version changes make in-flight executions uninterpretable.

## Scenario E: model-and-tool application flow

**Example:** retrieve controlled context, invoke a model, validate structured output, call a read-only tool, and send a draft to a human.

[Langflow documentation](https://docs.langflow.org/) and [Flowise documentation](https://docs.flowiseai.com/) represent visual AI-flow builders. Measure model/provider portability, prompt/config versioning, tool permission boundaries, output validation, trace export, evaluation hooks, and behavior when a provider is unavailable.

A canvas edge does not prove data typing, trust, or transactional behavior. Treat retrieved text and tool results as untrusted. The model must not expand its own permissions or approve its own output.

## Scenario F: multi-agent collaboration

**Example:** one agent proposes research, another critiques it, a deterministic validator checks artifacts, and a human approves the final action.

[AutoGen AgentChat documentation](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/index.html) documents an agent-conversation framework. Agent conversation is not a substitute for durable workflow state, policy, or independent verification. Define termination, budget, tool ownership, evidence handoff, and what happens when agents agree for the wrong reason.

Reject designs where “consensus” authorizes an external effect, where agents share a broad credential, or where the only audit trail is natural-language chat.

## The rejection funnel

Apply these tests in order. Stop when a candidate fails a mandatory contract.

### 1. Deployment and license fit

Record the exact edition, version, license, deployment topology, external services, and features available in the intended installation. “Open source,” “source available,” cloud, and self-hosted editions may not expose identical features or terms. Verify current upstream terms; this guide does not provide legal advice.

### 2. State semantics

Name the unit of state: event, node execution, flow run, DAG run, partition, event history, model turn, or agent conversation. Determine what persists through restarts and how upgrades affect in-flight work.

### 3. Effect semantics

For every external write, specify idempotency, deduplication, retry boundary, timeout, compensation, and ownership. “Retry three times” is dangerous when the operation already succeeded but the acknowledgment was lost.

### 4. Identity and secrets

Use least-privilege identities per environment and effect class. Prefer short-lived credentials. Test redaction and worker isolation with fake credentials before any production connection. A vault integration does not prove secrets never appear in logs, environment dumps, prompts, or connector errors.

### 5. Human control

Approval must show the exact target, parameters, expected effect, evidence, and rollback. It must expire and be re-requested after material changes. Model output, agent consensus, a scanner score, or a successful dry run cannot self-authorize production.

### 6. Portability and exit

Export flows, scripts, environment declarations, schedules, connection references, and run evidence. Identify proprietary nodes and hosted-only dependencies. Rebuild one small scenario elsewhere; an export file that only the same product can import is not full portability.

## A fair local evaluation

Use synthetic services and fake credentials. For each candidate, implement only the smallest representative scenario and inject the same failures:

1. duplicate trigger;
2. worker crash before acknowledgment;
3. external service timeout after an effect;
4. invalid or malicious untrusted input;
5. secret-like canary in an error;
6. approval parameter change;
7. version upgrade with an in-flight run;
8. telemetry backend outage.

Record expected and observed behavior, operator steps, artifacts retained, and unknowns. Do not compare execution speed across unlike workload semantics. Total operator effort and recoverability are often more relevant than node throughput.

## Safety and legal boundary

- Evaluate in an owned, disposable environment with synthetic connectors first.
- Do not connect production email, finance, customer, deployment, or messaging systems until identity, replay, rollback, and approval tests pass.
- Treat workflow definitions, imported templates, model text, webhooks, and connector output as untrusted.
- Limit outbound destinations and methods; prevent the model from creating new connectors or credentials.
- Keep human approval independent from the agent or workflow being approved.
- Stop and investigate unexpected effects, duplicate writes, secret exposure, silent field loss, or unbounded retries.
- A passing scenario is version-specific and does not authorize migration or publication.

## Upstream evidence and author design

The ten records in [`sources/sources.json`](sources/sources.json) are precise official documentation pages for representative systems. They support declared mechanisms only. The six scenarios, rejection funnel, failure injection set, and decision vocabulary are author recommendations.

## Change management

Re-evaluate when edition/license, version, connector, worker model, state backend, identity provider, secret path, retry policy, model/tool, approval mechanism, or deployment topology changes. Preserve failed scenario evidence. Never replace a failure with a rerun without linking both.

## Adjacent guides

- [agent-rules-portability-guide](https://github.com/Probstpp1/agent-rules-portability-guide) — Use it when selected agent products must consume consistent repository rules through reviewed adapters.
- [agent-execution-control-sandbox](https://github.com/MargaritaOfficial/agent-execution-control-sandbox) — Use it when orchestrated agents execute code, browse, or process data across a security boundary.
- [agent-observability-field-guide](https://github.com/duthdoll45/agent-observability-field-guide) — Use it when workflow and agent decisions need a trace model that survives tool changes.

## License

Original selector and scenario artifact: CC BY 4.0. Linked projects and documentation retain their own terms.

If this saved you a decision, star the repo — it is how the next reader finds it.
