---
title: "Hello, world — and a note on DNA tokenization"
date: 2026-05-08T10:00:00+08:00
draft: false
tags: ["meta", "dna-modeling", "tokenization"]
categories: ["notes"]
summary: "Why I'm starting this blog, and a quick orientation on the strange landscape of DNA tokenization."
ShowToc: true
TocOpen: false
math: false
---

This is the first post. The point of this blog is selfish: I read a lot of papers on DNA sequence modeling and forget most of what I read. Writing things down — in my own words, with code I've actually run — is the only way I've found to make a paper stick.

So expect mostly **paper notes** and **code walkthroughs**, with the occasional rant about what "understanding a sequence" should mean.

To make this post more than a placeholder, here's a small orientation post on something every DNA model has to decide first: how to tokenize.

## Why tokenization is weirder for DNA than for text

In NLP, byte-pair encoding (BPE) more or less won. The vocabulary it produces is vaguely interpretable — `playing` becomes `play`+`ing`, `unhappiness` becomes `un`+`happi`+`ness` — and the units roughly correspond to morphemes humans already use.

DNA does not work this way. The "alphabet" is four letters (`A`, `C`, `G`, `T`), the meaningful structures span scales from 1 bp (a SNP) to 10⁵+ bp (a topologically associating domain), and there are no natural word boundaries. Every choice you make about tokens is a bias.

Roughly, four families of choice are in use:

### 1. Single-nucleotide tokens

Treat each base as one token. Vocabulary size 4 (or 5 with `N`).

- ✓ No information loss; every position is addressable.
- ✗ Sequence length explodes. A 1 Mb region is 10⁶ tokens, which kills any quadratic-attention model.

This is what **HyenaDNA** and **Caduceus** do, leaning on sub-quadratic architectures (state-space / Mamba) so they can actually swallow long sequences at single-nt resolution.

### 2. k-mer tokens (overlapping or non-overlapping)

Slide a window of size *k* and emit each k-mer as a token. Vocabulary size 4ᵏ.

- ✓ Compresses the sequence by a factor of *k* (non-overlapping) or stays the same length (overlapping).
- ✗ With overlapping k-mers, adjacent tokens share `k-1` bases — there's massive information leakage into the input itself, which makes masked-language-modeling pretraining a bit dishonest. **DNABERT v1** used overlapping 6-mers and several follow-ups have argued this is a problem.

### 3. BPE on DNA

Treat the DNA string like a corpus and run BPE. The model learns its own vocabulary of variable-length motifs.

- ✓ Adaptive: motifs that are over-represented (e.g. `TATA`-box-like patterns, repeats) become single tokens.
- ✗ The vocabulary is opaque, and the same biological motif may end up split differently in different contexts.

**Nucleotide Transformer** is the prominent example of this approach (with non-overlapping 6-mers as a fallback option).

### 4. Hybrid / hierarchical

E.g. **Evo** uses byte-level tokenization (single-nt, vocabulary ~256 to also cover ambiguity codes), and relies on the architecture (StripedHyena) to handle the length blow-up.

## A useful mental model

When evaluating a paper, the question I find most clarifying is:

> **What's the smallest variation in the input that the model is allowed to *see* as a different input?**

For single-nt tokenization, that's 1 bp. For non-overlapping 6-mers, it's effectively 6 bp at the boundaries (a single SNP can either be invisible or shift the whole tokenization). This bounds the model's ceiling on tasks like variant effect prediction, regardless of how big the model is.

## Where I want to go next

A few posts I'm planning:

1. A code walkthrough of training a tiny single-nt transformer on the human genome (just to feel the pain of long contexts firsthand).
2. Notes on **Caduceus** — what reverse-complement equivariance actually buys you.
3. A skeptical look at the standard probing benchmarks (GUE, Genomics Benchmarks): what they measure vs. what they claim to measure.

If you have suggestions or find errors here, the email link is on the [About](/about/) page.

---

```python
# A toy: count how many tokens a 1 Mb region produces under each scheme
seq_len = 1_000_000
schemes = {
    "single-nt":              seq_len,
    "6-mer (overlapping)":    seq_len - 5,
    "6-mer (non-overlapping)": seq_len // 6,
    "BPE (avg ~4 bp/token)":  seq_len // 4,
}
for name, n in schemes.items():
    print(f"{name:30s} {n:>10,} tokens")
```

```text
single-nt                      1,000,000 tokens
6-mer (overlapping)              999,995 tokens
6-mer (non-overlapping)          166,666 tokens
BPE (avg ~4 bp/token)            250,000 tokens
```

The factor-of-six drop from non-overlapping k-mers is the only reason quadratic-attention DNA models can run on anything resembling a real genomic region. Whether it's worth the bp-resolution loss is the question.
