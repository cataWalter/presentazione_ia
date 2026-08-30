---
theme: default
background: '#ffffff'
class: text-center
colorSchema: light
highlighter: shiki
lineNumbers: false
info: |
  ## Local AI Field Guide
  A systems engineering guide to open-weight models: boundary architecture, inference physics, memory sizing, and private serving.
drawings:
  persist: false
transition: slide-left
title: "Running AI locally"
aspectRatio: 16/9
canvasWidth: 980
fonts:
  sans: 'Plus Jakarta Sans'
  mono: 'Geist Mono'
---

<div class="eyebrow">LOCAL AI FIELD GUIDE</div>

# Running AI locally

<h2 class="subtitle">A systems engineering guide to open-weight models: boundary architecture, inference physics, memory sizing, and private serving</h2>

<p class="source-note">First-principles systems view for engineering, architecture, and technology leadership · 2026</p>

---
layout: default
class: lead-alt
title: "A systems tour"
---

# A systems tour

<ul class="agenda">
  <li><span class="num">01</span><span><b>Boundaries &amp; Economics</b> — What private inference controls, and what your team must operate</span></li>
  <li><span class="num">02</span><span><b>Model Architectures</b> — Dense transformers, MoE routing, hybrid attention, and vision pipelines</span></li>
  <li><span class="num">03</span><span><b>Inference Physics &amp; Sizing</b> — Weights, KV cache math, quantization, bandwidth, and the memory wall</span></li>
  <li><span class="num">04</span><span><b>Serving &amp; Live Demo</b> — Test-time reasoning, serving runtimes, hardware tiers, and an end-to-end trace</span></li>
</ul>

<p class="source-note">Goal: A reusable sizing methodology and mental model for any open-weight deployment.</p>

---
layout: default
---

# 1. Local AI is a system property

<p>Local inference means the request path, model weights, compute execution, and operational policies stay inside a defined trust boundary. It is an architectural choice, not just running software on a workstation.</p>

<div class="grid3">
  <div class="card accent">
    <h3>Data boundary</h3>
    <p>Prompts, embeddings, proprietary documents, and generation streams never leave your VPC or local network.</p>
    <span class="big">Zero external egress</span>
  </div>
  <div class="card accent2">
    <h3>Compute boundary</h3>
    <p>The forward pass executes on dedicated silicon you control—guaranteeing deterministic performance without third-party rate limits.</p>
    <span class="big">Dedicated silicon</span>
  </div>
  <div class="card gold">
    <h3>Service boundary</h3>
    <p>Capacity, quantization levels, context budgets, version upgrades, and downtime fallbacks are direct engineering decisions.</p>
    <span class="big">Full operational control</span>
  </div>
</div>

<div class="callout">The fundamental question shifts from <b>"Can a model answer?"</b> to <b>"Can this workload run reliably, economically, and privately inside our boundary?"</b></div>

---
layout: default
---

# 2. Where does the request go?

<p>The deployment choice is a data-path and economics choice. Cloud APIs and local inference provide identical interfaces (e.g. OpenAI-compatible endpoints) with fundamentally different trade-offs.</p>

<div class="grid2">
  <div class="card warn">
    <h3>Hosted Cloud Inference</h3>
    <div class="path-row">
      <span class="path-node">Client</span>
      <span class="path-arrow">-&gt;</span>
      <span class="path-node">Public WAN</span>
      <span class="path-arrow">-&gt;</span>
      <span class="path-node">Multi-Tenant Gateway</span>
      <span class="path-arrow">-&gt;</span>
      <span class="path-node">Provider Cluster</span>
    </div>
    <ul>
      <li><b>Fastest time-to-market:</b> Zero hardware procurement or driver management.</li>
      <li><b>Variable OpEx:</b> Spend scales linearly with token volume and context depth.</li>
      <li><b>Trust &amp; latency:</b> Data crosses public networks; latency subject to shared multi-tenant queueing.</li>
    </ul>
  </div>
  <div class="card accent2">
    <h3>Local &amp; Private Inference</h3>
    <div class="path-row">
      <span class="path-node">Client</span>
      <span class="path-arrow">-&gt;</span>
      <span class="path-node">Private LAN / VPC</span>
      <span class="path-arrow">-&gt;</span>
      <span class="path-node">Dedicated Runtime</span>
      <span class="path-arrow">-&gt;</span>
      <span class="path-node">Owned Silicon</span>
    </div>
    <ul>
      <li><b>Total data sovereignty:</b> Data never leaves the trust perimeter; offline air-gapped support.</li>
      <li><b>Fixed CapEx:</b> Marginal cost per token approaches zero once hardware is amortized.</li>
      <li><b>Operational ownership:</b> Your team owns model serving, driver updates, and capacity planning.</li>
    </ul>
  </div>
