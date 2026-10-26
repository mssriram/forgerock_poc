# Resume-Based AI Engineer/SDE Hiring Manager Guide

## Purpose

This guide is tailored to this resume story:

> Backend engineer with 3.5+ years of Java/Spring Boot and AWS experience who built an agentic ITSM resolution assistant using Python, FastAPI, LangGraph, RAG, a vector database, human approval gates, and MCP-based tools, with an observed MTTR reduction from 35 minutes to 15 minutes.

The answers are designed for a hiring-manager interview, where the interviewer is likely to test ownership, judgment, trade-offs, production readiness, and communication more than framework trivia.

## Truth Guardrail

The resume establishes the technologies and outcome above, but it does not establish the details below. Fill these in before the interview and never invent them:

- `[LLM_AND_PROVIDER]`
- `[EMBEDDING_MODEL]`
- `[VECTOR_DATABASE]`
- `[SOP_FORMATS_AND_COUNT]`
- `[TICKET_TYPES]`
- `[MCP_TOOLS_AND_TARGET_SYSTEMS]`
- `[DEPLOYMENT_ENVIRONMENT]`
- `[YOUR_EXACT_OWNERSHIP]`
- `[TEAM_COMPOSITION]`
- `[EVALUATION_DATASET_SIZE]`
- `[PILOT_SIZE_AND_DURATION]`
- `[MTTR_MEAN_OR_MEDIAN]`
- `[MTTR_START_AND_END_EVENTS]`
- `[QUALITY_GUARDRAIL_METRICS]`
- `[REAL_FAILURE_OR_LIMITATION]`

When asked about a capability that was not implemented, use this wording:

> That was not part of the version I built. For a production rollout, I would design it as follows...

That answer demonstrates judgment without overstating experience.

## Tonight's Priority Path

If preparation time is short, rehearse these questions aloud first:

1. Q1: Tell me about yourself.
2. Q2: Tell me about the AI project.
3. Q3: What did you personally own?
4. Q11: Walk me through the architecture.
5. Q14: Why LangGraph?
6. Q31: Walk me through RAG.
7. Q33: Why RAG instead of fine-tuning?
8. Q46: What is MCP and why did you use it?
9. Q58: How did you evaluate the system?
10. Q61: Defend the MTTR improvement.
11. Q71: How did you prevent unsafe actions?
12. Q90: What was the hardest trade-off?
13. Q91: What would you improve?
14. Q97: Why move from backend engineering to AI engineering?
15. Q100: Why should we hire you?

Keep an initial answer to 60-90 seconds. Use this structure:

1. **Headline:** answer the question directly.
2. **Decision:** explain the requirement and trade-off.
3. **Implementation:** state what you personally did.
4. **Evidence:** give a metric, test, or observed outcome.
5. **Limitation:** identify what you would improve.

---

# 1. Opening, Resume Story, and Ownership

## Q1. Tell me about yourself.

**Interview-ready answer**

I am a backend engineer with 3.5+ years of experience building secure, high-traffic services using Java, Spring Boot, PostgreSQL, Redis, and AWS. At Deloitte, I have worked on a government health-benefits platform serving eight million users. My work has included microservices, identity federation, performance optimization, resilience, cloud modernization, and production quality controls.

I have increasingly applied that foundation to AI engineering. I built an ITSM resolution assistant using Python, FastAPI, LangGraph, RAG, and MCP-based tools. The model interpreted tickets and proposed resolutions, while deterministic application code controlled retrieval, tool validation, approvals, and execution. That combination of AI capability and production backend discipline is why an AI Engineer/SDE role is a natural next step for me.

## Q2. Tell me about your AI project.

**Interview-ready answer**

The problem was that support engineers spent significant time understanding tickets, locating the right SOP, gathering diagnostics, and performing routine resolution steps. I built an agentic ITSM support assistant using Python and FastAPI, with LangGraph orchestrating the workflow.

The system retrieved relevant SOP content through RAG, generated a grounded diagnosis and action proposal, and exposed controlled operational tools through MCP. I added human approval gates for high-impact actions because an operational agent must not turn model confidence into execution authority. For the measured workflow, the assisted process reduced MTTR from 35 minutes to 15 minutes. My strongest lesson was that production AI quality comes from the whole system: data, retrieval, orchestration, tools, policy, evaluation, and observability, not only the model.

## Q3. What did you personally own?

**Interview-ready answer template**

I personally owned `[YOUR_EXACT_OWNERSHIP]`. That included designing `[GRAPH_OR_RAG_OR_TOOL_COMPONENTS]`, implementing `[SPECIFIC_CODE_AND_INTEGRATIONS]`, and validating the flow using `[TESTS_OR_EVALUATIONS]`. I worked with `[TEAM_OR_DOMAIN_EXPERTS]` on `[THEIR_SCOPE]`.

The key decision I drove was `[DECISION]`, because `[CONSTRAINT_AND_TRADE_OFF]`. LangGraph supplied orchestration primitives, the vector database supplied similarity search, and MCP supplied a standard tool interface. My engineering contribution was defining how those components were composed, constrained, tested, and connected to the business workflow.

## Q4. Why was this problem suitable for AI?

**Interview-ready answer**

The workflow mixed ambiguous language with deterministic operations. Understanding free-form tickets, mapping different wording to an SOP, and synthesizing evidence benefited from an LLM and embeddings. Authorization, policy checks, retries, state transitions, and tool execution did not need probabilistic reasoning and belonged in conventional code.

That separation made AI appropriate without forcing every step through the model. If the tickets had been fully structured and the resolution path could be represented reliably as rules, I would have preferred a normal workflow engine.

## Q5. What made it agentic rather than a chatbot?

**Interview-ready answer**

A chatbot mainly generates a response. This system maintained state, retrieved evidence, selected from allowed tools, incorporated observations, and chose whether to continue, request approval, clarify, escalate, or terminate. That stateful action loop made it agentic.

I intentionally bounded the autonomy. It was not an open-ended loop. Allowed states and transitions were explicit, tools were allowlisted and typed, and execution had step, time, tool, and approval limits. I used agentic behavior only where the next step depended on context.

## Q6. What business value did it create?

**Interview-ready answer**

The immediate value was reducing time spent searching procedures, collecting diagnostics, and executing routine steps. For the measured ticket workflow, MTTR moved from 35 minutes to 15 minutes. The system also aimed to make recommendations more consistent and preserve an audit trail of the evidence and actions used.

I would always pair MTTR with quality guardrails such as reopen rate, escalation rate, engineer acceptance, incorrect-action rate, and policy violations. Faster closure is not valuable if it creates repeat incidents or unsafe actions.

## Q7. What was the hardest engineering problem?

**Interview-ready answer**

The hardest problem was not producing a plausible answer; it was making an AI-generated proposal safe enough to influence an operational system. A model can select a valid-looking tool and arguments that are still unauthorized, stale, or inappropriate.

I addressed that by separating proposal from execution. The model produced a typed proposal, application code validated it, policy checked the caller and target, high-risk actions required approval, and the tool boundary returned a structured result. The model never became the authorization boundary.

## Q8. What was the most important design principle?

**Interview-ready answer**

The model could propose, but deterministic software had to validate and dispose. I used the model for semantic interpretation and synthesis. I kept identity, authorization, business rules, budgets, retries, approvals, and side effects outside the model.

This created a probabilistic core inside a deterministic shell. It improved testability and made failures explicit: insufficient evidence caused abstention, invalid output caused rejection, and uncertain side effects caused reconciliation or escalation.

## Q9. What constraints shaped the design?

**Interview-ready answer**

The main constraints were operational risk, compliance, changing SOP knowledge, external-system reliability, model latency and cost, and the need to explain why an action was proposed. Those constraints led to RAG instead of storing changing knowledge in model weights, a state graph instead of an unrestricted loop, narrow MCP tools instead of generic execution, and approval gates for consequential actions.

I would confirm the exact compliance and latency constraints from the real project rather than claim a regulation or SLO that did not apply.

## Q10. What does production-ready mean for this system?

**Interview-ready answer**

Production-ready means measurable task quality, least-privilege access, typed contracts, safe failure behavior, durable state, idempotent actions, observability, versioning, controlled rollout, rollback, and clear operational ownership. It also means knowing when the system must abstain or escalate.

A successful demo proves that the happy path is possible. A production system proves that common and dangerous failure paths are contained, detectable, recoverable, and tested.

---

# 2. Architecture and System Design

