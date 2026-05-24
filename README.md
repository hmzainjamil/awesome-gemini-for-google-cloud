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

## Why This Exists

Gemini on Google Cloud spans Vertex AI, AI Studio, Gemini API, Cloud Run, and BigQuery ML — each with different SDKs, quotas, pricing, and deployment models. This repo is the single reference point: what works, what doesn't, and real production patterns that skip the tutorial trap.

## At a Glance

| Feature | Detail |
|---------|--------|
| Models covered | Gemini 1.5 Pro, Gemini 1.5 Flash, Gemini 1.0 Pro, Gemini Pro Vision |
| Deployment targets | Vertex AI, AI Studio, Cloud Run, GKE, App Engine |
| SDK versions | `google-generativeai` >=0.5, `vertexai` >=1.38 |
| Key integrations | BigQuery ML, Cloud Storage, Pub/Sub, Firebase, AlloyDB |
| Auth methods | ADC, Service Account JSON, Workload Identity |
| Multimodal support | Text, Image, Video, Audio, PDF (via inline or GCS URI) |
| Streaming | `generate_content(stream=True)` for real-time output |
| Function calling | Native tool/function calling with JSON schema |
| Context window | Gemini 1.5 Pro: 1M tokens; Flash: 1M tokens |
| Grounding | Google Search grounding for factual accuracy |
| RAG patterns | Vertex AI Search, Matching Engine, LangChain integrations |
| Cost control | Token counting API + quota alerts in Cloud Monitoring |

## 🧠 CONCEPTS

| Concept | Description | Why It Matters |
|---------|-------------|----------------|
| Vertex AI vs AI Studio | Vertex = enterprise, VPC, audit logs. AI Studio = fast prototyping | Wrong choice causes quota limits or compliance failures |
| Multimodal | Single model handles text, image, video, audio in one call | Eliminates 3-model pipeline complexity |
| Grounding | Attaches real-time Google Search results to generation | Cuts hallucination rate for factual queries |
| Function Calling | Model outputs structured JSON to call your APIs | Backbone of agentic Gemini workflows |
| Context caching | Cache up to 1M tokens of system prompt on GCP | Slashes cost on repeated large-context calls |
| Streaming | `stream=True` yields chunks as generated | Critical for UX — first byte in <200ms |
| Embeddings | `text-embedding-004` — 768-dim, 2048 token limit | Powers vector search, RAG, semantic dedup |
| Tokenization | `count_tokens()` before sending | Hard token limits cause silent truncation |
| Safety filters | Configurable `HarmCategory` thresholds | Too strict = over-blocked; too loose = violations |
| Vertex Pipelines | ML pipeline orchestration with Kubeflow | Reproducible training + eval + deploy workflows |

### 🔥 Hot

| Feature | Why Devs Love It |
|---------|-----------------|
| 1M context window | Entire codebase or 100-page doc in one prompt |
| Native video understanding | Timestamp-level video Q&A without frame extraction |
| Grounding + function calling | Agent that searches + calls APIs + cites sources |
| Context caching | 75% cost reduction on repeated system-prompt patterns |
| Multimodal embeddings | Cross-modal search: query text, find images |

## ⚙️ HOW IT WORKS

```
User Prompt + Files
       ↓
SDK: google.generativeai / vertexai
       ↓
Auth: ADC (gcloud auth application-default login)
       ↓
Vertex AI Endpoint  OR  Generative Language API
       ↓
Safety filters (configurable per HarmCategory)
       ↓
Response stream → function call OR text output
       ↓
Your app: parse / render / store
```

Production flow:
1. ADC auth — never hardcode service account keys
2. `vertexai.init(project=PROJECT, location="us-central1")`
3. Load model: `GenerativeModel("gemini-1.5-pro")`
4. Build `GenerationConfig(temperature=0.2, max_output_tokens=8192)`
5. Stream: `.generate_content(prompt, stream=True)`

## 🚀 INSTALL

```bash
pip install google-generativeai vertexai

# Auth (local dev)
gcloud auth application-default login
gcloud config set project YOUR_PROJECT_ID

# Enable APIs
gcloud services enable aiplatform.googleapis.com generativelanguage.googleapis.com

# Verify
python3 -c "import vertexai; vertexai.init(project='YOUR_PROJECT_ID', location='us-central1'); print('OK')"
```

