# Async Support Assistant POC - Project Brief

## Instructions to the Implementing Agent

Build the project described below end to end. Treat this document as the complete implementation contract. Do not add features outside the stated scope. Prefer the provider SDK's current documented APIs for tool calling and schema-constrained output; isolate provider-specific code so API-version changes do not affect tool execution or evaluation logic.

Use test-driven development through the deterministic evaluation checks described here: implement the local models, validation, retries, concurrency, and scripted fake path first; then connect the real model provider at the boundary. Do not require a real API call to verify local tool behavior.

## 1. Objective

Create a small command-line customer support assistant that:

1. Accepts a customer message.
2. Gives a real LLM access to three read-only tools.
3. Allows zero or more tool calls in the model's first response.
4. Validates every requested tool and its arguments in Python.
5. Executes independent tool calls concurrently with bounded retries.
6. Returns one output for every tool call, including failures.
7. Gives those results back to the model.
8. Allows no second tool-calling round.
9. Produces a schema-constrained final response.
10. Validates that response with Pydantic.
11. Runs five evaluation cases and prints a scorecard.

The exercise should take roughly 2-3 hours and demonstrate the mechanics beneath agent frameworks without using one.

## 2. Required Technology

- Python 3.11 or newer, so `enum.StrEnum` is available.
- `asyncio` from the standard library.
- Pydantic 2.x.
- One model-provider Python SDK that supports:
  - multiple tool calls in one model turn;
  - correlation identifiers for tool calls;
  - schema-constrained structured output;
  - asynchronous requests, either directly or through the SDK's async client.
- Environment variables for secrets and configuration.

Preferred provider: OpenAI, using the current official SDK and its current Responses API or equivalent supported API. If a different provider is selected, retain all provider-neutral contracts in this document.

Do not pin an obsolete SDK API copied from an old example. Consult the installed SDK or current provider documentation for exact request and response field names.

## 3. Strict Scope

Implement only:

- One command-line entry point.
- One real provider client.
- One optional scripted fake provider boundary for deterministic local development.
- Three fake asynchronous read-only tools with hard-coded data.
- One bounded async retry helper.
- One concurrent tool dispatcher.
- Pydantic models for tool arguments, tool calls, tool results, and the final response.
- Five evaluation cases and deterministic scoring.

Do not implement:

- RAG, embeddings, document ingestion, or a vector database.
- LangChain, LangGraph, MCP, semantic kernels, or multiple agents.
- FastAPI, a browser UI, a database, persistence, or authentication.
- Real refunds, writes, approvals, or other side effects.
- OpenTelemetry, dashboards, model routing, queues, or deployment.
- A general-purpose agent loop.
- Automatic JSON repair.
- More than one tool-calling round.

## 4. Project Shape

Create exactly this minimal application shape:

```text
async-support-assistant/
  app.py
  evals.py
```

A dependency manifest may be added only if the project environment needs one to install the provider SDK and Pydantic. Do not create a package hierarchy, web server, database, or framework configuration.

Responsibilities:

```text
app.py
  configuration
  Pydantic models
  domain exceptions
  fake async tools
  argument validation
  retry helper
  tool registry and dispatcher
  provider boundary
  orchestration
  command-line entry point

evals.py
  evaluation case model
  five cases
  deterministic checks
  concurrency check
  scorecard output
```

## 5. Environment Configuration

Read configuration from environment variables:

```text
OPENAI_API_KEY       required for real-provider mode
OPENAI_MODEL         model name; no secret or model name hard-coded
SUPPORT_MODEL_MODE   real or fake; default real
```

If another provider is used, replace the `OPENAI_*` names with that provider's conventional environment variables and document the substitution in code-level help text.

Rules:

- Never log an API key.
- Fail clearly at startup when real mode is selected and required configuration is absent.
- Fake mode must not require network access or an API key.
- Do not load secrets from source-controlled constants.

## 6. User Story

The primary demonstration message is:

