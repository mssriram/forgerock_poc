# Simple RAG Learning POC - Project Brief

## 1. Objective

Build a small command-line retrieval-augmented generation (RAG) application that answers questions over a reproducible subset of the SQuAD v2 dataset.

The POC must teach and demonstrate:

1. Preparing a document corpus and a labeled evaluation set.
2. Comparing two chunking strategies.
3. Creating local vector embeddings.
4. Storing chunks with useful metadata.
5. Running similarity search with optional metadata filters.
6. Building a grounded prompt from retrieved chunks.
7. Generating an answer with citations or abstaining when evidence is insufficient.
8. Measuring retrieval quality, answer quality, citation quality, abstention behavior, and latency.
9. Using an LLM as a secondary judge without making it the only source of truth.

The implementation must fit within a focused 5-6 hour build. It is a learning POC, not a production RAG platform.

## 2. Key Decisions

### 2.1 Local retrieval with Gemini generation

Run dataset preparation, chunking, embeddings, vector storage, and retrieval locally. Use the Gemini API free tier for answer generation and LLM judging.

Gemini is a proprietary hosted service, not an open-source model. It is included because the free tier satisfies the no-cost goal, but it has quotas, rate limits, account requirements, and terms that can change. The POC must not claim unlimited free access. Search and deterministic retrieval evaluation can run after local assets are cached; answer generation and judging require internet access, a valid Gemini API key, and available free-tier quota.

### 2.2 Dataset

Use a deterministic subset of **SQuAD v2** from Hugging Face Datasets.

SQuAD v2 is suitable because it provides:

- source article titles and passages;
- questions with reference answers;
- answer character offsets for identifying the correct chunk;
- unanswerable questions for testing abstention;
- a free, openly licensed dataset suitable for a learning exercise.

Use the validation split so the POC does not imply that it trains or fine-tunes a model. RAG ingestion is indexing, not model training.

### 2.3 Evaluation approach

Implement a small custom evaluation runner instead of adopting RAGAS or another evaluation framework. This keeps the metric calculations visible, avoids extra services, and fits the timebox.

Evaluation has two layers:

- **Primary deterministic metrics:** document/chunk retrieval, exact match, token F1, citation validity, abstention accuracy, and latency.
- **Secondary Gemini judge:** correctness, groundedness, and relevance using a schema-constrained score. Judge scores are reported separately because they are model-dependent and not perfectly reproducible.

## 3. No-Cost and Open Technology Stack

| Component | Choice | License/cost model | Purpose |
|---|---|---|---|
| Language | Python 3.11+ | PSF License | Application, ingestion, and evaluation |
| Dataset loader | `datasets` | Apache 2.0 | Download and cache SQuAD v2 |
| Dataset | `rajpurkar/squad_v2` | CC BY-SA 4.0 | Labeled corpus, answers, offsets, and unanswerable cases |
| Embeddings | `sentence-transformers/all-MiniLM-L6-v2` | Apache 2.0 | Small 384-dimensional local embedding model |
| Vector store | ChromaDB in persistent local mode | Apache 2.0 | Vector storage, metadata, and filtering |
| Provider SDK | `google-genai` | Open-source SDK; Gemini service terms apply | Gemini API integration |
| Generator and judge | Configurable Gemini Flash model; Flash-Lite fallback | Proprietary free tier with quotas | Grounded generation and secondary judging |
| Validation | Pydantic 2.x | MIT | Strict answer and judge-output schemas |
| Unit tests | pytest | MIT | Fast deterministic software tests |
| Metrics | Custom Python | Project code | Transparent retrieval and answer evaluation |

The local pipeline dependencies are open source. SQuAD content requires attribution and compliance with CC BY-SA 4.0. Gemini is the deliberate proprietary exception: it is used only through its no-cost free tier. Verify the current Gemini model availability, free-tier limits, data-use terms, and SDK license before implementation because hosted-service terms can change.

Do not use LangChain, LlamaIndex, a paid embedding endpoint, a paid model tier, or a hosted vector database for this POC. Implementing the small pipeline directly makes each RAG stage easier to inspect.

## 4. Hardware and Runtime Assumptions

Recommended local environment:

- Windows, macOS, or Linux;
- target development machine: 32 GB RAM;
- no GPU requirement;
- approximately 2 GB of free disk space for packages, dataset cache, the embedding model, and indexes;
- an internet connection and Google AI Studio account for generation and judging;
- enough Gemini free-tier quota for approximately 40 model calls in the completion run.

