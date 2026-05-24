# awesome-gemini-for-google-cloud

> **Curated Gemini on GCP** — production-grade patterns, SDKs, and architectures for Google's Gemini models on Cloud

<p align="center">
  <a href="https://github.com/hmzainjamil/awesome-gemini-for-google-cloud/stargazers"><img src="https://img.shields.io/github/stars/hmzainjamil/awesome-gemini-for-google-cloud?style=for-the-badge&labelColor=555&color=yellow" alt="Stars"/></a>
  <a href="https://github.com/hmzainjamil/awesome-gemini-for-google-cloud/network/members"><img src="https://img.shields.io/github/forks/hmzainjamil/awesome-gemini-for-google-cloud?style=for-the-badge&labelColor=555&color=blue" alt="Forks"/></a>
  <a href="https://github.com/hmzainjamil/awesome-gemini-for-google-cloud/issues"><img src="https://img.shields.io/github/issues/hmzainjamil/awesome-gemini-for-google-cloud?style=for-the-badge&labelColor=555&color=red" alt="Issues"/></a>
  <a href="https://github.com/hmzainjamil/awesome-gemini-for-google-cloud/pulls"><img src="https://img.shields.io/github/issues-pr/hmzainjamil/awesome-gemini-for-google-cloud?style=for-the-badge&labelColor=555&color=purple" alt="PRs"/></a>
  <a href="https://github.com/hmzainjamil/awesome-gemini-for-google-cloud/commits/main"><img src="https://img.shields.io/github/last-commit/hmzainjamil/awesome-gemini-for-google-cloud?style=for-the-badge&labelColor=555&color=green" alt="Last Commit"/></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Google_Cloud-4285F4?style=flat&labelColor=555&logo=google-cloud&logoColor=white"/>
  <img src="https://img.shields.io/badge/Gemini-8E75B2?style=flat&labelColor=555&logo=google&logoColor=white"/>
  <img src="https://img.shields.io/badge/Vertex_AI-4285F4?style=flat&labelColor=555"/>
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat&labelColor=555&logo=python&logoColor=white"/>
</p>

---

## Why This Exists

Gemini on Google Cloud spans Vertex AI, AI Studio, Gemini API, Cloud Run, and BigQuery ML — each with different SDKs, quotas, pricing, and deployment models. This repo is the single reference: what works, what doesn't, real production patterns, vetted code samples, and links to repos/tools that actually ship. Skip the tutorial trap.

No marketing. No beginner hand-holding. Just dense reference material for engineers deploying Gemini at scale.

---

## At a Glance

| Feature | Detail |
|---------|--------|
| Models covered | Gemini 1.5 Pro, Gemini 1.5 Flash, Gemini 1.0 Pro, Gemini Pro Vision |
| Deployment targets | Vertex AI, AI Studio, Cloud Run, GKE, App Engine |
| SDK versions | `google-generativeai` ≥0.5, `vertexai` ≥1.38 |
| Key integrations | BigQuery ML, Cloud Storage, Pub/Sub, Firebase, AlloyDB |
| Auth methods | ADC, Service Account JSON, Workload Identity Federation |
| Multimodal support | Text, Image, Video, Audio, PDF (inline or GCS URI) |
| Streaming | `generate_content(stream=True)` for real-time output |
| Function calling | Native tool/function calling with JSON schema |
| Context window | Gemini 1.5 Pro: 1M tokens; Flash: 1M tokens |
| Grounding | Google Search grounding for factual accuracy |
| RAG patterns | Vertex AI Search, Matching Engine, LangChain integrations |
| Cost control | Token counting API + quota alerts in Cloud Monitoring |
| Context caching | Cache system prompts/docs up to 1M tokens for 75% cost cut |
| Safety filters | Per-category `HarmBlockThreshold` — tune or disable per use case |

---

## 🧠 CONCEPTS