```text
My order ORD-123 is late and the status page says payments are degraded.
Can you check both and tell me what to do?
```

The first model response should request these two tools in the same turn:

```text
get_order_status(order_id="ORD-123")
get_service_status(service="payments")
```

Python must execute them concurrently. Their normal sleeps are 250 ms and 200 ms, so execution should be close to the slower call plus modest overhead, not the 450 ms sequential sum.

## 7. End-to-End Flow

```text
customer message
  -> first provider call with read-only tool definitions
  -> extract all requested tool calls
  -> validate names and arguments in Python
  -> run calls concurrently with bounded retries
  -> normalize every success or failure into ToolResult
  -> second provider call with correlated tool outputs
  -> reject any second-round tool request
  -> parse schema-constrained final output
  -> validate again with Pydantic
  -> return SupportResponse
```

When the first response contains no tool calls, request the final structured answer without executing tools. This final structured generation is still the terminal provider interaction.

## 8. Data Models

Use strict validation where practical. Do not silently coerce malformed tool arguments into valid values.

### 8.1 Final Response

```python
from enum import StrEnum
from pydantic import BaseModel, ConfigDict, Field


class IssueType(StrEnum):
    ORDER = "order"
    PAYMENT = "payment"
    REFUND = "refund"
    GENERAL = "general"


class SupportResponse(BaseModel):
    model_config = ConfigDict(extra="forbid")

    issue_type: IssueType
    summary: str = Field(min_length=1, max_length=300)
    recommended_action: str = Field(min_length=1, max_length=300)
    tools_used: list[str]
    needs_human: bool
```

`tools_used` must contain the unique names of tools actually attempted during the request. Preserve first-use order. A failed attempted tool still counts as used.

### 8.2 Tool Result

```python
class ToolResult(BaseModel):
    model_config = ConfigDict(extra="forbid")

    call_id: str = Field(min_length=1)
    tool_name: str = Field(min_length=1)
    success: bool
    data: dict[str, object] | None = None
    error: str | None = None
```

Enforce this invariant with a model validator:

- On success, `data` is present and `error` is `None`.
- On failure, `data` is `None` and `error` is a non-empty safe message.

Do not expose stack traces, SDK internals, credentials, or sensitive input in `error`.

### 8.3 Provider-Neutral Tool Call

Normalize provider output immediately:

```python
class ToolCall(BaseModel):
    model_config = ConfigDict(extra="forbid")

    call_id: str = Field(min_length=1)
    tool_name: str = Field(min_length=1)
    arguments: dict[str, object]
```

No downstream tool code should depend on provider response classes.

### 8.4 Argument Models

```python
class OrderStatusArgs(BaseModel):
    model_config = ConfigDict(extra="forbid", strict=True)
    order_id: str = Field(pattern=r"^ORD-[A-Z0-9]+$")


class ServiceStatusArgs(BaseModel):
    model_config = ConfigDict(extra="forbid", strict=True)
    service: str = Field(min_length=1, max_length=50)


class RefundPolicyArgs(BaseModel):
    model_config = ConfigDict(extra="forbid", strict=True)
    order_id: str = Field(pattern=r"^ORD-[A-Z0-9]+$")
```

Normalize `service` to lowercase only after validation. Supported services are `payments`, `checkout`, and `shipping`; any other value is a permanent invalid-argument failure.

## 9. Domain Exceptions

Define explicit categories:

```python
class TransientToolError(Exception):
    pass


class PermanentToolError(Exception):
    pass


class ModelRefusalError(Exception):
    pass


class StructuredOutputError(Exception):
    pass


class ProviderRequestError(Exception):
    pass
```

Unknown tools and invalid arguments are permanent failures. Provider timeout and rate-limit exceptions are retryable at the provider boundary. Convert provider-specific exceptions into provider-neutral categories as close to the SDK as possible.

Preserve the original exception as the cause when raising a final wrapper:

```python
raise ProviderRequestError("Provider request failed after 2 attempts") from original_error
```