</div>

<p class="source-note">Decision criteria: data compliance, token volume break-even point, latency predictability, and team operational capability.</p>

---
layout: default
---

# 3. Open weights: The model landscape

<p>Open-weight models span distinct size and capability tiers. Sizing local infrastructure begins by matching the task complexity to the minimum viable parameter scale.</p>

<div class="metric-strip">
  <div class="metric"><span class="metric-value">1B–3B</span><span class="metric-label">Edge &amp; Mobile</span></div>
  <div class="metric"><span class="metric-value">7B–14B</span><span class="metric-label">Single-GPU Fast</span></div>
  <div class="metric"><span class="metric-value">27B–32B</span><span class="metric-label">Reasoning Sweet Spot</span></div>
  <div class="metric"><span class="metric-value">70B+ / MoE</span><span class="metric-label">Frontier Datacenter</span></div>
</div>

<div class="grid3">
  <div class="card accent">
    <h3>Parameter Scale</h3>
    <p>Smaller models (3B–8B) excel at classification and summarization; 27B–70B+ models handle complex code, multi-step math, and nuanced reasoning.</p>
  </div>
  <div class="card accent2">
    <h3>Architectures</h3>
    <p>Available as dense transformers (all weights active), sparse Mixture-of-Experts (MoE), and vision-language models with native image/video encoders.</p>
  </div>
  <div class="card gold">
    <h3>Weight Ownership</h3>
    <p>Permissive licenses (Apache-2.0, MIT, OpenRAIL) provide the raw weight files (SafeTensors, GGUF) for air-gapped inspection, fine-tuning, and deployment.</p>
  </div>
</div>

<p class="source-note">Choosing a model tier sets your baseline VRAM requirement, memory bandwidth needs, and expected token throughput.</p>

---
layout: default
---

# 4. Architecture: Dense, MoE &amp; Hybrids

<p>The model backbone defines both computational efficiency and memory scaling. Different architecture families create distinct trade-offs on local hardware:</p>

<div class="architecture-flow">
  <div class="architecture-label">Hybrid Backbone Layout</div>
  <div class="architecture-group">
    <span class="architecture-chip delta">Linear / SSM</span><span class="architecture-chip ffn">FFN / Expert</span>
    <span class="architecture-chip delta">Linear / SSM</span><span class="architecture-chip ffn">FFN / Expert</span>
    <span class="architecture-chip delta">Linear / SSM</span><span class="architecture-chip ffn">FFN / Expert</span>
  </div>
  <div class="architecture-repeat">periodic mixing</div>
  <div class="architecture-group global-group">
    <span class="architecture-chip global">Global Attention</span><span class="architecture-chip ffn">FFN / Expert</span>
  </div>
</div>

<div class="grid3">
  <div class="card accent">
    <h3>Dense Transformers</h3>
    <p>Full pairwise self-attention across every layer. Maximizes associative recall, but KV cache grows with sequence length.</p>
    <span class="mono">High reasoning · Large KV cache</span>
  </div>
  <div class="card accent2">
    <h3>Mixture-of-Experts (MoE)</h3>
    <p>Routes each token to a subset of experts (e.g. 2 of 8). Fast compute speed per token, but requires enough VRAM to hold all experts.</p>
    <span class="mono">Fast decode · High VRAM floor</span>
  </div>
  <div class="card accent3">
    <h3>Linear &amp; Hybrid Cores</h3>
    <p>Mixes linear state spaces (constant state size $O(1)$ during decode) with periodic global attention layers for deep multi-hop reasoning.</p>
    <span class="mono">Ultra-compact cache · High throughput</span>
  </div>
</div>

<p class="source-note">Understanding the architecture determines whether your primary hardware bottleneck will be total VRAM capacity or memory bandwidth.</p>

---
layout: default
---

# 5. The multimodal pipeline &amp; token budget

