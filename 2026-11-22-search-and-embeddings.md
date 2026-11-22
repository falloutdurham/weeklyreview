# Search & Embeddings Weekly — 22 November 2026

> Papers from the past few weeks worth your attention, weighted toward late interaction, token pruning/compression, and query generation.
>
> **Note:** `xcancel.com` and direct `arxiv.org` are blocked in this cloud session's network policy. Twitter timelines for `@bclavie` and `@jobergum` could not be fetched directly — if you want those in future runs, ask the host to whitelist `xcancel.com` and `arxiv.org`. Papers below are sourced via web search.

---

## Late Interaction & Token Pruning

### ⭐ Voronoi Cell Formulation for Principled Token Pruning — `arXiv:2603.09933` (SIGIR 2026)
The best late-interaction token pruning paper of the cycle. Frames the token pruning problem geometrically: each retained token implicitly defines a Voronoi cell in embedding space, and tokens are dropped when their cell is "covered" by a neighbour. Works purely from the MaxSim mechanism, so it generalises beyond ColBERT to any late-interaction model. Accepted at SIGIR 2026.

### ⭐ EigenLI: Spectral Approximations to Late Interaction — `arXiv:2609.07561`
Approximates ColBERT-style MaxSim scoring via spectral decomposition, reducing the per-query compute significantly. Worth reading alongside the Voronoi paper for a complementary compression angle (storage vs. query-time).

### Query-Aware Token Budgeting for Visual Document Retrieval — `arXiv:2609.07262`
Addresses the specific pain of late-interaction visual retrievers (storing a dense embedding per *page patch* gets expensive fast). Allocates token budget dynamically per query. Cites Col-Bandit (`arXiv:2602.02827`) for zero-shot query-time pruning as a baseline.

### Generative Late-Interaction Embeddings for Visual Document Retrieval — `arXiv:2609.11808`
Applies generative modelling on top of a late-interaction architecture for visual/document retrieval. Pairs well with the token-budgeting paper above.

### LITTA: Late-Interaction & Test-Time Alignment for Multimodal Retrieval — `arXiv:2603.26683`
Query-expansion framework for evidence-page retrieval using a frozen Nemotron ColEmbed V2 backbone. No retriever retraining required — adds test-time alignment on top of an existing late-interaction model.

### AMES: Multimodal Enterprise Search via Late Interaction — `arXiv:2603.13537`
First unified multimodal late-interaction retrieval system built inside a Lucene-based enterprise search engine. Preserves metadata-aware filtering and operational compatibility. Practically relevant if you're thinking about production deployment.

### ColGraphRAG — `arXiv:2607.16208`
Replaces single-vector visual ranking in a multimodal GraphRAG pipeline with MaxSim scoring over graph-linked image nodes. Text and table retrieval stay dense/sparse; images get the late-interaction treatment.

---

## Query Generation & Expansion

### ⭐ DESA: Dense Expands, Sparse Anchors — `arXiv:2608.15851`
Shares LLM-generated reference text across both dense and sparse channels but assigns each a *different role*: dense uses the expansion for semantic lift, sparse uses the original query as anchor for precision. +3.82% nDCG@10, +2.38% Recall@20 across seven BEIR datasets. Cuts retrieval depth by ~37% for both channels. Clean idea worth stealing.

### AnchorQE: Query Expansion Is More Than Generation — `arXiv:2608.25521`
Argues that prior work over-indexes on *what* to generate and under-indexes on *how to integrate* the expansion into dense retrieval. Encodes the original query and LLM expansion separately, then interpolates embeddings. Training-free.

### Generated Query Expansion Still Helps SPLADE-v3 — `arXiv:2609.37911`
Controlled study: LLM-generated query expansion still lifts a strong learned sparse retriever in 103 of 114 interpolation settings. The key: keep at least 30% weight on the original query. Useful sanity-check before shipping any QE pipeline.

### HyPE: Hypothetical Prompt Embeddings — `arXiv:2607.29402`
Moves HyDE's hypothetical-document generation from *query time* to *index time*, eliminating runtime LLM calls. Practical efficiency gain if you like HyDE's idea but hate the latency. Several 2026 papers (AstroGenesis, multimodal RAG studies) confirm HyDE's gains are inconsistent at high retrieval depth — offline generation is a sensible hedge.

---

## Sparse Retrieval

### ⭐ SPARSEUP: Better Recipes for Modern Learned Sparse Retrievers — `arXiv:2611.09680`
November 2026. Training recipe improvements for SPLADE-style models. Key paper to read if you're maintaining a sparse retrieval pipeline.