## 10. Fake Async Tools

All tools are read-only, local, deterministic apart from explicitly controlled failure scenarios, and genuinely awaitable.

### 10.1 Hard-Coded Data

Use these records:

```python
ORDER_STATUSES = {
    "ORD-123": "delayed",
    "ORD-456": "shipped",
    "ORD-RETRY": "delayed",
    "ORD-FAIL": "unavailable",
}

SERVICE_STATUSES = {
    "payments": "degraded",
    "checkout": "operational",
    "shipping": "operational",
}

REFUND_ELIGIBILITY = {
    "ORD-123": "eligible",
    "ORD-456": "not_eligible",
    "ORD-RETRY": "eligible",
    "ORD-FAIL": "unknown",
}
```

### 10.2 Tool Contracts

```python
async def get_order_status(order_id: str) -> dict[str, object]:
    """Return the known shipping status for one order."""


async def get_service_status(service: str) -> dict[str, object]:
    """Return current health for one supported service."""


async def get_refund_policy(order_id: str) -> dict[str, object]:
    """Return refund eligibility for one order."""
```

Normal sleep durations:

- `get_order_status`: 0.25 seconds.
- `get_service_status`: 0.20 seconds.
- `get_refund_policy`: 0.15 seconds.

Return shapes:

```python
{"order_id": "ORD-123", "status": "delayed"}
{"service": "payments", "status": "degraded"}
{"order_id": "ORD-123", "eligibility": "eligible"}
```

Failure behavior:

- Unknown order IDs raise `PermanentToolError`.
- Unsupported services raise `PermanentToolError`.
- `get_order_status("ORD-RETRY")` fails with `TransientToolError` on its first attempt within one logical request, then succeeds on the second.
- `get_order_status("ORD-FAIL")` raises `TransientToolError` on every attempt, producing an exhausted typed failure after two attempts.

Do not implement failure state as an unbounded process-global counter. Pass or create request-scoped attempt state so repeated evaluation cases remain isolated and deterministic.

## 11. Tool Registry and Validation

Use an allowlisted registry. Never resolve a function from arbitrary model text through `globals()`, `getattr()`, `eval`, or dynamic import.

A suitable registry entry associates:

- tool name;
- argument model;
- async callable;
- provider-facing description and JSON Schema.

Conceptually:

```python
TOOLS = {
    "get_order_status": ToolDefinition(...),
    "get_service_status": ToolDefinition(...),
    "get_refund_policy": ToolDefinition(...),
}
```

Validation order:

1. Confirm the requested name exists in `TOOLS`.
2. Parse arguments into the registered Pydantic argument model.
3. Apply domain validation such as supported service names.
4. Call only the registered function with validated values.

Every provider-facing tool schema must set `additionalProperties` to `false`, require all mandatory fields, and describe when the tool should be used.

## 12. Retry Policy

Implement one reusable async retry helper.

Rules:

- Maximum two attempts total, not two retries.
- Retry only explicitly classified transient exceptions.
- Do not retry Pydantic validation errors, unknown tools, model refusals, `PermanentToolError`, or contract violations.
- Use deterministic exponential backoff for this POC:
  - no delay before attempt 1;
  - 0.10 seconds before attempt 2.
- Print or log the operation name, attempt number, and safe failure category.
- Preserve the final original exception as the cause.
- The helper must accept an async callable rather than containing tool-specific logic.

Conceptual signature:

```python
async def with_retry(
    operation_name: str,
    operation: Callable[[], Awaitable[T]],
    retryable: tuple[type[BaseException], ...],
    max_attempts: int = 2,
) -> T:
    ...
```

Use the same policy concept at both boundaries:

- Tools retry `TransientToolError`.
- Provider calls retry only the provider's timeout and rate-limit categories.

A production design would additionally use deadlines, jitter, retry budgets, idempotency checks, and cancellation propagation. Explain these in an interview, but do not add them to this POC.

## 13. Concurrent Dispatcher

