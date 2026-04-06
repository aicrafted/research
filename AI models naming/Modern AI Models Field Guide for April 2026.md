# Modern AI Models Field Guide for April 2026

## Executive summary

**Scope and date.** This guide maps the practical AI model landscape **as of April 6, 2026** (Europe/Kyiv), focusing on how to *read model names*, separate *architecture vs training vs packaging*, and make sense of the ecosystem without vendor hype. It is **not** a stack recommendation and **not** a benchmark-only comparison. Many examples use official model strings and commonly encountered open-weight artifact names. citeturn16search11turn17search2turn15search21turn3search4

### A high-value mental map of “the model world” today

Most confusion comes from mixing three different layers:

**Layer one: the model’s job (function).**  
General chat LLMs, coding LLMs, embeddings, rerankers, multimodal/VLMs, ASR/TTS, image/video generators.

**Layer two: the model’s nature (architecture + training).**  
Dense vs MoE, decoder-only vs encoder-only vs seq2seq, diffusion vs transformer-based diffusion, “reasoning tuning”, tool-use training, long-context training.

**Layer three: the artifact you actually run (packaging).**  
Original checkpoints (often safetensors), quantized variants (AWQ/GPTQ/GGUF/EXL2), engine-specific builds (TensorRT-LLM, ONNX), plus runtime constraints (vLLM vs llama.cpp vs MLX).

If you keep those layers separate, model “names” become far less intimidating.

### Major categories of AI models, in practical terms

The list below is intentionally “applied”: it’s about what you buy/run/call.

| Category | What it does | Typical input → output | Where you see it | Common evaluation pitfalls |
|---|---|---|---|---|
| General-purpose LLMs | Broad language tasks, chat, assistants | text → text (sometimes image/audio too) | API model IDs (e.g., GPT / Claude / Gemini families), open-weight “Instruct” models | Vendor benchmark cherry-picks; prompts differ; tool access changes outcomes citeturn16search11turn17search0turn16search14turn15search21 |
| Small language models (SLMs) | LLM-like behavior under tight latency/VRAM | text → text | “mini/nano/small” lines; 0.5B–~10B class open weights | “Small but tuned” can beat older bigger baselines; comparisons often unfair citeturn16search8turn19search0 |
| Reasoning-tuned models | Better multi-step reliability by training + test-time compute | text → text (sometimes with hidden reasoning tokens) | “reasoning,” “thinking,” “effort,” “verifier,” “trace” features | Longer answers ≠ better reasoning; hidden reasoning tokens complicate cost comparisons citeturn16search0turn14search7turn14search2turn12search27 |
| Coding models | Code completion, repo editing, agentic SWE tasks | text/code → code/text | “coder,” “codestral,” “deepseek-coder,” “devstral” | “HumanEval wins” ≠ “fixes your codebase”; tool-use matters citeturn20search1turn20search0turn20search6turn20search2 |
| Embedding models | Turn text (or multimodal inputs) into vectors for search/RAG | text → vector | “embedding,” “embed,” “text-embedding-*” | Dimensionality and pooling choices affect results; MTEB ≠ your domain citeturn9search4turn9search2turn9search0 |
| Rerankers | Re-score candidate docs for a query (often cross-encoders) | (query, docs) → scores | “rerank,” “ranker” endpoints | People skip reranking and blame embeddings; latency trade-off citeturn9search1turn9search25 |
| Multimodal models | Handle multiple modalities (text+image+audio+video) | multi → text (and sometimes audio/image outputs) | “omni,” “vl,” “vision,” audio models, “realtime” stacks | Capability varies by modality; “multimodal” can mean many different architectures citeturn16search3turn16search15turn14search9turn16search10 |
| Vision-language models (VLMs) | Image understanding + language | image+text → text | “vision,” “vl,” “vl-plus,” Llama 4, Gemini 3 Pro | “Vision” ≠ OCR quality; cropping/resize pipelines matter citeturn15search4turn16search10turn14search21 |
| ASR (speech-to-text) | Transcribe speech | audio → text | Whisper-like models; cloud ASR offerings | Noise/accents dominate; “WER on test set” ≠ your mic setup citeturn8search32turn8search0turn8search19turn8search3 |
| TTS (text-to-speech) | Generate speech audio | text → audio | TTS APIs + open models | Latency vs expressiveness trade-off; voice cloning policies vary citeturn8search2turn8search6turn8search10 |
| Image generation models | Generate/edit images | text/image → image | diffusion/transformer diffusion families (SD 3.5, FLUX) | “Pretty sample” ≠ consistency/editability; licenses vary citeturn7search6turn7search0turn7search10turn7search1 |
| Video generation models | Generate videos | text/image → video | Veo family; evolving vendor offerings | Availability and product status change quickly citeturn7search15turn7search18turn7search22turn7search2turn7search8 |
| On-device / edge models | Run locally (CPU/GPU/NPU) | local text/image/audio → local outputs | GGUF/MLX/ONNX/TensorRT ecosystems | “File size” ≠ runtime RAM; KV cache dominates longer contexts citeturn6search1turn5search31turn5search37turn13search27turn13search6 |

### What people confuse most: family vs checkpoint vs build

This distinction is the core “decoder ring.”

| Term | What it is | What changes | What does *not* change | Example of confusion |
|---|---|---|---|---|
| Model family | A branded lineage sharing training + design philosophy | Major training runs, tokenizer, architecture decisions | Not a single set of weights | “Claude” or “Llama” is not one model citeturn17search0turn15search8 |
| Checkpoint | A specific saved set of weights | Weights + sometimes config/tokenizer | The underlying architecture *type* (usually) | “Llama 4 Scout” checkpoint differs from “Maverick” citeturn15search4turn18search1 |
| Parameter-size variant | Same recipe, different scale | Layer width/depth, parameter count | Brand/family intention | “1B vs 70B” are siblings, not “updates” |
| Fine-tuned derivative | Base weights adapted (SFT/RLHF/DPO/LoRA) | Behavior, refusal style, tool use, instruction-following | Most core knowledge and architecture, though behavior can shift a lot | “Instruct” isn’t more knowledgeable; it’s more aligned citeturn19search0turn19search1turn19search2 |
| Distilled model | Student trained to imitate teacher outputs/trajectories | Often better “skill per parameter,” sometimes narrower | It may inherit teacher biases; not necessarily same factual knowledge | “Distill” can outperform older bigger baselines on some tasks citeturn12search0turn14search8turn14search1turn19search0 |
| Quantized build | Same model, weights stored at lower precision | Accuracy/speed/VRAM trade-offs | The intended behavior (in theory), but small behavior changes happen | “7B Q4” is the *same* model family but a different artifact citeturn3search5turn4search1turn4search0 |
| Serving/file format | How weights + metadata are packaged | Engine compatibility, load times, metadata inclusion | The conceptual model architecture | GGUF vs safetensors vs TensorRT engine are not “different models” citeturn3search4turn5search37turn4search3 |
| Benchmark claim | A result under a specific harness | Prompting, tools, temperature, dataset versions | General real-world performance | “#1 on X” may reflect harness quirks or tool access citeturn14search29turn16search8 |

### Preview: how to read model names (real examples)

Below are “name decoding” previews; the full decoder comes later.

