# awesome-gemini-for-google-cloud

> **The definitive operator's index for Gemini on Google Cloud** — production patterns, vertex AI recipes, and real deployment configs in one place.

<p align="center">
  <a href="https://github.com/hmzainjamil/awesome-gemini-for-google-cloud/stargazers"><img src="https://img.shields.io/github/stars/hmzainjamil/awesome-gemini-for-google-cloud?style=for-the-badge&labelColor=555&color=yellow" alt="Stars"></a>
  <a href="https://github.com/hmzainjamil/awesome-gemini-for-google-cloud/forks"><img src="https://img.shields.io/github/forks/hmzainjamil/awesome-gemini-for-google-cloud?style=for-the-badge&labelColor=555&color=blue" alt="Forks"></a>
  <a href="https://github.com/hmzainjamil/awesome-gemini-for-google-cloud/issues"><img src="https://img.shields.io/github/issues/hmzainjamil/awesome-gemini-for-google-cloud?style=for-the-badge&labelColor=555&color=red" alt="Issues"></a>
  <a href="https://github.com/hmzainjamil/awesome-gemini-for-google-cloud/pulls"><img src="https://img.shields.io/github/issues-pr/hmzainjamil/awesome-gemini-for-google-cloud?style=for-the-badge&labelColor=555&color=green" alt="PRs"></a>
  <a href="https://github.com/hmzainjamil/awesome-gemini-for-google-cloud/commits/main"><img src="https://img.shields.io/github/last-commit/hmzainjamil/awesome-gemini-for-google-cloud?style=for-the-badge&labelColor=555" alt="Last Commit"></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Gemini-4285F4?style=flat&labelColor=555&logo=google" alt="Gemini">
  <img src="https://img.shields.io/badge/Vertex_AI-34A853?style=flat&labelColor=555&logo=googlecloud" alt="Vertex AI">
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat&labelColor=555&logo=python" alt="Python">
  <img src="https://img.shields.io/badge/GCP-FBBC05?style=flat&labelColor=555&logo=googlecloud" alt="GCP">
  <img src="https://img.shields.io/badge/Terraform-7B42BC?style=flat&labelColor=555&logo=terraform" alt="Terraform">
</p>

---

## Why This Exists

Google's official docs scatter Gemini patterns across 40+ pages. No one curates the production-ready stuff. This repo is a single operator-grade index: working code, real configs, tested patterns. No tutorials. No "hello world". Only stuff that ships.

Built for ML engineers who bill by compute, not tutorials. Every link here has been run in a real GCP project.

---

## At a Glance

| Dimension | Detail |
|---|---|
| **Primary API** | Vertex AI Gemini — `aiplatform.googleapis.com` |
| **Models covered** | Gemini 2.0 Flash, Gemini 1.5 Pro, Gemini 1.5 Flash, Gemini Ultra |
| **Modalities** | Text, vision, audio, video, code, structured JSON |
| **Auth methods** | ADC, Service Account JSON, Workload Identity |
| **SDK** | `google-cloud-aiplatform>=1.49`, `google-generativeai>=0.5` |
| **Infra tools** | Terraform, Cloud Build, Artifact Registry |
| **Orchestration** | LangChain, LangGraph, VertexAI Pipelines, Kubeflow |
| **Cost control** | Quotas, committed use, batch prediction |
| **Regions** | `us-central1`, `us-east4`, `europe-west4`, `asia-northeast1` |
| **Latency targets** | Streaming first-token <400ms, batch throughput 1k req/min |
| **Safety** | DLP integration, safety filters, CSAM detection |
| **Compliance** | CMEK, VPC-SC, Audit Logs, HIPAA BAA eligible |

---

## 🧠 CONCEPTS

| Concept | Description | Why It Matters |
|---|---|---|
| **Vertex AI Endpoint** | Managed HTTPS endpoint for model inference | Prod-stable, autoscales, SLA-backed |
| **Grounding** | Augment model responses with Google Search or custom data | Reduces hallucination in factual tasks |
| **Function Calling** | Model emits structured tool-call payloads instead of text | Enables reliable agent actions |
| **Multimodal Input** | Pass image/audio/video alongside text in a single request | Vision, audio analysis, video QA |
| **Safety Filters** | Block/flag content by category (harassment, hate, sexual, dangerous) | Mandatory for consumer apps |
| **System Instruction** | Persistent context injected before every turn | Sets persona, constraints, output format |
| **Streaming** | Server-Sent Events (SSE) delivery of tokens as generated | Sub-second UX for chat apps |
| **Batch Prediction** | Async job over a JSONL file in GCS | Cost-efficient for bulk inference |
| **CMEK** | Customer-Managed Encryption Keys via Cloud KMS | Data sovereignty, compliance |
| **Workload Identity** | GKE pods auth to GCP APIs without service account keys | Key-less, zero-secret deployments |
| **Model Garden** | Curated registry of OSS + Google models on Vertex | One-click deploy of Llama, Gemma, etc. |
| **Eval Framework** | Built-in evaluation pipelines (ROUGE, BLEU, custom) | Measure quality before prod rollout |