<p>Vision-language models do not ingest raw pixels directly into attention. Visual inputs undergo a multi-stage preprocessing pipeline that converts images and video frames into standard LLM tokens.</p>

<div class="grid3">
  <div class="card accent">
    <div class="media-glyph">TXT</div>
    <h3>Text Ingestion</h3>
    <p>Direct tokenization via BPE/WordPiece. Fast, compute-light, and token counts directly match text volume.</p>
    <span class="mono">text -&gt; ~1.3 tokens / word</span>
  </div>
  <div class="card accent2">
    <div class="media-glyph">IMG</div>
    <h3>Vision Tokenization</h3>
    <p>A Vision Transformer (ViT) splits images into patches (e.g. 14x14 px), projecting each into embedding tokens.</p>
    <span class="mono">1 image -&gt; 500–2,000+ tokens</span>
  </div>
  <div class="card gold">
    <div class="media-glyph">VID</div>
    <h3>Video Sampling</h3>
    <p>Temporal frame extraction. Sampling 1–2 frames/sec across a 60-second video can generate 20K–60K+ tokens.</p>
    <span class="mono">video -&gt; sequence of image grids</span>
  </div>
</div>

<div class="callout"><b>The hidden cost of vision:</b> A single document screenshot consumes as much KV cache and context budget as a 1,500-word article. Factor visual token expansion into your VRAM sizing.</div>

<p class="source-note">Different vision encoders (SigLIP, CLIP, ViT) and dynamic patch splitting schemes determine the exact visual token count per image.</p>

---
layout: default
---

# 6. Inference mechanics: Prefill vs. Decode

<p>A single inference request consists of two mathematically distinct execution phases, each constrained by entirely different hardware bottlenecks.</p>

<div class="grid2">
  <div class="card accent">
    <div class="phase-kicker">PHASE 1 · PROMPT INGESTION</div>
    <h3>Prefill (Compute-Bound)</h3>
    <p>Processes the entire input prompt in parallel using dense matrix-matrix multiplications (GEMM). Saturated by GPU tensor compute cores.</p>
    <div class="phase-metric"><span>Hardware bottleneck</span><b>Tensor Core Compute (TFLOPS)</b></div>
    <div class="phase-metric"><span>User metric</span><b>Time-to-First-Token (TTFT)</b></div>
  </div>
  <div class="card accent2">
    <div class="phase-kicker">PHASE 2 · TOKEN GENERATION</div>
    <h3>Decode (Bandwidth-Bound)</h3>
    <p>Generates tokens sequentially one by one (GEMV). For every single token generated, the GPU must stream the entire model weights through memory.</p>
    <div class="phase-metric"><span>Hardware bottleneck</span><b>Memory Bandwidth (GB/s)</b></div>
    <div class="phase-metric"><span>User metric</span><b>Tokens-per-Second (TPS)</b></div>
  </div>
</div>

<div class="math-box">
  <div class="math-formula">$$\text{Total Latency} \approx \text{TTFT}_{\text{prefill}} + \frac{\text{Output Tokens}}{\text{Decode Speed (tokens/s)}}$$</div>
  <div class="math-explanation">Example: A 2,000-token prompt prefilled in 200 ms ($0.2\text{s}$) generating 200 output tokens at $25\text{ tok/s}$ ($8.0\text{s}$) results in an $8.2\text{s}$ total response time.</div>
</div>

---
layout: default
title: "7. Weight memory: Baseline VRAM sizing"
---

## 7. Weight memory: Baseline VRAM sizing

<p>The irreducible foundation of local inference is weight storage. The model weights must reside entirely in high-speed GPU VRAM (or Unified Memory) to achieve acceptable decode speeds.</p>

<div class="math-box">
  <div class="math-formula">$$M_{\text{weights}} \approx \frac{N \times 10^9 \times b}{8 \times 10^9} \text{ GB} = \frac{N \cdot b}{8} \text{ GB}$$</div>
  <div class="math-explanation">Where $N$ is the parameter count in billions and $b$ is the stored bits per parameter (16 for FP16, 8 for INT8/FP8, 4 for 4-bit):</div>
</div>