| Real name | Quick decode | Operational implication | What it does *not* imply |
|---|---|---|---|
| `gpt-5.4-mini` | GPT family, smaller tier (“mini”) | Lower cost/latency tier; may be used as subagent model citeturn16search8 | Not open-weight; not necessarily best at deep reasoning |
| `claude-sonnet-4-6` | Claude family; Sonnet line; version 4.6 | Use alias unless you need pinned versions; 1M context in beta is vendor-controlled citeturn17search8turn17search25turn17search30 | “4.6” doesn’t guarantee “better at everything” |
| `gemini-3-pro` | Gemini 3 generation; Pro tier | Multimodal + long context; model string patterns include stable/preview/latest labels citeturn16search10turn16search14 | “Pro” is vendor-relative branding|
| `meta-llama/Llama-4-Scout-17B-16E-Instruct` | Llama 4; Scout; MoE with 16 experts; 17B *active*; instruct-tuned | MoE means “total params” differs from “active params”; open-weight under a custom license citeturn15search24turn18search1turn15search3 | “16E” doesn’t mean 16× compute; only some experts are active |
| `deepseek-ai/DeepSeek-R1-Distill-Qwen-14B` | DeepSeek R1 distilled into a Qwen 14B base | “Distill” signals teacher/student pipeline; common for “smarter small” reasoning models citeturn14search0turn14search8 | Not the original teacher; architecture and tokenizer follow Qwen base |

---

## Big comparison tables

This section is intentionally “high density.” Use it as a fast reference.

### The ecosystem in one page: function × artifact × runtime

| You want to do… | Model type you likely need | Typical artifact | Common runtimes/serving | Biggest “gotcha” |
|---|---|---|---|---|
| General chat + writing + Q&A | General LLM (often “Instruct/Chat”) | API model ID or open-weight Instruct | API; vLLM/TGI; llama.cpp/MLX for local citeturn16search11turn6search1turn4search0turn5search37 | “Base” models feel unhelpful unless you know prompting |
| Tool-using agents | Tool-use capable LLM | API ID; sometimes instruct fine-tunes | API tools; agent frameworks | Tool access changes benchmark outcomes citeturn16search8turn14search6 |
| Semantic search / RAG retrieval | Embeddings (+ often reranker) | API embedding endpoint; open embedding checkpoint | Vector DB + embedding service | Embeddings alone often underperform without reranking citeturn9search4turn9search1turn9search25 |
| Sorting search results | Reranker (cross-encoder) | API rerank endpoint | Hosted rerank; sometimes local cross-encoder | Latency vs quality; needs candidate set first citeturn9search1turn9search25 |
| OCR-like extraction from images/PDFs | VLM + OCR/document models | VLM via API; specialized doc/OCR model | Vendor doc stacks; custom pipelines | “Vision model” may describe images well but miss tiny text |
| Transcription | ASR | Whisper-like checkpoint or cloud model | Local GPU/CPU; cloud ASR | Real-time constraints make “best WER” less relevant citeturn8search0turn8search19 |
| Voice agent | Low-latency audio model | Realtime speech-to-speech model ID | Realtime APIs | Context + latency constraints differ from text chat citeturn16search15turn16search3 |
| Image generation/editing | Image generator (diffusion/DiT-like) | Diffusers checkpoint; vendor API | Diffusers; vendor endpoints | Licenses and safety filters vary significantly citeturn7search0turn7search6turn7search10 |
| Video generation | Video generator | Vendor API | Cloud vendor APIs | Product availability is volatile (deprecations) citeturn7search15turn7search22turn7search2turn7search8 |
| Local offline assistant | SLM/LLM + quantized build (GGUF/AWQ/etc) | GGUF or GPU quant | llama.cpp / MLX / vLLM | KV cache can dominate memory for long contexts citeturn3search4turn6search1turn13search27turn13search6 |

### “What matters vs noise” (the brutally practical table)

The point of this table is to stop you from over-weighting marketing tokens.

| Label / feature | Usually important? | Why it matters (or not) | Common beginner mistake |
|---|---|---|---|
| Parameter count (e.g., 7B vs 70B) | Often | Correlates with capacity, but is not destiny; training and tuning dominate within a size band citeturn19search0turn10search0 | Assuming “bigger = always better” |
| MoE total params | Sometimes | Total parameters describe capacity; **active parameters** describe per-token compute citeturn10search3turn18search25turn18search1 | Comparing MoE total params to dense params directly |
| Active params (MoE) | Often | Better proxy for latency/compute than total params citeturn18search25turn18search1 | Ignoring that MoE can still be heavy on memory/bandwidth |
| Base vs Instruct | Extremely | Base = raw completion model; Instruct = trained to follow commands and chat norms citeturn19search0turn15search2 | Using base expecting ChatGPT-style behavior |
| “Chat” vs “Instruct” naming | Moderately | Often synonymous; vendor/community conventions differ | Treating them as strictly defined technical categories |
| Context length claim (32k/128k/1M/10M) | Often | Changes what you can feed at once; also changes cost and KV cache memory citeturn16search10turn15search1turn13search27 | Assuming you can *afford* max context in production |
| Benchmark score | Sometimes | Useful when independent and replicable; fragile otherwise citeturn14search29turn16search8 | Believing vendor single-number claims |
| “Reasoning” branding | Sometimes | Can indicate training + inference-time compute controls (effort/thinking) citeturn16search0turn14search7turn14search2 | Confusing “longer answers” with “better reasoning” |
| “Preview/experimental/latest” | Often | Indicates versioning stability and deprecation risk citeturn17search30turn16search14turn7search8 | Building production systems on unstable model strings |
| “Mini/flash/turbo” | Sometimes | Usually a latency/cost tier with vendor-specific meaning citeturn16search8turn16search14turn7search22 | Treating these suffixes as universal standards |
| Quantization method (AWQ/GPTQ/GGUF/EXL2) | Often | Determines hardware compatibility + quality/speed at same bit width citeturn4search0turn4search1turn3search4turn4search3 | Thinking “4-bit is 4-bit” regardless of method |
| Quantization level (Q4/Q5/Q8 etc) | Often | Affects accuracy and VRAM; Q8 is usually safer; Q4 often “good enough” for chat citeturn3search5turn4search1 | Picking lowest bits then blaming the base model |
| File format (GGUF/safetensors/ONNX) | Important for deployment | Controls which engines can load it and what metadata is embedded citeturn3search4turn5search37turn5search30 | Confusing “format” with “method” |
| Tokenizer differences | Sometimes | Token boundaries affect cost and context, and can break “drop-in” swaps | Assuming all tokenizers behave similarly |
| License | Extremely | Determines legal use, redistribution, fine-tuning and commercial constraints citeturn15search3turn15search6turn18search3turn7search0 | Assuming “open-weight = open-source” |

---

## Functional taxonomy

This taxonomy is **by job**, not vendor. Representative current families are included where they are materially relevant today.

### General-purpose LLMs

**What it is.** Broad language + instruction following, often the default “assistant brain.”  
**Used for.** Q&A, drafting, summarization, customer support, analysis, workflow automation.  
**Trade-offs.** Quality vs cost vs latency; tool access and safety behavior differ; long-context increases memory/cost. citeturn16search11turn17search0turn16search10turn15search4  
**Beginner misunderstanding.** Treating different *tiers* as “the same model, just faster.”