The 32 GB capacity is ample for the selected corpus, local Chroma index, and `all-MiniLM-L6-v2`. Retain MiniLM as the baseline because the POC is intended to expose chunking and retrieval behavior quickly; a larger embedding model would add runtime without teaching a new RAG stage. Gemini runs remotely, so local RAM does not increase generation-model capacity. Prefer a currently available free-tier Gemini Flash model for answer quality and use Flash-Lite only when Flash is unavailable or its quota is unsuitable.

Retrieval evaluation should be fast on CPU. Full answer generation and LLM judging depend on network and provider load, so generation latency is measured and reported but does not receive a strict pass threshold.

## 5. Strict Scope

Implement only:

- one dataset preparation command;
- two chunking strategies;
- one local embedding model;
- one local Chroma vector store;
- metadata storage and exact-match metadata filtering;
- top-k dense retrieval;
- one Gemini answer-generation model;
- one CLI query command;
- deterministic unit tests;
- one custom evaluation runner;
- one optional Gemini LLM-as-judge pass;
- JSON and Markdown evaluation reports.

Do not implement:

- model fine-tuning or training;
- hybrid BM25 plus vector search;
- reranking or query rewriting;
- agent loops or tool calling;
- multi-query retrieval;
- document upload or a web UI;
- authentication, persistence beyond the local index, or deployment;
- OCR, PDF parsing, crawling, or arbitrary user documents;
- hosted APIs other than the Gemini free tier or other cloud infrastructure;
- conversation memory;
- automatic prompt optimization;
- a production observability stack.

These are valid follow-up experiments but are outside the 5-6 hour POC.

## 6. Project Shape

Create the following small project:

```text
rag-learning-poc/
  app.py
  rag.py
  prepare_data.py
  evals.py
  requirements.txt
  README.md
  .gitignore
  tests/
    test_rag.py
  data/
    .gitkeep
```

Responsibilities:

```text
app.py
  command-line parsing
  query command
  human-readable and JSON output

rag.py
  Pydantic models
  chunking
  embedding and Chroma access
  metadata-filtered retrieval
  prompt construction
  Gemini generation and judge boundary
  answer orchestration
  metric helper functions

prepare_data.py
  deterministic SQuAD v2 selection
  corpus and evaluation JSONL creation
  chunking and indexing commands

evals.py
  retrieval evaluation
  answer evaluation
  LLM judge evaluation
  scorecards and report output

tests/test_rag.py
  hermetic unit tests with fake embeddings/model responses
```

Generated datasets, model caches, Chroma indexes, and evaluation reports must not be committed.

## 7. Configuration

Read configuration from environment variables with CLI flags allowed to override non-secret settings:

```text
RAG_EMBEDDING_MODEL   default: sentence-transformers/all-MiniLM-L6-v2
GEMINI_API_KEY        required for answer generation and judging
GEMINI_MODEL          required; prefer a currently available free-tier Flash model
GEMINI_JUDGE_MODEL    default: same as GEMINI_MODEL
GEMINI_MIN_INTERVAL   default: 6.5 seconds between provider calls
RAG_CHROMA_PATH       default: data/chroma
RAG_TOP_K             default: 5
```

Search and retrieval evaluation must not require an API key. Generation and judging require `GEMINI_API_KEY`. Fail clearly when it is absent, and never write it to source, reports, command output, or logs.

Keep the Gemini model identifier configurable because free-tier model availability changes. The README may show a currently verified model as an example, but the application must not depend on that example remaining available.

All commands must accept `--seed`, defaulting to `42`, wherever sampling occurs.

## 8. Dataset Contract

### 8.1 Reproducible subset

`prepare_data.py download` must:

1. Load the SQuAD v2 validation split using `datasets`.
2. Group duplicate rows by normalized `(title, context)`.
3. Assign a stable `document_id` derived from a hash of the title and context.
4. Shuffle document IDs with seed `42`.
5. Select 500 unique documents by default.
6. Write documents to `data/documents.jsonl`.
7. Select 100 evaluation questions from the chosen documents.
8. Include 80 answerable and 20 unanswerable questions when enough examples are available.
9. Write the selected examples to `data/eval_questions.jsonl`.
10. Refuse to overwrite files unless `--force` is supplied.

The selection must be stable for the same dataset version, seed, and limits.

### 8.2 Document record