<table class="estimate-table">
  <thead>
    <tr><th>Model Scale</th><th>FP16 (16-bit)</th><th>INT8 / FP8 (8-bit)</th><th>4-bit (AWQ/GGUF)</th><th>Target Hardware Class</th></tr>
  </thead>
  <tbody>
    <tr><td><b>8B</b></td><td>~16 GB</td><td>~8 GB</td><td>~4.5 GB</td><td>Consumer laptops / Apple M-series / 8GB VRAM</td></tr>
    <tr><td><b>14B</b></td><td>~28 GB</td><td>~14 GB</td><td>~8 GB</td><td>Single GPU (RTX 4070Ti / 4080 - 16GB)</td></tr>
    <tr><td><b>27B–32B</b></td><td>~60 GB</td><td>~30 GB</td><td>~15 GB</td><td>Single Workstation GPU (RTX 3090/4090 - 24GB)</td></tr>
    <tr><td><b>70B</b></td><td>~140 GB</td><td>~70 GB</td><td>~38 GB</td><td>Dual RTX 3090/4090 (48GB) or Mac Studio (64GB+)</td></tr>
  </tbody>
</table>

<p class="source-note estimate-label">ENGINEERING RULE: Total VRAM required = Weight Memory + KV Cache + Activation Memory + Runtime Buffers (~20% headroom recommended).</p>

---
layout: default
title: "8. KV Cache: Why context eats memory"
---

## 8. KV Cache: Why context eats memory

<p>In autoregressive generation, each new token must attend to all preceding tokens. Instead of recomputing previous keys ($K$) and values ($V$) on every step, engines store them in GPU memory as the <b>KV Cache</b>.</p>

<div class="context-visual">
  <div class="context-axis"><span>4K Context</span><span>32K Context</span><span>128K Context</span><span>256K–1M Window</span></div>
  <div class="context-track"><span class="context-fill native"></span><span class="context-marker native-marker">Standard Working Window</span></div>
  <div class="context-track"><span class="context-fill extended"></span><span class="context-marker extended-marker">High VRAM Memory Pressure</span></div>
</div>

<div class="grid2">
  <div class="card accent3">
    <h3>Linear Cache Growth</h3>
    <p>$$\text{KV Cache} \approx 2 \times n_{\text{layers}} \times d_{\text{model}} \times L_{\text{ctx}} \times B \times b_{\text{elem}}$$</p>
    <p>KV cache scales linearly with sequence length ($L_{\text{ctx}}$) and concurrency batch size ($B$). A 32B model at 128K context can consume <b>16GB+ VRAM per active session</b>.</p>
  </div>
  <div class="card gold">
    <h3>Cache Mitigation Techniques</h3>
    <ul>
      <li><b>Grouped-Query Attention (GQA):</b> Shares Key-Value heads across multiple query heads, cutting cache by 4x–8x.</li>
      <li><b>KV Quantization (FP8 / INT4):</b> Compresses cached keys and values, halving memory with negligible loss.</li>
      <li><b>PagedAttention (vLLM / SGLang):</b> Eliminates memory fragmentation like virtual memory pages in an OS.</li>
    </ul>
  </div>
</div>

<p class="source-note">Sizing principle: Maximum supported context window is not equal to default allocated context. Size cache buffers for actual user concurrency.</p>

---
layout: default
title: "9. Quantization mechanics & accuracy trade-offs"
---

## 9. Quantization mechanics &amp; accuracy trade-offs

<p>Quantization compresses weights from 16-bit floating point into 8-bit or 4-bit integers using scaling factors: $W_{\text{quant}} = \text{round}(W / S) + Z$. This cuts memory footprint and bus transfer by up to 75%.</p>

<QuantizerWidget />

<div class="grid3" style="margin-top: 8px;">
  <div class="card accent" style="padding: 8px 10px;"><b>16-bit (FP16)</b><br><small>Zero quality loss. Reference baseline for precision OCR and complex math.</small></div>
  <div class="card accent2" style="padding: 8px 10px;"><b>8-bit (FP8/INT8)</b><br><small>~99% accuracy retention. The industry standard sweet spot for local serving.</small></div>
  <div class="card accent3" style="padding: 8px 10px;"><b>4-bit (AWQ/GGUF)</b><br><small>Huge memory savings. Ideal for chat and extraction; slight loss on deep logic.</small></div>
</div>

<p class="source-note estimate-label">Quantization schemes like AWQ and GPTQ preserve critical salient weights to minimize perplexity degradation.</p>

---
layout: default
title: "10. Hardware architecture: Unified vs. Discrete"
---

## 10. Hardware architecture: Unified vs. Discrete

