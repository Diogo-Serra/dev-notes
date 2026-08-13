## Overview

This is background theory, not project code - it explains the ML/LLM concepts that the rest of the [[Projects/Call_Me_Maybe/Call_Me_Maybe|Call_Me_Maybe]] notes assume you already know. Read this first if any of "logits", "tokens", "autoregressive", or "BPE" are unfamiliar.

```
text prompt
   ↓ tokenization
token ids
   ↓ embedding lookup + transformer layers
hidden states
   ↓ output projection
logits (one score per vocabulary entry)
   ↓ sampling / argmax
next token id
   ↓ append, repeat
```

---

## 1. Tokens and Tokenization

Neural networks work on numbers, not text. **Tokenization** is the step that turns a string into a sequence of integers (**token ids**) the model understands.

Models don't tokenize by whole words - that would need a vocabulary of millions of entries to cover every word, typo, and language. Instead they use **subword tokenization**: a fixed vocabulary (tens of thousands of entries) of common whole words, word-pieces, and single characters as a fallback. Rare or made-up words get split into several tokens; common words are a single token.

> [!info] Why subword tokenization?
> It gives a small, fixed vocabulary size while still being able to represent *any* string (worst case: fall back to individual characters/bytes), unlike whole-word vocabularies which choke on anything unseen during training.

### Byte Pair Encoding (BPE)

BPE is the specific algorithm used to build that subword vocabulary, originally a data-compression technique:

1. Start with every individual character (or byte) as a token.
2. Count every adjacent pair of tokens across the training corpus.
3. Merge the single most frequent pair into one new token.
4. Repeat steps 2-3 thousands of times.

The result is a vocabulary that has learned common multi-character chunks (`"ing"`, `"tion"`, whole frequent words) while still being able to fall back to individual characters for anything unusual. The list of merge rules learned this way is what a `merges.txt` file stores.

### Byte-level BPE (GPT-2 / Qwen style)

Plain BPE over *characters* can still fail on unusual Unicode. GPT-2 (and the Qwen tokenizer this project uses) instead runs BPE over the 256 raw **bytes** of the UTF-8 encoding, so literally any input can always be tokenized.

The catch: many of those 256 byte values are unprintable (space, newline, control characters), which is awkward to store in a plain-text/JSON vocabulary file. GPT-2's fix is a second, unrelated trick: a fixed **byte ↔ unicode remapping table** that shifts the ugly bytes onto visible unicode characters purely for storage - e.g. a literal leading space becomes the printable character `Ġ` in `vocab.json`.

> [!info] Two different mappings, easy to confuse
> 1. **BPE merges** (`merges.txt`) - *how the vocabulary was built* from training data.
> 2. **Byte↔unicode remapping** - *how each vocab entry is written down* as printable text in `vocab.json`.
>
> This project never re-runs step 1 (that already happened when the tokenizer was trained, and `llm.encode()` does it for us). It only reverses step 2, to read `vocab.json` correctly - see [[Projects/Call_Me_Maybe/Models|Models]] (`Vocabulary._byte_decoder`).

---

## 2. Logits

Once tokenized, the input ids are looked up in an embedding table (turning each id into a vector) and pushed through the transformer's layers. The final layer projects the resulting hidden state back onto the vocabulary size, producing one raw score - a **logit** - per possible next token.

Logits are **unnormalized**: they can be any real number (positive, negative, huge, tiny) and don't sum to 1. Applying `softmax` to them would turn them into a proper probability distribution, but you don't have to - the token with the *highest logit* is also the token with the highest probability, so comparing raw logits is enough to find the model's top choice.

`Small_LLM_Model.get_logits_from_input_ids(ids)` returns exactly this: one float per vocabulary entry, for the position right after the given sequence.

---

## 3. Autoregressive Generation

LLMs generate text **one token at a time**, feeding each output back in as input for the next step:

```
ids = encode(prompt)
loop:
    logits = model(ids)          # scores for the *next* token
    next_id = pick_from(logits)  # argmax, sampling, or a constrained choice
    ids = ids + [next_id]
until stop condition
```

"Stop condition" is usually an end-of-sequence token or a max length. `pick_from(logits)` is usually `argmax` (greedy - always the single highest-scoring token) or a random sample weighted by probability (more varied, less deterministic).

This project always uses a variant of greedy selection, but restricted to a subset of tokens - which is exactly what constrained decoding is.

---

## 4. Constrained Decoding

The problem with letting an LLM freely generate structured output (like JSON) is that nothing stops it from producing invalid syntax, wrong types, or a nonexistent function name - small models in particular are unreliable at this.

**Constrained decoding** fixes this by intervening *before* the next token is chosen:

1. Get the logits for every possible next token (as above).
2. Decide, using outside knowledge (a grammar, a schema, an allowed-values list), which of those tokens would keep the output valid.
3. Discard every other token - conceptually, set their score to `-infinity` (or in this project's implementation, simply never consider them).
4. Pick the best-scoring token among the survivors.

This guarantees the output can never leave the space of valid answers, no matter how the model would have behaved unconstrained. See [[Projects/Call_Me_Maybe/Constrained Decoder|Constrained Decoder]] for exactly how this project applies that idea to function names, numbers, strings, and booleans.