| Concept | Description | Why It Matters |
|---------|-------------|----------------|
| Vertex AI vs AI Studio | Vertex = enterprise, VPC, audit logs, IAM. AI Studio = fast prototyping | Wrong choice → quota limits or compliance failures |
| Multimodal | Single model handles text, image, video, audio in one call | Eliminates 3-model pipeline complexity |
| Grounding | Attaches real-time Google Search results to generation | Cuts hallucination rate for factual queries |
| Function Calling | Model outputs structured JSON to call your APIs | Backbone of agentic Gemini workflows |
| Context caching | Cache up to 1M tokens of system prompt on GCP | Slashes cost on repeated large-context calls by 75% |
| Streaming | `stream=True` yields chunks as generated | Critical for UX — first byte <200ms |
| Embeddings | `text-embedding-004` — 768-dim, 2048 token limit | Powers vector search, RAG, semantic dedup |
| Tokenization | `count_tokens()` before sending | Hard token limits cause silent truncation |
| Safety filters | Configurable `HarmCategory` + `HarmBlockThreshold` | Too strict = over-blocked; too loose = policy violations |
| Vertex Pipelines | ML pipeline orchestration with Kubeflow | Reproducible training + eval + deploy workflows |

### 🔥 Hot

| Feature | Why Devs Love It |
|---------|-----------------|
| 1M context window | Entire codebase or 100-page doc in one prompt — no chunking |
| Native video understanding | Timestamp-level video Q&A without frame extraction |
| Grounding + function calling | Agent that searches + calls APIs + cites sources in one turn |
| Context caching | 75% cost reduction on repeated large system-prompt patterns |
| Multimodal embeddings | Cross-modal search: query text → find matching images |
| Thinking mode (Gemini 2.0) | Step-by-step reasoning before final answer |
| Batched requests | Process thousands of prompts offline at 50% cost via batch API |

---

## ⚙️ HOW IT WORKS

```
User Prompt + Optional Files (text/image/video/audio/pdf)
       ↓
SDK: google.generativeai OR vertexai Python SDK
       ↓
Auth: ADC (gcloud auth application-default login)
           OR Service Account JSON key (GOOGLE_APPLICATION_CREDENTIALS)
           OR Workload Identity (GKE/Cloud Run production)
       ↓
Vertex AI Endpoint  ──OR──  Generative Language API (AI Studio key)
       ↓
Optional: Grounding (Google Search), RAG (Vertex Search), Caching
       ↓
Safety filters: HarmCategory × HarmBlockThreshold per category
       ↓
Response: stream OR batch → function_call parts OR text parts
       ↓
Your app: parse function calls, render text, store to DB/GCS
```

**Production deployment pattern:**
1. Auth via Workload Identity — no service account keys in cluster
2. `vertexai.init(project=PROJECT, location="us-central1")` in app startup
3. Load model once: `model = GenerativeModel("gemini-1.5-pro")`
4. Per-request: build `GenerationConfig` with request-specific params
5. Stream with `.generate_content(prompt, stream=True, generation_config=config)`
6. Catch `ResourceExhausted` → exponential backoff with jitter
7. Log `response.usage_metadata` to BigQuery for cost attribution

---

## 🚀 INSTALL

```bash
# Core SDKs
pip install google-generativeai vertexai

# Optional integrations
pip install langchain-google-vertexai  # LangChain
pip install llama-index-llms-vertex    # LlamaIndex
pip install google-cloud-aiplatform    # Full Vertex AI SDK

# Auth (local dev)
gcloud auth application-default login
gcloud config set project YOUR_PROJECT_ID

# Enable required APIs
gcloud services enable aiplatform.googleapis.com
gcloud services enable generativelanguage.googleapis.com
gcloud services enable storage.googleapis.com

# Verify SDK works
python3 -c "
import vertexai
vertexai.init(project='YOUR_PROJECT_ID', location='us-central1')
from vertexai.generative_models import GenerativeModel
model = GenerativeModel('gemini-1.5-flash')
print(model.generate_content('Say OK').text)
"
```

---

## 📟 USAGE

### Basic text generation