Execute all first-round tool calls concurrently:

```python
async def execute_tool_calls(calls: list[ToolCall]) -> list[ToolResult]:
    tasks = [execute_one(call) for call in calls]
    raw_results = await asyncio.gather(*tasks, return_exceptions=True)
    return [
        normalize_result(call, raw_result)
        for call, raw_result in zip(calls, raw_results, strict=True)
    ]
```

Required behavior:

- Preserve input order in returned results.
- Preserve every provider `call_id` exactly.
- Produce exactly one `ToolResult` for every `ToolCall`.
- A failure in one call must not cancel or discard sibling results.
- Convert invalid names, invalid arguments, permanent failures, and exhausted transient failures into failed `ToolResult` values.
- Do not let expected per-tool failures escape from the dispatcher.
- Let cancellation propagate rather than converting `asyncio.CancelledError` into a tool result.

No semaphore is required for three local tools.

## 14. Provider Boundary

Keep provider-specific behavior in small functions or a small client adapter within `app.py`.

It must support:

```python
async def request_tool_calls(message: str) -> list[ToolCall]:
    ...


async def generate_structured_response(
    message: str,
    tool_results: list[ToolResult],
    tools_attempted: list[str],
) -> SupportResponse:
    ...
```

The exact signatures may include opaque provider conversation state if required by the selected API, but do not leak provider response types into the dispatcher or evaluation code.

### 14.1 First Call Instructions

The first model call must:

- Receive the customer message as untrusted data.
- Receive all three tool definitions.
- Request every independent read needed to answer the message in the same turn.
- Never invent tool results.
- Avoid tools for greetings or questions that do not require account or service data.
- Not request writes or unsupported actions.

Descriptions should make selection behavior clear:

```text
get_order_status:
Use when the customer asks about shipping, delivery, lateness, or the current
status of a specific order ID.

get_service_status:
Use when the customer asks whether payments, checkout, or shipping service is
operational, degraded, or unavailable.

get_refund_policy:
Use when the customer asks whether a specific order is eligible for a refund.
```

### 14.2 Second Call Instructions

The terminal structured-output call must:

- Receive the original customer message.
- Receive one correlated result for each tool request.
- Treat failed tool results as unavailable evidence rather than facts.
- Set `needs_human=True` when a required tool remains unavailable or evidence is insufficient.
- List only actually attempted tool names in `tools_used`.
- Produce concise, useful fields within schema limits.
- Never request another tool.

If the provider returns a tool request during this terminal call, raise `StructuredOutputError` because a second tool round violates the POC contract.

### 14.3 Structured Output Handling

Use the provider's schema-constrained feature, ideally by supplying the Pydantic-derived JSON Schema or the SDK's typed parsing facility. Merely telling the model to "return JSON" is not sufficient.

Even after provider parsing succeeds:

1. Convert the response to plain data if necessary.
2. Run `SupportResponse.model_validate(...)`.
3. Reject extra fields and invalid lengths.

Explicitly handle:

- valid structured output;
- provider refusal;
- missing or incomplete structured output;
- Pydantic validation failure;
- provider timeout;
- provider rate limit;
- other provider errors after classification.

Do not repair malformed JSON. Retry one time only when the failure is classified as provider timeout or rate limit. A refusal or schema-invalid output is a terminal, clearly reported failure.

## 15. Orchestration

Keep orchestration bounded and easy to read:

```python
async def answer_customer(message: str) -> SupportResponse:
    if not message.strip():
        raise ValueError("Customer message must not be empty")

    first_response = await call_model_with_tools(message)
    tool_calls = extract_tool_calls(first_response)

    if not tool_calls:
        return await generate_structured_response(message, [], [])

    tool_results = await execute_tool_calls(tool_calls)
    tools_attempted = unique_tool_names_in_order(tool_calls)
    return await generate_structured_response(
        message,
        tool_results,
        tools_attempted,
    )
```

This pseudocode may be adapted to preserve provider conversation state.

