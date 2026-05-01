# Chunking Strategy Recommendations

## Summary

This document consolidates findings from three chunking experiments —
fixed-size, recursive character, and token-based — plus a boundary
quality evaluation, across two content types: a structured PDF
(EU AI Ethics Guidelines) and a conversational podcast transcript
(Blueprint for Trustworthy AI).

---

## Strategy Comparison Table

| Strategy | Chunk Precision | Boundary Quality | Token Control | Speed | Best For |
|---|---|---|---|---|---|
| Fixed-size (char) | Low (±30% token variance) | Poor — frequent mid-sentence cuts | None | Very fast | Prototyping, uniform content |
| Recursive (char) | Low (±30% token variance) | Good for PDFs, fair for podcasts | None | Fast | Structured documents |
| Token-based | Exact (±1–2 tokens) | Poor — no separator awareness | Full | Fast | LLM integration, context budgeting |
| Recursive + token verify | Exact | Good | Full | Moderate | Production RAG pipelines |

---

## For PDF Documents

**Recommended strategy:** Recursive character splitting + token verification

**Reasoning:**

- PDF text extracted from structured documents (headings, sections,
  paragraphs) contains natural `\n\n` boundaries that
  `RecursiveCharacterTextSplitter` detects as first-priority split
  points. This means section headers reliably land at chunk starts
  rather than mid-chunk.
- The boundary quality audit showed fixed-size chunking breaks
  mid-sentence in a high proportion of chunks at size 500. Recursive
  splitting reduces this significantly by preferring paragraph and
  sentence boundaries first.
- Token verification with `tiktoken` is still necessary because
  character-based recursive splitting has ±30% token variance — a
  500-character chunk can contain anywhere from ~120 to ~180 tokens
  depending on word density. Without verification, a long technical
  section will silently overflow the LLM's context budget.

**Optimal settings:**

- Chunk size: **1000 characters** (≈ 250–300 tokens) — balances
  retrieval granularity with coherent context per chunk.
- Chunk overlap: **200 characters** — enough to carry sentence
  continuity across boundaries without excessive duplication.
- Separators: `["\n\n", "\n", ". ", " ", ""]`
- Post-split: verify all chunks are under your embedding model's token
  limit (typically 512 tokens for `text-embedding-ada-002`).

**Key advantages for PDFs:**

- Section headers are preserved at chunk starts, making retrieved
  chunks self-explanatory without surrounding context.
- Low overlap sensitivity at large chunk sizes (only +5% chunk count
  from overlap 0 → 100 at size 2000) means you can tune overlap
  aggressively without exploding your vector store size.
- The structured prose of academic/technical PDFs means sentence
  boundaries are well-punctuated, giving recursive splitting reliable
  split signals.

---

## For Podcast Transcripts

**Recommended strategy:** Pre-process transcript → then recursive
character splitting with speaker-turn separators

**Reasoning:**

- Raw podcast transcripts have almost no `\n\n` boundaries and use
  long, unpunctuated run-on sentences — the two signals that recursive
  splitting relies on most. Without pre-processing, recursive splitting
  falls back to splitting on spaces (`" "`), which is barely better
  than fixed-size.
- The boundary quality audit confirmed this: podcasts showed higher
  mid-sentence break rates than PDFs at every chunk size and strategy
  combination.
- Overlap sensitivity is also higher for podcasts (+25% chunk count
  from overlap 0 → 100 at size 500) because conversational speech
  flows continuously without natural stopping points — meaning more
  overlap is needed to recover split context, which in turn inflates
  chunk count significantly.

**Recommended pre-processing steps (before splitting):**

```python
import re

def clean_transcript(text: str) -> str:
    # 1. Normalise speaker labels to consistent separators
    text = re.sub(r'\n?(HOST|GUEST|SPEAKER \d+):', r'\n\n\1:', text)
    # 2. Add sentence-ending punctuation to lines that lack it
    text = re.sub(r'([a-z])\n', r'\1.\n', text)
    # 3. Collapse excessive whitespace
    text = re.sub(r'\n{3,}', '\n\n', text)
    return text.strip()

podcast_text_clean = clean_transcript(podcast_text)
```