## Architecture Reference

Use this diagram to keep the walkthrough consistent. Describe only the components you actually built as implemented; frame the rest as the production target.

```text
Support engineer
    |
    v
Gateway + authentication
    |
    v
FastAPI API ---> LangGraph orchestrator <--> Durable state + action ledger
              |
              v
        Classify + normalize
              |
              v
        Authorized retrieval <--- SOP store + vector index
              |
              v
        LLM diagnosis + proposal
              |
              v
        Deterministic policy + validation
              |
          +---------+----------+
          |                    |
     Low-risk action      High-impact action
          |                    |
          |              Human approval
          |                    |
          +---------+----------+
              |
              v
           MCP tool server
              |
              v
          ITSM + operational systems
              |
              v
         Resolve, clarify, or escalate

Metrics, traces, and audit span the API, orchestrator, retrieval, and tools.
```

## Q11. Walk me through the architecture end to end.

**Interview-ready answer**

A request enters FastAPI, where the service authenticates the caller, validates the ticket payload, derives tenant and permission context, and assigns a correlation ID. LangGraph initializes typed state and classifies the incident. The retrieval layer searches authorized SOP content and returns passages with document IDs, versions, and scores.

The LLM uses that evidence to produce a structured diagnosis and proposed plan. If a tool is needed, deterministic code validates its schema, authorization, target, policy, and risk. Read-only or low-risk operations can proceed within policy; high-impact actions pause for an authorized human approval. The MCP tool returns a structured observation, after which the graph resolves, asks for clarification, or escalates. State transitions, evidence, approvals, and actions are recorded for audit and debugging.

## Q12. What did the LangGraph state contain?

**Interview-ready answer**

I would keep explicit fields for the run and ticket IDs, trusted user and tenant context, normalized incident details, retrieved document references, diagnosis, proposed action, tool result, approval status, attempt counters, deadlines, and terminal outcome.

The transcript should not be the only source of state. Explicit typed fields make validation, redaction, migration, replay, and testing easier. Sensitive content should be minimized or referenced by ID, and identity must never be accepted from model-generated state.

## Q13. Which decisions were model-driven and which were deterministic?

**Interview-ready answer**

The model handled narrow semantic decisions: classifying an ambiguous ticket, summarizing evidence, identifying whether an SOP path applied, and proposing a diagnosis or tool call. Deterministic code handled identity, authorization, data access, schema validation, business invariants, approval policy, retries, deadlines, budgets, and terminal states.

An unrecognized classification, insufficient evidence, malformed output, denied permission, or exhausted budget did not give the model more authority. It moved the run to clarification or escalation.

## Q14. Why did you choose LangGraph?

**Interview-ready answer**

The use case required branching, state, tool observations, human interruption, and resume. A simple chain would become difficult to reason about, while a free-form agent loop would allow more autonomy than the workflow needed. LangGraph let me express explicit nodes and conditional edges and gave useful checkpoint and interrupt primitives.

The trade-off was framework complexity and another state model to operate. If the flow were only retrieve and answer, I would use a simpler pipeline. LangGraph was justified because intermediate observations changed the next step and approval could interrupt execution.

## Q15. Why not use a fully deterministic workflow?

**Interview-ready answer**

I would use a deterministic workflow wherever the ticket fields and resolution rules are sufficiently structured. The model was useful because incident descriptions and SOP language were variable and the relevant path could depend on semantic context.

The resulting design was hybrid: deterministic nodes and policy contained a small number of model-driven decisions. If evaluation showed that a model decision could be replaced by a reliable classifier or rule, I would simplify that part.

## Q16. Why FastAPI?

**Interview-ready answer**

FastAPI fit the Python AI ecosystem and provided typed request and response models, dependency injection, automatic API schemas, and good asynchronous I/O support. Model, retriever, and tool calls are largely network-bound, so async clients can improve concurrency.

FastAPI does not automatically make blocking work safe. Synchronous SDK calls, heavy parsing, or local embedding computation can block the event loop. I would use native async clients and bounded concurrency, and move CPU-heavy work to separate workers or services.

## Q17. Would the entire workflow run in one HTTP request?

**Interview-ready answer**

Only a short, bounded recommendation path should remain synchronous. Tool-heavy requests, retries, or human approval can exceed gateway timeouts and tie correctness to one process and connection.

For production, I would persist the run, enqueue it, and return `202 Accepted` with a run ID. A client could poll or receive SSE progress. Workers would checkpoint meaningful transitions and resume after approval or transient failure. This also makes client retries less likely to duplicate logical work.

## Q18. What would you persist?

**Interview-ready answer**

I would persist run metadata, current typed state, version numbers, authorized document references, proposals, policy decisions, approvals, action status, idempotency keys, and terminal outcomes. Optimistic concurrency would prevent duplicate workers or callbacks from overwriting newer state.

I would not persist secrets, unrestricted prompts, hidden reasoning, or unnecessary sensitive document content. Auditability requires structured evidence and action records, not every token the model processed.

## Q19. What happens if the process crashes halfway through a run?

**Interview-ready answer**

The worker resumes from the last durable checkpoint. Before retrying a step, it checks whether the step was read-only or could have produced an external side effect. Read operations can normally be retried within policy. Mutating operations require reconciliation against an action ledger or target system.

The dangerous window is after an external system accepts an action but before success is checkpointed. A stable idempotency key and persisted action state prevent blind duplicate execution.

## Q20. How do you make tool execution idempotent?

**Interview-ready answer**

Each logical action gets an idempotency key derived from the run and action identity, not from an individual attempt. The key is persisted before dispatch and passed to the tool or downstream API when supported. Repeated calls return or reconcile the prior result.

For systems without native idempotency, I prefer state-setting operations, such as setting a service to a desired state, and verify current state before retrying. The realistic goal is at-least-once processing with effectively-once business effects, not a claim of universal exactly-once delivery.

## Q21. What consistency model would you use?

**Interview-ready answer**

Run and action state needs strong enough consistency to prevent duplicate transitions, typically through transactions, unique constraints, and optimistic compare-and-set updates. The vector index and caches can be eventually consistent because they are derived views, but their freshness must be visible and bounded by an SLO.

Before a consequential action, the service revalidates current resource state, authorization, approval, and source-document status rather than relying only on an earlier model context.

## Q22. How do you deploy a graph or state-schema change with runs in flight?

**Interview-ready answer**

Each run records its graph, prompt, model, tool-contract, and state-schema versions. Additive changes can be backward compatible. Breaking changes require an explicit state migration or side-by-side workers that finish old runs under the old definition.

When an approval resumes later, the service still revalidates current permissions, policy, schema, and target state. Versioning preserves replay; revalidation preserves safety.

## Q23. How would you scale the system to 100 times the traffic?

**Interview-ready answer**

I would first measure the bottleneck by stage: API concurrency, queue age, retrieval latency, token throughput, provider limits, tool quotas, and database contention. Stateless API and worker instances can scale horizontally. Queue-backed execution absorbs bursts, while per-tenant quotas, bulkheads, and concurrency limits protect downstream dependencies.

I would scale retrieval separately, batch offline embeddings, cache only safe tenant-scoped data, route simpler cases to smaller models when evaluation supports it, and autoscale using queue age and token demand rather than CPU alone.

## Q24. Estimate capacity for 150-200 tickets per day.

**Interview-ready answer**

I would first ask for peak distribution, tokens per call, model-call count, tool fan-out, latency SLO, and approval percentage. For 150-200 tickets per day, the 24-hour average is only about 0.0017-0.0023 tickets per second. That average is not useful for sizing, so I would assume 70% arrive during eight business hours and the busiest hour is 4 times the business-hours average. That gives a peak of roughly 0.15-0.20 tickets per second. If each ticket makes two five-second model calls, there are roughly 1.5-2 concurrent model calls before headroom.

I would then calculate tokens per minute, retrieval QPS, worker occupancy, and provider quotas from measured percentiles. With two model calls per ticket, the peak is roughly 0.3-0.4 model calls per second, or 18-24 model calls per minute. Little's Law, $L = \lambda W$, is a useful check, but provider quotas and p95 latency usually define the practical limit.

## Estimation Playbook: How to Answer Capacity Questions

For estimation questions, the interviewer is usually evaluating your method more than the final number. Use this sequence:

1. **Clarify the unit:** tickets per day, peak requests per second, tokens per minute, concurrent runs, storage, or cost.
2. **Convert the workload:** daily volume to average rate, then apply a peak factor.
3. **State assumptions:** model calls per ticket, tokens per call, latency, tool fan-out, cache hit rate, and approval percentage.
4. **Calculate the bottleneck:** use one simple equation and show the units.
5. **Add headroom:** account for retries, bursts, failures, and uneven traffic.
6. **Name the validation plan:** load test, provider quota check, production percentiles, or a pilot.

### Useful Mental Conversions

Memorize these rough conversions:

- 1 day = 86,400 seconds.
- 150-200 requests/day is about 0.0017-0.0023 requests/second on a 24-hour average.
- 150-200 requests/day concentrated into 8 business hours is about 0.0052-0.0069 requests/second.
- A 4x busiest-hour factor makes that roughly 0.021-0.028 requests/second for a typical hour.
- 1 million tokens/day is about 11.6 tokens/second averaged across 24 hours.
- 1 million tokens/day concentrated into 8 hours is about 34.7 tokens/second.
- Requests per second and tokens per second are different bottlenecks. LLM providers usually constrain both.

Use round numbers in the interview. Say “roughly” and explain what would replace the estimate with measured data.

### Core Equations

**Average request rate**

$$
	ext{average RPS} = \frac{\text{requests per day}}{86{,}400}
$$

**Peak request rate**

$$
	ext{peak RPS} = \text{average RPS} \times \text{peak factor}
$$

Use a higher rate if the workload is concentrated in business hours:

$$
	ext{business-hours RPS} = \frac{\text{daily requests}}{\text{business-hours seconds}}
$$

**Concurrency using Little's Law**

$$
L = \lambda W
$$

Here, $L$ is average in-flight work, $\lambda$ is throughput in requests per second, and $W$ is average time in seconds. For capacity planning, use a peak rate and an appropriate service-time percentile, then add headroom.

**Token throughput**

$$
	ext{tokens per minute} = \text{requests per minute} \times \text{tokens per request}
$$

Separate input and output tokens because providers often price or limit them differently.

**Worker count**

$$
	ext{workers} \approx \frac{\text{peak RPS} \times \text{average job seconds}}{\text{target utilization}}
$$

For example, target utilization of 0.7 leaves room for bursts and slow requests. This is a starting estimate, not a final autoscaling number.

**Storage**

$$
	ext{daily storage} = \text{items per day} \times \text{bytes per item}
$$

For vectors, estimate dimensions multiplied by bytes per dimension, then add index overhead, metadata, replicas, and versioned indexes. Float32 uses about 4 bytes per dimension before overhead.

**Cost**

$$
	ext{daily model cost} = \frac{\text{input tokens}}{1{,}000{,}000} \times \text{input price}
+ \frac{\text{output tokens}}{1{,}000{,}000} \times \text{output price}
$$

Then add retrieval, tool, storage, worker, and retry costs. Prefer cost per successful resolution over cost per raw request.

### Example 1: 150-200 Tickets per Day

Assume 150-200 tickets per day, 70% arrive during eight business hours, the busiest hour is 4 times the average business-hour rate, and a ticket makes two model calls.

1. Business-hours volume: $150-200 \times 0.70 = 105-140$ tickets.
2. Business-hours rate: $105-140 / (8 \times 3{,}600) \approx 0.0036-0.0049$ tickets/second.
3. Busiest-hour rate: $0.0036-0.0049 \times 4 \approx 0.0146-0.0194$ tickets/second.
4. Model request rate: $0.0146-0.0194 \times 2 \approx 0.029-0.039$ model calls/second, or about 1.75-2.33 calls/minute.
5. If each call averages 4,000 input tokens and 500 output tokens, token rate is approximately 117-156 input tokens/second and 15-19 output tokens/second.

An interview-ready conclusion is:

> I would plan for roughly 0.015-0.020 tickets per second at the busiest hour and about 0.03-0.04 model calls per second. If a model call occupies a worker for 5 seconds, Little's Law gives roughly 1.5-2 concurrent model calls. I would provision about 3-4 model-call slots initially to absorb retries and short bursts, then validate against real traces and provider quotas.

The important correction is to avoid sizing directly from the daily average. Even at this volume, a short incident burst, a batch import, or a provider retry storm can temporarily dominate normal traffic.

### Example 2: Scaling to 100 Times Traffic

Suppose the current system handles 150-200 tickets per day and the target is 15,000-20,000 per day. That is a 100x increase in volume, but not necessarily a 100x increase in every component.

Break the system into independently scaling dimensions:

- API requests scale with ticket rate.
- Model tokens scale with ticket rate multiplied by tokens per ticket.
- Retrieval queries scale with retrieval calls per ticket and cache hit rate.
- Tool calls scale with tool calls per ticket, but may have much lower downstream quotas.
- SOP ingestion scales with document changes, not ticket traffic.
- Approval work scales with high-risk-action percentage and may become a human bottleneck.

Example assumptions at peak: 15-20 tickets/second, 2 model calls/ticket, 3,000 input tokens/model call, 400 output tokens/model call, and 6-second average model occupancy.

- Model calls: $15-20 \times 2 = 30-40$ calls/second.
- Input tokens: $30-40 \times 3{,}000 = 90{,}000-120{,}000$ tokens/second.
- Output tokens: $30-40 \times 400 = 12{,}000-16{,}000$ tokens/second.
- In-flight model calls: $30-40 \times 6 = 180-240$.
- At 70% target utilization: roughly $180-240 / 0.7 \approx 257-343$ model-call capacity units.

The answer is not “add 100 times more API servers.” It is:

> I would scale stateless API and worker pools horizontally, but I would separately capacity-plan model tokens, provider quotas, retrieval, tool systems, state storage, and human approvals. I would add queues, per-tenant limits, bulkheads, and load shedding because the first constrained dependency may be the model provider or an operational tool rather than the web tier.

### Example 3: Model Cost at 150-200 Tickets per Day

Assume one ticket uses two model calls. Each call averages 4,000 input tokens and 600 output tokens. Suppose the provider price is $5 per million input tokens and $15 per million output tokens.

- Input per ticket: $8{,}000$ tokens.
- Output per ticket: $1{,}200$ tokens.
- Input cost: $8{,}000 / 1{,}000{,}000 \times \$5 = \$0.04$.
- Output cost: $1{,}200 / 1{,}000{,}000 \times \$15 = \$0.018$.
- Approximate model cost: $0.058$ per ticket.
- At 150-200 tickets/day: approximately $8.70-$11.60/day, or roughly $226-$302 per 26-day working month, before retries, tools, retrieval, and infrastructure.

Then explain the engineering levers:

> I would measure cost per successful resolution, not just model spend. I could reduce cost by trimming irrelevant context, routing simple classifications to a smaller model, limiting output length, caching safe retrieval results, and reducing unnecessary tool or reasoning steps. Each change must be checked against groundedness, escalation, reopen, and unsafe-action metrics.

### Example 4: Vector Storage Estimate

Assume 500,000 indexed SOP chunks, 1,536-dimensional embeddings, float32 storage, two replicas, and 2.5x overhead for metadata and the approximate-nearest-neighbor index.

- Raw vector size: $1{,}536 \times 4 = 6{,}144$ bytes, or about 6 KB.
- Raw corpus: $500{,}000 \times 6$ KB, approximately 3 GB.
- With two replicas and 2.5x overhead: roughly $3 \times 2 \times 2.5 = 15$ GB.

The estimate excludes backups, old embedding versions during migration, and growth. A good answer adds:

> I would reserve additional storage for a parallel index during embedding-model migration and for tombstones, metadata, backups, and replication. I would validate the estimate against the selected vector database's actual index overhead because HNSW or other ANN structures can materially change memory use.

### Example 5: Queue and Worker Capacity at Current Volume

Assume a peak of 0.15-0.20 tickets/second and an average end-to-end worker occupancy of 20 seconds because the workflow includes model and tool calls. At 70% target utilization:

- In-flight jobs: $0.15-0.20 \times 20 = 3-4$.
- Worker capacity with headroom: $3-4 / 0.7 \approx 4-6$ concurrent job slots.

This does not mean 57 machines. A worker may process multiple I/O-bound jobs, while provider and tool semaphores limit downstream calls. I would load-test with p50 and p95 job durations, because a few long approval or retry paths can dominate queue age.

### Example 6: Human Approval Capacity at Current Volume

Assume 0.15-0.20 tickets/second at peak, 10% require human approval, and one reviewer can complete 6 approvals per minute.