Each document record must contain:

```json
{
  "document_id": "doc_<stable hash>",
  "title": "Beyonce",
  "title_slug": "beyonce",
  "source": "squad_v2",
  "dataset_split": "validation",
  "context": "..."
}
```

### 8.3 Evaluation record

Each evaluation record must contain:

```json
{
  "question_id": "...",
  "question": "...",
  "document_id": "doc_<stable hash>",
  "title": "Beyonce",
  "answers": ["..."],
  "answer_starts": [123],
  "is_answerable": true
}
```

Never use generated answers as ground truth.

## 9. Chunking Experiment

Implement two strategies behind one `Chunker` interface. Both must use the embedding model's tokenizer so chunk limits are measured in model tokens rather than characters.

### 9.1 Strategy A: fixed-token windows

- Maximum size: 180 tokens.
- Overlap: 30 tokens.
- Preserve tokenizer offset mappings to calculate source character ranges.
- Advance by `chunk_size - overlap` tokens.
- Never emit an empty chunk.

This is the simple baseline and may split sentences.

### 9.2 Strategy B: sentence-aware windows

- Split text into sentences using a small documented regular expression.
- Add whole sentences until adding another would exceed 180 tokens.
- Carry the final complete sentence into the next chunk as overlap.
- If one sentence exceeds 180 tokens, split only that sentence using the fixed-token implementation.
- Never emit an empty chunk.

This strategy is still intentionally simple; it is not expected to handle every abbreviation or language.

### 9.3 Required chunk metadata

Every stored chunk must contain:

```text
chunk_id
document_id
title
title_slug
source
dataset_split
chunk_strategy
chunk_index
start_char
end_char
token_count
```

`chunk_id` must be stable and unique for a document, strategy, and chunk index.

Store the original chunk text as the Chroma document. Embed this text:

```text
Title: <title>
<chunk text>
```

Prepending the title gives short chunks useful document-level context. Hold this behavior constant when comparing chunking strategies so the comparison isolates chunking rather than prompt differences.

### 9.4 Gold chunk mapping

For answerable evaluation examples, a chunk is relevant when:

1. its `document_id` matches the evaluation record; and
2. its `[start_char, end_char)` range fully contains at least one reference answer span.

This mapping must be calculated from source offsets, not by asking an LLM.

For unanswerable examples, evaluate document retrieval but do not invent a gold answer-bearing chunk.

## 10. Embedding and Indexing

Use `sentence-transformers/all-MiniLM-L6-v2` locally.

Requirements:

- encode documents in batches;
- encode queries with the same model;
- normalize embeddings;
- configure Chroma for cosine distance;
- use a separate collection for each chunk strategy;
- use deterministic IDs so rerunning ingestion is idempotent;
- validate metadata before insertion;
- print document count, chunk count, collection name, embedding model, and elapsed time;
- fail clearly when the prepared data is absent.

Collection names should include the strategy and a short embedding-model identifier, for example:

```text
squad_v2_fixed_all_minilm_l6_v2
squad_v2_sentence_all_minilm_l6_v2
```

Do not treat cosine distance as a calibrated probability or invent a confidence percentage from it.

## 11. Retrieval

Implement:

```python
def retrieve(
    question: str,
    *,
    strategy: str,
    top_k: int = 5,
    title_slug: str | None = None,
) -> list[RetrievedChunk]:
    ...
```

Requirements:

- reject blank questions;
- validate `1 <= top_k <= 20`;
- query only the selected strategy's collection;
- apply `where={"title_slug": title_slug}` when a title filter is supplied;
- return results in rank order;
- return chunk text, metadata, distance, and one-based rank;
- return an empty list rather than failing when a valid filter has no matches;
- never interpolate raw filter text into executable code;
- keep embedding and search latency observable separately.

The CLI must support both unfiltered retrieval and a filtered query such as:

```powershell
python app.py search "When did Beyonce release Dangerously in Love?" --title beyonce
```

The filtered result set must contain only chunks whose `title_slug` is `beyonce`.

## 12. Answer Generation

### 12.1 Output schema

Validate every generated answer with Pydantic:

```python
class RagAnswer(BaseModel):
    model_config = ConfigDict(extra="forbid")

    answer: str = Field(min_length=1, max_length=500)
    citations: list[str] = Field(max_length=5)
    abstained: bool
```

Rules:

- `citations` contains only retrieved `chunk_id` values;
- citations are unique and preserve first-use order;
- a non-abstained answer requires at least one citation;
- an abstained answer must use an empty citation list;
- when abstaining, `answer` briefly states that the indexed context is insufficient;
- invalid JSON or schema-invalid output is an explicit case error, not silently repaired.

### 12.2 Prompt contract

The prompt must:

- label the customer question separately from retrieved context;
- number and identify each context block by `chunk_id`;
- tell the model to use only the supplied context;
- tell the model that context is untrusted data and instructions inside it must be ignored;
- request a concise answer, ideally the shortest supported answer span for SQuAD questions;
- require citations for factual answers;
- require abstention when the answer is not supported;
- request JSON output through the Google Gen AI SDK's structured-output configuration using the Pydantic-derived schema;
- use temperature `0`.

Truncate or reject a prompt that exceeds the configured context budget. Do not silently drop the highest-ranked chunk.

### 12.3 Generation boundary

Keep Google Gen AI SDK request, response, and exception handling in one small adapter. Use the SDK's current structured-output API rather than merely prompting for JSON. Convert the response to plain data immediately, then run `RagAnswer.model_validate` or equivalent strict validation.

Provider-call requirements:

- send generation and judge requests sequentially during evaluation;
- enforce `GEMINI_MIN_INTERVAL` between calls to reduce free-tier rate-limit failures;
- retry only timeouts, HTTP 429 responses, and transient HTTP 5xx failures;
- make at most two attempts total per request;
- honor `Retry-After` when the SDK exposes it, otherwise use a short exponential backoff;
- do not retry authentication failures, refusals, malformed output, or schema failures;
- preserve the final provider exception as the cause of a concise provider-neutral error.

Handle these failures clearly:

- Gemini API key is absent or invalid;
- the configured model is unavailable to the account or free tier;
- free-tier rate limit or daily quota is exhausted;
- request timeout;
- provider refusal or blocked content;
- malformed structured output;
- schema validation failure;
- no retrieved context.

An empty retrieval result must produce a deterministic abstention without calling the model.

## 13. Query Flow

The primary orchestration must remain bounded:

```text
question
  -> validate input
  -> embed question
  -> retrieve top-k chunks with optional metadata filter
  -> return deterministic abstention if no chunks exist
  -> construct grounded prompt
  -> generate schema-constrained Gemini answer
  -> validate answer and citations
  -> return answer plus diagnostics
```

Return an internal run record so evaluation does not scrape console text:

```python
class RagRun(BaseModel):
    response: RagAnswer
    retrieved_chunks: list[RetrievedChunk]
    embedding_seconds: float
    retrieval_seconds: float
    generation_seconds: float
```

## 14. Command-Line Interface

Required commands:

```powershell
python prepare_data.py download
python prepare_data.py index --strategy fixed
python prepare_data.py index --strategy sentence

python app.py search "What year was the university founded?" --top-k 5
python app.py ask "What year was the university founded?" --strategy sentence
python app.py ask "When was Dangerously in Love released?" --title beyonce --json

python evals.py retrieval --strategy fixed
python evals.py retrieval --strategy sentence
python evals.py full --strategy sentence --limit 20 --judge
python evals.py compare
```

CLI requirements:

- print concise progress to standard error and results to standard output;
- support `--json` for machine-readable query output;
- exit `0` on success and nonzero on configuration, index, model, or schema failure;
- do not print a stack trace for expected user-facing failures;
- make the selected model, chunk strategy, top-k, and filter visible in diagnostics.

## 15. Deterministic Evaluation Set

Use the 100 selected evaluation questions.

- Retrieval evaluation runs all 100 questions without generation.
- Full RAG evaluation defaults to a fixed 20-question subset: 15 answerable and 5 unanswerable.
- The 20-question subset is selected by stable question ID, not by runtime ordering.
- A `--limit` flag may reduce local iteration time but the completion run must use all required cases.
- Record dataset revision, seed, model names, strategy, top-k, and case counts in every report.

Do not tune against the final score repeatedly without recording the changes. Reserve the first 20 non-evaluation examples as a small calibration set for prompt debugging.

## 16. Primary Metrics

### 16.1 Retrieval metrics

Calculate:

- **Document Hit@1, Hit@3, Hit@5:** whether a chunk from the labeled source document appears in the first $k$ results.
- **Answer Chunk Hit@1, Hit@3, Hit@5:** for answerable questions, whether an answer-bearing chunk appears in the first $k$ results.
- **MRR@5:** reciprocal rank of the first answer-bearing chunk, or zero when absent.
- **Context coverage:** percentage of answerable questions for which at least one retrieved chunk contains a gold answer span.
- **Filter precision:** percentage of filtered results whose metadata matches the requested title.

For $N$ questions, Hit@$k$ is:

$$
\operatorname{Hit@k} = \frac{1}{N}\sum_{i=1}^{N}\mathbb{1}[\text{relevant result occurs in top }k]
$$

MRR@$k$ is:

$$
\operatorname{MRR@k} = \frac{1}{N}\sum_{i=1}^{N}
\begin{cases}
\frac{1}{r_i}, & r_i \le k \\
0, & \text{otherwise}
\end{cases}
$$

where $r_i$ is the rank of the first answer-bearing chunk.

### 16.2 Answer metrics

For answerable questions, calculate SQuAD-style:

- **Exact Match (EM):** normalized generated answer exactly matches any reference answer.
- **Token F1:** maximum token overlap F1 against all reference answers.

Normalization must lowercase text and remove punctuation, articles, and extra whitespace using one tested function.

For unanswerable questions, calculate:

- **Abstention accuracy:** generated `abstained` equals the expected unanswerable label.

Also calculate overall answerability classification accuracy across answerable and unanswerable cases.

### 16.3 Citation metrics

Calculate:

- **Citation validity:** every cited ID exists in the retrieved result set.
- **Gold citation rate:** an answerable response cites at least one gold answer-bearing chunk.
- **Citation completeness:** every non-abstained answer has at least one citation.

Do not claim that citation validity alone proves the cited text supports every phrase in an answer.

### 16.4 Performance metrics

Record:

- embedding latency;
- retrieval latency;
- generation latency;
- end-to-end latency;
- p50 and p95 latency;
- chunks indexed;
- average retrieved chunks per query.

Exclude one warm-up query from reported retrieval latency and identify that choice in the report.

## 17. Gemini LLM Judge

The judge is a secondary diagnostic. It must never replace deterministic retrieval, answer, and citation metrics.

For each generated answer, give the judge:

- question;
- reference answers or the unanswerable label;
- retrieved context;
- generated answer;
- citations.

Require schema-constrained output:

```python
class JudgeResult(BaseModel):
    model_config = ConfigDict(extra="forbid")

    correctness: int = Field(ge=0, le=2)
    groundedness: int = Field(ge=0, le=2)
    relevance: int = Field(ge=0, le=1)
    rationale: str = Field(min_length=1, max_length=300)
```

Rubric:

- correctness `0`: wrong or unsupported; `1`: partially correct; `2`: correct;
- groundedness `0`: contradicted or unsupported; `1`: partly supported; `2`: fully supported;
- relevance `0`: does not directly answer or appropriately abstain; `1`: relevant and concise.

Judge requirements:

- use temperature `0`;
- record judge model and SDK version;
- treat schema failures as judge errors rather than zero scores;
- keep the deterministic case score even if judging fails;
- report judge coverage, average score out of 5, and score distribution;
- label results as model judgments, not objective ground truth.

Using the same Gemini model for generation and judging is acceptable for this timeboxed POC but must be documented as a bias risk. A future experiment should use a different judge model or human labels.

## 18. Chunking Comparison

`python evals.py compare` must run retrieval evaluation for both strategies with all other settings fixed.

Print a table containing:

```text
strategy
chunk count
average tokens per chunk
Document Hit@1/3/5
Answer Chunk Hit@1/3/5
MRR@5
context coverage
p50 retrieval latency
p95 retrieval latency
```

The report must state which strategy performed better for:

- Answer Chunk Hit@5;
- MRR@5;
- index size;
- retrieval latency.

Do not declare a universal winner from this one dataset. The goal is to observe tradeoffs.

## 19. Scorecard and Reports

Write timestamped reports under `data/reports/` in both JSON and Markdown.

Each case row must include:

```text
question_id
answerable
expected document_id
retrieved chunk IDs and ranks
document hit
answer-chunk hit
generated answer
reference answers
exact match
token F1
abstained expected/actual
citations
citation validity
gold citation result
latencies
judge scores or judge error
run error, if any
```

Summary output must include:

```text
cases attempted
cases completed
retrieval metrics
answer metrics
citation metrics
abstention metrics
latency percentiles
judge coverage and average
errors by category
configuration and model versions
```