Responsibilities that must remain deterministic Python behavior:

- Non-empty input validation.
- Tool allowlisting.
- Argument validation.
- Retry classification and attempt limits.
- Concurrent dispatch.
- Tool-call/result correlation.
- Tool failure normalization.
- Unique `tools_used` derivation.
- Pydantic final validation.
- One-round termination.

Do not implement a `while` loop.

## 16. Command-Line Interface

Running `app.py` should support either:

```powershell
python app.py "My order ORD-123 is late. Is payments degraded?"
```

or, when no positional message is supplied, one interactive prompt from standard input.

CLI behavior:

- Print the validated final response as indented JSON.
- Exit with code 0 on success.
- Exit nonzero with a concise safe message for configuration, provider, refusal, or structured-output failures.
- Do not print stack traces by default for expected failures.
- Include enough attempt output to demonstrate retry behavior.

Example successful shape:

```json
{
  "issue_type": "order",
  "summary": "Order ORD-123 is delayed and payments are degraded.",
  "recommended_action": "Wait for the payment service to recover and contact support if the order status does not update.",
  "tools_used": [
    "get_order_status",
    "get_service_status"
  ],
  "needs_human": false
}
```

Exact wording is nondeterministic and is not part of the contract.

## 17. Scripted Fake Provider Mode

Provide a minimal scripted fake mode if needed to complete and verify deterministic mechanics before real-provider integration. It must use the same normalized `ToolCall` and `SupportResponse` boundaries.

The fake should deterministically recognize only the five evaluation messages in Section 18 and return their prescribed tool calls and final fields. It is not a general natural-language parser and must not replace the real provider in the definition of done.

Purpose:

- Test dispatch without network access.
- Verify retry behavior.
- Verify partial failure normalization.
- Verify scorecard formatting.
- Demonstrate concurrency reliably.

## 18. Evaluation Framework

Do not install an evaluation framework. Implement a small runner in `evals.py`.

### 18.1 Case Model

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class EvalCase:
    name: str
    message: str
    expected_issue_type: str
    required_tools: frozenset[str]
    expect_human: bool
```

### 18.2 Required Cases

Use exactly these semantic cases:

| Name | Message intent | Expected issue | Exact tools expected | Human |
|---|---|---|---|---|
| Late order | Check delayed `ORD-123` | `order` | `get_order_status` | `False` |
| Order plus outage | Check `ORD-123` and `payments` together | `order` | `get_order_status`, `get_service_status` | `False` |
| Refund request | Check refund eligibility for `ORD-123` | `refund` | `get_refund_policy` | `False` |
| General greeting | Greet the assistant without asking for account information | `general` | none | `False` |
| Tool unavailable | Check status of `ORD-FAIL` | `order` | `get_order_status` | `True` |

Use clear messages containing the exact identifiers so failures are attributable to model behavior rather than ambiguity.

Recommended wording:

```python
CASES = [
    EvalCase(
        name="Late order",
        message="My order ORD-123 is late. Please check its status.",
        expected_issue_type="order",
        required_tools=frozenset({"get_order_status"}),
        expect_human=False,
    ),
    EvalCase(
        name="Order plus outage",
        message=(
            "My order ORD-123 is late and the status page says payments are "
            "degraded. Check both and tell me what to do."
        ),
        expected_issue_type="order",
        required_tools=frozenset(
            {"get_order_status", "get_service_status"}
        ),
        expect_human=False,
    ),
    EvalCase(
        name="Refund request",
        message="Is order ORD-123 eligible for a refund?",
        expected_issue_type="refund",
        required_tools=frozenset({"get_refund_policy"}),
        expect_human=False,
    ),
    EvalCase(
        name="General greeting",
        message="Hello, I hope you are doing well.",
        expected_issue_type="general",
        required_tools=frozenset(),
        expect_human=False,
    ),
    EvalCase(
        name="Tool unavailable",
        message="Please check the current status of order ORD-FAIL.",
        expected_issue_type="order",
        required_tools=frozenset({"get_order_status"}),
        expect_human=True,
    ),
]
```

### 18.3 Observability for Evaluation

`answer_customer` must make the actual attempted tool names and elapsed tool-dispatch time observable to `evals.py` without scraping console output. Use a small returned run record or an optional diagnostics object while keeping the CLI's user-facing result as `SupportResponse`.

One acceptable internal model is:

```python
class SupportRun(BaseModel):
    response: SupportResponse
    tool_results: list[ToolResult]
    tool_elapsed_seconds: float
