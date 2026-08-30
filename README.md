# Local AI Field Guide — Systems, Architecture & Deployment

A 20-slide systems engineering guide to open-weight models: boundary architecture, inference physics, memory sizing, and private serving. Built with [Slidev](https://sli.dev/).

---

## 🚀 Quick Start

### 1. Install dependencies
```bash
npm install
```

### 2. Start the development presentation server
```bash
npm run dev
```

The presentation will be available at `http://localhost:3030`.

---

## 🛠️ Commands

- `npm run dev` — Start the local development server with live reload and presenter mode.
- `npm run build` — Build static single-page application into `dist/`.
- `npm run export` — Export slides to PDF or PNG format.

---

## 📋 Slide Structure & Contents

1. **Cover** — *Running AI locally (Local AI Field Guide)*
2. **Agenda** — *A systems tour (Boundaries, Architectures, Physics, Serving)*
3. **Local AI is a system property** — *Data, compute, and service boundaries*
4. **Where does the request go?** — *Hosted Cloud Inference vs. Local & Private Inference*
5. **Open weights: The model landscape** — *1B–3B to 70B+ model parameter tiers*
6. **Architecture: Dense, MoE & Hybrids** — *Attention backbones & linear state spaces*
7. **The multimodal pipeline & token budget** — *Text, image patching, and video tokenization*
8. **Inference mechanics: Prefill vs. Decode** — *Compute-bound prefill vs. bandwidth-bound decode*
9. **Weight memory: Baseline VRAM sizing** — *Irreducible weight sizing formula and hardware tiers*
10. **KV Cache: Why context eats memory** — *Linear context scaling and mitigation (GQA, PagedAttention)*
11. **Quantization mechanics & accuracy trade-offs** — *Interactive 16-bit, 8-bit, and 4-bit precision simulator*
12. **Hardware architecture: Unified vs. Discrete** — *Interactive hardware VRAM tiers and GPU bandwidth comparison*
13. **The Memory Wall: Why bandwidth bounds decode** — *Memory bus bandwidth limits and batching saturation*
14. **Reasoning & Test-Time Compute** — *Interactive reasoning effort simulator and agent logic*
15. **Context Scaling: RoPE & YaRN** — *Native context vs. YaRN frequency extension*
16. **The Local Serving Architecture** — *Decoupled artifacts, engines (vLLM, llama.cpp), and OpenAI API contract*
17. **Where local AI excels (and where cloud wins)** — *Data sovereignty, agentic tools, vision, and air-gapped ops*
18. **A local request, end to end** — *Interactive live case study with animated terminal endpoint trace*
19. **The 6-Point Production Checklist** — *Technical & operational readiness gates*
20. **The Local AI Mental Models** — *Core takeaways, summary pillars & discussion*

---

## 🧩 Interactive Components

- `<QuantizerWidget />` — Precision trade-off simulator (FP16, INT8/FP8, AWQ/GGUF).
- `<BarChartWidget />` — Animated hardware memory tier chart (24 GB, 48 GB, 80 GB+, Multi-GPU).
- `<TerminalWidget />` — Live terminal trace with typewriter playback and re-run trigger.
- `<ThinkingControl />` — Reasoning effort selector with reactive explanations.