A failed case must remain in the denominator for applicable deterministic metrics. Provider or schema errors must be listed separately and must not silently disappear from averages.

## 20. Software Tests

Write tests before or alongside each implementation step. Unit tests must not download models, call Gemini, or depend on an existing Chroma directory.

Required tests:

1. Blank questions are rejected.
2. Fixed-token chunks respect token limits and overlap.
3. Sentence-aware chunks respect token limits and preserve source order.
4. An oversized sentence uses the fixed-token fallback.
5. Chunk IDs are stable and unique.
6. Character offsets map each chunk back to the original context.
7. Required metadata is present and valid.
8. Dataset selection is stable for the same seed.
9. Gold answer spans map to the expected chunk.
10. `top_k` outside `1..20` is rejected.
11. Metadata filtering returns only matching records.
12. Retrieval results preserve rank order.
13. SQuAD answer normalization, EM, and token F1 match known examples.
14. `RagAnswer` rejects extra fields and inconsistent citation/abstention combinations.
15. Generated citations not present in retrieved results are rejected.
16. Empty retrieval produces a deterministic abstention without invoking Gemini.
17. Invalid model JSON is surfaced as a case error.
18. One failed evaluation case does not stop later cases.

Use a small fake tokenizer, fake embedder, temporary Chroma collection or vector-store fake, and fake model adapter to keep tests fast and hermetic.

## 21. Quality Gates

The completion run uses 500 documents, 100 retrieval questions, 20 full RAG questions, `top_k=5`, and seed `42`.

The POC passes when:

| Gate | Required result |
|---|---:|
| Unit tests | 100% pass |
| Retrieval cases completed | 100/100 |
| Best strategy Document Hit@5 | at least 0.90 |
| Best strategy Answer Chunk Hit@5 | at least 0.80 |
| Best strategy MRR@5 | at least 0.55 |
| Metadata filter precision | 1.00 |
| Full RAG cases completed | 20/20 |
| Answerable Token F1 | at least 0.55 |
| Citation validity | 1.00 |
| Gold citation rate | at least 0.80 |
| Answerability classification accuracy | at least 0.75 |
| LLM judge coverage | at least 0.90 |
| Average LLM judge score | at least 3.5/5 |
| Warm retrieval p95 | less than 1.0 second on the stated test machine |

Exact Match is reported but is not a pass gate because a concise generative answer may be correct without exactly matching the reference string.

If a quality threshold is missed, the POC is still runnable but not complete. The final report must identify the missed gate and one evidence-based next experiment. Do not lower a threshold merely to make the run pass.

## 22. Failure Semantics

| Failure | Required behavior |
|---|---|
| Dataset download unavailable | Explain that the initial cache is missing; exit nonzero |
| Prepared data missing | Print the exact preparation command; exit nonzero |
| Index missing | Print the exact indexing command; exit nonzero |
| Unsupported strategy | Reject before loading a model |
| Blank question | Reject before embedding |
| Invalid metadata filter | Return no matches or a concise validation error |
| Gemini API key absent or invalid | Preserve retrieval diagnostics; print safe configuration guidance; exit nonzero |
| Model unavailable on free tier | Record a provider error and identify the configured model without exposing credentials |
| Free-tier rate limit | Retry within the bounded policy, then record a rate-limit error |
| Daily quota exhausted | Stop new provider calls, preserve completed results, and report a resumable run error |
| Model timeout | Record a case error; continue remaining evaluation cases |
| Invalid structured output | Record a schema error; do not auto-repair |
| No retrieved chunks | Return deterministic abstention |
| Invalid citation | Reject the answer as a contract failure |
| Judge failure | Keep primary metrics and mark judge result unavailable |

Do not catch `BaseException`, suppress cancellation, or print sensitive environment data.

## 23. Five-to-Six-Hour Implementation Plan

### Hour 1: setup and deterministic data

- Create the project and dependency manifest.
- Add Pydantic records and configuration.
- Download and deterministically subset SQuAD v2.
- Write dataset-selection and validation tests.

### Hour 2: chunking

- Implement fixed-token chunking.
- Implement sentence-aware chunking and offsets.
- Add chunk metadata and gold-span mapping.
- Finish focused chunking tests.

### Hour 3: embeddings, storage, and retrieval

- Integrate Sentence Transformers.
- Create both Chroma collections.
- Implement top-k retrieval and title filters.
- Verify filter behavior and retrieval result models.