<p>Hardware selection involves two main paradigms: pooled high-capacity Unified Memory (Apple Silicon) versus dedicated ultra-high-bandwidth Discrete GPUs (Nvidia RTX / Enterprise).</p>

<BarChartWidget />

<div class="grid2" style="margin-top: 6px;">
  <div class="card accent2">
    <h3>Apple Silicon (Unified Memory)</h3>
    <p><b>Pros:</b> Huge addressable memory pools (up to 192GB) at desktop price points. Runs 70B models easily.<br><b>Trade-off:</b> Moderate bandwidth (300–800 GB/s) yields lower peak decode speeds.</p>
  </div>
  <div class="card warn">
    <h3>Discrete GPUs (Dedicated VRAM)</h3>
    <p><b>Pros:</b> Blazing memory bandwidth (1,000–3,350 GB/s) delivering 50–150+ tokens/sec decode speeds.<br><b>Trade-off:</b> Strict physical VRAM limits per card; requires multi-GPU scaling for &gt;70B models.</p>
  </div>
</div>

<p class="source-note estimate-label">Sizing methodology: First ensure Weights + KV Cache + 20% Headroom fits in total VRAM, then calculate bandwidth throughput.</p>

---
layout: default
title: "11. The Memory Wall: Why bandwidth bounds decode"
---

## 11. The Memory Wall: Why bandwidth bounds decode

<p>In single-user decode, the GPU executes a matrix-vector multiplication for each new token. Arithmetic intensity is extremely low ($\approx 1 \text{ FLOP / byte}$). The compute cores spend most cycles waiting for weight bytes to travel across the memory bus.</p>

<div class="math-box">
  <div class="math-formula">$$\text{Theoretical Max Decode (tokens/s)} \approx \frac{\text{Memory Bandwidth (GB/s)}}{\text{Model Footprint in VRAM (GB)}}$$</div>
  <div class="math-explanation">Example: An RTX 4090 ($1,008\text{ GB/s}$) serving a 27B model quantized to 4-bit ($14\text{ GB}$) has an upper speed limit of $\frac{1008}{14} \approx 72\text{ tokens/s}$.</div>
</div>

<div class="grid2">
  <div class="card accent">
    <h3>Hardware Bandwidth Spectrum</h3>
    <ul>
      <li><b>PCIe Bus Transfer (Offloading):</b> 16–64 GB/s $\rightarrow$ <i>Unusable (1–3 tok/s)</i></li>
      <li><b>DDR5 System RAM (CPU only):</b> 60–100 GB/s $\rightarrow$ <i>Slow (4–8 tok/s)</i></li>
      <li><b>Apple Unified Memory (M-Max/Ultra):</b> 300–800 GB/s $\rightarrow$ <i>Good (15–40 tok/s)</i></li>
      <li><b>Workstation GDDR6X (RTX 4090):</b> 1,008 GB/s $\rightarrow$ <i>Fast (50–90 tok/s)</i></li>
      <li><b>Datacenter HBM3 (H100 / H200):</b> 3,350–4,800 GB/s $\rightarrow$ <i>Ultra (100–250+ tok/s)</i></li>
    </ul>
  </div>
  <div class="card accent2">
    <h3>How Batching Breaks the Wall</h3>
    <p>When serving $B$ concurrent users simultaneously, the engine loads model weights <b>once</b> to process $B$ tokens together in parallel.</p>
    <span class="mono">Batching transforms memory-bound decode into compute-saturated matrix multiplication.</span>
  </div>
</div>

<p class="source-note">Key lesson: Adding a faster GPU with more CUDA cores does not speed up single-user decode—increasing memory bandwidth or quantizing does.</p>

---
layout: default
title: "12. Reasoning & Test-Time Compute"
---

## 12. Reasoning &amp; Test-Time Compute

<p>Modern reasoning models leverage test-time compute: generating internal thinking tokens in a hidden scratchpad to decompose problems, self-correct, and verify logic before answering.</p>

<ThinkingControl />

<div class="grid2">
  <div class="card gold">
    <h3>Preserve Thinking in Agents</h3>
    <p>Retaining prior thinking blocks across multi-turn tool interactions maintains intermediate logic and prevents agent hallucinations.</p>
  </div>
  <div class="card warn">
    <h3>System-Level Trade-off</h3>
    <p>Extra thinking tokens increase per-request latency, but drastically cut down on downstream retry loops and prompt engineering overhead.</p>
  </div>