**Optimal settings (after pre-processing):**

- Chunk size: **1000 characters** (≈ 220 tokens for conversational
  text, which averages ~4.5 chars/token vs ~3.5 for PDFs).
- Chunk overlap: **100–150 characters** — more than PDF because
  conversational context is harder to recover mid-thought.
- Separators: `["\n\nHOST:", "\n\nGUEST:", "\n\n", "\n", ". ", " ", ""]`
  (add speaker labels as top-priority separators if your transcript
  has them).

**Key advantages after pre-processing:**

- Speaker-turn boundaries become reliable split points, keeping each
  chunk within a single speaker's turn rather than blending two voices.
- Normalised punctuation gives the `". "` separator a signal to work
  with, reducing mid-sentence breaks significantly.
- For downstream RAG, chunks that start at speaker turns are much more
  interpretable — the retriever can surface who said what rather than
  a fragment of an answer.

---

## Trade-offs Summary

| Strategy | Pros | Cons | Best For |
|---|---|---|---|
| Fixed-size (char) | Simple, no dependencies, predictable chunk count | Breaks sentences and paragraphs freely; ±30% token variance makes context budgeting unreliable | Rapid prototyping, uniform content where boundary quality does not matter |
| Recursive (char) | Respects `\n\n` → `\n` → `. ` → ` ` hierarchy; meaningful improvement for structured docs; configurable separators | Still has token variance; podcasts get little benefit without pre-processing; slightly more complex to configure | Structured documents (PDFs, articles, reports) where paragraph and sentence boundaries are well-defined |
| Token-based | Exact token count per chunk (±1–2 tokens); eliminates context window overflow risk; essential for LLM integration | No boundary awareness — as structure-blind as fixed-size; requires `tiktoken` or equivalent; 500 tokens ≠ 500 characters | Any pipeline where the LLM context budget must be respected precisely; embedding models with strict token limits |
| Recursive + token verify | Best of both worlds: boundary-aware splitting with exact token count validation; production-grade | Two-step pipeline; slightly more code complexity; re-splitting oversized chunks can degrade boundary quality | Production RAG pipelines where both retrieval quality and context budgeting matter |

---

## Decision Guide

```
Does your content have clear structural markers (\n\n, headers)?
├── YES (PDF, article, report)
│   └── Use: Recursive + token verify
│       Size: 1000 chars / 200 overlap
│
└── NO (podcast, raw transcript, chat logs)
    ├── Can you pre-process the transcript?
    │   ├── YES → Clean + add speaker separators → Recursive + token verify
    │   │         Size: 1000 chars / 100-150 overlap
    │   └── NO  → Token-based (at least controls context budget)
    │             Size: 256–512 tokens / 50 overlap
```

---

## Key Findings from the Experiments

1. **Fixed-size chunking is never production-ready.** It is useful only
   for learning and rapid iteration. Its ±30% token variance makes it
   unreliable for any LLM pipeline.

2. **Chars per token varies by content type.** PDFs average ~3.5
   chars/token (dense technical vocabulary); podcasts average ~4.5
   chars/token (short common words). The same character chunk size
   sends very different token loads to the model depending on the source.

3. **Overlap sensitivity reveals content structure.** Low overlap
   sensitivity (PDF at size 2000: only +5% chunk count from ov0 → ov100)
   means the content has natural split points. High overlap sensitivity
   (podcast at size 500: +25%) means the content flows without
   boundaries — a warning sign that boundary-aware splitting will
   struggle.

4. **The ideal pipeline is recursive split → token verify → embed.**
   Use `RecursiveCharacterTextSplitter` for boundary quality, then
   verify every chunk's token count with `tiktoken`, and discard or
   re-split any chunk that exceeds your embedding model's token limit.
