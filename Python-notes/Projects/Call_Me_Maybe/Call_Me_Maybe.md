---
tags: [projects, call-me-maybe, index]
---

# Call_Me_Maybe - Index

A Python **function calling** project. Given natural language prompts and a set of available function definitions, it uses a **small local LLM** (Qwen/Qwen3-0.6B) to pick the right function and extract typed arguments - guaranteeing 100% valid JSON through **constrained decoding** instead of hoping the model writes correct JSON on its own.

---

## Execution Flow

```
uv run python -m src
     ↓
Init() → parse_args() → resolve_paths()
     ↓
Small_LLM_Model()          ← loads the local LLM (llm_sdk)
     ↓
FunctionCallEngine
  ┌──────────────────────────────────────────────────────────┐
  │ load_inputs()                                            │
  │   read + validate JSON → list[FunctionDefinition]        │
  │   Vocabulary(llm).build()  → id_to_token map             │
  │   ConstrainedDecoder(vocabulary, definitions)            │
  │                                                          │
  │ run()                                                    │
  │   for each prompt:                                       │
  │     decoder.select_function_name()  → FunctionDefinition │
  │     decoder.generate_parameters()    → dict              │
  │     → FunctionCallResult                                 │
  │                                                          │
  │ write_output() → function_calling_results.json           │
  └──────────────────────────────────────────────────────────┘
```

---

## Notes

| Note                                                                                   | Description                                                              |
| --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| [[Projects/Call_Me_Maybe/LLM Fundamentals\|LLM Fundamentals]]                            | Background: tokenization, logits, the autoregressive loop, byte-level BPE |
| [[Projects/Call_Me_Maybe/Models\|Models]]                                                | `models.py` - `FunctionDefinition`, `FunctionCallResult`, `Vocabulary`    |
| [[Projects/Call_Me_Maybe/Constrained Decoder\|Constrained Decoder]]                      | `decoder.py` - the core constrained-decoding algorithm                   |
| [[Projects/Call_Me_Maybe/Engine\|Engine]]                                                | `engine.py` - orchestrates loading, the generation loop, and output      |
| [[Projects/Call_Me_Maybe/Config\|Config]]                                                | `config.py` + `__main__.py` - env/CLI setup and layered error handling   |
| [[Projects/Call_Me_Maybe/Resources\|Resources]]                                          | Reference links + AI usage disclosure, mirrored from the project README |

---

## Project Structure

```
src/
  __init__.py
  __main__.py            ← CLI entry point (python -m src), one function: main()
  classes/
    config.py            ← Init (env + CLI args), CliArgs
    constants.py          ← shared regex constants
    decoder.py            ← ConstrainedDecoder
    engine.py             ← FunctionCallEngine
    models.py             ← FunctionDefinition, FunctionCallResult, Vocabulary
  data/
    input/
      function_calling_tests.json
      functions_definition.json
    output/               ← generated at runtime, not versioned
  llm_sdk/
    llm_sdk/
      __init__.py         ← Small_LLM_Model (provided SDK)
```

---

## Input / Output File Formats

### `functions_definition.json` - available functions

```json
{
  "name": "fn_add_numbers",
  "description": "Add two numbers together and return their sum.",
  "parameters": { "a": { "type": "number" }, "b": { "type": "number" } },
  "returns": { "type": "number" }
}
```

| Field         | Type                | Description                                  |
| ------------- | ------------------- | --------------------------------------------- |
| `name`        | `str`               | Function identifier, e.g. `fn_add_numbers`    |
| `description` | `str`               | Natural language description shown to the LLM |
| `parameters`  | `dict[str, dict]`   | Parameter name → `{"type": "number/string/boolean"}` |
| `returns`     | `dict`              | Declared return type (not used by the decoder) |

### `function_calling_tests.json` - prompts to answer

```json
{ "prompt": "What is the sum of 2 and 3?" }
```

### `function_calling_results.json` - what the program produces

```json
{
  "prompt": "What is the sum of 2 and 3?",
  "name": "fn_add_numbers",
  "parameters": { "a": 2.0, "b": 3.0 }
}
```

> The output is never parsed out of raw LLM text - it is assembled in Python from the function name and parameter values the decoder generates. See [[Projects/Call_Me_Maybe/Constrained Decoder|Constrained Decoder]] for why that matters.