### 🔥 Hot

| Resource | Type | Why |
|---|---|---|
| `vertexai.generative_models.GenerativeModel` | Python class | Core SDK entry point — wrap once, reuse everywhere |
| Gemini 2.0 Flash | Model | Best cost-to-performance ratio for production |
| Function Calling + Grounding combo | Pattern | Agentic grounded search in 20 lines |
| Batch Prediction via GCS JSONL | Pattern | 10x cheaper than online for offline workloads |
| VPC-SC + Private Service Connect | Security | Required for financial/healthcare data |

---

## ⚙️ HOW IT WORKS

```
Developer → Vertex AI API → Gemini Model
                ↑                ↓
         Auth (ADC/SA)      Response (text/JSON/stream)
                ↑
         System Instruction + Safety Config
```

1. **Auth**: ADC (`gcloud auth application-default login`) or service account key exported as `GOOGLE_APPLICATION_CREDENTIALS`
2. **Initialize SDK**: `vertexai.init(project=PROJECT_ID, location=REGION)`
3. **Load model**: `model = GenerativeModel("gemini-2.0-flash-001")`
4. **Generate**: `response = model.generate_content(contents)`
5. **Stream**: `for chunk in model.generate_content(contents, stream=True)`
6. **Functions**: Pass `tools=[Tool(function_declarations=[...])]` to enable structured output
7. **Grounding**: Add `tools=[Tool.from_google_search_retrieval(GoogleSearchRetrieval())]`

---

## 🚀 INSTALL

### Prerequisites

```bash
# GCP project with billing enabled
gcloud projects create $PROJECT_ID
gcloud config set project $PROJECT_ID

# Enable required APIs
gcloud services enable \
  aiplatform.googleapis.com \
  storage.googleapis.com \
  cloudbuild.googleapis.com \
  artifactregistry.googleapis.com
```

### Python SDK

```bash
pip install google-cloud-aiplatform>=1.49 google-generativeai>=0.5
```

### Auth

```bash
# Local dev
gcloud auth application-default login

# CI/CD — create service account
gcloud iam service-accounts create gemini-runner \
  --display-name "Gemini Runner"

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member "serviceAccount:gemini-runner@${PROJECT_ID}.iam.gserviceaccount.com" \
  --role "roles/aiplatform.user"

gcloud iam service-accounts keys create key.json \
  --iam-account gemini-runner@${PROJECT_ID}.iam.gserviceaccount.com

export GOOGLE_APPLICATION_CREDENTIALS=key.json
```

### Terraform (infra-as-code)

```hcl
resource "google_project_service" "vertex" {
  service = "aiplatform.googleapis.com"
}

resource "google_service_account" "gemini" {
  account_id   = "gemini-runner"
  display_name = "Gemini Runner SA"
}

resource "google_project_iam_member" "gemini_user" {
  project = var.project_id
  role    = "roles/aiplatform.user"
  member  = "serviceAccount:${google_service_account.gemini.email}"
}
```

---

## 📟 USAGE

### Basic generation

```python
import vertexai
from vertexai.generative_models import GenerativeModel

vertexai.init(project="my-project", location="us-central1")
model = GenerativeModel("gemini-2.0-flash-001")

response = model.generate_content("Summarize the Transformer architecture in 3 bullets.")
print(response.text)
```

### Multimodal (image + text)

```python
from vertexai.generative_models import GenerativeModel, Image

model = GenerativeModel("gemini-2.0-flash-001")
image = Image.load_from_file("screenshot.png")
response = model.generate_content(["What's wrong with this UI?", image])
print(response.text)
```

### Streaming

```python
for chunk in model.generate_content("Write a 500-word essay.", stream=True):
    print(chunk.text, end="", flush=True)
```

### Function calling

```python
from vertexai.generative_models import FunctionDeclaration, Tool

get_weather = FunctionDeclaration(
    name="get_weather",
    description="Returns current weather for a city",
    parameters={"type": "object", "properties": {"city": {"type": "string"}}, "required": ["city"]}
)
tool = Tool(function_declarations=[get_weather])
response = model.generate_content("What's the weather in Tokyo?", tools=[tool])
print(response.candidates[0].content.parts[0].function_call)
```