**Representative current families.**  
Closed: GPT-5.4 line (with mini/nano tiers) citeturn16search8turn16search24; Claude 4.6 line (Opus/Sonnet/Haiku tiers) citeturn17search10turn17search8turn17search25; Gemini 3 Pro/Flash family citeturn16search10turn16search14.  
Open-weight: Llama 4 Scout/Maverick (MoE, multimodal) citeturn18search1turn15search4; Mistral Large 3 (open-weight multimodal) citeturn20search4turn1search4; Qwen 3.5 Omni (omni-modal line) citeturn14search9.

### Small local / edge LLMs

**What it is.** Models designed to fit consumer GPUs, laptops, and sometimes mobile/NPUs — often via quantization and engine-specific formats.  
**Used for.** Private/local assistants, offline workflows, on-device summarization, local code helpers.  
**Trade-offs.** Smaller context and weaker robustness vs privacy/latency/cost control. Engines matter. citeturn6search1turn3search4turn5search37  
**Beginner misunderstanding.** Equating “download size” with “it will fit in VRAM.”

Representative families (open-weight oriented): Llama family (including Llama 4 and smaller earlier variants distributed for local use) citeturn15search21turn6search1; Gemma family citeturn1search7turn1search3; Phi family for small efficient reasoning variants citeturn2search4turn2search0.

### Reasoning-oriented models

**What it is.** Models tuned and/or served with mechanisms that allocate more “thinking” (extra inference-time compute or reasoning tokens) to hard problems. citeturn16search0turn12search27turn12search3  
**Used for.** Math, multi-hop planning, hard coding/debugging, structured decision tasks.  
**Trade-offs.** Higher latency and cost; sometimes worse at “simple chat”; potential for confident but wrong long rationales. citeturn16search0turn12search27  
**Beginner misunderstanding.** “Reasoning” = “always better.”

Representative families: OpenAI reasoning models with `reasoning.effort` controls citeturn16search0turn16search4; Mistral Magistral (reasoning traces) citeturn14search7turn14search18; Qwen “thinking mode” model variants (explicit “Thinking” columns in some listings) citeturn14search2turn14search6; DeepSeek reasoning lines including distilled R1 derivatives citeturn14search8turn14search0.

### Code-specialized models

**What it is.** Models trained for code completion (FIM), repo understanding, tool-using SWE agents, and debugging.  
**Used for.** IDE completion, code review, test generation, multi-file refactors, agentic coding.  
**Trade-offs.** Some code models are brittle at general chat; licensing and tool schemas differ.  
**Beginner misunderstanding.** Treating “code completion” and “agentic SWE” as the same need.

Representative families: Codestral (code completion + FIM) citeturn20search1turn20search17; Devstral 2 (agentic coding, multi-file) citeturn20search0turn20search16; DeepSeek-Coder family incl. MoE variants and reported code-heavy pretraining mix for earlier DeepSeek Coder series citeturn20search6turn20search2; Qwen Coder lines (e.g., “coder-plus/next”) citeturn20search7turn20search15.

### Long-context models

**What it is.** Models engineered to accept very long inputs (tens of thousands to millions of tokens).  
**Used for.** Large document synthesis, codebase-level Q&A, long-horizon planning, multi-document RAG with fewer retrieval hops.  
**Trade-offs.** Longer context increases KV cache memory and compute; “supports 1M” does not mean “cheap at 1M.” citeturn13search27turn13search6turn15search1turn16search10  
Representative examples: Gemini 3 Pro (1M context) citeturn16search10; Llama 4 Scout (10M context marketed; model family is MoE) citeturn15search1turn18search1; Claude 4.6 models with 1M context in beta (vendor controlled) citeturn17search10turn17search8.

### Multimodal models and VLMs

**What it is.** Models that can read images (and sometimes audio/video) and respond in language (and sometimes produce audio).  
**Architectural reality check.** “Multimodal” can mean: (a) a vision encoder + projector + LLM, or (b) more native fusion; vendors differ. (See architecture section.) citeturn15search4turn14search9turn16search15  
Representative examples: Llama 4 models (native multimodal positioning; MoE) citeturn18search1turn15search4; Gemini 3 Pro (text, audio, image, video, PDFs) citeturn16search10; OpenAI realtime/audio models (audio in/out) citeturn16search3turn16search15turn16search7.

### Embeddings and reranking (retrieval stack)

**Embeddings.** Vector representations for similarity search and clustering. OpenAI’s embedding guide explicitly frames embeddings as numerical representations used for relatedness and retrieval use cases. citeturn9search4turn9search0  
**Rerankers.** Models that sort candidate passages by relevance; Cohere describes rerankers as sorting inputs by semantic relevance to a query, commonly used after first-stage retrieval. citeturn9search1turn9search17  
Representative APIs: OpenAI `text-embedding-3-*` citeturn9search0turn9search4; Vertex AI embedding models including `gemini-embedding-001` being generally available per Google Developer Blog citeturn9search30turn9search2; Cohere Rerank model family docs citeturn9search1turn9search17; open embedding families like BGE-M3 citeturn9search3turn9search7.

### Speech models: ASR and TTS

ASR: Whisper is described by its creators as a general-purpose speech recognition model trained on diverse audio and capable of multilingual recognition and translation. citeturn8search0turn8search32  
Cloud ASR: Google’s Chirp 3 “Transcription” appears in Speech-to-Text release notes as GA and positioned as a multilingual ASR-specific generative model in API v2. citeturn8search19turn8search11  
TTS: ElevenLabs positions multiple TTS models with explicit latency tiers (Flash/Turbo/etc.) and provides developer docs for TTS capabilities. citeturn8search2turn8search6turn8search10

### Image and video generation

Image: Stable Diffusion 3.5 Large is described in its model card as a “Multimodal Diffusion Transformer (MMDiT)” text-to-image model; Stability AI published SD 3.5 variants in an open release under its Community License. citeturn7search6turn7search0  
Image: FLUX.1 dev model card describes it as a “rectified flow transformer” and Black Forest Labs docs position FLUX as their model family. citeturn7search10turn7search4turn7search1  
Video: Veo is explicitly documented in Vertex AI’s Veo video generation reference; Google also announced Veo 3.1 Lite on Vertex AI in early April 2026. citeturn7search15turn7search22turn7search18  
Video volatility: OpenAI published deprecations and a help article with concrete shutdown dates for Sora experiences and API, demonstrating how quickly video products can change. citeturn7search2turn7search8

---

## Architectural taxonomy

The goal here is to let you infer real operational differences — **latency, memory, quantization friendliness, and serving complexity** — from architecture class.

image_group{"layout":"carousel","aspect_ratio":"16:9","query":["decoder-only transformer architecture diagram","mixture of experts transformer diagram router experts","diffusion model denoising process diagram","kv cache transformer inference diagram"],"num_per_query":1}

### Dense decoder-only transformers

**What it is.** The “GPT-like” family: autoregressive next-token prediction. Introduced by the Transformer architecture. citeturn10search0  
**Why it exists.** Strong general text generation; straightforward scaling.  
**Strengths.** High-quality generation; good for chat and code.  
**Weaknesses.** Attention cost and KV cache grow with sequence length; long-context inference becomes memory-heavy. citeturn13search27turn13search6turn13search2  
**Serving implications.** Continuous batching and KV cache management are major levers for throughput (vLLM is explicitly built around PagedAttention to reduce KV cache waste). citeturn13search27turn6search34