### LACONIC: Dense-Level Effectiveness for Scalable Sparse Retrieval — `arXiv:2601.01684`
Two-phase training curriculum to close the effectiveness gap between learned sparse (SPLADE-style) and dense retrievers while keeping sparse inference costs. Worth pairing with SPARSEUP.

### Rescaling MLM Head for Neural Sparse Retrieval — `arXiv:2606.18811`
Post-hoc calibration of the MLM projection head used in SPLADE to fix distribution mismatch. Low-engineering-effort improvement.

---

## Reranking

### ⭐ Yuanbao-Reranker (0.6B / 4B / 8B) — `arXiv:2611.10405`
November 2026. Dense reranker family built on Qwen3 at three sizes. Training: retrieval pools filtered by teacher score margins, evaluated on Chinese, multilingual, English, long-document, and code benchmarks. If you need a strong Chinese/multilingual reranker this is the one to benchmark.

### Preference Is Not Position: Reranking on NevIR — `arXiv:2610.26260`
Tests rerankers on near-identical passage pairs differing only by a decisive detail (negation, qualifier). Pairwise-trained reranker improves strict per-query preference but *lowers* preferred-passage MRR — a good reminder that reranker eval metrics diverge on nuanced pairs.

---

## Dense Embeddings & Compression

### H+ Embedding: Harmonizing Global and Token-Level Retrieval — `arXiv:2608.00065`
Unifies a global (single-vector) and token-level (multi-vector) representation in one encoder with context-dependent phrase pooling. Effectively a lightweight late-interaction path without full ColBERT cost.

### Relevance-Guided Residual Calibration for Dense Retrieval Compression — `arXiv:2610.37707`
Spectral compression of dense retrieval embeddings using residual calibration guided by relevance signals. Useful for anyone trying to shrink embedding storage.

### Embedding Compression via Spherical Coordinates — `arXiv:2602.00079`
Lossless compression for unit-norm embeddings: 1.5× compression, 25% better than the previous best lossless method. If you're storing millions of embeddings this is a quick win.

### LEAF: Knowledge Distillation for Retrieval — `arXiv:2509.12539` (ICLR 2026)
State-of-the-art on BEIR for models ≤100M parameters. Worth looking at the distillation recipe if you're building compact retrievers.

### Is Dimensionality a Barrier for Retrieval Models? — `arXiv:2605.23556`
Theoretical analysis of what dense embedding dimensions can and cannot represent. Pairs well with the ICLR paper on theoretical limitations (`arXiv:2508.21038`) for anyone thinking hard about capacity vs. generalization.

---

## Multimodal / Visual Document Retrieval

### EVIE: Evidence-Vector-Informed Embeddings — `arXiv:2610.11553`
October 2026. Visual document retrieval that injects explicit evidence vectors into the embedding process. Interesting if you're working in the visual RAG space.

### PUMA: Post-Hoc Sparsification of Universal Multimodal Embeddings — `arXiv:2608.25780`
Takes dense multimodal embeddings and sparsifies them post-hoc for efficient retrieval. Bridges the dense-multimodal and learned-sparse worlds.

### UEmbed: Unified Sparse and Dense Multimodal Embeddings — `arXiv:2608.02583`
Single model outputting both sparse and dense representations for multimodal inputs. Good companion to PUMA for understanding the design space.

---

## RAG & Context Compression

### OpticalRAG: Pixel-Space Compression for Token-Efficient RAG — `arXiv:2611.13675`
November 2026. Compresses retrieved documents via visual encoding instead of text pruning or single-vector summarisation. Avoids the information loss that comes with high-ratio text-space compression. Novel angle — the demo idea is clever even if production viability needs more evidence.

---

## Community / Meta

### LIR Workshop @ ECIR 2026 — `arXiv:2511.00444`
The first dedicated Late Interaction and Multi-Vector Retrieval workshop proceedings. Good overview of where the community is pointing.

---

## Quick Priority List

| Must Read | Worth Skimming |
|---|---|
| Voronoi Token Pruning (2603.09933) | Query-Aware Token Budgeting (2609.07262) |
| DESA: Dense Expands, Sparse Anchors (2608.15851) | HyPE offline HyDE (2607.29402) |
| SPARSEUP (2611.09680) | Preference Is Not Position reranking (2610.26260) |
| Yuanbao-Reranker (2611.10405) | H+ Embedding (2608.00065) |
| EigenLI (2609.07561) | LEAF distillation (2509.12539) |