### Hour 4: generation and CLI

- Integrate the Google Gen AI SDK behind a small adapter.
- Implement the grounded prompt and structured answer.
- Validate citations and abstention.
- Add `search` and `ask` CLI commands.

### Hour 5: evaluation

- Implement retrieval, answer, citation, and latency metrics.
- Produce JSON and Markdown scorecards.
- Run the chunking comparison.

### Hour 6: judge and completion run

- Add the schema-constrained Gemini judge.
- Run the 20-case full evaluation.
- Fix only contract failures.
- Record actual results, limitations, and reproduction commands in the README.

If free-tier limits interrupt the run, preserve partial results and resume after the quota window resets. If time is short, keep the judge to the required 20 full cases; do not judge all 100 retrieval cases.

## 24. Definition of Done

The POC is complete only when all of the following are true:

- all local dependencies are open source, with licenses documented;
- Gemini uses only the no-cost free tier, with its proprietary-service exception documented;
- the API key is loaded only from the environment and never exposed;
- the SQuAD v2 subset is reproducible from a fixed seed;
- both chunking strategies produce valid chunks with source offsets and metadata;
- both strategies are embedded and stored in local Chroma collections;
- unfiltered and title-filtered retrieval work from the CLI;
- metadata filtering is verified by tests and evaluation;
- generated answers conform to the Pydantic schema;
- every factual answer cites a retrieved chunk;
- insufficient evidence produces an explicit abstention;
- deterministic retrieval, answer, citation, abstention, and latency metrics are reported;
- the Gemini judge produces separate structured scores for at least 90% of full cases;
- chunking strategies are compared under identical retrieval settings;
- all required software tests pass without network or model calls;
- the completion run meets every quality gate in Section 21;
- README instructions reproduce preparation, indexing, querying, testing, and evaluation;
- known limitations and the exact test-machine configuration are recorded.

## 25. Expected Demonstration

PowerShell example:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt

# Set GEMINI_API_KEY through a secure environment mechanism.
# Select a model currently listed for the Gemini free tier.
$env:GEMINI_MODEL = "<free-tier-flash-model>"

python prepare_data.py download
python prepare_data.py index --strategy fixed
python prepare_data.py index --strategy sentence

python -m pytest

python app.py search "When was Dangerously in Love released?" --top-k 5
python app.py ask "When was Dangerously in Love released?" --strategy sentence --title beyonce

python evals.py retrieval --strategy fixed
python evals.py retrieval --strategy sentence
python evals.py compare
python evals.py full --strategy sentence --limit 20 --judge
```

The demonstration must show:

1. At least one question where both strategies retrieve the answer-bearing chunk.
2. At least one question where chunking strategy changes rank or success.
3. A metadata-filtered query whose results all match the filter.
4. A supported answer with a valid chunk citation.
5. An unanswerable question that produces abstention.
6. A scorecard with deterministic metrics and separate LLM-judge results.
7. The measured latency and hardware/model configuration.

## 26. Learning Review

After implementation, be ready to explain:

1. Why chunk size and overlap affect recall, precision, index size, and prompt cost.
2. Why character offsets are useful for objective retrieval labels.
3. Why document metadata belongs alongside vectors.
4. Why metadata filtering should happen before or during vector search.
5. Why embedding similarity is not factual confidence.
6. Why retrieval metrics and generation metrics diagnose different failure stages.
7. Why exact match can understate generative answer quality.
8. Why citations must be validated against retrieved IDs.
9. Why an LLM judge is useful but biased and nondeterministic.
10. Why unanswerable cases and abstention are essential to RAG evaluation.
11. Why a small labeled evaluation set is more useful than anecdotal demos.
12. What production RAG would add: access controls, ingestion versioning, hybrid search, reranking, prompt-injection defenses, tracing, caching, larger evaluations, and rollout monitoring.

## 27. Constraints for the Implementing Agent

- Keep each stage readable enough to explain line by line.
- Prefer small typed functions over framework abstractions.
- Keep model and vector-store adapters separate from metric logic.
- Do not hide model downloads or network access.
- Do not use mutable global state for run results.
- Do not calculate evaluation metrics with an LLM when deterministic labels exist.
- Do not silently remove failed cases from score denominators.
- Do not tune both chunking and generation settings at the same time when comparing results.
- Do not expand scope after the definition of done is met.