### Mixture-of-Experts transformers (MoE)

**What it is.** Sparse activation: router selects subsets (“experts”) per token. Switch Transformer frames MoE as a way to get huge parameter counts with roughly constant per-token compute via sparse activation. citeturn10search3turn10search7  
**Why it exists.** More “capacity” without full dense compute; better quality/compute trade-offs in some regimes.  
**What matters operationally.**  
- **Total params**: storage/capacity.  
- **Active params**: per-token compute proxy.  
Mixtral’s paper describes selecting 2 experts per token and distinguishes total vs active parameters. citeturn18search25  
**Examples.**  
- Mixtral 8x7B: paper describes 47B total with ~13B active per token. citeturn18search25  
- Llama 4 Maverick: Meta describes 17B active and 400B total parameters (MoE) in the Llama 4 release. citeturn18search1turn15search4  
**Weaknesses.** Routing overhead, memory/bandwidth complexity; can be harder to deploy efficiently.

### Encoder-only transformers

**What it is.** BERT-style bidirectional encoders, typically for representations (classification, embeddings). citeturn10search1  
**Why it exists.** For tasks where you want a fixed vector or token-level representations.  
**Where you see it.** Many rerankers and embedding families are encoder-only or encoder-dominant.

### Encoder–decoder (seq2seq) transformers

**What it is.** T5-style: encoder reads input, decoder generates output. citeturn10search6turn10search2  
**Why it exists.** Strong for translation/summarization; retriever-augmented generation historically used seq2seq components. The original RAG paper combines parametric seq2seq with retrieval over a Wikipedia index. citeturn11search3turn11search7

### Multimodal transformer compositions

In practice you’ll see at least two common shapes:

**Vision encoder + projector + LLM hybrid.**  
A dedicated vision backbone produces embeddings; a projector aligns them to the LLM token space. Many “VLM” products work like this even if not described in detail publicly.

**More native multimodal fusion.**  
Meta’s Llama 4 documentation emphasizes “natively multimodal” positioning and MoE structure; the Llama 4 model card materials and release language describe multimodal capability and MoE. citeturn15search4turn18search1turn15search24  
(When internal fusion details are not fully public, treat “native” as a *capability label*, not a guaranteed architecture.)

### Diffusion and transformer-diffusion image families

**Diffusion basics.** Diffusion probabilistic models generate by iteratively denoising; DDPM is a canonical reference. citeturn11search0turn11search4  
**Transformer diffusion (DiT).** DiT replaces U-Nets with transformer backbones; the DiT paper describes diffusion models with transformers. citeturn11search1turn11search37  
**Modern image families.** SD 3.5 Large is explicitly described as MMDiT; FLUX.1 dev is described as a rectified flow transformer. citeturn7search6turn7search10

### Speech architectures

Whisper is publicly released and described as a multitask speech recognition model; it is often discussed as an encoder-decoder Transformer approach for ASR/translation. citeturn8search0turn8search4turn8search32  
Large vendor speech stacks (e.g., Chirp) are typically accessed as APIs with limited architectural disclosure; use official release notes and docs for confirmed claims. citeturn8search19turn8search3

### Emerging non-standard sequence models (state-space)

Mamba is explicitly presented as a selective state space model architecture aiming for linear-time sequence modeling. citeturn11search2turn11search30  
**Practical relevance.** These models matter conceptually for long-context efficiency, but most mainstream “model names” you encounter day-to-day in 2026 are still transformer-centered.

### Retrieval-augmented stacks (deployment pattern, not one model)

RAG is best understood as **a stack**: embed → retrieve → rerank → generate, often with citations and caching. The original RAG formulation is a canonical reference for combining parametric generation with retrieval. citeturn11search3turn11search7  
In production, the “model name” may hide this: “the LLM” is only one component.

---

## Naming and suffix decoding guide

This is the **decoder ring** section. The guiding rule:

> A model name is usually a **bag of hints** about (1) scale, (2) tuning, (3) modality/specialization, and (4) packaging — but vendors mix engineering signal with branding.

To meet your requirement on uncertainty: throughout this section, labels follow:
- **Confirmed fact**: directly supported by docs/papers.  
- **Strong public evidence**: multiple reliable public sources or explicit vendor statements.  
- **Informed inference**: deduction from common practice + partial disclosure.  
- **Speculation**: explicitly uncertain; avoided unless necessary.

### How to parse a model name: a practical token grammar

Most names can be understood as:

**[Family]-[Generation]-[Specialization]-[Size]-[Tuning]-[Context]-[Quant/Format]-[Version/Snapshot]**

Not all parts appear, and order differs by vendor/community.

### Size / scale indicators (0.5B … 100B+)

**What “B” means.** Usually “billions of parameters,” but MoE complicates it: total vs active. Switch Transformer explains sparse activation and why MoE can have huge total parameters with sparse per-token compute. citeturn10search3turn10search7  
**MoE examples in the wild.**
- Mixtral 8x7B paper: 47B total parameters, ~13B active per token. citeturn18search25  
- Llama 4 Maverick: Meta explicitly cites 17B active vs 400B total. citeturn18search1  

**Why bigger isn’t always better.** InstructGPT showed that a **1.3B** fine-tuned model could be preferred to a **175B** base model in human evaluations — a canonical demonstration that *post-training and alignment* can dominate size in perceived helpfulness. citeturn19search0

### Common suffixes and descriptors (base/instruct/reasoning/vl/etc.)

This table is the “what it usually means” guide — **not** a claim of universal standards.

| Token you see | Usually means | Operational implication | What it does *not* imply |
|---|---|---|---|
| `base` / `pretrained` | Raw next-token model | Needs prompting; best for further fine-tuning | Not safe/harmless by default |
| `instruct` / `chat` / `it` | Instruction-tuned via SFT and often preference optimization | Works better as assistant | Not more knowledgeable than base |
| `sft` | Supervised fine-tuning stage | Improved instruction following | Not necessarily safe/aligned alone citeturn19search0 |
| `rlhf` / `rl` | Preference tuning with human feedback (often RLHF-like) | Changes refusal/helpfulness style | Not a guarantee against hallucinations citeturn19search0 |
| `dpo` | Direct Preference Optimization | Preference alignment without explicit RL loop | Not automatically “more factual” citeturn19search1 |
| `reasoning` / `thinking` | Reasoning-tuned or inference-time scaling controls | Higher latency/cost; better on hard problems | Not always better on easy tasks citeturn16search0turn12search27turn14search2 |
| `coder` / `code` | Code-specialized training | Better code completion/debugging | Not necessarily good at chit-chat citeturn20search6turn20search1 |
| `vl` / `vision` / `vlm` | Vision-language capability | Image+text inputs | Not equal to OCR/document parsing quality citeturn15search4turn16search10 |
| `omni` | Multiple modalities (often text+image+audio; vendor-specific) | Broader I/O options | Not a specific architecture standard citeturn14search9turn16search15 |
| `embed` / `embedding` | Embedding/vector model | Use for search, clustering, RAG | Not a chat model citeturn9search4turn9search0 |
| `rerank` / `ranker` | Relevance scoring model | Used after retrieval | Not a first-stage retriever citeturn9search1turn9search25 |
| `distill` / `distilled` | Student trained from teacher outputs/trajectories | Better “skill per parameter” possible | Not the same as “compressed weights” citeturn12search0turn14search8 |
| `teacher/student` | Explicit distillation framing | Expect imitation of teacher behavior | Not necessarily same knowledge cutoff |
| `32k/128k/1M/10M` | Context window tier | Changes input capacity + KV cache footprint | Not “free”; cost/memory scale with length citeturn16search10turn15search1turn13search27 |
| `tool-use` / `function calling` | Tool invocation trained/supported | Better structured actions | Not guaranteed “agent reliability” citeturn16search8turn16search0 |
| `preview` / `experimental` / `latest` | Versioning stability tier | Deprecation risk; alias may shift | Not a quality guarantee citeturn16search14turn17search30turn7search8 |
| `mini` / `nano` / `flash` / `turbo` | Vendor tiering (speed/cost) | Usually cheaper/faster | Not comparable across vendors citeturn16search8turn16search14turn7search22 |

