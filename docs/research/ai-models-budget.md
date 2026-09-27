# AI models that fit a near-zero budget

Research for the ticket "AI models that fit a near-zero budget" on the map "Map: smart-modeling v1 spec".
All prices, limits and terms were checked on **2026-09-26** on the vendors' own pages, linked inline.
Anything I could not confirm is marked **unverified**. Paragraphs marked **Assessment** are my judgment, not facts from a source.

## The question

Which language and vision models can power the tool with a running budget close to zero, understand Portuguese and English plus images, and allow commercial use ([ADR-0001](../adr/0001-reuse-only-commercially-licensed-parts.md))?
The ticket asks for free API tiers (limits, terms, commercial use, what they do with our data), open-weight models that run on the owner's PC (GTX 1650 with 4 GB of video memory, 16 CPU threads, 19 GB RAM under WSL2), the cost of a design session on paid fallbacks, and which models reliably return structured output (JSON or code) for pattern parameters.

A few terms used below:

- **Token**: a word piece. 1,000 tokens is roughly 750 English words. APIs charge per million tokens ("MTok"), separately for input (what we send) and output (what the model writes).
- **Open weights**: the model file itself is published, so it can run on our own machine under its license.
- **Quantization**: storing the model's numbers at about 4 bits instead of 16, which cuts memory about 4x for a small quality loss. **GGUF** is the file format that the local runtimes llama.cpp and Ollama load; a vision model needs an extra vision file (called mmproj).
- **Structured output / constrained decoding**: the runtime only lets the model produce text that matches a JSON schema we supply, so the reply always parses.

## Short answer

- **Free hosted tiers that see images and allow commercial use:** Cloudflare Workers AI (Gemma 4 26B A4B, Qwen 3.8 27B and others), Google's Gemini API free tier (Gemini 3.x Flash models and Gemma 4), OpenRouter's free models (Qwen 3.8 27B, Gemma 4) and Z.ai (GLM-4.6V-Flash). Groq's free plan is very fast and has guaranteed JSON, but its only vision model is a preview "for evaluation purposes only".
- **What they do with our data differs sharply:** Workers AI, Groq and Z.ai (for API customers) do not train on prompts. Gemini's free tier uses prompts to improve Google products and lets human reviewers read them. Mistral's free mode may train on them unless you opt out. OpenRouter depends on which provider serves the free model.
- **Gone or unusable:** GitHub Models was retired on 2026-07-30. NVIDIA's API catalog is evaluation-only and uses your content to improve its models. Cerebras is now a 30-day, $5 trial. Hugging Face gives free users $0.10 of credit a month.
- **Free capacity per 20-call design session:** Workers AI's 10,000 free "neurons" a day cover about 9 sessions with Gemma 4 26B A4B (about 1 with Qwen 3.8 27B). Groq's free plan covers about 2 sessions a day. OpenRouter covers 2.5 sessions a day, or 50 after a one-off $10 credit purchase. Gemini's and Mistral's free limits are only shown inside their consoles (unverified).
- **Local:** the GTX 1650 fits 2 to 4 billion-parameter vision models at 4 bits (Qwen 3.5 2B and 4B, Ministral 3 3B), all Apache 2.0. The CPU with 19 GB RAM can hold Gemma 4 26B A4B (15.6 GB at 4 bits) or gpt-oss-20b (text only), but I estimate each call would take minutes, too slow for a chat-driven session.
- **Paid fallbacks per session** (20 calls of 3,000 input and 1,000 output tokens, plus 3 images): GPT-6 Luna $0.02, Mistral Small 4 $0.02, Gemini 3.5 Flash-Lite $0.07, Gemini 3.8 Flash $0.12 (doubling on 2027-01-01), Claude Haiku 4.5 $0.16, GPT-6 Sol $0.33, Claude Sonnet 5 $0.42, Claude Opus 5.5 $0.85. A heavy session (images kept in every call, extra hidden reasoning) costs about twice as much.
- **Structured output:** Anthropic, OpenAI, Gemini, Groq (strict mode, 3 models), Mistral and the local runtimes llama.cpp and Ollama can force replies to match a JSON schema. Workers AI's JSON mode covers only a few older models and is not guaranteed. A schema guarantees the shape of the answer, not correct values, and each provider supports a different subset of JSON Schema.
- **Recommendation (Assessment):** prototype on Workers AI with Gemma 4 26B A4B (free, no training on our data, Apache 2.0 weights we can also run locally). Keep Gemini 3.8 Flash on a $5 prepaid paid tier and Claude Sonnet 5 as quality fallbacks. Let the ticket "Choose the pattern-generation route and its AI model" pick the final model with a small test set of real Portuguese and English descriptions.