- Approval requests: $0.15-0.20 \times 60 \times 0.10 = 0.9-1.2$ per minute.
- Review capacity per reviewer: 6 per minute.
- Minimum reviewers: $0.9-1.2 / 6 = 0.15-0.20$, so at least 1 reviewer is needed for coverage.

Then add coverage and burst headroom, perhaps planning for 1-2 reviewer-equivalents depending on support hours and response-time expectations. At 100x growth, the same calculation becomes a real bottleneck: approval load scales with high-risk-action percentage and cannot be solved only by adding API workers. You can reduce pressure by improving risk classification, making requests concise, and automating only proven low-risk actions.

### Example 7: Latency Budget

Suppose the user-facing p95 target is 10 seconds for a recommendation:

- API and authentication: 0.3 seconds.
- Retrieval and reranking: 1.5 seconds.
- First model call: 3 seconds.
- Parallel diagnostic tools: 2 seconds.
- Final model call: 2.5 seconds.
- Network and scheduling margin: 0.7 seconds.

The total is 10 seconds. If the tools are independent, their contribution is the slowest tool rather than the sum. If the final model call is already the bottleneck, adding more parallel tools may not improve user latency and may increase cost or tail risk.

Say:

> I would budget latency by stage and track p50, p95, and p99. The budget must include retries and queue time, not only successful dependency latency. Approval wait time should be reported separately from machine-processing latency because it is a business workflow delay.

### Estimation Hacks That Sound Rigorous

- **Anchor every number to a unit:** “2 calls per ticket,” “4,000 input tokens per call,” or “5 seconds of occupancy.”
- **Use ranges when uncertainty is high:** “I would start with 2-3 model calls per ticket and validate from traces.”
- **Separate average from peak:** daily traffic is not capacity; peak rate and tail latency drive capacity.
- **Separate compute from quotas:** adding workers cannot overcome a provider tokens-per-minute limit.
- **Identify the human bottleneck:** approvals, incident review, and content maintenance may not scale automatically.
- **Include retries:** a 10% retry rate turns 100 logical calls into roughly 110 physical calls.
- **Include headroom explicitly:** “I would add 30% for burst and failure headroom,” then validate it with load testing.
- **Name what would change the answer:** cache hit rate, model choice, token distribution, peak factor, and tool latency.
- **End with a measurement plan:** load test, shadow traffic, quota check, tracing, and a canary.

### A Reusable 45-Second Answer

> I would first clarify whether the number is average or peak and identify the workload unit. Then I would convert volume into peak requests per second, multiply by model calls and tokens per request, and use Little's Law for concurrency. I would separately check retrieval QPS, tool quotas, state-store writes, and human approval capacity because the bottleneck may not be the API tier. I would add explicit headroom for retries and bursts, then validate the estimate with realistic load tests and production p95 traces. The estimate is a starting capacity model, not a substitute for measurement.

## Q25. How would you design for high availability and disaster recovery?

**Interview-ready answer**

I would define RTO and RPO first. APIs and workers are stateless and reproducibly deployed. Run state, action ledgers, audit records, and source-document metadata use replicated storage and tested backups. Vector indexes can be replicated or rebuilt from versioned sources depending on the recovery target.

Failover of mutating work requires reconciliation; I would not simply run the same in-flight action in another region. Recovery exercises should prove that runs either resume safely or terminate with enough context for human handoff.

## Q26. What would you cache?

**Interview-ready answer**

Good candidates are embeddings keyed by content hash and model version, stable prompt prefixes, and short-lived read-only tool results when freshness allows it. Retrieval caching requires tenant, authorization scope, normalized query, filters, and index version in the key.

I would be cautious with semantic or final-answer caches because similar tickets may have different users, permissions, or current resource states. Every cache needs a TTL or invalidation event, tenant boundary, and explicit stale-data behavior.

## Q27. What would you monitor?

**Interview-ready answer**

I would trace each run across the API, graph nodes, retrieval, model, approval, and tools using one correlation ID. Telemetry would include graph and prompt version, termination reason, retrieval IDs and scores, token usage, model latency, tool latency and result category, retry count, approval wait time, end-to-end latency, estimated cost, and final business outcome.

Dashboards should track p50, p95, and p99 latency; error and timeout rates; no-result retrieval; escalation; approval rejection; unsafe-action attempts; cost per successful resolution; MTTR; and reopen rate. Sensitive prompts, credentials, protected documents, and hidden reasoning should not be logged.

## Q28. How would you control latency and cost?

**Interview-ready answer**

I would decompose both by stage before optimizing. Common improvements are smaller context windows, a bounded candidate set, parallel independent reads, concise output schemas, prompt-prefix caching, fewer agent steps, and model routing based on evaluated task difficulty.

Each run needs token, tool, step, time, and monetary budgets. I optimize cost per successful resolution, not cost per model call. A cheaper model that increases escalations, reopens, or wrong actions can be more expensive overall.

## Q29. How would you avoid provider lock-in?

**Interview-ready answer**

I would define provider-neutral contracts for structured responses, tool calls, usage, refusals, and errors, with thin adapters around SDKs. Prompts and capabilities remain versioned because provider behavior is not identical.

An abstraction reduces code coupling but does not make models interchangeable. Every provider or fallback model must pass the same quality, safety, latency, and tool-use evaluations before receiving traffic.

## Q30. What are the main failure modes?

**Interview-ready answer**

I group them by layer: bad or incomplete input; missing, stale, unauthorized, or conflicting retrieval; unsupported model output; invalid tool selection; loops or stale state; timeout or uncertain side effects; permission failures; provider outage; and cost or capacity overload.

For each class, I want a detection signal, safe fallback, owner, and regression test. When evidence, authorization, or action state is uncertain, the default is to stop automation, preserve useful context, and escalate.

---

# 3. RAG, Embeddings, and Knowledge Architecture

## Q31. Walk me through the RAG pipeline.

**Interview-ready answer**

The offline pipeline ingested approved SOPs, parsed and normalized them, split them on procedural boundaries, attached metadata such as service, incident type, version, owner, validity, and access scope, generated embeddings using `[EMBEDDING_MODEL]`, and indexed them in `[VECTOR_DATABASE]`.

At query time, the service normalized the ticket, created a retrieval query, applied authorization and metadata filters, retrieved candidates, optionally reranked them, and assembled a context window within a token budget. The generation step had to use that evidence, cite source IDs and versions, and abstain or escalate when evidence was missing or contradictory.

## Q32. Why use a vector database?

**Interview-ready answer**

Ticket language often differs from SOP language, so dense embeddings help retrieve semantically related procedures. A vector database supports efficient nearest-neighbor search and metadata filtering as the corpus grows.

Dense retrieval is not enough for exact error codes, hostnames, or acronyms. For ITSM, I would normally evaluate hybrid dense and lexical search followed by reranking. If the corpus were small and exact-term-heavy, relational full-text search might be simpler and sufficient.

## Q33. Why RAG instead of fine-tuning?

**Interview-ready answer**

SOPs are factual, versioned, private, and change over time. RAG can use current documents, preserve citations, enforce document-level access, and update knowledge without retraining. Fine-tuning is better for stable behavior, format, style, or repeated task patterns, not for acting as a frequently changing knowledge store.

I would consider fine-tuning only after evaluation identified a persistent behavioral gap that prompting, structured output, and retrieval could not solve. I would still use RAG for current SOP facts.

## Q34. How did you choose chunk size and overlap?

**Interview-ready answer**

I would not choose one arbitrary token size. SOPs have semantic units such as prerequisites, diagnostic checks, ordered steps, rollback, and escalation criteria. I would split on those boundaries, retain the heading and parent-document context, and add limited overlap only where a procedure crosses a boundary.

I would test variants using labeled tickets and compare retrieval recall@k, ranking, grounded answer quality, latency, and token use. Chunks should be as small as possible while preserving a complete decision or procedure.

## Q35. Dense, sparse, or hybrid retrieval?

**Interview-ready answer**

Dense retrieval handles semantic paraphrases; sparse retrieval handles exact codes, product names, and rare terms. ITSM tickets commonly contain both, so hybrid retrieval is usually a strong candidate. I would merge ranks, apply metadata and authorization filters, and rerank a small candidate set.

The choice must be evidence-based. I would compare retrieval metrics by query slice and accept added latency only when it improves end-to-end resolution quality.

## Q36. Why add a reranker?

**Interview-ready answer**