### Batch prediction

```bash
# Upload input JSONL to GCS
gsutil cp requests.jsonl gs://my-bucket/input/

# Submit batch job
gcloud ai batch-prediction-jobs create \
  --display-name="gemini-batch-01" \
  --model=publishers/google/models/gemini-2.0-flash-001 \
  --bigquery-source=bq://project.dataset.table \
  --bigquery-destination-prefix=bq://project.dataset.output \
  --region=us-central1
```

### Grounded search

```python
from vertexai.generative_models import Tool, grounding

model = GenerativeModel("gemini-2.0-flash-001")
tool = Tool.from_google_search_retrieval(grounding.GoogleSearchRetrieval())
response = model.generate_content("Latest Gemini 2.0 benchmarks?", tools=[tool])
print(response.text)
```

---

## ⚙️ CONFIGURATION

| Parameter | Type | Default | Description |
|---|---|---|---|
| `temperature` | float | 1.0 | Sampling temperature — lower = deterministic |
| `top_p` | float | 0.95 | Nucleus sampling threshold |
| `top_k` | int | 40 | Top-K sampling vocab size |
| `max_output_tokens` | int | 8192 | Max tokens in response |
| `candidate_count` | int | 1 | Number of candidate responses |
| `stop_sequences` | list[str] | [] | Strings that halt generation |
| `safety_settings` | dict | medium | Per-category harm thresholds |
| `system_instruction` | str | None | Injected before every request |
| `response_mime_type` | str | text/plain | Force `application/json` for structured output |
| `response_schema` | Schema | None | Pydantic/JSON schema for constrained output |
| `grounding_config` | Tool | None | Attach Google Search or custom retrieval |
| `tool_config` | ToolConfig | AUTO | Force/disable specific tool calls |

```python
from vertexai.generative_models import GenerationConfig, SafetySetting, HarmCategory, HarmBlockThreshold

config = GenerationConfig(
    temperature=0.2,
    top_p=0.9,
    max_output_tokens=4096,
    response_mime_type="application/json"
)

safety = [
    SafetySetting(category=HarmCategory.HARM_CATEGORY_HATE_SPEECH, threshold=HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE),
    SafetySetting(category=HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT, threshold=HarmBlockThreshold.BLOCK_ONLY_HIGH),
]

model = GenerativeModel("gemini-2.0-flash-001", generation_config=config, safety_settings=safety)
```

---

## 💡 TIPS AND TRICKS

### Cost Optimization