```python
import vertexai
from vertexai.generative_models import GenerativeModel, GenerationConfig

vertexai.init(project="YOUR_PROJECT_ID", location="us-central1")
model = GenerativeModel("gemini-1.5-pro")
config = GenerationConfig(temperature=0.1, max_output_tokens=2048)

response = model.generate_content("Summarize GCP Vertex AI in 3 bullets", generation_config=config)
print(response.text)
```

### Streaming

```python
for chunk in model.generate_content("Write a Python web scraper", stream=True):
    print(chunk.text, end="", flush=True)
print()
```

### Multimodal — image + text

```python
from vertexai.generative_models import Part

image_part = Part.from_uri("gs://my-bucket/screenshot.jpg", mime_type="image/jpeg")
response = model.generate_content(["What errors are visible in this screenshot?", image_part])
print(response.text)
```

### PDF analysis

```python
pdf_part = Part.from_uri("gs://my-bucket/contract.pdf", mime_type="application/pdf")
response = model.generate_content([
    "Extract all payment terms and deadlines from this contract.",
    pdf_part
])
print(response.text)
```

### Function calling (agentic)

```python
from vertexai.generative_models import FunctionDeclaration, Tool

get_stock = FunctionDeclaration(
    name="get_stock_price",
    description="Get current stock price for a ticker symbol",
    parameters={
        "type": "object",
        "properties": {
            "ticker": {"type": "string", "description": "Stock ticker, e.g. GOOG"},
            "exchange": {"type": "string", "enum": ["NYSE", "NASDAQ"]}
        },
        "required": ["ticker"]
    }
)
tool = Tool(function_declarations=[get_stock])
response = model.generate_content(
    "What's Google's current stock price?",
    tools=[tool]
)
func_call = response.candidates[0].content.parts[0].function_call
print(f"Call: {func_call.name}({dict(func_call.args)})")
```

### Embeddings

```python
from vertexai.language_models import TextEmbeddingModel

emb = TextEmbeddingModel.from_pretrained("text-embedding-004")
results = emb.get_embeddings(["deploy gemini to cloud run", "vertex ai deployment"])
vectors = [r.values for r in results]  # list of 768-dim floats
```

### Context caching (75% cost reduction)

```python
import datetime
from vertexai.preview.caching import CachedContent

cached = CachedContent.create(
    model_name="gemini-1.5-pro-001",
    system_instruction="You are a legal document expert.",
    contents=[Part.from_uri("gs://bucket/1000-page-manual.pdf", mime_type="application/pdf")],
    ttl=datetime.timedelta(hours=24),
)
cached_model = GenerativeModel.from_cached_content(cached)
response = cached_model.generate_content("What are the warranty terms in section 4?")
print(response.text)
```

### Token counting

```python
tokens = model.count_tokens("Your long document text here")
print(f"Tokens: {tokens.total_tokens} (max 1M for gemini-1.5-pro)")
```

---

## ⚙️ CONFIGURATION

| Config Key | Type | Default | Description |
|-----------|------|---------|-------------|
| `temperature` | float | 0.9 | Creativity: 0=deterministic, 1=max creative |
| `max_output_tokens` | int | 2048 | Max tokens in response (up to 8192) |
| `top_p` | float | 0.95 | Nucleus sampling — lower = more focused |
| `top_k` | int | 40 | Top-k token sampling |
| `candidate_count` | int | 1 | Number of response candidates (max 8) |
| `stop_sequences` | list | [] | Stop generation at these strings |
| `location` | str | us-central1 | GCP region for Vertex AI |
| `project` | str | required | GCP project ID |
| `model_name` | str | gemini-1.5-pro | Model version to use |
| `safety_threshold` | enum | BLOCK_MEDIUM_AND_ABOVE | Per-category harm threshold |
| `stream` | bool | False | Enable streaming response |
| `GOOGLE_APPLICATION_CREDENTIALS` | env | - | Path to service account JSON |
| `GOOGLE_CLOUD_PROJECT` | env | - | Default project (overrides gcloud config) |