</div>

<p class="source-note">Local serving APIs allow configuring reasoning token budgets per request via standard flags (e.g. reasoning_effort).</p>

---
layout: default
title: "13. Context Scaling: RoPE & YaRN"
---

## 13. Context Scaling: RoPE &amp; YaRN

<p>Rotary Position Embedding (RoPE) encodes token sequence order into attention vectors. Frequency interpolation methods allow scaling native context windows to 1,000,000+ tokens.</p>

<div class="grid2">
  <div class="card accent2 context-card">
    <div class="big">32K–256K</div>
    <h3>Native Context Window</h3>
    <p>The pre-trained attention limit. Delivers maximum associative accuracy for high-density document Q&amp;A and complex coding tasks without position distortion.</p>
    <span class="status-chip">native baseline</span>
  </div>
  <div class="card gold context-card">
    <div class="big">1M+</div>
    <h3>Extended Window (YaRN / NTK)</h3>
    <p>Compresses high-frequency attention dimensions to ingest entire repositories or books without retraining. Requires careful validation of retrieval needle-in-a-haystack accuracy.</p>
    <span class="status-chip">frequency scaling</span>
  </div>
</div>

<div class="callout"><b>Engineering Reality:</b> Context length directly drives KV cache memory footprint. Configure runtime context limits for actual workload requirements rather than theoretical maximums.</div>

<p class="source-note">Runtimes like vLLM and llama.cpp expose rope_scaling parameters (type: yarn, factor, original_max_position_embeddings) to activate extension.</p>

---
layout: default
title: "14. The Local Serving Architecture"
---

## 14. The Local Serving Architecture

<p>An enterprise-grade local AI service decouples model artifacts, inference runtimes, standard API contracts, and operational governance.</p>

<div class="architecture-flow serving-flow">
  <div class="architecture-stage"><span class="stage-number">01</span><b>Artifacts</b><small>SafeTensors / GGUF</small></div>
  <span class="flow-arrow">-&gt;</span>
  <div class="architecture-stage"><span class="stage-number">02</span><b>Engine</b><small>vLLM / SGLang / llama.cpp</small></div>
  <span class="flow-arrow">-&gt;</span>
  <div class="architecture-stage"><span class="stage-number">03</span><b>API Layer</b><small>OpenAI-compatible /v1</small></div>
  <span class="flow-arrow">-&gt;</span>
  <div class="architecture-stage"><span class="stage-number">04</span><b>Operations</b><small>Auth, Quotas, Telemetry</small></div>
</div>

```bash
# High-throughput production serving with continuous batching
vllm serve <org/model-id> \
  --host 0.0.0.0 --port 8000 \
  --max-model-len 32768 --gpu-memory-utilization 0.90
```

<div class="grid3" style="margin-top: 8px;">
  <div class="card accent" style="padding: 8px 10px;"><b>vLLM / SGLang</b><br><small>High-concurrency continuous batching, PagedAttention, Tensor Parallelism.</small></div>
  <div class="card accent2" style="padding: 8px 10px;"><b>llama.cpp / Ollama</b><br><small>Lightweight C/C++ engine, Apple Metal &amp; CPU support, zero dependencies.</small></div>
  <div class="card gold" style="padding: 8px 10px;"><b>OpenAI API Contract</b><br><small>Drop-in compatibility with existing SDKs, LangChain, LlamaIndex, and agents.</small></div>
</div>

<p class="source-note">Choosing the runtime depends on target hardware (Nvidia CUDA vs Apple Metal), concurrency requirements, and batching needs.</p>

---
layout: default
title: "15. Where local AI excels (and where cloud wins)"
---

## 15. Where local AI excels (and where cloud wins)

<p>Evaluate local deployment through the lens of data sovereignty, predictable unit economics, and operational constraints.</p>