```

The public orchestration may return `SupportRun`, with the CLI printing `run.response`, or expose a diagnostic orchestration function used by `answer_customer`. Keep the design small and avoid global mutable telemetry.

### 18.4 Four Checks Per Case

Score four binary checks:

1. `schema_valid`: final output is a valid `SupportResponse`.
2. `issue_correct`: `issue_type` equals the expected value.
3. `tools_correct`: the set of actual attempted tools equals the expected set; missing and extra tools both fail.
4. `escalation_correct`: `needs_human` equals the expected value.

```python
case_score = sum(
    [schema_valid, issue_correct, tools_correct, escalation_correct]
)
```

A semantic mismatch is an evaluation failure, not a process crash. Provider errors, malformed outputs, and unexpected exceptions must be reported separately as run errors.

### 18.5 Concurrency Check

Add one deterministic check using direct tool dispatch, preferably in fake mode:

- Dispatch `get_order_status("ORD-123")` and `get_service_status("payments")` together.
- Sequential nominal sleep is 0.45 seconds.
- Assert only that elapsed time is materially below sequential execution.
- Use a tolerant threshold such as less than 0.40 seconds under normal local conditions, or compare measured concurrent time against a separately measured sequential run.
- Print both elapsed values.

Do not assert an exact millisecond value. The check should demonstrate concurrency without becoming fragile under ordinary scheduling noise.

### 18.6 Scorecard

Print one row per case with:

```text
case name
score out of 4
schema result
issue result
actual tools
expected tools
escalation result
latency
run error, if any
```

Print totals:

```text
checks passed
checks attempted
pass percentage
average latency
average tool-call count
concurrency check result
```

The normal denominator is $5 \times 4 = 20$ semantic checks. Clearly distinguish cases that could not be scored because of provider or program errors.

## 19. Required Deterministic Checks Before Real Calls

Before spending provider tokens, verify locally that:

1. Each argument model rejects extra or malformed fields.
2. Unknown tools produce a failed `ToolResult` and are never executed.
3. `ORD-RETRY` succeeds on attempt 2.
4. `ORD-FAIL` exhausts two attempts and becomes a failed `ToolResult`.
5. One failed call does not cancel a successful sibling call.
6. Output result order and `call_id` values match input calls.
7. `ToolResult` rejects inconsistent success/data/error combinations.
8. The two demonstration tools execute concurrently.
9. Empty customer input is rejected.
10. A simulated second-round tool request is rejected.

These checks may live as small deterministic functions invoked by `evals.py`; do not add a test framework solely for this exercise.

## 20. Implementation Order

Use this sequence to keep failures local and diagnosable:

1. Define Pydantic models and domain exceptions.
2. Define argument models and the tool registry.
3. Implement hard-coded async tools.
4. Implement and verify the retry helper.
5. Implement one-call validation and execution.
6. Implement the concurrent dispatcher and timing check.
7. Implement the scripted fake provider boundary.
8. Implement orchestration and the five-case scorecard in fake mode.
9. Add the real provider adapter using current SDK APIs.
10. Run one real tool call, then the two-tool example.
11. Run all five real-provider evaluations.
12. Stop when the definition of done is met.

## 21. Definition of Done

The POC is complete only when all of the following are demonstrated:

- A real model produces a Pydantic-validated `SupportResponse` through schema-constrained output.
- The `ORD-123` plus payments prompt produces both required tool calls in the same model turn.
- Those calls run concurrently.
- `ORD-RETRY` fails once and succeeds on its second attempt.
- `ORD-FAIL` exhausts retries and becomes a typed failed `ToolResult`.
- A failed call does not crash or cancel sibling calls.
- Every tool output preserves the provider's original `call_id`.
- Invalid arguments and unknown tools are rejected by deterministic Python code.
- A second tool-calling round is rejected.
- `evals.py` runs five cases and prints the requested scorecard.
- Real mode obtains secrets and the model name only from environment variables.
- Fake mode runs without network access.

Do not expand the project after these conditions pass.

## 22. Expected Demonstration Commands

PowerShell examples:

```powershell
$env:SUPPORT_MODEL_MODE = "fake"
python evals.py