### Quantization naming: method vs format vs runtime build

This is where most people get lost. Three different things get mashed together:

1) **Quantization method** (math + calibration + kernels): GPTQ, AWQ, etc.  
2) **File/packaging format**: GGUF, safetensors, ONNX, TensorRT engines.  
3) **Runtime-specific build**: e.g., a TensorRT-LLM engine compiled for a GPU; or a GGUF tuned for llama.cpp.

**GGUF (format).** GGUF is a binary file format designed by the llama.cpp project to store tensors and metadata in one file for fast loading and quantization workflows. citeturn3search4turn6search1  
**llama.cpp quant types like `Q4_K_M`.** llama.cpp documents quantization options and the existence of the “K” variants used in GGUF quantization. citeturn3search5turn3search4  
**GPTQ / AWQ (methods).** GPTQ is a post-training quantization approach for GPT-like transformers; AWQ is activation-aware weight quantization. citeturn3search8turn3search6turn3search7turn3search0  
**EXL2 (format + method family).** ExLlama is explicitly oriented around efficient local LLM inference with EXL2 quantization artifacts. citeturn4search2turn4search3  
**bitsandbytes / QLoRA (training + runtime quantization tooling).** QLoRA describes NF4 4-bit quantization plus training tricks; it’s a training method, not just a file format. citeturn19search3turn19search7

#### Quick cheat sheet: common tokens you’ll see in local model names

| Token | Category | Meaning in practice | Commonly paired with |
|---|---|---|---|
| `BF16` / `FP16` / `FP8` | precision | Weight precision tier | Vendor APIs; TensorRT; some open distributions citeturn15search10turn18search5turn4search7 |
| `INT8` / `INT4` | precision | Quantized weights | ONNX/TensorRT; GPTQ/AWQ; GGUF citeturn4search3turn4search7turn3search4 |
| `Q8_0` / `Q6_K` / `Q4_K_M` | llama.cpp quant type | GGUF quant variants (runtime-specific naming) | llama.cpp / GGUF ecosystems citeturn3search5turn3search4 |
| `GGUF` | file format | llama.cpp-oriented format embedding tokenizer+metadata | llama.cpp, Ollama (often), LM Studio |
| `AWQ` | method + kernels | Activation-aware weight quant (often 4-bit) | vLLM/TensorRT/TGI support matrices citeturn4search6turn5search28turn4search7 |
| `GPTQ` | method | Post-training quantization for GPT-like models | vLLM/TGI/TensorRT; often 4-bit citeturn3search8turn5search28turn4search7 |
| `QAT` | training recipe | Quantization-aware training (or finetuning) | Often appears in research and some model releases citeturn21academia10 |
| `PTQ` | recipe class | Post-training quantization | GPTQ/AWQ are PTQ-style families citeturn3search8turn3search0 |
| `ONNX` | format | Exported graph format; can be quantized | ONNX Runtime, edge deployment citeturn5search30turn5search26 |
| `TensorRT-LLM` | runtime/build | NVIDIA optimized serving/build system | GPU inference, FP8/INT8/4-bit options citeturn4search7 |
| `vLLM` | runtime | High-throughput serving engine using PagedAttention | OpenAI-style serving; continuous batching citeturn6search34turn6search2 |
| `MLX` | runtime | Apple silicon-focused ML framework | MLX LM, mlx-lm packages citeturn5search31turn5search37 |

### Fully decoded example breakdowns (template-style)

These are “real-looking” and designed to teach pattern recognition. When tokens are runtime-specific (e.g., `Q4_K_M`), that is called out.

#### Example: `Model-X-7B-Base`
- `Model-X`: family label (could be vendor/community)  
- `7B`: ~7 billion parameters (dense unless otherwise stated)  
- `Base`: pretrained completion model  
**Implies:** best for fine-tuning; not necessarily chat-friendly.  
**Does NOT imply:** safety alignment, tool-use, or that it’s comparable to another 7B trained differently.

#### Example: `Model-X-7B-Instruct`
- `Instruct`: instruction-tuned (often via SFT, sometimes preference optimization) citeturn19search0turn19search1  
**Implies:** reads prompts like “do X” better; more assistant-like.  
**Does NOT imply:** superior raw knowledge vs base.

#### Example: `Model-X-4B-Distilled-Instruct`
- `Distilled`: trained to imitate a larger teacher (could be behavior or trajectory distillation) citeturn12search0turn12search1  
**Implies:** may outperform older untuned 7B on reasoning-like tasks. citeturn19search0  
**Does NOT imply:** same tokenizer or same architecture as teacher.

#### Example: `Model-X-32B-MoE`
- `MoE`: mixture-of-experts; expect “total vs active” parameter split citeturn10search3turn18search25  
**Implies:** deployment complexity; “32B” may be active or total depending on convention (often unclear).  
**Does NOT imply:** 32B compute per token.

#### Example: `Model-X-7B-GGUF-Q4_K_M`
- `GGUF`: file format for llama.cpp-like flows citeturn3search4  
- `Q4_K_M`: llama.cpp quantization type naming citeturn3search5  
**Implies:** loads in llama.cpp-family engines; quality close to 4-bit with K-variant choice.  
**Does NOT imply:** “AWQ quality” or “GPTQ quality” — those are different quant pipelines.

#### Example: `Model-X-Coder-7B-Instruct-AWQ`
- `Coder`: code-specialized  
- `AWQ`: activation-aware weight quantization method citeturn3search0turn4search6  
**Implies:** likely intended for GPU kernels that support AWQ; faster than generic 4-bit in some stacks.  
**Does NOT imply:** GGUF compatibility (unless explicitly provided separately).

---

## Lifecycle, distillation, and reasoning

This section explains how the thing you download (or call) is derived.

### The canonical derivation chain (what changes at each stage)

A practical, modern lifecycle is:

**Pretrained base** → **SFT / instruct** → **preference tuning (RLHF/DPO/etc.)** → **reasoning tuning / test-time compute hooks** → **domain specialization** → **distillation (optional)** → **quantization (optional)** → **format conversion / engine build** → **deployment**

Key references for the “alignment” and “preference” stages:
- InstructGPT: SFT then RLHF-style fine-tuning with human feedback. citeturn19search0turn19search12  
- DPO: preference optimization framed as a simpler alternative to RLHF pipelines. citeturn19search1turn19search5  
- LoRA: parameter-efficient finetuning by injecting low-rank adapters into frozen weights. citeturn19search2turn19search14  
- QLoRA: finetuning with 4-bit quantized weights (NF4) and associated training methods. citeturn19search3turn19search7  