<div class="grid2">
  <div class="card accent">
    <h3>Private Document Intelligence</h3>
    <p>Analyze confidential contracts, financial audits, and medical records where external data transmission is legally restricted.</p>
    <span class="use-case-tag">Sovereignty · Zero Egress</span>
  </div>
  <div class="card accent2">
    <h3>Agentic Codebase Tools</h3>
    <p>Integrate deep repository indexing, automated refactoring, and test generation inside your engineering security boundary.</p>
    <span class="use-case-tag">Low Latency · Developer Privacy</span>
  </div>
  <div class="card accent3">
    <h3>Multimodal Visual Inspection</h3>
    <p>Process high-resolution schematics, OCR documents, and factory inspection frames locally without WAN upload latency.</p>
    <span class="use-case-tag">Vision · Edge Processing</span>
  </div>
  <div class="card gold">
    <h3>Air-Gapped Operations</h3>
    <p>Critical infrastructure, defense, maritime, and field environments requiring 100% offline autonomy and uptime.</p>
    <span class="use-case-tag">Offline · Total Resilience</span>
  </div>
</div>

<div class="callout"><b>When hosted cloud still wins:</b> Rapid MVP exploration, unpredictable extreme traffic spikes, or workloads requiring &gt;400B frontier models.</div>

---
layout: default
class: lead-alt
title: "16. A local request, end to end"
---

<div class="demo-flag">LIVE CASE STUDY · QWEN 3.8</div>

# A local request, end to end

<h2 class="subtitle">Execution trace walkthrough running a local OpenAI-compatible endpoint with Qwen 3.8 (27B) as the multimodal serving target.</h2>

<ul class="demo-steps">
  <li><span class="tick">1</span><span>Client sends JSON request with multimodal prompt &amp; reasoning controls to <span class="mono">localhost:8000/v1/chat/completions</span></span></li>
  <li><span class="tick">2</span><span>Local engine executes ViT image patching, runs parallel prefill, and initiates test-time reasoning</span></li>
  <li><span class="tick">3</span><span>Response streams back token-by-token privately over Server-Sent Events (SSE) with zero WAN latency</span></li>
</ul>

<TerminalWidget />

---
layout: default
title: "17. The 6-Point Production Checklist"
---

## 17. The 6-Point Production Checklist

<p>Before procuring hardware or committing infrastructure, validate these 6 technical and operational readiness gates:</p>

<div class="checklist-grid">
  <div class="check-item"><span class="check-mark">01</span><div><b>Data Boundary</b><small>Which compliance or privacy policies mandate on-premise execution?</small></div></div>
  <div class="check-item"><span class="check-mark">02</span><div><b>Workload Metrics</b><small>What are average prompt length, output token counts, and peak concurrency?</small></div></div>
  <div class="check-item"><span class="check-mark">03</span><div><b>VRAM Capacity</b><small>Does Weight Memory + KV Cache + 20% Headroom fit in GPU VRAM?</small></div></div>
  <div class="check-item"><span class="check-mark">04</span><div><b>Decode Bandwidth</b><small>Does GPU memory bus bandwidth deliver the target Tokens-per-Second?</small></div></div>
  <div class="check-item"><span class="check-mark">05</span><div><b>Serving Engine</b><small>Is the runtime configured with continuous batching and optimal quantization?</small></div></div>
  <div class="check-item"><span class="check-mark">06</span><div><b>Fallback &amp; HA</b><small>What is the failover strategy when local GPU capacity saturates?</small></div></div>
</div>

<div class="callout"><b>Production Principle:</b> Model cards define theoretical capabilities; empirical traces on your target hardware determine production viability.</div>

---
layout: default
class: lead-alt
title: "18. The Local AI Mental Models"
---

<div class="eyebrow">THE DECISION IN ONE MINUTE</div>

# The Local AI Mental Models

<ul class="demo-steps takeaway-list">
  <li><span class="tick">1</span><span><b>The System Lens</b> — Local AI is an architectural commitment to data sovereignty, dedicated silicon, and operational ownership.</span></li>
  <li><span class="tick">2</span><span><b>The Sizing Law</b> — Total VRAM = Weight Footprint + KV Cache + Activations. Long context and concurrency can easily exceed weight size.</span></li>
  <li><span class="tick">3</span><span><b>The Memory Wall</b> — Single-token decode is strictly bounded by memory bandwidth (GB/s). Quantization and batching are the primary levers for throughput.</span></li>
  <li><span class="tick">4</span><span><b>Workload-First Design</b> — Benchmark empirical traces on representative hardware before making infrastructure commitments.</span></li>
</ul>

<h2 class="subtitle" style="margin-top: 14px;">Questions &amp; Discussion</h2>

<p class="source-note">Local AI Field Guide · Open Systems, Architectures &amp; Inference Physics · 2026</p>