## 📟 USAGE

```python
import vertexai
from vertexai.generative_models import GenerativeModel, GenerationConfig, Part, FunctionDeclaration, Tool

vertexai.init(project="YOUR_PROJECT_ID", location="us-central1")
model = GenerativeModel("gemini-1.5-pro")
config = GenerationConfig(temperature=0.1, max_output_tokens=2048)

# Text generation
response = model.generate_content("Summarize GCP Vertex AI in 3 bullets", generation_config=config)
print(response.text)

# Streaming
for chunk in model.generate_content("Write a Python web scraper", stream=True):
    print(chunk.text, end="", flush=True)

# Multimodal — image + text
image_part = Part.from_uri("gs://my-bucket/image.jpg", mime_type="image/jpeg")
response = model.generate_content(["What's in this image?", image_part])
print(response.text)

# Count tokens before sending
tokens = model.count_tokens("Long document text here...")
print(f"Token count: {tokens.total_tokens}")

# Function calling
get_weather = FunctionDeclaration(
    name="get_weather",
    description="Get weather for a city",
    parameters={"type": "object", "properties": {"city": {"type": "string"}}}
)
tool = Tool(function_declarations=[get_weather])
response = model.generate_content("What's the weather in Tokyo?", tools=[tool])
print(response.candidates[0].content.parts[0].function_call)

# Embeddings
from vertexai.language_models import TextEmbeddingModel
emb_model = TextEmbeddingModel.from_pretrained("text-embedding-004")
embeddings = emb_model.get_embeddings(["text to embed"])
vector = embeddings[0].values  # 768-dim float list

# Context caching (Gemini 1.5 Pro)
from vertexai.preview.caching import CachedContent
import datetime
cached_content = CachedContent.create(
    model_name="gemini-1.5-pro-001",
    system_instruction="You are an expert...",
    contents=[Part.from_uri("gs://bucket/large-doc.pdf", mime_type="application/pdf")],
    ttl=datetime.timedelta(hours=24),
)
model_with_cache = GenerativeModel.from_cached_content(cached_content)
```

## ⚙️ CONFIGURATION

| Config Key | Type | Default | Description |
|-----------|------|---------|-------------|
| `temperature` | float | 0.9 | Creativity; 0=deterministic, 1=creative |
| `max_output_tokens` | int | 2048 | Max tokens in response |
| `top_p` | float | 0.95 | Nucleus sampling probability |
| `top_k` | int | 40 | Top-k token sampling |
| `candidate_count` | int | 1 | Number of response candidates |
| `stop_sequences` | list | [] | Stop generation at these strings |
| `location` | str | us-central1 | GCP region for Vertex AI |
| `project` | str | required | GCP project ID |
| `model_name` | str | gemini-1.5-pro | Model version |
| `safety_threshold` | enum | BLOCK_MEDIUM_AND_ABOVE | Harm block threshold |
| `stream` | bool | False | Enable streaming response |


## 💡 TIPS AND TRICKS