### What “distilled” means in modern practice

**Confirmed core idea.** Distillation transfers behavior/knowledge from a larger teacher into a smaller student; the Hinton et al. paper is canonical for distillation principles. citeturn12search0turn12search8  
**Modern variants you actually encounter in names:**
- **Compression-style distillation:** make a smaller network approximate teacher outputs. citeturn12search0  
- **Behavior distillation:** student learns “how the teacher answers,” including style and refusal patterns (often via synthetic data). (Informed inference; patterns widely used in open releases.)  
- **Chain-of-thought / trajectory distillation:** student trains on teacher reasoning traces or intermediate steps; recent work focuses on avoiding “overthinking” and improving efficiency. citeturn12search2turn12search10turn12search14  
- **Synthetic data distillation:** teacher generates large datasets that become SFT corpora for the student (common in practice and explicitly referenced in some distill releases). citeturn14search8turn14search1turn14search30  

**Why a distilled 4B can beat an older plain 7B (mechanism).** InstructGPT provides a concrete demonstration that tuning and feedback can outweigh raw size in preference/utility. citeturn19search0 A distilled student may inherit higher-quality behaviors and problem-solving patterns than an older baseline trained without those signals.

### What “reasoning model” means today (and common failure modes)

**Confirmed vendor pattern.** Some vendors explicitly expose *reasoning budgets* (e.g., `reasoning.effort`) and distinguish reasoning tokens from output tokens. citeturn16search0turn16search4  
**Research framing.** Test-time compute scaling is an active area; papers describe improving reasoning by spending more inference compute and verification. citeturn12search27turn12search3turn12search31  

**Key distinctions you should make:**
- **Stronger base model**: better representation and generalization.  
- **Step-by-step tuned model**: more inclined to structured reasoning.  
- **More test-time compute**: sampling/verification/verifiers improve results. citeturn12search3turn12search27  
- **Hidden reasoning traces**: vendor may discard reasoning tokens and only expose summaries. citeturn16search0  
- **Just longer answers**: verbosity without correctness improvement (common failure).

**Common failure modes of “reasoning-branded” models (practical).**
- Overthinking easy tasks → wasted tokens/latency (explicitly discussed as a risk in CoT distillation literature). citeturn12search2  
- Confident wrong multi-step explanations (long rationales can be persuasive but incorrect; verification is non-trivial). citeturn12search19  
- Tool misuse: “agentic” models can fail by calling tools incorrectly if tool schemas are weak (informed inference; tool schema quality is known to matter in practice).

---

## Quantization, runtimes, and practical decoding checklist

This section is intentionally “operational.” It explains why the same “7B” can behave differently depending on build and runtime.

### Why quantization exists (and why it’s not just “compression”)

Quantization reduces precision (e.g., FP16 → INT4), shrinking weight memory and often improving speed — at some quality cost. GGUF is explicitly designed around quantization workflows for llama.cpp-family inference. citeturn3search4turn3search5  
For GPU serving, frameworks like vLLM and TensorRT-LLM support multiple quantization strategies and kernel paths. citeturn4search6turn4search7turn5search28

### The quality vs VRAM trade-off (what “Q4 good enough” actually means)

A practical mental model:

- **Weights memory**: roughly proportional to parameters × bits-per-weight.  
- **KV cache memory**: grows with context length, batch size/concurrency, and model dimensions; vLLM emphasizes KV cache efficiency with PagedAttention. citeturn13search27turn6search34turn13search6  
- **Long context** makes KV cache a first-class constraint; vLLM explicitly supports FP8 KV cache quantization to reduce memory footprint. citeturn13search6  
- FlashAttention reduces memory traffic for attention, enabling longer contexts and better efficiency in many transformer setups. citeturn13search2turn13search5

**Rule of thumb (informed inference, widely observed):**
- Q4-class can be “good enough” for casual chat and some coding, but sensitive tasks (math proofs, tricky code edits, multilingual nuance) benefit from higher precision (Q6/Q8, FP16/BF16). The exact breakpoints depend on model family and kernels; method matters at equal bits. citeturn3search5turn3search8turn3search0turn4search6

### Why “7B GGUF Q4” is not the same as “7B AWQ”

Because these differ in **both** packaging **and** intended kernels:

- **GGUF Q4_K_M**: llama.cpp-oriented file format and quant type naming. citeturn3search4turn3search5  
- **AWQ**: activation-aware weight quantization method typically paired with GPU kernels; vLLM/TGI mention specific handling of GPTQ/AWQ models and kernels. citeturn5search28turn4search6turn4search7  

Same base model weights can yield different output quirks after quantization (small distribution shifts, rounding effects) — especially for borderline prompts. (Informed inference.)

### Runtime glossary (what each tool actually is)

- **llama.cpp**: C/C++ inference engine emphasizing broad hardware support and local inference; explicitly supports CPU+GPU hybrid inference and multiple backends. citeturn6search1  
- **GGUF**: llama.cpp’s file format storing tensors + metadata; designed for fast loading and quantization workflows. citeturn3search4  
- **vLLM**: serving engine built around PagedAttention to reduce KV cache waste and improve throughput. citeturn6search34turn6search2  
- **TGI (Text Generation Inference)**: Hugging Face serving stack; launcher docs show quantization handling and sharding options. citeturn5search28turn5search32  
- **TensorRT-LLM**: NVIDIA inference/serving framework with multiple quantization and precision options. citeturn4search7  
- **ONNX / ONNX Runtime**: portable graph format and runtime; supports quantized export paths. citeturn5search30turn5search26  
- **MLX / mlx-lm**: Apple silicon-focused ML framework; Transformers integration notes explain shared-memory behavior and safetensors support. citeturn5search37turn5search31  
- **Ollama**: packaging/UX layer; Modelfile is a blueprint to build/share models, and can build from GGUF or safetensors. citeturn6search0turn6search20  

### A seven-step checklist for decoding any unfamiliar model name

Use this every time; it prevents most mistakes.

1) **Identify the family and issuer** (vendor or org).  
   - Is it a vendor API model ID (stable/preview/latest)? citeturn16search14turn17search30turn16search11  
   - Or an open hub path (e.g., `org/model` on Hugging Face)?

2) **Determine whether the name is an alias or a pinned version.**  
   - Anthropic explicitly notes that aliases can point to latest; use full names to pin. citeturn17search30turn17search2  
   - Google Gemini docs describe version naming patterns (stable/preview/latest/experimental). citeturn16search14

3) **Extract the *function* tokens.**  
   `coder`, `embed`, `rerank`, `vl`, `omni`, `realtime`, `tts`, `asr`, etc. citeturn20search1turn9search0turn9search1turn16search3turn8search0

4) **Extract scale tokens.**  
   `7B`, `70B`, or MoE patterns like `8x7B`, `17B-16E`. If MoE, look for **active vs total**. citeturn18search25turn18search1turn15search4

5) **Extract tuning tokens.**  
   `base` vs `instruct/chat`, plus distill/reasoning markers. citeturn19search0turn14search8turn16search0

6) **Extract deployment tokens (quantization + format + runtime).**  
   `GGUF` + `Q4_K_M` means llama.cpp-land; `AWQ/GPTQ` means PTQ method families likely targeting GPU kernels; `ONNX/TensorRT` signals runtime builds. citeturn3search4turn3search5turn5search28turn4search7turn5search30