| Tip | Detail | Source |
|---|---|---|
| Use Flash for bulk tasks | Gemini 2.0 Flash is 10x cheaper than Pro with 90% of the quality | [HMZ](https://github.com/hmzainjamil) |
| Enable batch prediction | Async JSONL batch jobs are ~50% cheaper than online prediction | [HMZ](https://github.com/hmzainjamil) |
| Cache system instructions | System instructions count toward input tokens every request — keep them short | [HMZ](https://github.com/hmzainjamil) |

### Latency Reduction

| Tip | Detail | Source |
|---|---|---|
| Stream all responses | SSE streaming makes UX feel 5x faster — first token <400ms | [HMZ](https://github.com/hmzainjamil) |
| Use `us-central1` region | Lowest latency for most North America traffic | [HMZ](https://github.com/hmzainjamil) |
| Avoid large context re-sends | Use multi-turn chat history, not repeated full-document sends | [HMZ](https://github.com/hmzainjamil) |

### Safety & Compliance

| Tip | Detail | Source |
|---|---|---|
| Set safety filters explicitly | Don't rely on defaults — set each category threshold in code | [HMZ](https://github.com/hmzainjamil) |
| Use VPC-SC for sensitive data | Prevents data exfiltration to unauthorized Google APIs | [HMZ](https://github.com/hmzainjamil) |
| Enable Cloud Audit Logs | Log every API call for SOC 2 / HIPAA evidence | [HMZ](https://github.com/hmzainjamil) |

### Production Patterns

| Tip | Detail | Source |
|---|---|---|
| Wrap in retry logic | Vertex returns 429/503 under load — use `tenacity` with exponential backoff | [HMZ](https://github.com/hmzainjamil) |
| Use `response_schema` for agents | Pydantic schema enforces structured tool output — prevents parsing errors | [HMZ](https://github.com/hmzainjamil) |
| Pin model versions | Use `gemini-2.0-flash-001` not `gemini-2.0-flash` — avoid silent upgrades | [HMZ](https://github.com/hmzainjamil) |

---

## 🔧 TROUBLESHOOTING

| Issue | Cause | Fix |
|---|---|---|
| `403 Permission denied` | Service account missing `roles/aiplatform.user` | `gcloud projects add-iam-policy-binding ... --role roles/aiplatform.user` |
| `429 Resource exhausted` | Quota exceeded for requests-per-minute | Request quota increase or add exponential backoff |
| `API not enabled` | `aiplatform.googleapis.com` not enabled | `gcloud services enable aiplatform.googleapis.com` |
| `Blocked by safety filters` | Response triggered harm threshold | Lower threshold or rephrase prompt |
| `Empty response text` | Function call returned instead of text | Check `response.candidates[0].content.parts[0].function_call` |
| `ADC not found` | No credentials configured | Run `gcloud auth application-default login` |
| `Region not supported` | Model not available in that region | Use `us-central1` or check Vertex AI region table |
| `Max tokens exceeded` | Input + output > context limit | Chunk input or use a model with larger context window |

---

## 📊 ARCHITECTURE

```
┌─────────────────────────────────────────────────────────────┐
│                        Your Application                      │
│   Python / Node / Java / REST                                │
└────────────────────────┬────────────────────────────────────┘
                         │
              google-cloud-aiplatform SDK
                         │
┌────────────────────────▼────────────────────────────────────┐
│                    Vertex AI API                             │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────────┐  │
│  │  Gemini 2.0 │  │  Grounding   │  │  Function Calling  │  │
│  │  Flash/Pro  │  │  (Search)    │  │  (Tool Use)        │  │
│  └─────────────┘  └──────────────┘  └────────────────────┘  │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────────┐  │
│  │  Safety     │  │  Batch Pred  │  │  Eval Framework    │  │
│  │  Filters    │  │  (GCS/BQ)    │  │  (ROUGE/Custom)    │  │
│  └─────────────┘  └──────────────┘  └────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
         │                   │                    │
    Cloud Storage       BigQuery            Cloud Logging
    (batch I/O)         (results)           (audit trail)
```

---

## 🗺️ ROADMAP

| Priority | Feature | Status |
|---|---|---|
| P0 | Gemini 2.0 Flash recipes | ✅ Done |
| P0 | Function calling patterns | ✅ Done |
| P0 | Grounding with Google Search | ✅ Done |
| P1 | Batch prediction cookbook | 🔄 In Progress |
| P1 | LangChain + Vertex integration | 🔄 In Progress |
| P1 | Terraform module for Vertex setup | 📅 Planned |
| P2 | RAG with Vertex AI Search | 📅 Planned |
| P2 | Gemma 2 fine-tuning on Vertex | 📅 Planned |
| P2 | Multi-agent patterns (ADK) | 📅 Planned |
| P3 | Cost dashboard with BigQuery | 📅 Planned |

---

## ☠️ STARTUPS / BUSINESSES

| Old Way | What This Replaces | Disruption Level |
|---|---|---|
| OpenAI API | Drop-in replacement for GPT-4 with GCP native billing | 🔥 High |
| Custom ML infra | Replace on-prem GPU servers with Vertex managed endpoints | 💀 Total |
| Proprietary RAG vendors | Vertex AI Search + Gemini = enterprise RAG in 1 day | 🔥 High |
| Manual QA annotation | Gemini batch prediction scores quality at scale | 🔥 High |
| OCR SaaS tools | Gemini vision handles document parsing natively | 🔥 High |
| Translation APIs | Gemini 1.5 Pro matches DeepL quality with added context awareness | 🔥 High |
| Data extraction contractors | Batch JSONL + Gemini extracts structured data from unstructured docs | 💀 Total |

---

## Resources

- [Vertex AI Gemini API Docs](https://cloud.google.com/vertex-ai/generative-ai/docs)
- [google-cloud-aiplatform Python SDK](https://github.com/googleapis/python-aiplatform)
- [Gemini Cookbook (Official)](https://github.com/google-gemini/cookbook)
- [Vertex AI Pricing](https://cloud.google.com/vertex-ai/pricing)
- [Model Garden](https://cloud.google.com/vertex-ai/docs/model-garden/explore-models)
- [Grounding with Google Search](https://cloud.google.com/vertex-ai/generative-ai/docs/grounding/overview)
- [Function Calling Guide](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/function-calling)
- [Batch Prediction](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/batch-prediction-gemini)
- [Safety Filters Reference](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/configure-safety-filters)
- [Vertex AI Quotas](https://cloud.google.com/vertex-ai/quotas)
- [VPC Service Controls](https://cloud.google.com/vertex-ai/docs/general/vpc-service-controls)
- [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation)
- [CMEK for Vertex AI](https://cloud.google.com/vertex-ai/docs/general/cmek)
- [Cloud Audit Logs](https://cloud.google.com/logging/docs/audit)
- [ADK (Agent Development Kit)](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-builder/overview)
- [LangChain Vertex Integration](https://python.langchain.com/docs/integrations/llms/google_vertex_ai_palm)
- [Gemini for Workspace](https://workspace.google.com/products/gemini/)
- [Vertex AI Pipelines](https://cloud.google.com/vertex-ai/docs/pipelines/introduction)
- [Model Evaluation](https://cloud.google.com/vertex-ai/generative-ai/docs/models/evaluate-models)
- [Fine-tuning Gemini](https://cloud.google.com/vertex-ai/generative-ai/docs/models/tune-models)

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=hmzainjamil/awesome-gemini-for-google-cloud&type=Date)](https://star-history.com/#hmzainjamil/awesome-gemini-for-google-cloud&Date)

---

Built by [HMZ](https://github.com/hmzainjamil)

---

## Contributing

PRs welcome. Requirements:
- Resource must be tested in a real GCP project
- Include the exact API/SDK version it was tested with
- No affiliate links, no tutorials, no vendor marketing copy
- Add to the correct section — don't create new sections without discussion

```bash
# Fork, clone, edit, PR
gh repo fork hmzainjamil/awesome-gemini-for-google-cloud --clone
cd awesome-gemini-for-google-cloud
# edit README.md
gh pr create --title "add: [resource name]" --body "tested on gemini-2.0-flash-001, us-central1"
```

### Formatting rules

- Links: `[Display Name](url)` — no raw URLs
- Code: always specify language in fenced blocks
- Tables: align columns, test render on GitHub
- Descriptions: ≤15 words, start with verb or noun phrase

---

## Related Awesome Lists

| List | Focus |
|---|---|
| [awesome-gemini](https://github.com/martinlindhe/awesome-gemini) | Gemini Protocol (not the model) |
| [awesome-google-cloud](https://github.com/GoogleCloudPlatform/awesome-google-cloud) | GCP services broad |
| [awesome-llm](https://github.com/Hannibal046/Awesome-LLM) | All LLMs |
| [awesome-langchain](https://github.com/kyrolabs/awesome-langchain) | LangChain ecosystem |
| [awesome-claude](https://github.com/hmzainjamil/awesome-claude) | Claude / Anthropic |

---

## License

CC0 1.0 — public domain. No rights reserved. Use freely.

---

## Vertex AI SDK Cheatsheet

```python
# Init
import vertexai
vertexai.init(project="PROJECT", location="us-central1")

# Models
from vertexai.generative_models import GenerativeModel, Part, Image, Video, Audio
model = GenerativeModel("gemini-2.0-flash-001")
pro   = GenerativeModel("gemini-1.5-pro-002")

# Text
r = model.generate_content("prompt")
print(r.text)

# Image
img = Image.load_from_file("img.jpg")
r   = model.generate_content(["describe", img])

# Video
vid = Part.from_uri("gs://bucket/video.mp4", mime_type="video/mp4")
r   = model.generate_content(["summarize this video", vid])

# Audio
aud = Part.from_uri("gs://bucket/audio.mp3", mime_type="audio/mpeg")
r   = model.generate_content(["transcribe", aud])

# Chat (multi-turn)
chat = model.start_chat()
r    = chat.send_message("Hello")
r2   = chat.send_message("What did I just say?")

# JSON output
from vertexai.generative_models import GenerationConfig
cfg = GenerationConfig(response_mime_type="application/json")
r   = model.generate_content("Extract name, age from: John is 25", generation_config=cfg)

# Async
import asyncio
async def gen():
    r = await model.generate_content_async("prompt")
    return r.text

# Count tokens
tokens = model.count_tokens("Your prompt here")
print(tokens.total_tokens)
```

### Environment variable pattern

```bash
export GOOGLE_CLOUD_PROJECT="my-project"
export GOOGLE_CLOUD_REGION="us-central1"
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/key.json"
```

```python
import os, vertexai
vertexai.init(
    project=os.environ["GOOGLE_CLOUD_PROJECT"],
    location=os.environ["GOOGLE_CLOUD_REGION"]
)
```