The first-stage retriever is optimized for high-recall, low-latency candidate generation. A cross-encoder reranker can jointly examine the ticket and each passage and improve precision before expensive generation.

The trade-off is added latency and cost. I would rerank only a small top set and retain it if the gain in relevant evidence, groundedness, or task success justifies the p95 latency increase.

## Q37. How do you detect that no relevant SOP exists?

**Interview-ready answer**

I would use calibrated retrieval signals, reranker scores, required metadata, and evidence-coverage checks rather than a model's self-reported confidence. Thresholds should be tuned on answerable and unanswerable cases.

Below the threshold, the agent should state that evidence is insufficient, ask for a specific missing detail, or escalate. Abstention is a valid outcome and must be evaluated for both false abstentions and unsafe answers.

## Q38. How do you reduce hallucinations?

**Interview-ready answer**

I treat hallucination as several failure classes. Missing knowledge needs better data or retrieval. Unsupported generation needs evidence-linked output and abstention. Invalid actions need typed schemas, policy checks, and authorization. Stale facts need versioning and freshness controls.

Low temperature may reduce variation, but it does not guarantee truth. The strongest controls combine good evidence, constrained output, narrow tools, deterministic validation, and a safe fallback.

## Q39. The correct document was retrieved, but the answer was wrong. How do you debug it?

**Interview-ready answer**

I trace the pipeline: was the correct chunk high enough in the ranking, did context construction preserve the relevant steps, was it displaced by irrelevant text, did sources conflict, and did the model ignore or misinterpret the evidence? I inspect the exact version tuple for the run.

The fix might be context ordering, deduplication, a stronger synthesis prompt, structured evidence mapping, or a model change. I add the case to the regression set and verify the change does not damage other slices.

## Q40. The correct SOP was not retrieved. How do you debug it?

**Interview-ready answer**

I check whether the source was present, parsed correctly, current, authorized, and indexed under the expected embedding version. Then I inspect query construction, metadata filters, exact identifiers, chunk boundaries, candidate count, and ranking.

The appropriate fix could be ingestion repair, metadata correction, hybrid retrieval, query rewriting, chunking changes, or reranking. Prompt tuning is not the first fix for a retrieval miss.

## Q41. How do you handle stale or conflicting SOPs?

**Interview-ready answer**

Every chunk should include source ID, version, effective date, owner, authority, and status. Retrieval excludes retired content and prefers current authoritative sources. If active sources conflict, the system exposes the conflict and escalates instead of silently choosing the closest vector.

The ingestion pipeline needs incremental updates, tombstones, reindexing, and a measurable freshness SLO from source change to searchable index.

## Q42. How do you keep the source store and vector index consistent?

**Interview-ready answer**

The document repository is the source of truth and the vector index is a derived, versioned view. A source event or transactional outbox drives parse, chunk, embed, and upsert steps. Each chunk stores the source version, embedding version, checksum, and access metadata.

Deletes propagate as tombstones, and reconciliation jobs compare source and index manifests. Sensitive actions can revalidate source status before execution because the index is normally eventually consistent.

## Q43. What happens when the embedding model changes?

**Interview-ready answer**

Document and query embeddings must use the same model and version. I would create a parallel index, re-embed the corpus, run retrieval and end-to-end evaluations, shadow or canary traffic, and then switch an alias if quality, latency, and cost meet thresholds.

The previous index remains available for rollback. I would not mix vectors from incompatible embedding spaces.

## Q44. How do you secure RAG for multiple tenants or teams?

**Interview-ready answer**

Authorization is applied before or during retrieval, never after the model has seen content. The API derives tenant, role, team, and environment from trusted identity, and those values become mandatory retrieval filters. Returned document IDs are rechecked before context construction.

Depending on risk, I would use separate indexes or strongly enforced partitions, encrypt data, isolate caches, minimize provider data, and test cross-tenant and revoked-access cases explicitly.

## Q45. How do you protect against prompt injection in retrieved documents?

**Interview-ready answer**

Retrieved content is untrusted data even when it comes from an internal repository. It is delimited as evidence and cannot change system policy, grant permission, or directly trigger a tool. Authorization and action policy remain in deterministic code.

I would add trusted-source allowlists, provenance, ingestion scanning, narrow tools, egress controls, argument validation, and adversarial evaluation. A stronger prompt helps interpretation but is not the security boundary.

---

# 4. MCP and Tool-Assisted Resolution

## Q46. What is MCP, and how did you use it?

**Interview-ready answer**

The Model Context Protocol standardizes how an AI host discovers and invokes external tools and accesses contextual resources. In my project, MCP exposed narrow ITSM or operational capabilities such as `[MCP_TOOLS_AND_TARGET_SYSTEMS]` through typed contracts.

LangGraph selected a potential tool path, but the host validated the call before invocation. MCP reduced provider-specific integration coupling; it did not provide authorization, approval, idempotency, or audit automatically.

## Q47. Why MCP instead of direct API integrations?

**Interview-ready answer**

Direct integration is reasonable for one or two stable APIs. MCP becomes valuable when multiple tools need a consistent discovery and invocation contract, tools have independent owners, or capabilities should be reusable across AI hosts.

The cost is another protocol, compatibility surface, and trust boundary. I would use MCP only when interoperability and reuse justify that complexity, not simply because it is popular.

## Q48. Where does authorization belong?

**Interview-ready answer**

Authorization belongs at trusted application and tool-server boundaries, never in model judgment. The host determines which tools are visible and permitted for the authenticated workflow. The tool server independently verifies caller identity, tenant, target resource, action, and scopes.

User, role, tenant, and approval values must come from signed credentials or trusted state, not model arguments. Tool discovery means a capability exists; it does not mean the caller may invoke it.

## Q49. What makes a tool safe for model use?

**Interview-ready answer**

I prefer narrow domain operations with strict schemas, enums, bounds, and stable structured results. Read and write tools are separated. Each tool has a permission scope, risk classification, deadline, retry policy, audit context, and safe error categories.

I would not expose generic shell, SQL, or arbitrary HTTP tools when a purpose-built operation is possible. Mutating tools require an idempotency key, and high-impact calls require an approval reference bound to exact arguments.

## Q50. What if the model chooses the right tool with the wrong arguments?

**Interview-ready answer**

The validator rejects the call before execution. For a safe read, the graph can request a missing value or allow one bounded regeneration using the validation error. It should not silently invent or coerce a sensitive identifier.

For writes, regenerated arguments still pass authorization, policy, and approval checks. I would improve ambiguous schemas, add enums or descriptions, and add the case to tool-selection evaluation.

## Q51. How do you handle tool timeouts and failures?

**Interview-ready answer**

I classify failures as transient, permanent, or uncertain. A connection reset may be retried with bounded exponential backoff and jitter. Invalid arguments and denied permission are permanent. A timeout after dispatching a write is uncertain and requires status reconciliation before retry.

The graph receives normalized outcomes, not raw SDK exceptions. One propagated deadline prevents nested retries from exceeding the request budget.

## Q52. Can independent tools run concurrently?

**Interview-ready answer**

Yes, read-only tools with no dependency can run concurrently under a shared deadline, which reduces latency. Results remain correlated to tool-call IDs and one failure should not discard successful siblings.

Concurrency must be bounded by provider and downstream capacity. Mutating or dependent actions should not be parallelized unless ordering, consistency, rollback, and idempotency are explicitly handled.

## Q53. How do you handle a tool schema change?

**Interview-ready answer**

Tool contracts should be versioned and covered by consumer-driven contract tests. Additive optional fields may be backward compatible; changed semantics or required inputs need a new version or coordinated rollout.

Long-running runs record the tool version used for the proposal. On resume, the system invokes a compatible version or regenerates and revalidates the proposal. Old arguments must never execute silently under new semantics.

## Q54. How do you audit an AI-triggered action?

**Interview-ready answer**

I would record user and tenant identity, ticket and run IDs, model and prompt version, evidence IDs and versions, proposed action, redacted validated arguments, policy decision, approval identity and timestamp, tool version, idempotency key, result, and terminal outcome.

Audit events should be immutable and searchable by correlation ID. I would not retain credentials, unrestricted sensitive content, or hidden chain-of-thought.

## Q55. How do human approval gates work?

**Interview-ready answer**

When policy classifies a proposal as high-impact, the graph persists state and creates an approval request showing the target, action, evidence, expected impact, risk, and rollback path. An authorized reviewer can approve, reject, or modify it. The run resumes from the checkpoint.