7) **Check the license and the hardware fit — including KV cache.**  
   Llama community licenses are explicitly not “free software” per FSF commentary; treat “open-weight” as “source-available unless proven otherwise.” citeturn15search6turn15search3  
   For long contexts and concurrency, account for KV cache scaling and engine features like PagedAttention / KV cache quantization. citeturn13search27turn13search6

### Worked examples: decoding real model names start-to-finish

#### Example one: `gpt-5.4-mini`
1) Family/issuer: GPT line by entity["company","OpenAI","ai lab, san francisco, us"]. citeturn16search8turn16search11  
2) Alias vs pinned: vendor model ID; OpenAI also uses dated snapshots/deprecations for some models (general platform pattern). citeturn7search8turn16search11  
3) Function: general LLM with tool use and multimodal reasoning positioning. citeturn16search8  
4) Scale/tier: “mini” tier indicates cost/latency tiering. citeturn16search8  
5) Tuning: marketed as capable for coding/tool use; not labeled “base,” so you should treat as assistant-ready. citeturn16search8  
6) Packaging: API-only (you don’t choose GGUF/AWQ).  
7) Fit: context window and pricing are given in the launch post (400k context for mini in that announcement). citeturn16search8  
**Not implied:** Open weights, local deployability, or that “mini” equals another vendor’s “mini.”

#### Example two: `claude-sonnet-4-6`
1) Family/issuer: Claude family by entity["company","Anthropic","ai lab, san francisco, us"]. citeturn17search0turn17search8  
2) Alias vs pinned: Claude Code docs explicitly note aliases always point to latest; pin with full model name. citeturn17search30  
3) Function: general LLM; Sonnet line optimized for broad tasks; supports tool calling. citeturn17search25turn17search8  
4) Scale: not encoded as “B”; Anthropic uses product-line naming rather than parameter counts.  
5) Tuning: assistant-ready; “Sonnet” implies the “mid-tier” family line in their lineup. (Confirmed lineup approach via official overview.) citeturn17search0turn17search10  
6) Packaging: API; can also be accessed through partner platforms with explicit model IDs. citeturn17search25  
7) Fit: context window features are described as “1M token context window in beta” for Sonnet 4.6 announcement. citeturn17search8  
**Not implied:** That every request truly uses 1M tokens cheaply; beta features can change.

#### Example three: `gemini-3-pro`
1) Family/issuer: Gemini line by entity["company","Google","technology company, mountain view, ca, us"] (with Gemini models also associated with entity["organization","Google DeepMind","ai research lab, london, uk"]). citeturn16search10turn7search18  
2) Versioning: Gemini docs explain stable/preview/latest/experimental patterns. citeturn16search14  
3) Function: multimodal reasoning model; Vertex AI docs describe multimodal inputs and 1M token context for Gemini 3 Pro. citeturn16search10  
4) Scale: not encoded in name; tier is “Pro.”  
5) Tuning: assistant-ready.  
6) Packaging: API; can be used via Vertex AI endpoints. citeturn16search10  
7) Fit: long context is a feature; costs and latency scale accordingly (KV cache implications apply in self-hosting contexts). citeturn13search27turn6search34  
**Not implied:** That “Pro” equals “Opus” or “GPT frontier” across vendors.

#### Example four: `meta-llama/Llama-4-Scout-17B-16E-Instruct`
1) Family/issuer: Llama by entity["company","Meta","technology company, menlo park, ca, us"]. citeturn15search24turn18search1  
2) Versioning: Llama provides model cards/prompt formats and versioning guidance. citeturn15search4turn18search11  
3) Function: general + multimodal (image input), text output. citeturn15search4turn15search1  
4) Scale: `17B` active parameters; `16E` = 16 experts; total params are larger (~109B total per public release descriptions). citeturn18search1turn18search4turn15search1  
5) Tuning: `Instruct` means assistant-tuned.  
6) Packaging: open-weight; you can find BF16 and quantized distributions in some releases (e.g., HF model card notes quantization options for Llama 4 releases). citeturn18search5turn15search24  
7) License: Llama 4 Community License Agreement is explicitly provided on llama.com; it is not necessarily OSI-open-source. citeturn15search3turn15search6  
**Not implied:** That it runs locally on consumer GPUs without heavy quantization; also “10M context” is a claim that must be considered with cost/memory realities. citeturn15search1turn13search27

#### Example five: `unsloth/Llama-3.3-70B-Instruct-GGUF` (plus `…Q4_K_M.gguf`)
1) Family/issuer: Llama 3.3 family; instructions tuned. citeturn15search2turn21search0  
2) Artifact type: this is a **converted/packaged** GGUF artifact, not the canonical training checkpoint. citeturn3search4turn21search0  
3) Function: general chat LLM (instruct).  
4) Scale: 70B dense model (text-only). citeturn15search2turn21search0  
5) Tuning: instruct (SFT/RLHF stated in the model info text on this distribution). citeturn21search0turn19search0  
6) Quantization tokens: `Q4_K_M` in file names match llama.cpp quant naming conventions. citeturn3search5turn21search3  
7) Runtime: intended for llama.cpp-family engines; will not “just work” in vLLM unless converted back to a supported checkpoint format. citeturn3search4turn6search1turn4search6  
**Not implied:** That it matches BF16 performance; quant level matters.

### Current model family survey (curated, “major families only”)

This is not exhaustive; it’s the set you’re most likely to encounter in 2026 product and open-weight ecosystems.