---

## 💡 TIPS AND TRICKS

### Prompt Engineering Tips
1. **System instruction > system prompt** — use `system_instruction=` in `GenerativeModel()` not in the user turn; it's processed differently and improves instruction following. Source → [HMZ](https://github.com/hmzainjamil)
2. **Temperature 0 for extraction** — JSON extraction, classification, structured output: set `temperature=0`. Creativity prompts: 0.7-1.0. Source → [HMZ](https://github.com/hmzainjamil)
3. **Few-shot beats long instructions** — 3-5 input/output examples in context outperform 500-word instruction paragraphs for complex formatting tasks. Source → [HMZ](https://github.com/hmzainjamil)

### Cost Optimization Tips
1. **Count tokens before sending** — `model.count_tokens(prompt)` is free; use it to gate expensive calls and avoid surprise overages. Source → [HMZ](https://github.com/hmzainjamil)
2. **Use Flash for classification** — Gemini 1.5 Flash costs ~10x less than Pro; route simple classify/extract tasks to Flash, complex reasoning to Pro. Source → [HMZ](https://github.com/hmzainjamil)
3. **Context caching for repeated docs** — if your system prompt + context exceeds 32k tokens and is reused, caching saves 75% on input token cost. Source → [HMZ](https://github.com/hmzainjamil)

### Production Tips
1. **Workload Identity > service account keys** — on GKE/Cloud Run, Workload Identity eliminates key rotation headache and reduces breach surface area. Source → [HMZ](https://github.com/hmzainjamil)
2. **Retry with exponential backoff** — wrap all API calls with `tenacity.retry(wait=wait_exponential(min=1, max=60))`; quota errors are transient. Source → [HMZ](https://github.com/hmzainjamil)
3. **Log usage_metadata per request** — `response.usage_metadata.total_token_count` → BigQuery; enables per-tenant cost attribution and anomaly alerts. Source → [HMZ](https://github.com/hmzainjamil)

### Safety + Compliance Tips
1. **Tune safety thresholds per use case** — medical/legal: keep defaults; internal dev tools: can lower to `BLOCK_ONLY_HIGH`; adult platforms: follow GCP ToS carefully. Source → [HMZ](https://github.com/hmzainjamil)
2. **Inspect finish_reason always** — `response.candidates[0].finish_reason == FinishReason.SAFETY` means content was blocked; handle gracefully vs crashing. Source → [HMZ](https://github.com/hmzainjamil)
3. **Audit logs via Cloud Logging** — Vertex AI writes every API call to Cloud Audit Logs; enable for compliance without code changes. Source → [HMZ](https://github.com/hmzainjamil)

---

## 🔧 TROUBLESHOOTING

| Issue | Cause | Fix |
|-------|-------|-----|
| `google.api_core.exceptions.PermissionDenied` | API not enabled or wrong project | `gcloud services enable aiplatform.googleapis.com` |
| `ResourceExhausted: 429` | Quota exceeded | Add retry with exponential backoff; request quota increase in GCP console |
| `InvalidArgument: 400` on multimodal | File too large or wrong MIME | Max inline: 20MB; use GCS URI for larger files |
| Empty `response.text` | Safety filter blocked | Check `response.candidates[0].finish_reason` and `safety_ratings` |
| `ValueError: context_window exceeded` | Prompt > 1M tokens | Count tokens first; split or summarize input |
| `DefaultCredentialsError` | No ADC set up | Run `gcloud auth application-default login` |
| Slow cold start on Cloud Run | SDK init per request | Move `vertexai.init()` and `GenerativeModel()` to module level |
| High latency on large context | Network transfer of tokens | Use GCS URIs for files instead of inline bytes |
| `StopCandidateException` | Response stopped unexpectedly | Increase `max_output_tokens`; check stop_sequences |
| Embeddings dimension mismatch | Model version changed | Pin to `text-embedding-004`; check output_dimensionality param |

---

## 📊 ARCHITECTURE

```
┌──────────────────────────────────────────────────────────┐
│                   Client Application                     │
│   Cloud Run / GKE Pod / App Engine / Local Python        │
│   Auth: Workload Identity / ADC / Service Account        │
└──────────────────────────┬───────────────────────────────┘
                           │ vertexai SDK calls
                           ↓
┌──────────────────────────────────────────────────────────┐
│                  Google Cloud Platform                   │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────────┐ │
│  │ Vertex AI   │  │  Gen Lang   │  │  Cloud Storage   │ │
│  │ Gemini 1.5  │  │  API (AI    │  │  (files, PDFs,   │ │
│  │ Pro / Flash │  │  Studio)    │  │   video, audio)  │ │
│  └─────────────┘  └─────────────┘  └──────────────────┘ │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────────┐ │
│  │  Embeddings │  │  Grounding  │  │  Context Cache   │ │
│  │ text-emb-004│  │ Google Srch │  │  (75% cost cut)  │ │
│  └─────────────┘  └─────────────┘  └──────────────────┘ │
└──────────────────────────┬───────────────────────────────┘
                           │
         ┌─────────────────┴──────────────────┐
         ↓                                    ↓
┌─────────────────────┐          ┌────────────────────────┐
│  Downstream APIs    │          │  BigQuery / AlloyDB    │
│  (function calling) │          │  (store embeddings,    │
│  your backend       │          │   usage logs, RAG)     │
└─────────────────────┘          └────────────────────────┘
```

---

## 🗺️ ROADMAP

| Priority | Feature | Status |
|----------|---------|--------|
| P0 | Gemini 2.0 Flash + Thinking mode examples | In Progress |
| P0 | Vertex AI RAG Engine (managed) reference | In Progress |
| P1 | Multi-turn chat with persistent history | Planned |
| P1 | Batch prediction API for offline workloads | Planned |
| P2 | Gemini Code Assist + custom model fine-tuning | Planned |
| P2 | AlloyDB AI + pgvector RAG pattern | Planned |
| P2 | Vertex AI Agent Engine (managed agents) | Planned |
| P3 | Cost calculator notebook (token → $ estimate) | Planned |
| P3 | Multi-region failover pattern | Exploring |
| P3 | Vertex AI Model Garden integration guide | Exploring |

---

## ☠️ STARTUPS / BUSINESSES

| What This Replaces | Old Cost | With Gemini + GCP | Saving |
|-------------------|----------|------------------|--------|
| OpenAI GPT-4 at scale | $15-30/M tokens input | Gemini Flash ~$0.075/M | 90%+ |
| Custom ML team for NLP pipeline | $300k-500k/yr (2 engineers) | One Vertex AI endpoint | 95% |
| Separate OCR + vision + LLM pipeline | 3 services + glue code | One Gemini multimodal call | 80% infra cost |
| 3rd-party video analytics SaaS | $2k-8k/mo | Gemini 1.5 Pro + GCS | 85% |
| Manual document review (legal/finance) | $300-500/hr consultant | Gemini + PDF context | 90% |

**Why GCP + Gemini beats alternatives for businesses:**
- Single vendor — IAM, billing, compliance, SLA all in one GCP contract
- Gemini 1.5 Pro 1M context = process entire contracts, codebases, meetings in one call
- Grounding = real-time factual answers without hallucination for customer-facing products
- No per-seat pricing — Vertex AI is token-based, scales to zero

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=hmzainjamil/awesome-gemini-for-google-cloud&type=Date)](https://star-history.com/#hmzainjamil/awesome-gemini-for-google-cloud&Date)

---

Built by [HMZ](https://github.com/hmzainjamil)

---

## 🔬 DEEP DIVE — GCP PRODUCTION PATTERNS

### Workload Identity Setup (GKE)

```bash
# Create GCP service account
gcloud iam service-accounts create gemini-app-sa \
  --display-name="Gemini App Service Account"

# Grant Vertex AI User role
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:gemini-app-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"

# Link to Kubernetes service account
gcloud iam service-accounts add-iam-policy-binding gemini-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/KSA_NAME]"

# Annotate K8s SA
kubectl annotate serviceaccount KSA_NAME \
  --namespace=NAMESPACE \
  iam.gke.io/gcp-service-account=gemini-app-sa@PROJECT_ID.iam.gserviceaccount.com
```

### Retry Pattern for Production

```python
from tenacity import retry, wait_exponential, stop_after_attempt, retry_if_exception_type
from google.api_core.exceptions import ResourceExhausted, ServiceUnavailable

@retry(
    retry=retry_if_exception_type((ResourceExhausted, ServiceUnavailable)),
    wait=wait_exponential(multiplier=1, min=1, max=60),
    stop=stop_after_attempt(5)
)
def call_gemini(model, prompt, config):
    return model.generate_content(prompt, generation_config=config)
```

### Cost Attribution to BigQuery

```python
from google.cloud import bigquery

bq = bigquery.Client()

def log_usage(request_id: str, model: str, usage_metadata, user_id: str):
    rows = [{
        "request_id": request_id,
        "model": model,
        "prompt_tokens": usage_metadata.prompt_token_count,
        "response_tokens": usage_metadata.candidates_token_count,
        "total_tokens": usage_metadata.total_token_count,
        "user_id": user_id,
        "timestamp": "AUTO"
    }]
    errors = bq.insert_rows_json("PROJECT.DATASET.gemini_usage", rows)
    if errors:
        raise RuntimeError(f"BigQuery insert failed: {errors}")
```

### Grounding Configuration

```python
from vertexai.generative_models import GenerativeModel, Tool, grounding

# Enable Google Search grounding
google_search_tool = Tool.from_google_search_retrieval(
    grounding.GoogleSearchRetrieval()
)

model = GenerativeModel("gemini-1.5-pro")
response = model.generate_content(
    "What are the latest changes in Python 3.13?",
    tools=[google_search_tool]
)
print(response.text)
# response.candidates[0].grounding_metadata has source citations
```

---

## 🧪 TESTING GEMINI INTEGRATIONS

```python
# tests/test_gemini.py
from unittest.mock import MagicMock, patch
import pytest

@pytest.fixture
def mock_model():
    model = MagicMock()
    model.generate_content.return_value.text = "Mock response"
    model.count_tokens.return_value.total_tokens = 100
    return model

def test_generate_content(mock_model):
    response = mock_model.generate_content("Test prompt")
    assert response.text == "Mock response"
    mock_model.generate_content.assert_called_once_with("Test prompt")

def test_token_counting(mock_model):
    tokens = mock_model.count_tokens("Test input")
    assert tokens.total_tokens == 100

# Integration test (requires real credentials)
@pytest.mark.integration
def test_real_gemini_call():
    import vertexai
    from vertexai.generative_models import GenerativeModel
    vertexai.init(project="TEST_PROJECT", location="us-central1")
    model = GenerativeModel("gemini-1.5-flash")
    response = model.generate_content("Say 'test OK'")
    assert "OK" in response.text
```

---

## 📊 MONITORING + ALERTING

### Cloud Monitoring Alerts

```bash
# Alert on high token usage (cost control)
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="Gemini High Token Usage" \
  --condition-display-name="Total tokens > 1M/hr" \
  --condition-filter='resource.type="aiplatform.googleapis.com/Endpoint"' \
  --condition-threshold-value=1000000

# Alert on error rate
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="Gemini Error Rate High" \
  --condition-filter='metric.type="aiplatform.googleapis.com/prediction/online/error_count"'
```

### Key Metrics to Track

| Metric | Normal Range | Alert Threshold |
|--------|-------------|-----------------|
| Input tokens/hr | < 500K | > 1M (cost spike) |
| Latency P95 | < 3s | > 8s |
| Error rate | < 0.1% | > 1% |
| Safety blocks | < 0.5% | > 5% (prompt engineering issue) |
| Cache hit rate | > 50% | < 20% (caching misconfigured) |