On resume, the service verifies the approver, expiry, exact approved arguments, current policy, and current resource state. Approval supplements authorization; it never replaces it.

## Q56. Which actions require approval?

**Interview-ready answer**

Read-only diagnostics can generally run automatically. Reversible, low-blast-radius actions may become autonomous after strong evaluation. Destructive, privileged, security-sensitive, externally visible, unusual, or broad-impact actions require approval.

Policy should consider action type, environment, resource criticality, reversibility, evidence quality, and caller role. Model confidence alone should not determine autonomy.

## Q57. How do you prevent approval fatigue?

**Interview-ready answer**

I would not put every tool behind the same dialog. Approval should focus on decisions where a human can materially reduce risk. The request should concisely present action, target, reason, supporting SOP, impact, and rollback.

I would track approval volume, decision time, rejection and modification rates, and incidents by action type. Frequently approved low-risk actions can be candidates for controlled automation; frequent rejection indicates a model, retrieval, schema, or policy defect.

---

# 5. Evaluation, Testing, and Metrics

## Q58. How did you evaluate the system?

**Interview-ready answer**

For my implementation, we evaluated `[EVALUATION_DATASET_SIZE_AND_METHOD]`. For a production-grade approach, I would use an evaluation pyramid. Unit tests cover validators, state transitions, authorization, policy, retries, and tool adapters. Contract tests cover structured model output and MCP schemas. Component evaluations measure retrieval, classification, grounding, and tool selection. Trajectory evaluations assess whether the agent followed an acceptable path. End-to-end evaluations measure task success, safety, latency, cost, and escalation.

The dataset should contain de-identified real tickets, edge cases, unanswerable requests, stale or conflicting SOPs, permission failures, tool outages, and prompt-injection attempts. Every model, prompt, graph, retriever, index, and tool version must be recorded.

## Q59. How do you test a nondeterministic system?

**Interview-ready answer**

I make the shell deterministic and evaluate the probabilistic component against defined outcomes. Orchestration tests use scripted model fakes. Assertions target schemas, evidence IDs, allowed actions, policy behavior, and terminal states rather than exact prose.

For real-model evaluations, I use a versioned dataset, repeat cases where variance matters, compare slices and confidence intervals, and use calibrated human or model review for semantic qualities. Exact matching remains appropriate for IDs, enums, citations, and tool arguments.

## Q60. Which RAG metrics matter?

**Interview-ready answer**

Retrieval metrics include recall@k, precision@k, mean reciprocal rank, nDCG, and no-result rate. Generation metrics include groundedness, correctness, citation correctness and coverage, completeness, and abstention accuracy.

Business and safety metrics include resolution success, engineer acceptance and edit rate, escalation, reopen rate, incorrect-action rate, policy violations, MTTR, latency, and cost per successful resolution. I would inspect them by incident type, severity, team, document version, and tool path.

## Q61. How do you defend the MTTR reduction from 35 to 15 minutes?

**Interview-ready answer template**

We defined MTTR as `[MTTR_START_EVENT]` to `[MTTR_END_EVENT]` and compared `[SAME_SUPPORTED_TICKET_TYPES]` across `[PILOT_SIZE_AND_DURATION]`. The baseline `[MEAN_OR_MEDIAN]` was 35 minutes and the assisted workflow measured 15 minutes. The main savings came from SOP search, diagnostic collection, and routine tool execution. We also checked `[QUALITY_GUARDRAIL_METRICS]` so faster handling did not hide poorer outcomes.

The limitation was `[PILOT_LIMITATION]`, so I describe it as an observed result for that workflow rather than a universal causal claim. A broader rollout should use matched cohorts or an A/B design and report confidence intervals and severity slices.

## Q62. How do you prove quality did not fall while MTTR improved?

**Interview-ready answer**

I would pair MTTR with reopen rate, repeat incidents, escalation, approval rejection, engineer edits, incorrect actions, policy violations, and satisfaction. Results should be sliced by ticket type and severity rather than reported only as an aggregate.

Rollout would progress from shadow recommendations to human-approved actions and then to narrowly scoped automation. Each increase in autonomy requires evidence that outcome and safety thresholds remain satisfied.

## Q63. How do you build a golden evaluation dataset?

**Interview-ready answer**

I would start with representative, de-identified real tickets plus known failure cases. Domain experts define expected evidence, acceptable diagnoses and actions, prohibited actions, and when escalation is correct. I would include common, rare, ambiguous, adversarial, and unanswerable cases.

The set needs development and holdout portions, versioning, ownership, slice labels, and documented provenance. Production failures should be reviewed and added as regression cases rather than copied automatically.

## Q64. Would you use an LLM as a judge?

**Interview-ready answer**

An LLM judge can scale evaluation of relevance, completeness, and groundedness when several outputs are valid. I would give it a specific rubric and evidence, calibrate it against domain-expert labels, randomize candidate order for comparisons, and inspect disagreements.

I would not use an LLM judge for facts normal code can verify, such as schemas, citations, permissions, exact IDs, or whether an unauthorized tool ran.

## Q65. How do you regression-test a prompt change?

**Interview-ready answer**

Prompts are versioned artifacts. A change runs against the same development, holdout, safety, and trajectory suites and is compared on quality, tool behavior, refusal, latency, tokens, and cost by slice.

After offline success, I would use shadow or canary traffic and retain a rollback path. A few handpicked examples are useful for diagnosis but are not sufficient evidence for release.

## Q66. How do you evaluate tool selection?

**Interview-ready answer**

For each ticket, the dataset defines acceptable tools, required tools, forbidden tools, argument constraints, and whether no tool is correct. Metrics include tool precision and recall, argument validity, unnecessary-call rate, unsafe-call rate, and task completion.

Tool implementation is tested independently with deterministic unit and contract tests. This separation tells me whether a failure came from selection, argument generation, execution, or result interpretation.

## Q67. How do you evaluate an agent trajectory?

**Interview-ready answer**

I define invariants and acceptable path families instead of one exact sequence. Examples include retrieve before proposing an SOP-based action, never execute a write before approval, stay within step and tool budgets, preserve tool-call correlation, and terminate safely on denied permission.

A trajectory can be valid even if it uses a different safe diagnostic order. Scoring should penalize unsafe, redundant, looping, or excessively costly paths and reward completion with correct evidence.

## Q68. How do you detect model or provider drift?

**Interview-ready answer**

I continuously run a stable evaluation set and monitor production outcomes by model, prompt, graph, retriever, and tool version. Signals include changes in task success, groundedness, schema adherence, tool accuracy, refusal, tokens, latency, and cost.

Canary or shadow releases and replayable de-identified cases help identify regressions. Because a provider may change behavior behind an API, monitoring and rollback matter even without an application deployment.

## Q69. What test strategy would you use without real network calls?

**Interview-ready answer**

I would inject fakes for the model, retriever, clock, state store, and tools. Unit tests remain fast and hermetic. Scripted model outputs exercise success, malformed output, refusal, retries, approval, loops, and escalation deterministically.

Contract tests verify provider and MCP translations. A small integration suite can use local ephemeral dependencies, and carefully budgeted live-provider tests validate only the external boundary.

## Q70. What should happen after a production failure?

**Interview-ready answer**

First contain impact: disable the affected action or switch to recommendation-only mode, preserve audit evidence, identify affected runs, and reconcile external state. Then classify the failure across input, retrieval, model, policy, orchestration, approval, and tools.

The reviewed incident becomes a regression case. Re-enabling autonomy requires passing the relevant evaluation and rollout gates, not only changing a prompt until the example looks correct.

---

# 6. Security, Privacy, Compliance, and Responsible AI

## Q71. How did you prevent unsafe actions?

**Interview-ready answer**

The model did not execute arbitrary commands. It proposed an allowlisted typed action. Deterministic code validated arguments, authenticated and authorized the caller, checked tenant and resource scope, applied risk policy, required approval where necessary, and passed an idempotency key to the tool.

The tool server repeated critical authorization checks. Invalid, unauthorized, unsupported, or uncertain actions failed closed and moved to escalation.

## Q72. Why is schema validation not enough?

**Interview-ready answer**

A schema proves only that data has the expected shape. A syntactically valid action may still target the wrong customer, violate business state, exceed permission, use stale evidence, or require approval.

After schema validation, application code must enforce identity, authorization, business invariants, current state, policy, and audit requirements. Structured output is a reliability feature, not a security boundary.

## Q73. How do you handle prompt injection?

