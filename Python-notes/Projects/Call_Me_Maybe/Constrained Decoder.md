## Overview

`decoder.py` is the algorithmic core of Call_Me_Maybe. Given a prompt already turned into token ids, it figures out which function to call and what arguments to pass - one constrained field at a time, never trusting the model to write JSON syntax on its own.

```
ConstrainedDecoder
   ↓
1. select_function_name()
   → constrained-decode the "name" field against the real function names
   ↓
2. generate_parameters()
   → for each declared parameter, constrained-decode a typed value
   ↓
FunctionCallEngine assembles the result (see Engine note)
```

Read [[Projects/Call_Me_Maybe/LLM Fundamentals|LLM Fundamentals]] first if "logits" or "constrained decoding" aren't already familiar - this note assumes that background.

---

## `ConstrainedDecoder`

```python
class ConstrainedDecoder(BaseModel):
    vocabulary: Vocabulary
    function_definitions: list[FunctionDefinition]
    model_config = ConfigDict(arbitrary_types_allowed=True)
```

Holds the `Vocabulary` (id ↔ string map) and the list of callable functions. Every generation method takes the `Small_LLM_Model` instance and the current token-id sequence as parameters, and returns `(value, updated_ids)` - the growing id list is threaded through the whole pipeline so later fields are generated with full knowledge of everything asked and answered before them.

---

## Stage 1 - `select_function_name(llm, input_ids)`

```python
def select_function_name(
    self, llm: Small_LLM_Model, input_ids: list[int]
) -> tuple[FunctionDefinition, list[int]]:
```

Calls `_generate_enum()` with the list of every real function name as candidates, then matches the resulting string back to a `FunctionDefinition` (exact match first, prefix match as a fallback, first definition as a last resort if generation stalls).

> [!info] Why an enum, not free text?
> The function name is not a name the model can just make up - it must be exactly one of the strings already known from `functions_definition.json`. Restricting generation to that fixed candidate set is what guarantees the LLM's choice is always a real, callable function.

---

## Stage 2 - `generate_parameters(llm, input_ids, function_def)`

```python
def generate_parameters(
    self, llm, input_ids, function_def
) -> dict:
```

Walks `function_def.parameters` **in order**. For each parameter:

1. Appends a short sub-prompt to the running id sequence: `Value for parameter "a" (number): `.
2. Dispatches to the generator matching the parameter's declared `"type"`:

| Declared type | Generator          | Constraint                                             |
| ------------- | ------------------- | ------------------------------------------------------- |
| `number`      | `_generate_number`   | only digits / `.` / leading `-` allowed                  |
| `boolean`     | `_generate_enum`     | restricted to the two-item candidate list `["true", "false"]` |
| anything else | `_generate_string`   | any token allowed except ones containing `"` or a newline |

> [!info] Why keep extending the same `ids` list across parameters?
> For a prompt like *"sum of 265 and 345"*, generating `a` and `b` from two independent, context-free prompts made the model repeat `265` for both. Appending each new parameter's sub-prompt after the *previous* parameter's generated answer gives the model the context it needs to move on to the next value instead of repeating itself.

---

## `_generate_enum(llm, input_ids, candidates)`

```python
def _generate_enum(self, llm, input_ids, candidates) -> tuple[str, list[int]]:
```

Used by both function-name selection and boolean parameters. Walks a **token trie** over the fixed candidate strings:

```
loop (up to longest candidate length + 2 steps):
    if partial already equals a full candidate: stop
    remaining = candidates that still start with partial
    if none remain: stop (nothing valid left to generate)
    logits = llm.get_logits_from_input_ids(ids)
    for every (token_id, token_text) in vocabulary:
        candidate_text = partial + token_text
        if candidate_text is a prefix of any string in `remaining`:
            keep it as a contender, ranked by its logit
    append the highest-logit contender; partial += its text
```

This is the literal masking step from constrained decoding: every token that would step outside all remaining candidates is discarded before comparing scores at all.

---

## `_generate_number(llm, input_ids)`

```python
def _generate_number(self, llm, input_ids) -> tuple[float, list[int]]:
```

Numbers don't have a fixed candidate list, so instead of an enum trie, each step is validated against `NUMBER_PREFIX_RE` (`^-?\d*\.?\d*$` - a valid, possibly-incomplete number so far):

```
loop (up to 12 steps):
    logits = llm.get_logits_from_input_ids(ids)
    peek at the model's own unconstrained top choice (highest logit, no filter)
    if partial is non-empty AND that top choice would break the number format:
        stop - the model itself is signalling "the number is done"
    otherwise, among only `vocabulary.numeric_token_ids` (precomputed subset),
    keep the highest-logit token whose text still matches NUMBER_PREFIX_RE
    append it; partial += its text
```

> [!info] Why peek at the unconstrained top choice to decide when to stop?
> Enum fields (name, booleans) have an obvious end: the text exactly matches a candidate. Numbers don't - `"26"` and `"265"` are both valid partial numbers. Letting the model's own preferred next token (ignoring constraints for a moment) signal completion avoids a hardcoded max length while still respecting the number-format constraint for every token actually appended.

Falls back to `0.0` if the accumulated text never parses as a float (defensive; should not happen given the character-level constraint above).

---

## `_generate_string(llm, input_ids)`

```python
def _generate_string(self, llm, input_ids) -> tuple[str, list[int]]:
```

Same "peek the model's unconstrained choice to detect the stopping point" pattern as `_generate_number`, but the constraint itself is much looser: **any** token is allowed except ones containing a literal `"` or a newline, so the generated value can never break out of its JSON string boundary. Generation stops as soon as the model's own top pick would introduce one of those characters.

The final string is `.strip(" '\"")`-ed, trimming stray leading/trailing quote characters the model sometimes copies from the natural-language prompt (e.g. `'hello` → `hello`).

---

## Why this guarantees valid JSON

None of these four generation methods ever produce a `{`, `}`, `:`, or `,` - the model is only ever asked for the *value* of one field at a time. [[Projects/Call_Me_Maybe/Engine|Engine]] assembles the actual `FunctionCallResult` object in Python from those values, so invalid JSON structure is structurally impossible, not just unlikely.