## What the tool asks of a model

In a design session the model reads the owner's description (Portuguese or English) and optional reference images, asks or answers questions in chat, and returns pattern parameters (for example garment family, lengths, ease, neckline type, dart positions) that a pattern engine turns into a pattern.
The ticket's working assumption is about 20 model calls per session, each around 3,000 input and 1,000 output tokens, plus a few images. The route decision (pattern-first or mesh-first) is out of scope here; this report only covers which models can do the language and vision work, at what cost, and how reliably they return structured data.

## 1. Free API tiers

| Service | Free models that accept images | Free limits | Prompts used for training? | Commercial use (ADR-0001) | Structured output |
|---|---|---|---|---|---|
| **Cloudflare Workers AI** | Gemma 4 26B A4B, Qwen 3.8 27B, Llama 4 Scout, Llama 3.2 11B Vision ([Gemma 4 page](https://developers.cloudflare.com/workers-ai/models/gemma-4-26b-a4b-it/), [Qwen 3.8 page](https://developers.cloudflare.com/workers-ai/models/qwen3.8-27b/), [Llama 4 Scout page](https://developers.cloudflare.com/workers-ai/models/llama-4-scout-17b-16e-instruct/), [Llama 3.2 Vision page](https://developers.cloudflare.com/workers-ai/models/llama-3.2-11b-vision-instruct/)) | 10,000 neurons a day, on the free plan too; $0.011 per 1,000 neurons above that on the paid plan ([pricing](https://developers.cloudflare.com/workers-ai/platform/pricing/)) | No: Cloudflare "does not use your Customer Content to (1) train any AI models made available on Workers AI or (2) improve any Cloudflare or third-party services" ([data usage](https://developers.cloudflare.com/workers-ai/platform/data-usage/)) | Yes; each model's own license applies ([data usage](https://developers.cloudflare.com/workers-ai/platform/data-usage/)) | JSON mode only on 6 older text models, and "Workers AI can't guarantee" the schema ([JSON mode](https://developers.cloudflare.com/workers-ai/features/json-mode/)); Gemma 4 and Qwen 3.8 list function calling ([model pages](https://developers.cloudflare.com/workers-ai/models/gemma-4-26b-a4b-it/)) |
| **Google Gemini API, free tier** | Gemini 3.8, 3.7, 3.6 and 3.5 Flash, 3.5 Flash-Lite, 3.1 Flash-Lite, 3 Flash Preview, Gemma 4 31B and 26B A4B; not Gemini 3.1 Pro ([pricing](https://ai.google.dev/gemini-api/docs/pricing), [Gemma on the Gemini API](https://ai.google.dev/gemma/docs/core/gemma_on_gemini_api)) | Per-project limits are shown only in AI Studio and "are not guaranteed" ([rate limits](https://ai.google.dev/gemini-api/docs/rate-limits)); numbers **unverified** | Yes: free-tier prompts and responses are used "to provide, improve, and develop Google products", "human reviewers may read" them, and Google says "Do not submit sensitive, confidential, or personal information" ([terms](https://ai.google.dev/gemini-api/terms)) | Yes, "for professional or business purposes"; 18+ only; users in the EEA, Switzerland or UK must be served on the paid tier; Brazil is an available region ([terms](https://ai.google.dev/gemini-api/terms), [regions](https://ai.google.dev/gemini-api/docs/available-regions)) | JSON Schema subset including `enum`, `minimum`, `maximum`; output is "syntactically correct JSON", values must still be validated ([structured outputs](https://ai.google.dev/gemini-api/docs/structured-output)) |
| **OpenRouter free models** (ids ending in `:free`) | Qwen 3.8 27B, Gemma 4 31B and 26B A4B, Nemotron 3 Nano Omni, dots 3 note preview, Inkling ([models API](https://openrouter.ai/api/v1/models)) | 20 requests a minute; 50 a day, or 1,000 a day once at least $10 of credits has ever been bought ([limits](https://openrouter.ai/docs/api-reference/limits)); OpenRouter itself calls free models "usually not suitable for production use" ([FAQ](https://openrouter.ai/docs/faq)) | OpenRouter stores prompts only if you opt in ([data collection](https://openrouter.ai/docs/guides/privacy/data-collection)); each provider's own policy applies, and an account setting blocks providers that train ([provider logging](https://openrouter.ai/docs/guides/privacy/provider-logging)). The free Gemma 4 endpoints are served by Google AI Studio, and the free Nemotron endpoint links NVIDIA's evaluation-only trial terms ([providers API](https://openrouter.ai/api/v1/providers)) | Yes, including inside our own product, subject to each model's terms; no reselling of API access ([terms](https://openrouter.ai/terms)) | Per endpoint: the free Qwen 3.8 endpoint supports structured outputs, the free Gemma 4 endpoints only `response_format` ([endpoints API](https://openrouter.ai/api/v1/models/qwen/qwen3.8-27b:free/endpoints)) |
| **Groq, free plan** | Only `qwen/qwen3.8-27b`, a Preview model "intended for evaluation purposes only and should not be used in production" ([vision](https://console.groq.com/docs/vision), [models](https://console.groq.com/docs/models)); the production models gpt-oss-120b and gpt-oss-20b are text only | 30 requests a minute, 1,000 a day, 8,000 tokens a minute, 200,000 tokens a day per model ([rate limits](https://console.groq.com/docs/rate-limits)) | No: "Groq is not permitted to use Inputs or Outputs for training" ([Services Agreement](https://console.groq.com/docs/legal/services-agreement)); inference data is not kept by default and zero data retention is open to all ([your data](https://console.groq.com/docs/your-data)) | Yes; the same agreement covers free use, and model terms apply ([Services Agreement](https://console.groq.com/docs/legal/services-agreement)) | Strict mode (constrained decoding, "100% schema adherence") on gpt-oss-20b, gpt-oss-120b and qwen3.8-27b, but not with streaming or tool use ([structured outputs](https://console.groq.com/docs/structured-outputs)) |
| **Mistral, Studio free mode** | Ministral 3 (3B, 8B, 14B), Mistral Small 4, Mistral Medium 3.5 accept images ([Ministral 3 card](https://huggingface.co/mistralai/Ministral-3-3B-Instruct-2512), [Small 4 card](https://huggingface.co/mistralai/Mistral-Small-4-119B-2603), [models](https://docs.mistral.ai/models)) | "The lowest limits, intended for evaluation and prototyping"; numbers only in the admin panel ([help](https://help.mistral.ai/en/articles/698531-why-am-i-hitting-api-rate-limits-and-how-do-i-increase-them)); **unverified** | Yes unless you opt out: "we may use your data (input and output) to train our artificial intelligence models" ([help](https://help.mistral.ai/en/articles/347617-do-you-use-my-user-data-to-train-your-artificial-intelligence-models)) | Yes under the Commercial Terms, effective 2026-09-25 ([terms](https://legal.mistral.ai/terms/commercial-terms-of-service)) | Structured outputs listed for Small 4, Medium 3.5 and Ministral 3 ([Small 4](https://docs.mistral.ai/models/mistral-small-4-0-26-03), [Ministral 3 3B](https://docs.mistral.ai/models/ministral-3-3b-25-12)) |
| **Z.ai** | GLM-4.6V-Flash (vision) free; GLM-4.7-Flash and GLM-4.5-Flash (text) free ([pricing](https://docs.z.ai/guides/overview/pricing)) | Not published; **unverified** | For API customers: "we will not use your User Content for developing or improving Services unless you explicitly agree"; for individual users it may ([terms](https://docs.z.ai/legal-agreement/terms-of-use)) | Integration into your own business is allowed; Singapore law; no use to build competing models ([terms](https://docs.z.ai/legal-agreement/terms-of-use)) | Not checked |
| **Hugging Face Inference Providers** | Routes to many providers | $0.10 of credit a month for free users, $2 for PRO ([pricing](https://huggingface.co/docs/inference-providers/pricing)) | Hugging Face keeps no request bodies; each provider's policy applies ([security](https://huggingface.co/docs/inference-providers/security)) | Provider terms | Provider-dependent |

**Not usable for this product:**

- **GitHub Models**: "As of July 30, 2026, GitHub Models has been fully retired" ([docs](https://docs.github.com/en/github-models)).
- **NVIDIA API catalog** (build.nvidia.com): without a paid subscription "you may only use the API Service for internal testing and evaluation purposes, not in production", and NVIDIA collects "User Content and Generated Content to improve NVIDIA products and services, including AI models" ([API Trial Terms](https://assets.ngc.nvidia.com/products/api-catalog/legal/NVIDIA%20API%20Trial%20Terms%20of%20Service.pdf)). This fails ADR-0001 for production use.
- **Cerebras**: "Is there a permanently free tier? No"; new accounts get $5 of credits that expire after 30 days ([rate limits](https://inference-docs.cerebras.ai/support/rate-limits)).
- **Cohere**: free evaluation keys are "limited to 1,000 API calls a month" ([rate limits](https://docs.cohere.com/docs/rate-limits)).

**How far the free tiers go** (my arithmetic from the published limits, for the ticket's 20-call session with 3 images at about 1,300 tokens each):

- **Workers AI**: about 1,130 neurons per session with Gemma 4 26B A4B, so about 9 sessions a day for free. Qwen 3.8 27B costs about 8,400 neurons (1 session a day) and Llama 4 Scout about 3,100 (3 a day). Beyond the free allowance, a Gemma 4 session costs about $0.01.
- **Groq**: a session is about 86,000 tokens, so the 200,000 tokens-a-day cap allows about 2 sessions. The 8,000 tokens-a-minute cap allows only two 4,000-token calls a minute, so a session takes at least 10 minutes of waiting. Groq is the fastest provider checked: its models page lists about 1,000 tokens a second for gpt-oss-20b, 500 for gpt-oss-120b and 450 for qwen3.8-27b ([models](https://console.groq.com/docs/models)).
- **OpenRouter**: 2.5 sessions a day for free, or 50 a day after a one-off $10 purchase. The $10 stays usable for paid models; OpenRouter adds a 5.5% fee on credit purchases ([pricing](https://openrouter.ai/pricing)).
- **Gemini free tier**: cannot be computed without logging into AI Studio.

**Free tiers churn quickly.** In the last three months GitHub Models was retired, Google limited Gemini 2.5 to projects that already used it ("For any new projects, use our latest models: 3.5 Flash-Lite or 3.8 Flash", [changelog](https://ai.google.dev/gemini-api/docs/changelog), 2026-09-18), and Cerebras's free tier became a trial.

**Assessment:** for a single user, Workers AI with Gemma 4 26B A4B is the strongest truly free option: enough daily capacity, images, Portuguese-capable weights, no training on our data, and an Apache 2.0 model we can move to our own machine if Cloudflare changes terms. It also sits next to Cloudflare's free web hosting, which matters for the open question about tech stack and hosting. Gemini's free tier gives stronger models but trains on the owner's designs and allows human review, so it suits test prompts, not real designs or, later, customers' photos. OpenRouter's free models are a useful backup but depend on third-party providers with uneven uptime and data policies.

## 2. Open-weight models on the owner's PC

**Hardware facts:**

- The GTX 16 series is built on NVIDIA's Turing architecture ([GTX 16 series](https://www.nvidia.com/en-in/geforce/graphics-cards/16-series/)); NVIDIA's compute-capability table lists the GeForce GTX 1650 Ti under 7.5 and does not list the plain GTX 1650 separately ([CUDA GPUs](https://developer.nvidia.com/cuda/gpus)). Ollama supports NVIDIA GPUs "with compute capability 5.0+" ([Ollama GPU](https://docs.ollama.com/gpu)). CUDA inside WSL2 works on "Pascal and later" GeForce cards using only the Windows driver ([CUDA on WSL](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)).
- Measured data point: a GTX 1650 **Mobile** running a 7-billion-parameter model at 4 bits (3.56 GiB) with 30 of 33 layers on the GPU read the prompt at about 310 tokens a second and wrote at about 22 tokens a second (llama.cpp, Vulkan backend, [benchmark comment](https://github.com/ggml-org/llama.cpp/discussions/10879#discussioncomment-13503953)).
- The local runtimes llama.cpp and Ollama are both MIT-licensed ([llama.cpp](https://github.com/ggml-org/llama.cpp), [Ollama](https://github.com/ollama/ollama)). llama.cpp can keep the "Mixture of Experts" weights of a MoE model in system RAM and the rest on the GPU (`--cpu-moe`, [server docs](https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md)).

**Candidates** (sizes are real file sizes of 4-bit GGUF builds; "vision file" is the separate mmproj file):

| Model | License | Images | Portuguese | 4-bit file + vision file | Fits where on this PC |
|---|---|---|---|---|---|
| [Qwen 3.5 2B](https://huggingface.co/unsloth/Qwen3.5-2B-GGUF) | Apache 2.0 | Yes | "201 languages and dialects" ([card](https://huggingface.co/Qwen/Qwen3.5-4B)); Portuguese not named | 1.28 + 0.67 GB | GPU, with room for context |
| [Qwen 3.5 4B](https://huggingface.co/Qwen/Qwen3.5-4B) | Apache 2.0 | Yes | Same | 2.74 + 0.67 GB ([GGUF](https://huggingface.co/unsloth/Qwen3.5-4B-GGUF)) | GPU, tight |
| [Ministral 3 3B](https://huggingface.co/mistralai/Ministral-3-3B-Instruct-2512) | Apache 2.0 | Yes | Portuguese listed | 2.15 + 0.84 GB ([official GGUF](https://huggingface.co/mistralai/Ministral-3-3B-Instruct-2512-GGUF)) | GPU |
| [Gemma 4 E2B](https://huggingface.co/google/gemma-4-E2B-it-qat-q4_0-gguf) | Apache 2.0 | Yes (and audio) | "35+ languages" out of the box ([card](https://huggingface.co/google/gemma-4-E4B-it)); Portuguese not named | 3.35 + 0.99 GB (official 4-bit build) | GPU plus some RAM |
| [Gemma 4 E4B](https://huggingface.co/google/gemma-4-E4B-it-qat-q4_0-gguf) | Apache 2.0 | Yes (and audio) | Same | 5.15 + 0.99 GB | CPU, or GPU and RAM split |
| [Qwen 3.5 9B](https://huggingface.co/unsloth/Qwen3.5-9B-GGUF) | Apache 2.0 | Yes | As Qwen above | 5.68 + 0.92 GB | CPU, or split |
| [Ministral 3 8B](https://huggingface.co/mistralai/Ministral-3-8B-Instruct-2512-GGUF) | Apache 2.0 | Yes | Portuguese listed | 5.20 + 0.86 GB | CPU, or split |
| [Gemma 4 12B](https://huggingface.co/google/gemma-4-12B-it-qat-q4_0-gguf) | Apache 2.0 | Yes | As Gemma above | 6.98 + 0.18 GB | CPU |
| [Gemma 4 26B A4B](https://huggingface.co/google/gemma-4-26B-A4B-it-qat-q4_0-gguf) (MoE, 3.8B active) | Apache 2.0 | Yes | As Gemma above | 14.44 + 1.19 GB | CPU, fits 19 GB with little spare |
| [gpt-oss-20b](https://huggingface.co/openai/gpt-oss-20b) (MoE, 3.6B active) | Apache 2.0 | No (text only) | Not checked | "runs within 16GB of memory" | CPU, tight |
| [Qwen 3.6 35B A3B](https://huggingface.co/unsloth/Qwen3.6-35B-A3B-GGUF) | Apache 2.0 | Yes | As Qwen above | 17.7 to 22 GB at about 4 bits | Does not fit comfortably |
| [Qwen 3.8 27B](https://huggingface.co/Qwen/Qwen3.8-27B) | Apache 2.0 | Yes | As Qwen above | 16.5 GB ([GGUF](https://huggingface.co/unsloth/Qwen3.8-27B-GGUF)) | Too big and too slow locally; use it hosted |

Notes on these models:

- **Gemma 4 is now Apache 2.0** ([license page](https://ai.google.dev/gemma/docs/gemma_4_license)); earlier versions such as Gemma 3 and 3n stay under Google's Gemma Terms of Use and its Prohibited Use Policy ([Gemma terms](https://ai.google.dev/gemma/terms)). All Gemma 4 sizes read images, have "native function-calling support", and Google publishes official 4-bit builds ([card](https://huggingface.co/google/gemma-4-E4B-it)).
- **Qwen 3.5** (February 2026) is a vision-language family from 0.8B to 397B parameters, all tagged Apache 2.0 on Hugging Face ([Qwen models](https://huggingface.co/Qwen)). Qwen does not publish its own GGUF builds of Qwen 3.5; the sizes above come from a community build (Unsloth).
- **Ministral 3 and Mistral Small 4** list Portuguese by name and describe "native function calling and JSON output" ([Ministral 3 3B card](https://huggingface.co/mistralai/Ministral-3-3B-Instruct-2512), [Small 4 card](https://huggingface.co/mistralai/Mistral-Small-4-119B-2603)). Mistral Small 4 has 119B parameters, too large locally.

**Licenses to avoid or watch** under ADR-0001:

- **Llama 3.2 Vision**: "for image+text applications, English is the only language supported" ([model card](https://github.com/meta-llama/llama-models/blob/main/models/llama3_2/MODEL_CARD_VISION.md)), and its multimodal rights are not granted to individuals or companies based in the EU ([use policy](https://github.com/meta-llama/llama-models/blob/main/models/llama3_2/USE_POLICY.md)). Meta has published no newer Llama on Hugging Face since Llama 4 in April 2025 ([meta-llama](https://huggingface.co/meta-llama)), and Llama 4 (17B active parameters, 16 or 128 experts) is too large for this PC.
- **Phi-4-multimodal** (MIT): vision supports English only ([card](https://huggingface.co/microsoft/Phi-4-multimodal-instruct)).
- **Qwen licenses vary by model**: Qwen 3.8 Flash-Next uses a "qwen-community-1.0" license (not reviewed, [card](https://huggingface.co/Qwen/Qwen3.8-Flash-Next)), and older Qwen 2.5 3B is tagged "other" rather than Apache 2.0 ([Qwen GGUF list](https://huggingface.co/Qwen)). Check the tag of each exact model.
- **Mistral Medium 3.5** ("Modified MIT"): no rights if the company's monthly revenue exceeds $20 million ([license](https://huggingface.co/mistralai/Mistral-Medium-3.5-128B/blob/main/LICENSE)). Irrelevant today, worth recording before a sale.

**Speed on this PC (Assessment, not measured):** from the measured 22 tokens a second above, a 2 to 4B model fully on the GPU should write roughly 25 to 40 tokens a second, so a 1,000-token reply takes 25 to 40 seconds and a 20-call session spends 10 to 15 minutes waiting on the model. On the CPU, token speed is limited by memory bandwidth; Gemma 4 26B A4B touches only 3.8B parameters per token, so I would expect single-digit to low double-digit tokens a second for writing, but reading a 3,000-token prompt on the CPU could take a minute or more, making each call take minutes. Local models also tie the web app to the owner's PC being switched on.

**Assessment:** local models are a good zero-cost, private option for development, offline work and simple steps (classifying a request, pulling named values out of a description), using Qwen 3.5 4B or Ministral 3 3B on the GPU. They are a poor main engine for a chat-driven design session. The published scores of the small sizes are far below the larger ones: on Gemma 4's own card, the E2B and E4B models score 24.5% and 42.2% on the Tau2 tool-use benchmark versus 68.2% for 26B A4B and 76.9% for 31B, and 44% and 52% on LiveCodeBench versus 77.1% and 80% ([card](https://huggingface.co/google/gemma-4-E4B-it)).

## 3. Paid fallbacks: cost per design session

Assumptions: 20 calls of 3,000 input and 1,000 output tokens, plus 3 reference images resized to about 1 megapixel and sent once. Image tokens: about 1,300 on Claude ([vision](https://platform.claude.com/docs/en/build-with-claude/vision): a 1000x1000 image is 1,296 tokens), 1,120 on Gemini 3 at default resolution ([media resolution](https://ai.google.dev/gemini-api/docs/media-resolution)), 2,048 on Groq ([vision](https://console.groq.com/docs/vision)), and about 1,300 assumed elsewhere. Claude 4.7 and later models use a tokenizer that "produces approximately 30% more tokens for the same text" ([pricing](https://platform.claude.com/docs/en/about-claude/pricing)), so their text tokens are counted at 1.3x. The **heavy** column keeps the 3 images in context on every call and adds 1,000 hidden reasoning ("thinking") tokens per call, which most vendors bill as output. No prompt caching in either column.

| Model | Price per MTok (input / output) | Takes images | Base session | Heavy session |
|---|---|---|---|---|
| GPT-6 Luna | $0.10 / $0.50 ([pricing](https://developers.openai.com/api/docs/pricing), [model](https://developers.openai.com/api/docs/models/gpt-6-luna)) | Yes | $0.016 | $0.034 |
| Mistral Small 4 | $0.15 / $0.60 ([model](https://docs.mistral.ai/models/mistral-small-4-0-26-03)) | Yes | $0.022 | $0.045 |
| Workers AI Gemma 4 26B A4B, beyond the free allowance | $0.10 / $0.30 ([pricing](https://developers.cloudflare.com/workers-ai/platform/pricing/)) | Yes | $0.012 | $0.026 |
| Gemini 3.5 Flash-Lite | $0.30 / $2.50 ([pricing](https://ai.google.dev/gemini-api/docs/pricing)) | Yes | $0.069 | $0.138 |
| Gemini 3.8 Flash, through 2026-12-31 | $0.75 / $3.75 | Yes | $0.123 | $0.245 |
| Gemini 3.8 Flash, from 2027-01-01 | $1.50 / $7.50 | Yes | $0.245 | $0.491 |
| Groq Qwen 3.8 27B (paid plan, preview model) | $0.80 / $4.00 ([models](https://console.groq.com/docs/models)) | Yes | $0.133 | $0.306 |
| Claude Haiku 4.5 | $1 / $5 ([pricing](https://platform.claude.com/docs/en/about-claude/pricing)) | Yes | $0.164 | $0.338 |
| Mistral Medium 3.5 | $1.50 / $7.50 ([model](https://docs.mistral.ai/models/mistral-medium-3-5-26-04)) | Yes | $0.246 | $0.507 |
| GPT-6 Sol | $2 / $10 ([model](https://developers.openai.com/api/docs/models/gpt-6-sol)) | Yes | $0.328 | $0.676 |
| Gemini 3.1 Pro Preview | $2 / $12 | Yes | $0.367 | $0.734 |
| Claude Sonnet 5 | $2 / $10 | Yes | $0.424 | $0.832 |
| Claude Opus 5.5 | $4 / $20 | Yes | $0.848 | $1.664 |
| Claude Opus 5 | $5 / $25 | Yes | $1.060 | $2.080 |

What else moves the bill:

- **Prompt caching** cuts repeated input: a cache hit costs 0.1x the input price on most Claude models and 0.05x on Opus 5.5 ([pricing](https://platform.claude.com/docs/en/about-claude/pricing)); Gemini and OpenAI list cached-input prices too. A chat session resends its history each call, so caching matters.
- **Hidden reasoning** is billed as output: Gemini's price is "Output price (including thinking tokens)", and Gemini 3.8 Flash cannot switch thinking below "low" ([model page](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash)).
- **Minimum top-ups**: Gemini's paid tier needs a prepayment of at least $5, and prepaid credits expire after 12 months ([billing](https://ai.google.dev/gemini-api/docs/billing)). Anthropic gives new users "a small amount of free credits" ([pricing](https://platform.claude.com/docs/en/about-claude/pricing)).
- **Data on paid tiers**: Anthropic "may not train models on Customer Content" ([commercial terms](https://www.anthropic.com/legal/commercial-terms)); OpenAI API data "is not used to train or improve OpenAI models (unless you explicitly opt in)" ([your data](https://developers.openai.com/api/docs/guides/your-data)); on Gemini's paid tier "Google doesn't use your prompts ... or responses to improve our products" ([terms](https://ai.google.dev/gemini-api/terms)).
- **Portuguese** tends to use more tokens than English for the same meaning on most tokenizers; I found no vendor figure for this, so the table does not include it (**unverified**).

**Assessment:** every mainstream paid option costs under $1 for a base session, and 30 sessions a month on Claude Sonnet 5 would be about $13. A $5 Gemini prepayment covers about 40 base sessions on Gemini 3.8 Flash at the current price, with no training on the owner's designs. Paid use is therefore a small, controllable cost rather than a budget risk, as long as the app caps calls per session.

## 4. Structured output for pattern parameters

| Where | Mechanism | Guarantee | Limits that matter for pattern parameters |
|---|---|---|---|
| Anthropic (Haiku 4.5, Sonnet 5, Opus 5 and 5.5, others) | Structured outputs (`output_config.format`) and strict tool use, generally available ([docs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)) | Schema-valid output by constrained decoding | No `minimum` / `maximum` / `multipleOf`, no string length limits, no recursive schemas (the Python and TypeScript SDKs move such limits into field descriptions and check the reply locally); a reply cut off by `max_tokens` can be incomplete |
| OpenAI GPT-6 Luna and Sol | Structured Outputs listed as supported ([Luna](https://developers.openai.com/api/docs/models/gpt-6-luna), [Sol](https://developers.openai.com/api/docs/models/gpt-6-sol)) | Schema-valid (per OpenAI) | Schema subset not reviewed here |
| Gemini 3.x | `response_format` with a JSON Schema subset ([docs](https://ai.google.dev/gemini-api/docs/structured-output)) | "Syntactically correct JSON" | Supports `enum`, `minimum`, `maximum`, `minItems`, `maxItems`; "very large or deeply nested schemas may be rejected" |
| Groq | Strict mode on gpt-oss-20b, gpt-oss-120b, qwen3.8-27b ([docs](https://console.groq.com/docs/structured-outputs)) | "100% schema adherence" | All fields must be required; no streaming or tool use with it |
| Mistral | Structured outputs on Small 4, Medium 3.5, Ministral 3 ([model pages](https://docs.mistral.ai/models/mistral-small-4-0-26-03)) | Not stated on the pages I read | **Unverified** |
| OpenRouter | Per endpoint (see the table in section 1) | Depends on the provider | Free Gemma 4 endpoints list only `response_format`, not structured outputs |
| Workers AI | JSON mode on 6 older models ([docs](https://developers.cloudflare.com/workers-ai/features/json-mode/)) | "Can't guarantee" | Newer vision models list function calling instead |
| llama.cpp (local) | JSON Schema converted to a grammar (`response_format`, `json_schema`, [docs](https://github.com/ggml-org/llama.cpp/blob/master/grammars/README.md)) | Output matches the grammar | `minimum` / `maximum` only for integers; unsupported features "are skipped silently"; the schema is not shown to the model, so the prompt must describe it |
| Ollama (local) | `format` takes a JSON Schema ([docs](https://docs.ollama.com/capabilities/structured-outputs)) | Output matches the schema | Ollama suggests also putting the schema in the prompt |

Evidence beyond vendor pages:

- A benchmark of 10,000 real-world JSON schemas across Guidance, Outlines, llama.cpp, XGrammar, OpenAI and Gemini found that constrained decoding frameworks differ in which schema features they cover and in output quality ([JSONSchemaBench, 2025](https://arxiv.org/abs/2501.10868)).
- Forcing a strict output format can cause "a significant decline in LLMs reasoning abilities" ([Tam et al., 2024](https://arxiv.org/abs/2408.02442)). Models with a separate thinking phase, or a two-step "reason, then fill the schema" call, avoid most of this.
- Tool-use scores reported by vendors show small models trail: Gemma 4 E2B 24.5% and E4B 42.2% on Tau2 versus 68.2% for 26B A4B ([Gemma 4 card](https://huggingface.co/google/gemma-4-E4B-it)); Qwen 3.5 4B scores 50.3 and 9B 66.1 on BFCL-V4 ([Qwen 3.5 card](https://huggingface.co/Qwen/Qwen3.5-4B)). Vendors measure differently, so compare within a family only.
- No provider guarantees working **code**. Code quality follows model size and coding scores (for example LiveCodeBench on the Gemma 4 card).

**Assessment:**

- Ask the model for a bounded JSON object of pattern parameters (enums for choices such as neckline or sleeve type, numbers for measurements and ease), not free-form pattern code, whenever the chosen route allows it. JSON can be validated; generated code needs a sandbox and fails more often on small or free models.
- Enforce numeric ranges in our own validator for every provider, because Anthropic's schemas cannot express ranges and llama.cpp only bounds integers.
- Let the model reason before it emits the JSON (thinking mode or two calls), and retry once on validation failure.

## 5. Recommendations (Assessment)

1. **Prototype on Cloudflare Workers AI with Gemma 4 26B A4B**: free for about 9 sessions a day, reads images, Apache 2.0 weights, and Cloudflare does not train on our data. The same weights run on the owner's CPU (15.6 GB at 4 bits) if the free tier changes.
2. **Keep a paid fallback ready for quality**: Gemini 3.8 Flash on a $5 prepaid paid tier (about $0.12 a session now, $0.25 from 2027) or Claude Sonnet 5 (about $0.42 a session) for the hardest step, such as turning a vague description plus a photo into a complete, consistent set of pattern parameters.
3. **Use Gemini's free tier, Mistral's free mode and data-training OpenRouter endpoints only with test descriptions**, never with the owner's real designs or, later, customers' photos.
4. **Do not rely on**: GitHub Models (retired), NVIDIA's trial, Cerebras's trial, Groq's preview vision model, Llama 3.2 Vision (English-only with images) or Phi-4-multimodal (English-only vision).
5. **Keep the model behind one small interface in the code** (most providers above offer an OpenAI-style chat API), because free tiers have changed several times in the last three months.
6. **Choose the final model with a test set**: 20 to 30 real descriptions in Portuguese and English, some with images, scored on valid JSON rate and on whether the parameters match what the owner meant (ideally checked with the pattern maker). Run Gemma 4 26B A4B, Gemini 3.8 Flash, Claude Sonnet 5 and one local 4B model. This belongs to the ticket "Choose the pattern-generation route and its AI model".

## Confidence and gaps

**Solid:** prices, free allowances, data-use clauses and license tags above were read directly from vendor pages, terms and Hugging Face on 2026-09-26; file sizes come from the Hugging Face file listings; per-session costs are arithmetic on those prices.

**Unverified or missing:**

- Exact free-tier limits for Gemini (shown only in AI Studio), Mistral (admin panel) and Z.ai (not published).
- Portuguese quality: model cards give multilingual averages or language counts, not Portuguese-specific scores, except Anthropic's table (Portuguese (Brazil) at 97.8% of English for Sonnet 4.5, [multilingual](https://platform.claude.com/docs/en/build-with-claude/multilingual-support)).
- Local speeds other than the one GTX 1650 Mobile measurement are estimates; the owner's RAM speed and CPU model are unknown.
- Whether Mistral's structured outputs guarantee schema validity, and the data policy of ModelRun, the provider behind OpenRouter's free Qwen 3.8 endpoint.
- Image token counts on Workers AI, OpenAI and Mistral (assumed about 1,300 per image).
- No test of how well any model maps garment descriptions to pattern parameters; that needs the test set in recommendation 6.
