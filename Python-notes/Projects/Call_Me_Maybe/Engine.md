## Overview

`engine.py` owns everything that isn't the constrained-decoding algorithm itself: reading and validating both input files, running the per-prompt generation loop, and writing the final output file.

| Method          | Role                                                              |
| --------------- | ------------------------------------------------------------------ |
| `load_inputs()` | Parse + validate both JSON files, build `Vocabulary` and `ConstrainedDecoder` |
| `run()`         | Process every prompt, skipping failures without crashing            |
| `write_output()`| Serialize `results` to `function_calling_results.json`              |

---

## `FunctionCallEngine`

```python
class FunctionCallEngine(BaseModel):
    model_config = ConfigDict(arbitrary_types_allowed=True)

    llm: Small_LLM_Model
    definitions_path: Path
    prompts_path: Path
    output_path: Path

    function_definitions: list[FunctionDefinition] = []
    prompts: list[str] = []
    vocabulary: Vocabulary | None = None
    decoder: ConstrainedDecoder | None = None
    results: list[FunctionCallResult] = []
```

A pydantic `BaseModel` like every other class in this project. `vocabulary` and `decoder` start as `None` and are only populated once `load_inputs()` runs - `run()` guards against being called before that with a `RuntimeError`.

---

## `load_inputs()`

```python
def load_inputs(self) -> None:
```

1. Parses every entry of `definitions_path` into a `FunctionDefinition` - this is where a malformed entry (missing `name`/`description`/`parameters`/`returns`) raises a `pydantic.ValidationError`, caught and reported field-by-field in [[Projects/Call_Me_Maybe/Config|Config]]'s `main()`.
2. Raises `ValueError` if the definitions file parsed to an empty list - there is nothing to call without at least one function.
3. Parses `prompts_path` into a plain `list[str]`, skipping any entry that isn't a `{"prompt": ...}` object instead of raising.
4. Builds the `Vocabulary` (downloads and decodes the LLM's vocab file - see [[Projects/Call_Me_Maybe/Models|Models]]) and constructs the `ConstrainedDecoder` from it.

---

## `run()`

```python
def run(self) -> list[FunctionCallResult]:
```

Loops over `self.prompts`, calling `_process_prompt()` for each one inside a `try/except Exception`. A single bad prompt is logged (`Skipping prompt ...`) and skipped rather than aborting the whole batch - the subject requires the program to never crash on bad input, and one prompt failing shouldn't cost every other result.

### `_process_prompt(prompt)`

```python
def _process_prompt(self, prompt: str) -> FunctionCallResult:
```

1. Builds a header prompt listing the request and every available function + description.
2. Encodes it (`llm.encode(header)`) to get the starting token-id sequence.
3. `decoder.select_function_name()` → chosen `FunctionDefinition` + updated ids.
4. `decoder.generate_parameters()` → `dict` of typed argument values.
5. Wraps `prompt`, the chosen function's `name`, and the generated `parameters` into a `FunctionCallResult`.

See [[Projects/Call_Me_Maybe/Constrained Decoder|Constrained Decoder]] for what happens inside steps 3-4.

---

## `write_output()`

```python
def write_output(self) -> None:
```

Creates `output_path`'s parent directory if needed, then writes `[r.model_dump() for r in self.results]` as indented JSON. Using pydantic's `model_dump()` (rather than hand-building dicts) guarantees the output always has exactly the three required keys (`prompt`, `name`, `parameters`) in the shape declared by `FunctionCallResult` - no extra keys, no missing ones.

---

## `_read_json_array(path)`

```python
@staticmethod
def _read_json_array(path: Path) -> list[Any]:
```

Shared file-loading helper for both input files. Translates the two ways file-reading can fail into a single `ValueError` with a clear message:

| Failure                    | Raised as                                  |
| -------------------------- | -------------------------------------------- |
| File does not exist        | `ValueError("Input file not found: ...")`   |
| File is not valid JSON     | `ValueError("Invalid JSON in ...: ...")`    |
| Valid JSON, but not a list | `ValueError("Expected a JSON array in ...")`|

All three end up as plain `ValueError`s, which is exactly the exception type [[Projects/Call_Me_Maybe/Config|Config]]'s `main()` catches and reports without a traceback.