**Interview-ready answer**

I treat user input, documents, web content, and tool results as untrusted data. They are clearly delimited, but protection does not rely on prompting alone. Retrieval is access-controlled, tools are least-privilege and allowlisted, authorization is server-side, arguments are validated, risky actions need approval, and egress is restricted.

I also test direct and indirect injection cases. Prompt injection is a system threat that requires defense in depth.

## Q74. What data would you send to a hosted model?

**Interview-ready answer**

Only the minimum required data after checking classification, provider terms, retention, training use, residency, and contractual controls. I would remove secrets and unnecessary personal data, use references or pseudonymous IDs where possible, and avoid sending unrestricted logs or credentials.

For sensitive workloads, options include a managed model in the required region with appropriate privacy terms, private connectivity, or self-hosting. The decision balances risk, quality, latency, cost, and operational capability.

## Q75. How do you support privacy deletion?

**Interview-ready answer**

I would maintain lineage from a source record to chunks, embeddings, caches, traces, and approved evaluation copies. A deletion workflow removes or tombstones the source, blocks future retrieval immediately, propagates to indexes and caches, and records completion.

Backups follow a defined expiry and restore process in which deletion tombstones are replayed before restored data becomes available. Production tickets must not silently become permanent training or evaluation data.

## Q76. How do you prevent cross-tenant data leakage?

**Interview-ready answer**

Tenant identity is derived at the authenticated boundary and propagated as trusted context. It is enforced in source storage, retrieval filters, caches, state stores, tool authorization, logs, and evaluation access. The model cannot select or override a tenant.

I would test direct cross-tenant queries, guessed IDs, cache-key collisions, revoked access, batch retrieval, and administrative paths. Plausible output must not conceal a failed authorization test.

## Q77. How would you threat-model MCP?

**Interview-ready answer**

I would map assets, actors, trust boundaries, and data flows across host, client, server, target system, and identity provider. Threats include malicious tool descriptions, compromised servers, token theft, confused-deputy behavior, argument tampering, replay, schema drift, result leakage, denial of service, and supply-chain compromise.

Controls include server allowlisting, mutual identity, encrypted transport, short-lived audience-bound credentials, independent authorization, versioned deployments, schema validation, egress controls, quotas, idempotency, redaction, and immutable audit events.

## Q78. How do you prevent denial-of-wallet or runaway cost?

**Interview-ready answer**

I use per-user and per-tenant rate limits, bounded queues, model and tool concurrency limits, maximum steps, token and cost budgets, deadlines, and duplicate-action detection. Tool catalogs and context are minimized for each workflow.

Alerts monitor cost per run, token anomalies, loop rate, retry amplification, and provider spend. When capacity or cost budgets are exhausted, low-priority work is rejected or deferred and consequential automation stops safely.

## Q79. What does compliance mean here?

**Interview-ready answer**

Compliance comes from system and organizational controls, not from telling the model to comply. Relevant controls include authenticated access, least privilege, separation of read and write capabilities, approval for defined high-risk actions, data minimization, encryption, retention, audit trails, change management, incident response, and the ability to disable automation.

I would map exact controls to the applicable policy or regulation. Human approval alone is not a compliance guarantee.

## Q80. What should never be logged?

**Interview-ready answer**

Credentials, tokens, secrets, unrestricted sensitive tickets, protected document contents, unredacted tool arguments, and hidden chain-of-thought should not be logged. Logs should contain redacted operational facts, IDs, version metadata, timing, result categories, and policy decisions needed for debugging and audit.

Access control, encryption, sampling, and retention apply to AI traces just as they do to other sensitive production telemetry.

## Q81. Would you let the agent write long-term memory?

**Interview-ready answer**

Not without a defined product need and governance. Working state, approved semantic knowledge, user preferences, and episodic history have different owners and retention rules. The ticket system and approved SOP repository should remain sources of truth.

The model should not freely convert its own conclusions into trusted memory. Candidate memories require validation, provenance, access control, deduplication, deletion, and protection from poisoning.

---

# 7. AI Fundamentals Likely to Be Probed

## Q82. Explain how an LLM works.

**Interview-ready answer**

An LLM tokenizes text, maps tokens to vectors, and processes them through transformer layers using attention and feed-forward networks. During inference it produces a probability distribution for the next token, selects a token according to decoding settings, appends it, and repeats.

It learns statistical patterns rather than consulting a guaranteed source of truth. That explains why it can generalize and synthesize but also produce plausible unsupported claims.

## Q83. What are embeddings?

**Interview-ready answer**

Embeddings are dense vectors that represent semantic characteristics of text or other data. Semantically related items tend to be closer under a measure such as cosine similarity, allowing a query embedding to retrieve related document embeddings.

Similarity is not proof of factual relevance. Exact identifiers, access rules, source authority, and current validity still require lexical search, filters, metadata, reranking, and evaluation.

## Q84. What do temperature and top-p do?

**Interview-ready answer**

Temperature reshapes token probabilities: lower values favor high-probability tokens and higher values add variation. Top-p restricts sampling to a cumulative-probability set.

For classification, extraction, and tool arguments, I use low variation and schema constraints. These settings affect diversity, not truth, authorization, or safety.

## Q85. Why does more context not always improve quality?

**Interview-ready answer**

More context can add irrelevant or conflicting content, increase latency and cost, dilute important evidence, and expand data-exposure and prompt-injection surfaces. Models may also underuse information buried in the middle of a long context.

I prefer the smallest sufficient authorized evidence set and test whether added context improves task metrics.

## Q86. What is structured output, and why use it?

**Interview-ready answer**

Structured output constrains generation to a schema with named fields, types, enums, and required properties. It is stronger than asking the model to print JSON and makes integration and validation more reliable.

Application code must still validate semantic and business rules. A valid schema can contain an unauthorized or incorrect action.

## Q87. Hosted versus self-hosted models?

**Interview-ready answer**

Hosted models offer fast adoption, strong managed quality, and less infrastructure ownership. Self-hosting offers more control over data, versions, customization, and potentially economics at sustained scale, but requires GPU capacity, serving optimization, patching, and specialized operations.

I would compare both on representative quality, latency, throughput, privacy, residency, reliability, team capability, and total cost, not only token price.

## Q88. When would you fine-tune?

**Interview-ready answer**

I would fine-tune when a stable, repeated behavioral gap remains after prompting, structured output, and retrieval, and when enough high-quality training examples and evaluation data exist. Examples include consistent classification, extraction, style, or tool-selection patterns.

I would not fine-tune merely to store changing SOP facts. RAG and tools remain appropriate for current, attributable information.

## Q89. One agent or multiple agents?

**Interview-ready answer**

I prefer one orchestrator with explicit specialized steps unless separate agents have independent goals, tools, ownership, or parallel work that measurably improves outcomes. Multi-agent designs add coordination cost, latency, state complexity, and difficult debugging.

The burden of proof is empirical. If a deterministic node or simple router performs the task, that is usually the stronger production design.

---

# 8. Hiring Manager, Behavioral, and Resume Cross-Examination

## Q90. Tell me about the hardest trade-off you made.

**Interview-ready answer**

The central trade-off was autonomy versus operational risk. Full automation might reduce handling time further, but the cost of an incorrect production action is much higher than the cost of an approval step. I separated recommendations, diagnostics, low-risk reversible actions, and high-impact actions into different policy levels.

That preserved speed in evidence gathering while retaining accountable control over consequential changes. I would expand autonomy only using approval, incident, quality, and latency data.

## Q91. What would you improve if you rebuilt it?

**Interview-ready answer**

I would establish the evaluation dataset and failure taxonomy earlier, before optimizing prompts or demos. That makes retrieval, tool, and workflow decisions evidence-driven from the beginning.

I would also separate short synchronous recommendations from durable long-running action workflows earlier. The broader lesson was to design measurement and failure semantics at the same time as the happy path.

## Q92. Tell me about a failure.

**Interview-ready answer template**

In `[REAL_FAILURE_OR_LIMITATION]`, the system `[OBSERVED_FAILURE]`. I traced it to `[RETRIEVAL_MODEL_TOOL_ORCHESTRATION_CAUSE]` rather than treating it as a generic model problem. I contained the risk by `[FALLBACK_OR_CONTROL]`, changed `[SPECIFIC_COMPONENT]`, and added the case to `[TEST_OR_EVALUATION_SET]`.

The result was `[HONEST_OUTCOME]`. The lesson I carried forward was `[SPECIFIC_ENGINEERING_LEARNING]`.