| Family | Issuer | Open-weight? | Primary roles | Architectural notes (public) | Distinct naming conventions you’ll see |
|---|---|---|---|---|---|
| GPT-5.4 line | entity["company","OpenAI","ai lab, san francisco, us"] | Closed | General LLM, reasoning/tiering (mini/nano), tool use | Reasoning tokens + `reasoning.effort` controls for reasoning models (public API docs) citeturn16search0turn16search8turn16search24 | `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.4-nano` citeturn16search8turn16search24 |
| Realtime/audio models | OpenAI | Closed | Voice agents, audio in/out | Realtime API supports speech-to-speech; model pages list context windows and modalities citeturn16search15turn16search3turn16search7 | `gpt-realtime-*`, `gpt-audio-*` citeturn16search11 |
| Claude 4.6 line | entity["company","Anthropic","ai lab, san francisco, us"] | Closed | General LLM, agents, coding | Model IDs and version pinning practices are explicitly documented citeturn17search2turn17search30turn17search25 | `claude-opus-4-6`, `claude-sonnet-4-6`, `claude-haiku-4-5` citeturn17search25turn17search10turn17search8turn17search3 |
| Gemini 3 | entity["company","Google","technology company, mountain view, ca, us"] / entity["organization","Google DeepMind","ai research lab, london, uk"] | Closed | Multimodal reasoning, long context | Vertex AI docs describe multimodal inputs and 1M context for Gemini 3 Pro citeturn16search10turn16search14 | `gemini-3-pro`, versioning labels (stable/preview/latest/experimental) citeturn16search14 |
| Llama 4 | entity["company","Meta","technology company, menlo park, ca, us"] | Open-weight (custom license) | General LLM + VLM | MoE with active vs total params; Llama site documents models and licensing citeturn18search1turn15search4turn15search3 | `Llama-4-Scout-17B-16E`, `…Maverick-17B-128E` citeturn15search24turn18search8 |
| Mistral Large/Small + Mixtral | entity["company","Mistral AI","ai company, paris, france"] | Mixed (some open, some premier) | General LLM, MoE, code, reasoning | Mixtral paper and docs describe MoE and active params; Mistral docs provide model lists and features citeturn18search25turn18search3turn14search3turn20search4turn14search7 | Names like `mistral-*-latest`, `mixtral-8x7b`, `magistral-*` citeturn14search7turn18search3 |
| Qwen 3.x / 3.5 | entity["company","Alibaba Cloud","cloud provider, hangzhou, china"] | Mixed (open + proprietary variants) | Chat, coder, omni, thinking mode | Public model listings show “thinking mode” and very large context windows for some variants citeturn14search2turn14search9turn20search7turn20search15 | `qwen3.5-*`, `qwen3-*-thinking`, `qwen3-coder-*` citeturn14search2turn20search7 |
| DeepSeek R1 / Coder | entity["company","DeepSeek","ai company, china"] | Mixed (some open weights) | Reasoning + code | DeepSeek R1 repo describes distill variants; code family describes code-heavy training mix citeturn14search8turn20search6 | `DeepSeek-R1-Distill-*`, `deepseek-coder-*` citeturn14search1turn20search22 |
| Phi | entity["company","Microsoft","technology company, redmond, wa, us"] | Mixed | Small efficient reasoning | Microsoft research posts describe Phi variants and capabilities citeturn2search4turn2search0 | `phi-*` tiers |
| Command / Rerank | entity["company","Cohere","ai company, toronto, canada"] | Closed | Enterprise LLM + retrieval stack | Cohere docs describe rerank models and language support citeturn9search1turn9search17turn2search1 | `rerank-v*`, embed lines |
| Grok | entity["company","xAI","ai company, san francisco, us"] | Closed | General LLM + tools | Official xAI docs list model families and usage patterns citeturn2search6turn2search14 | `grok-*` variants |
| Nemotron | entity["company","NVIDIA","technology company, santa clara, ca, us"] | Mixed | Foundation + optimized deployments | NVIDIA pages describe Nemotron families and NIM distribution ecosystem citeturn2search3turn2search19turn14search30 | Often appears via NIM model cards |
| SD 3.5 | entity["company","Stability AI","ai company, london, uk"] | Open-ish (community license) | Image generation | SD 3.5 model card describes MMDiT; Stability AI blog describes release variants citeturn7search6turn7search0 | `sd3.5-large`, `turbo`, etc. |
| FLUX | entity["company","Black Forest Labs","ai company, germany"] | Mixed | Image generation/editing | FLUX.1 dev described as rectified flow transformer; official docs position FLUX family citeturn7search10turn7search1turn7search4 | `FLUX.1 [dev]`, `FLUX.2 *` |

### Glossary of common terms (short, precise)

| Term | Meaning (practical) | Anchor reference |
|---|---|---|
| LLM | Large language model (general text generation) | Transformer lineage citeturn10search0 |
| SLM | Small language model (practically: runs cheaper/local; not a strict cutoff) | InstructGPT size-vs-preference example citeturn19search0 |
| VLM | Vision-language model (image+text → text) | Gemini 3 Pro, Llama 4 docs citeturn16search10turn15search4 |
| MoE | Mixture-of-Experts; sparse activation | Switch Transformers, Mixtral, Llama 4 citeturn10search3turn18search25turn18search1 |
| Dense | All parameters active per token | Standard transformer usage citeturn10search0 |
| Checkpoint | Saved weights/config snapshot | Open model distribution practice citeturn15search21turn5search37 |
| Tokenizer | Text→token mapping; affects cost/context | (General concept; vendor-specific) |
| Embedding | Vector representation for similarity search | OpenAI embeddings guide citeturn9search4 |
| Reranker / cross-encoder | Scores query-doc pairs for relevance | OpenSearch reranking tutorial explains cross-encoder role citeturn9search25 |
| Decoder-only | Autoregressive generation architecture | Transformer paper lineage citeturn10search0 |
| Encoder-only | Bidirectional encoding for representations | BERT citeturn10search1 |
| Seq2seq | Encoder-decoder input→output mapping | T5 citeturn10search6 |
| Context window | Max tokens processed at once | Gemini 3 Pro 1M; Llama 4 10M marketing citeturn16search10turn15search1 |
| KV cache | Stored keys/values for faster autoregressive decoding; memory grows with context/concurrency | vLLM paper + quantized KV cache docs citeturn13search27turn13search6 |
| SFT | Supervised fine-tuning | InstructGPT citeturn19search0 |
| RLHF | Preference alignment via human feedback and RL loop | InstructGPT citeturn19search0 |
| DPO | Preference optimization via classification objective | DPO paper citeturn19search1 |
| Distillation | Teacher→student imitation | Hinton distillation paper citeturn12search0 |
| QAT | Quantization-aware training/fine-tuning | Edge quantization research line citeturn21academia10 |
| PTQ | Post-training quantization | GPTQ/AWQ are PTQ-family methods citeturn3search8turn3search0 |
| Quantization | Lower-precision representation for weights/activations | GGUF + PTQ methods citeturn3search4turn3search8turn3search0 |
| Hallucination | Confident-sounding incorrect output | Mitigated via retrieval/verification; not eliminated citeturn11search3turn12search19 |
| Tool/function calling | Structured outputs to invoke tools/APIs | OpenAI reasoning/tooling docs; Mistral and Qwen tool-use docs citeturn16search0turn20search0turn20search7 |
| Agent | System that uses model + tools + memory + planning | Vendor “agentic” positioning; not one model feature alone citeturn16search8turn14search6 |
| RAG | Retrieval-augmented generation stack | RAG paper citeturn11search3 |
| LoRA | Low-rank adapters for efficient fine-tuning | LoRA paper citeturn19search2 |
| QLoRA | LoRA applied with 4-bit quantization + training tricks | QLoRA paper citeturn19search3 |
| Speculative decoding | Draft+verify acceleration technique | Recent speculative decoding work + vLLM context citeturn6search3turn6search34 |
| Test-time compute | Spending more inference compute to improve answers | Test-time scaling literature citeturn12search27turn12search3 |

### Key takeaways and rules of thumb

1) **Always separate: function → architecture → tuning → packaging.** Most “model name confusion” is those layers getting mixed.  
2) **Instruct beats base for most users.** If you’re not training, start from instruct/chat variants. citeturn19search0  
3) **MoE requires two numbers in your head:** total params and active params. Compare accordingly. citeturn18search25turn18search1  
4) **Quantization is not one thing.** “INT4” is a precision; GPTQ/AWQ/GGUF are pipelines; TensorRT/vLLM/llama.cpp are runtimes. citeturn3search4turn4search7turn5search28turn6search1  
5) **Context length is a budget, not a trophy.** KV cache and throughput constraints dominate real deployments. citeturn13search27turn13search6turn6search34  
6) **Treat vendor “mini/flash/turbo” as branding until proven otherwise.** Use docs for actual modality/context/limits. citeturn16search14turn16search8turn7search22  
7) **Prefer pinned versions for production.** Alias strings move; deprecations happen. citeturn17search30turn7search8turn16search14