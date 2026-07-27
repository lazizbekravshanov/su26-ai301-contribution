_Ready to post the moment mteb #5026 is approved & merged. Lead maintainer already reviewed it "looks good, a few minor comments" (2026-07-26) — this fires on the merge._

---

🎉 Cycle 7 merged: SciRepEval is now in MTEB! (embeddings-benchmark/mteb #5026)

🔬 MTEB is the standard benchmark for text-embedding models. My PR adds four SciRepEval scientific-paper classification tasks — DRSM (5-class research state), Biomimicry (relevance), Fields of Study (multi-label), and MeSH descriptors (30-class) — so embedding models can now be evaluated on scientific-document understanding.

✅ That's 7 merged PRs and counting, across 6 projects & 3 languages (Python, Rust, TypeScript). I opened this with a single task; the maintainer asked me to cover the whole classification family, so I expanded it to four — each verified against a real model vs a random baseline to prove it measures signal, not noise.

💡 Takeaway: scope grows through review. What shipped is 4× what I opened — because a maintainer saw the pattern was worth generalizing, and said so. Reviews aren't a gate to clear; they're where the contribution gets better.