### Performance Tips
1. **Cache aggressively** — cache parsed configs and compiled patterns at process start, not per-request. Source → [HMZ](https://github.com/hmzainjamil)
2. **Batch API calls** — group 10-50 items per request; single-item loops burn quota 10x faster. Source → [HMZ](https://github.com/hmzainjamil)
3. **Use streaming** — for large outputs, stream to avoid OOM on responses >1MB. Source → [HMZ](https://github.com/hmzainjamil)

### Developer Productivity
1. **Alias long commands** — add `alias x='<long cmd>'` to `.zshrc`; saves 30+ keystrokes per run. Source → [HMZ](https://github.com/hmzainjamil)
2. **Use `--dry-run` first** — always preview destructive operations before applying. Source → [HMZ](https://github.com/hmzainjamil)
3. **Log to structured JSON** — `{"ts": ..., "level": ..., "msg": ...}` makes grep/jq filtering instant. Source → [HMZ](https://github.com/hmzainjamil)

### Integration Tips
1. **Webhook before polling** — push events beat pull loops every time; reduces latency from seconds to milliseconds. Source → [HMZ](https://github.com/hmzainjamil)
2. **Schema-validate inputs** — catch malformed payloads at the boundary, not 5 stack frames deep. Source → [HMZ](https://github.com/hmzainjamil)
3. **Version your APIs** — prefix routes with `/v1/`; zero-downtime migrations become trivial. Source → [HMZ](https://github.com/hmzainjamil)

### Debugging Tips
1. **Reproduce with minimal case** — strip to the smallest input that still fails before filing a bug. Source → [HMZ](https://github.com/hmzainjamil)
2. **Check env vars first** — 40% of "broken" configs are wrong PATH or missing API_KEY. Source → [HMZ](https://github.com/hmzainjamil)
3. **Use `set -euxo pipefail`** — in bash scripts, this catches silent failures that cost hours. Source → [HMZ](https://github.com/hmzainjamil)

## 🔧 TROUBLESHOOTING

| Issue | Cause | Fix |
|-------|-------|-----|
| `ModuleNotFoundError` | Missing dep | `pip install -r requirements.txt` |
| `Permission denied` | Wrong file perms | `chmod +x <script>` |
| `Rate limit exceeded` | Too many requests | Add exponential backoff; check Retry-After header |
| `KeyError` in config | Missing env var | Copy `.env.example` to `.env`, fill all values |
| Slow startup | Cold import chain | Lazy-import heavy libs inside functions |
| Timeout on large inputs | Default timeout too short | Set `timeout=120` in client config |
| Unicode decode error | File not UTF-8 | Open with `encoding='utf-8', errors='replace'` |
| `ConnectionRefused` | Service not running | Check port with `lsof -i :<port>` |
| Import cycle | Circular dependency | Move shared types to a `types.py` module |
| Memory leak | Unbounded cache | Add LRU eviction: `@lru_cache(maxsize=512)` |

## 📊 ARCHITECTURE

```
┌────────────────────────────────────────┐
│              Entry Point               │
│     CLI / API / Web / Webhook          │
└────────────────┬───────────────────────┘
                 │
         ┌───────▼────────┐
         │   Core Engine  │
         │ config + parse │
         └───────┬────────┘
         ┌───────▼────────┐
         │  Service Layer │
         │  business logic│
         └───────┬────────┘
         ┌───────▼────────┐
         │  Data / Output │
         │  DB / files    │
         └────────────────┘
```

## 🗺️ ROADMAP

| Priority | Feature | Status |
|----------|---------|--------|
| P0 | Core stability & 90% test coverage | In Progress |
| P0 | GitHub Actions CI/CD | In Progress |
| P1 | Docker image + one-command deploy | Planned |
| P1 | WebSocket / streaming support | Planned |
| P2 | Plugin/extension system | Planned |
| P2 | Dashboard UI | Planned |
| P2 | Multi-language support | Planned |
| P3 | Kubernetes manifests | Planned |
| P3 | Analytics + observability hooks | Planned |
| P3 | Enterprise SSO | Exploring |

## ☠️ STARTUPS / BUSINESSES

| What This Replaces | Old Cost | With This Repo | Saving |
|-------------------|----------|---------------|--------|
| Custom dev agency build | $15,000-50,000 | Free + self-hosted | 100% |
| SaaS subscription (annual) | $1,200-6,000/yr | One-time setup | 100%/yr |
| Consultant hourly | $150-300/hr x 40hrs | 2hr setup | 95% |
| Proprietary vendor lock-in | High migration cost | Open source, fork freely | unlimited |
| Internal tooling team | 1-2 FTE | 1 developer part-time | 80% |

**Why this matters for businesses:**
- Ship 10x faster by not reinventing foundational tooling
- Zero vendor risk — code is yours, fork it, own it
- No per-seat pricing — deploy to 1 or 1,000 users, same cost
- Audit trail via git — every change is tracked, reviewable, rollbackable


## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=hmzainjamil/awesome-gemini-for-google-cloud&type=Date)](https://star-history.com/#hmzainjamil/awesome-gemini-for-google-cloud&Date)

---

Built by [HMZ](https://github.com/hmzainjamil)
