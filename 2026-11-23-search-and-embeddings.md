# Search & Embeddings Weekly — 23 November 2026

> Delta report on top of [last week's](2026-11-22-search-and-embeddings.md) (not repeated here). Focus: late interaction, token pruning/compression, query generation.
>
> **Coverage caveat:** `arxiv.org` and `xcancel.com` are still blocked by this session's network policy (DNS fails), so there is **no direct read of cs.IR listings or of @bclavie / @jobergum**. Web search surfaced no verifiable posts from either account or any `2611.*` papers this week. Everything below comes from search results; summaries are from snippets/abstracts, not full reads. To fix: allow `arxiv.org` and `xcancel.com` in the environment network settings.

## Late interaction

- **Quantifying and Expanding the Theoretical Capacity of Late-Interaction Retrieval Models** — [`2607.05803`](https://huggingface.co/papers/2607.05803.md). Capacity analysis of multi-vector scoring; pairs with last week's dimensionality-limits papers. *Read.*
- **Exploring the Capabilities and Limits of Late Interaction** — [ACM DL, 6 Nov 2026](https://dl.acm.org/doi/10.1145/3799682.3840035) (not on arXiv). Argues late interaction beats single-vector scoring mainly through higher representational capacity or lexically bound token embeddings. *Read; directly relevant to why MaxSim works.*
- **Fully Open Dense and Late-Interaction Models for Multilingual, Long-Context, and Code Search** — [`2607.27178`](https://huggingface.co/papers/2607.27178.md). Open model family with both dense and ColBERT-style variants. *Check for checkpoints and PyLate compatibility.*
- **SMART: "Your Embedding Model is SMARTer Than You Think"** — [`2605.24938`](https://arxiv.org/html/2605.24938). Derives 128-d token vectors from jina-embeddings-v4's frozen single-vector states, scoring 83.98 vs 83.19 for its trained native multi-vector mode (16x smaller token dim). *Interesting cheap route to multi-vector.*
- Also surfaced: a comparison of training-free multi-vector compression methods and a study of length bias in late-interaction scoring (titles not captured; [`2601.05200`](https://arxiv.org/html/2601.05200v1) appeared in the same results, unverified).

## Token pruning / compression (background that resurfaced)

- Voronoi pruning [`2603.09933`](https://arxiv.org/pdf/2603.09933): keeps 98.0% of unpruned ColBERTv2 effectiveness at a 50% token budget (last week's pick, now with numbers).
- Lossless pruning via vector dominance [`2504.12778`](https://arxiv.org/html/2504.12778) and CRISP clustering [`2505.11471`](https://arxiv.org/pdf/2505.11471): older, but the natural comparisons for the Voronoi work.
- **ReinPool** [`2601.07125`](https://arxiv.org/html/2601.07125): RL-trained pooling of multi-vector into one vector; 746–1249x compression recovering 76–81% of multi-vector quality on ViDoRe V2, but poor for a 3072-d model.
- **ResComEmb** [`2609.37225`](https://arxiv.org/pdf/2609.37225): residual homogeneity compression for multimodal embeddings, evaluated on MMEB and ViDoRe against ColPali/ColQwen2.
- HPC-ColPali [`2506.21601`](https://arxiv.org/pdf/2506.21601): patch compression + pruning + quantization for ColPali.

## Query / data generation

- **SYNTRA: Self-improving data synthesis for embedding models** — [`2610.14730`](https://arxiv.org/html/2610.14730v1). Self-improving synthetic-data loop; 24 retrieval datasets across 15 domains; MS MARCO gains at every scale over a 32x range of synthetic data size. *Most relevant new item for query generation.*
- Recipes worth revisiting alongside it: CADET [`2505.19274`](https://arxiv.org/html/2505.19274v1) (diverse query types beat single types), Arctic-Embed [`2405.05374`](https://arxiv.org/html/2405.05374) (query generation grounded by mined hard negatives), Qwen3 Embedding [`2506.05176`](https://arxiv.org/pdf/2506.05176) (two-stage synthetic pipeline: sample type/difficulty/persona, then generate).
- Multi-query retrieval: [`2511.02770`](https://arxiv.org/html/2511.02770v1), capturing diverse targets beyond a single embedding.

## Tooling / meta

- PyLate [`2508.03555`](https://ar5iv.labs.arxiv.org/html/2508.03555) and [Sentence Transformers multi-vector blog](https://huggingface.co/blog/multi-vector-encoder) for training late-interaction models.
- Simple Projection Variants Improve ColBERT [`2510.12327`](https://arxiv.org/html/2510.12327v1).
- LIR workshop [`2511.00444`](https://arxiv.org/abs/2511.00444).
- Choosing a Text Embedding Model: benchmarking and decision framework [`2607.23507`](https://arxiv.org/pdf/2607.23507).

## Priority this week

1. 2607.05803 (late-interaction capacity) + the ACM "limits of late interaction" paper
2. SYNTRA 2610.14730 (synthetic query/data generation)
3. SMART 2605.24938 and ReinPool 2601.07125 (cheap multi-vector / compression routes)
4. 2607.27178 (open dense + late-interaction models)