## Q93. Tell me about disagreement over a design choice.

**Interview-ready answer template**

The proposed direction was `[DECISION]`, and my concern was `[SECURITY_RELIABILITY_COST_OR_DELIVERY_RISK]`. I gathered `[DATA_PROTOTYPE_OR_FAILURE_CASES]`, presented the options and trade-offs, and recommended `[YOUR_DECISION]`.

The team decided `[OUTCOME]`. I then `[SUPPORTED_OR_IMPLEMENTED_IT]`, and we measured `[RESULT]`. Keep this story factual; the approval boundary, RAG design, Java migration, Lambda modernization, or resilience work may provide a real example.

## Q94. How did you work with domain experts?

**Interview-ready answer**

Support engineers and SOP owners define what a correct resolution means, which sources are authoritative, and which actions are too risky to automate. I would involve them in selecting representative tickets, labeling evidence and outcomes, reviewing failure clusters, and defining approval policy.

Their feedback should become structured artifacts: corrected metadata, new evaluation cases, SOP ownership tasks, or policy changes. A generic thumbs-up signal is not enough to diagnose the system.

## Q95. How would you explain the project to a nontechnical leader?

**Interview-ready answer**

The assistant shortens the repetitive portion of incident resolution. It reads a support ticket, finds the approved procedure, gathers information from operational tools, and recommends the next step. Routine low-risk steps can be assisted, while sensitive actions stay under authorized human control.

We judge it on faster and more consistent resolution, but also on reopens, incorrect actions, escalation, and policy compliance. The objective is trustworthy operational improvement, not maximum autonomy.

## Q96. How does your Java backend experience help in AI engineering?

**Interview-ready answer**

AI services inherit normal distributed-systems concerns: API contracts, concurrency, retries, timeouts, authentication, authorization, state, caching, observability, testing, deployment, and incident response. My Java/Spring background gives me a strong production foundation for those concerns.

For example, my experience with WebClient, Resilience4j, Redis, PostgreSQL, OIDC, and high-scale services directly informs how I design model-provider calls, tool isolation, cache boundaries, identity propagation, and failure handling around an LLM.

## Q97. Why move from backend engineering to AI engineering?

**Interview-ready answer**

I am not moving away from backend engineering; I am applying it to a new class of systems. The model call is only one component. The hard production work is building secure, observable, testable workflows around probabilistic behavior and connecting them to real systems.

The role fits because I can contribute to both layers: AI patterns such as RAG and agent orchestration, and the software-engineering discipline needed to operate them reliably.

## Q98. Why use Python when your strongest language is Java?

**Interview-ready answer**

Python provided the strongest ecosystem and fastest iteration for LangGraph, model SDKs, retrieval, and evaluation. Java remained a good fit for established domain services. I would use versioned APIs or events between them rather than rewrite stable services only for stack uniformity.

Language choice should follow ecosystem, team capability, performance evidence, and ownership boundaries. If profiling showed a Python CPU bottleneck, I would optimize or isolate that work instead of assuming a rewrite is automatically required.

## Q99. Why are you interested in this role?

**Interview-ready answer template**

I am interested because the role combines production software engineering with applied AI. My experience has shown me that the difficult part is not calling a model API; it is building secure, measurable, and reliable systems around probabilistic behavior. The team's work on `[COMPANY_PRODUCT_OR_PROBLEM]` is compelling because `[SPECIFIC_REASON]`.

I would bring backend, cloud, identity, resilience, and performance experience together with hands-on RAG and agent orchestration. I am looking for ownership from use-case definition and evaluation through controlled production rollout.

## Q100. Why should we hire you?

**Interview-ready answer**

I bring a useful combination of production backend engineering and hands-on applied AI. I have built services for a sensitive platform serving eight million users, worked on identity, cloud modernization, resilience, caching, performance, and quality gates, and then applied that discipline to a RAG and tool-using agent.

I do not treat an LLM demo as a finished system. I think about evidence, authorization, failure recovery, evaluation, observability, and measurable outcomes from the beginning. I can contribute across both the AI workflow and the surrounding SDE architecture.

---

# 9. Rapid-Fire Follow-Ups

## RAG or fine-tuning?

RAG for current, private, citable facts; fine-tuning for stable behavior that prompting cannot achieve reliably.

## Dense or hybrid retrieval?

Hybrid for ITSM because semantic descriptions and exact error codes both matter. Prove it with retrieval and end-to-end evaluation.

## Why not put every SOP in a long prompt?

It raises cost, latency, conflict, authorization, and lost-in-the-middle risk. Retrieve the smallest sufficient evidence set.

## What temperature would you use?

Low for extraction, classification, and tool proposals, but temperature is not a factuality or security control.

## What if no SOP matches?

Abstain, preserve diagnostics, ask for a specific missing detail, or escalate. Do not manufacture a procedure.

## Can MCP secure tools?

No. MCP standardizes integration; identity, authorization, validation, approval, network controls, and audit remain application responsibilities.

## Why is human approval not enough?

Approvers can rubber-stamp unclear requests. Approval must be authorized, risk-targeted, informed, expiring, bound to exact arguments, and followed by current-state revalidation.

## How do you stop loops?

Explicit terminal states plus step, token, tool, cost, and deadline budgets; detect repeated actions and escalate when a budget is exhausted.

## How do you prevent duplicate actions?

Stable idempotency keys, persisted action state, unique constraints, desired-state operations, and reconciliation of uncertain outcomes.

## What is the safest rollout?

Offline evaluation, shadow mode, recommendation-only use, human-approved actions, then canary automation for proven low-risk operations.

## What is the most important metric?

Task success under quality and safety constraints. MTTR is valuable only alongside reopens, errors, escalation, and policy compliance.

## What is a good fallback?

A safe partial result with supporting evidence, completed diagnostics, explicit uncertainty, and a routed human escalation.

## What should the model never decide?

Identity, authorization, tenant, approval validity, business invariants, retry safety, or permission to perform a consequential side effect.

## One agent or many?

Prefer one explicit orchestrator unless multiple agents measurably improve a problem with genuinely independent roles or parallel work.

## LangGraph or Temporal?

LangGraph for bounded AI state and transitions; a durable workflow engine for long-lived execution and strict recovery semantics. They can be composed.

## Circuit breaker or retries?

Retries handle isolated transient failures. A circuit breaker stops repeated calls to an unhealthy dependency. Both require bounded budgets and careful recovery behavior.

## Mean or median MTTR?

Use the one actually measured and state it. Median is less sensitive to outliers, but never relabel the project metric after the fact.

## Model confidence?

Do not trust self-reported confidence for action policy. Use calibrated evidence quality, validation, historical slice performance, and human review.

## What is production-ready AI?

Versioned and evaluated behavior, deterministic controls, least privilege, durable state, safe side effects, observability, rollback, auditability, and owned operations.

---

# 10. Questions to Ask the Hiring Manager

Choose two or three that fit the conversation:

1. How does the team measure AI feature success beyond model-level benchmarks?
2. Which decisions are model-driven today, and which remain deterministic?
3. How does the team convert production failures into evaluation cases?
4. What autonomy is currently allowed, and how was that boundary chosen?
5. Who owns prompts, retrieval, model changes, and operational reliability after launch?
6. How do domain experts participate in evaluation and policy design?
7. What are the hardest reliability or security problems the team has faced with RAG or tools?
8. How does the team manage model-provider changes and fallback behavior?
9. What would success for this role look like in the first 90 days?
10. What distinguishes engineers who earn broader technical ownership on this team?

---

# 11. Final Pre-Interview Checklist

Be able to say these without notes:

1. A 90-second project story with problem, ownership, architecture, controls, and outcome.
2. The actual LLM, embedding model, vector database, MCP tools, and deployment environment.
3. The exact graph nodes and which decisions were probabilistic versus deterministic.
4. One real technical decision and the rejected alternative.
5. One real failure or limitation and what you learned.
6. The exact actions that required approval.
7. How retrieval and generation were evaluated separately.
8. How identity and authorization remained outside model control.
9. The evidence behind the 35-to-15-minute MTTR result.
10. Your exact personal contribution versus team and framework contributions.

Use this closing summary when asked for the main lesson:

> The project taught me to treat the LLM as a probabilistic component rather than the application. The production value came from combining retrieval and tool use with deterministic validation, least-privilege authorization, human control over high-risk actions, observability, and measurable business outcomes.