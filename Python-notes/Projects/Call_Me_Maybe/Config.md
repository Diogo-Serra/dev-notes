## Overview

`config.py` and `__main__.py` together handle everything that is not the algorithm - environment setup, CLI argument parsing, default path resolution, and top-level error reporting.

```
uv run python -m src [--flags]
     ↓
Init()                  ← loads .env, computes default paths
     ↓
config.parse_args()     ← reads --functions_definition/--input/--output
     ↓
config.resolve_paths()  ← CLI value, or default, for each of the 3 paths
     ↓
main()                  ← wires Small_LLM_Model + FunctionCallEngine together
```

`src/__main__.py` intentionally contains exactly one function, `main()` - every other piece of logic lives on a class.

---

## `CliArgs`

```python
class CliArgs(BaseModel):
    functions_definition: Path | None
    input: Path | None
    output: Path | None
```

A small pydantic model representing the three optional CLI flags, once parsed. Exists so the rest of the code deals with a typed, named object instead of `argparse`'s generic `Namespace` (which is just a plain attribute bag with no fixed shape).

---

## `Init`

```python
class Init(BaseModel):
    _SRC_DIR = Path(__file__).parent.parent

    hf_token: str | None = None
    data_dir: Path = _SRC_DIR / "data" / "input"
    output_file: Path = (
        _SRC_DIR / "data" / "output" / "function_calling_results.json"
    )
```

> `_SRC_DIR` has **no type annotation**, so pydantic doesn't collect it as a model field - it stays a normal Python class attribute, computed once from `config.py`'s own location (`classes/config.py` → two `.parent`s up → `src/`).

### `model_post_init(self, __context)`

```python
def model_post_init(self, __context: Any) -> None:
    load_dotenv()
    if self.hf_token is None:
        self.hf_token = environ.get("HF_TOKEN")
```

Pydantic's post-construction hook - runs automatically right after all fields are set. Loads `.env` and fills `hf_token` from the environment, since that value can't be computed as a plain field default (it depends on `load_dotenv()` having already run).

### `parse_args()`

```python
def parse_args(self) -> CliArgs:
```

Builds an `argparse.ArgumentParser` with the three required flags (`--functions_definition`, `--input`, `--output`, all optional, all `type=Path`), calls `parser.parse_args()`, and immediately converts the resulting `Namespace` into a `CliArgs` via `CliArgs(**vars(parser.parse_args()))`-style unpacking - `Namespace` is never touched again after this method returns.

### `resolve_paths(args)`

```python
def resolve_paths(self, args: CliArgs) -> tuple[Path, Path, Path]:
```

For each of the three paths, uses the CLI value if one was given, otherwise falls back to the corresponding default under `data_dir` / `output_file`.

---

## `main()` (`__main__.py`)

```python
def main() -> int:
    config = Init()
    args = config.parse_args()
    definitions_path, prompts_path, output_path = config.resolve_paths(args)
    ...
```

Wires `Init`, `Small_LLM_Model`, and `FunctionCallEngine` together, then reports errors in three tiers before returning an exit code (`0` success, `1` failure) for `sys.exit()`:

| Exception            | Handling                                                          |
| --------------------- | -------------------------------------------------------------------- |
| `pydantic.ValidationError` | Prints `Error: invalid data in <file>:` then every failing field's `loc` + `msg`, one per line |
| `(OSError, ValueError)`     | Prints the message directly - covers missing files, bad JSON, empty definitions (all raised as `ValueError` by [[Projects/Call_Me_Maybe/Engine|Engine]]'s `_read_json_array`) |
| `Exception` (catch-all)     | Prints `Unexpected error (<type>): <message>` - guarantees the program never crashes silently |

```python
if __name__ == "__main__":
    exit_code = main()
    sys.exit(exit_code)
```

Kept as two explicit lines rather than `sys.exit(main())` in one line, so the control flow (call, then exit) reads clearly at a glance.
