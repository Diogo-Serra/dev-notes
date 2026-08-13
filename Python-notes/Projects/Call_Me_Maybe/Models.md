## Overview

`models.py` holds every pydantic model that describes data flowing through the pipeline, plus `Vocabulary` - the class that turns the LLM's raw vocab file into something the decoder can actually query.

| Class               | Role                                                          |
| ------------------- | -------------------------------------------------------------- |
| `FunctionDefinition` | One entry parsed from `functions_definition.json`               |
| `FunctionCallResult` | One resolved answer, written to `function_calling_results.json` |
| `Vocabulary`         | Token-id ↔ string map built from the LLM's vocab file           |

---

## `FunctionDefinition`

```python
class FunctionDefinition(BaseModel):
    name: str
    description: str
    parameters: dict[str, dict]
    returns: dict
```

A direct, minimally-typed mirror of the JSON schema in `functions_definition.json`. `parameters` stays a plain `dict[str, dict]` (e.g. `{"a": {"type": "number"}}`) rather than a nested model - there's no need for a dedicated `FunctionParameter` class since the only field ever read off it is `"type"`.

---

## `FunctionCallResult`

```python
class FunctionCallResult(BaseModel):
    prompt: str
    name: str
    parameters: dict[str, Any]
```

The object that becomes one entry of `function_calling_results.json`. Built entirely in Python (see [[Projects/Call_Me_Maybe/Engine|Engine]]) from the original prompt, the function name chosen by the decoder, and the parameter values it generated - never parsed out of raw model text.

---

## `Vocabulary`

```python
class Vocabulary(BaseModel):
    model_config = ConfigDict(arbitrary_types_allowed=True)

    llm: Small_LLM_Model
    vocab_path: str | None = None
    token_to_id: dict[str, int] = {}
    id_to_token: dict[int, str] = {}
    numeric_token_ids: list[int] = []
```

Wraps the LLM's `get_path_to_vocab_file()` and turns it into two lookup structures every step of [[Projects/Call_Me_Maybe/Constrained Decoder|Constrained Decoder]] depends on.

> `arbitrary_types_allowed=True` is required because `llm: Small_LLM_Model` is not itself a pydantic model - it's a plain class from the provided SDK, and pydantic would otherwise refuse to accept it as a field type.

### `build()`

```python
def build(self) -> None:
```

1. Downloads `vocab.json` via `self.llm.get_path_to_vocab_file()` and loads it into `token_to_id` (raw byte-level-BPE key → id, as stored on disk).
2. Runs every key through `_byte_decoder()` to recover the *real* text each token represents, producing `id_to_token` (id → real string, e.g. `1234 → " hello"`).
3. Precomputes `numeric_token_ids` - every token id whose real text is composed only of digits, `.`, and `-` (matched against `NUMERIC_TOKEN_RE` from `constants.py`). This lets number generation scan a few hundred candidates instead of the full ~150k-entry vocabulary on every step.

See [[Projects/Call_Me_Maybe/LLM Fundamentals|LLM Fundamentals]] for why `vocab.json`'s keys aren't plain text to begin with.

### `_byte_decoder()`

```python
@staticmethod
def _byte_decoder() -> dict[str, int]:
```

The exact reverse of GPT-2's byte-level BPE byte→unicode remapping table: builds the same 256-entry `byte → printable unicode character` table the tokenizer used to write `vocab.json`, then flips it into `character → byte`. `build()` uses this to turn each vocab key back into raw bytes, then UTF-8-decodes those bytes into the real string.

```python
raw_bytes = bytes(byte_decoder[c] for c in raw_token)
id_to_token[token_id] = raw_bytes.decode("utf-8", errors="ignore")
```