$env:SUPPORT_MODEL_MODE = "real"
$env:OPENAI_MODEL = "<supported-model-name>"
# Set OPENAI_API_KEY securely in the environment; never place it in source.
python app.py "My order ORD-123 is late and payments are degraded. Check both."
python evals.py
```

Dependency installation should contain only the selected provider SDK and Pydantic, for example:

```powershell
python -m pip install openai pydantic
```

Use the environment's secure secret-entry mechanism. Do not include a real key in documentation, source, shell history, or output.

## 23. Failure Semantics

Use these expected outcomes:

| Failure | Retry? | Result |
|---|---:|---|
| Transient fake tool error | Yes, once | Success or failed `ToolResult` after exhaustion |
| Permanent fake tool error | No | Failed `ToolResult` |
| Unknown tool | No | Failed `ToolResult` |
| Invalid tool arguments | No | Failed `ToolResult` |
| Provider timeout | Yes, once | Final provider error after exhaustion |
| Provider rate limit | Yes, once | Final provider error after exhaustion |
| Provider refusal | No | `ModelRefusalError` |
| Invalid structured output | No | `StructuredOutputError` |
| Second-round tool request | No | `StructuredOutputError` |
| Empty user message | No | `ValueError` before provider call |

The terminal response may still be valid when a tool fails. When a tool needed to answer the question remains unavailable, the model should summarize the limitation, recommend human support or retrying later, and set `needs_human=True`.

## 24. Interview Discussion Points

After implementation, be ready to explain:

1. Why provider-enforced schema output is stronger than prompting for JSON.
2. Why Pydantic validation is still necessary after provider schema enforcement.
3. Why names and arguments are validated outside the model.
4. Why the tool registry is an allowlist rather than dynamic dispatch.
5. Why maximum two attempts means one retry, not two.
6. Why permanent failures, refusals, and invalid arguments are not retried.
7. How retries can amplify overload without deadlines, jitter, and retry budgets.
8. Why `asyncio.gather(..., return_exceptions=True)` fits independent read tools.
9. How `TaskGroup` would differ by cancelling siblings after an unhandled task failure.
10. Why preserving `call_id` is necessary for correlation.
11. Why failed calls still require tool outputs.
12. Why one tool round makes termination deterministic.
13. Why a fake provider is useful but does not satisfy real-integration acceptance.
14. What the five-case evaluation fails to measure, including semantic nuance, security, cost variance, provider drift, and representative production traffic.
15. What production would add: end-to-end deadlines, jitter, distributed rate limits, idempotency for writes, tracing with redaction, authorization, durable execution, larger versioned evaluations, and rollout controls.

## 25. Final Constraints for the Implementing Agent

- Keep the implementation readable enough to explain line by line in an interview.
- Use clear names and small functions.
- Keep provider code separate from deterministic domain behavior.
- Do not suppress exceptions broadly.
- Do not catch `BaseException`.
- Do not swallow cancellation.
- Do not use mutable default arguments.
- Do not use global mutable attempt counters.
- Do not use arbitrary dynamic function execution.
- Do not add a framework to solve a two-file exercise.
- Report any provider SDK limitation that prevents exact compliance instead of silently weakening the contract.
