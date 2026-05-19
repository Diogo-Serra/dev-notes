## Overview

`parser.py` handles everything that happens **before** the maze is generated - reading `config.txt`, parsing its key-value pairs, and routing execution between startup and in-app reload.

It exposes two functions:

| Function            | Role                                                                          |
| ------------------- | ----------------------------------------------------------------------------- |
| `load_settings()`   | Central dispatcher - validates, builds maze, launches app, handles all errors |
| `settings_parser()` | Low-level file reader - parses `config.txt` into a `Settings` object          |

---

## Execution Flow

```
a-maze-ing.py  →  main()  →  load_settings(argv[0], argv[1], "init")
                                       ↓
                              settings_parser("config.txt")
                                       ↓
                              Settings(**parsed_dict)   ← Pydantic validates
                                       ↓
                              MazeGenerator(settings).generate()
                                       ↓
                              app.run(settings, maze)
```

On in-app reload (after first init):

```
app.run()  →  load_settings("", "config.txt", "run")
                       ↓
              settings_parser("config.txt")
                       ↓
              Settings(**parsed_dict)
                       ↓
              return settings   ← back to app.run()
```

---

## `load_settings(script_name, config_file, flag)`

```python
def load_settings(script_name: str, config_file: str, flag: str) -> Settings:
```

The central entry point. Controls two execution paths via the `flag` parameter.

### Parameters

| Parameter     | Type  | Description                                      |
| ------------- | ----- | ------------------------------------------------ |
| `script_name` | `str` | `argv[0]` - validated only on `"init"`           |
| `config_file` | `str` | path to the config file (usually `"config.txt"`) |
| `flag`        | `str` | `"init"` for startup, `"run"` for in-app reload  |

### `flag = "init"` (startup)

```python
if flag == "init":
    if script_name.strip() != "a-maze-ing.py":
        raise ValueError(...)
    settings = settings_parser(config_file)
    maze = MazeGenerator(settings)
    maze.generate()
    run(settings, maze)
```

1. Validates the script name is exactly `a-maze-ing.py`
2. Parses the config file
3. Builds and generates the maze
4. Launches `app.run()` - the interactive loop

### `flag = "run"` (in-app reload)

```python
elif flag == "run":
    settings = settings_parser(config_file)
return settings
```

Skips script name validation and the app launch. Just re-parses and returns the new `Settings` object to `app.py` for comparison.

---

### Error Handling

All errors are caught here and converted to clean printed messages. The program exits gracefully on any failure.

```python
except ValidationError as error:
    for _error in error.errors():
        if _error['loc']:
            print(f"Error location: {_error['loc'][0]} (config.txt).")
        print(f"Error message: {_error['msg']}.")
    exit(1)
except (FileNotFoundError, FileExistsError, PermissionError) as file_error:
    print(f"File error: {file_error}")
    exit(1)
except ValueError as value_error:
    print(f"Error message: {value_error}")
    exit(1)
except (Exception, BaseException) as exceptional_error:
    print(f"\n\nUnexpected Error: {exceptional_error}")
    exit(1)
```

| Exception           | Cause                                       | Output                          |
| ------------------- | ------------------------------------------- | ------------------------------- |
| `ValidationError`   | Pydantic field/model validator failed       | Prints field location + message |
| `FileNotFoundError` | `config.txt` not found                      | Prints file error               |
| `PermissionError`   | No read access to config                    | Prints file error               |
| `ValueError`        | Wrong script name, bad `=` syntax in config | Prints error message            |
| `Exception`         | Anything unexpected                         | Prints unexpected error         |

> `ValidationError` loops through all errors in the exception, so if multiple fields fail at once, all of them are reported before exiting.

---

## `settings_parser(file_name)`

```python
def settings_parser(file_name: str) -> Settings:
```

Reads and parses `config.txt` into a dict, then passes it to `Settings(...)` for Pydantic validation.

### How it parses the file

```python
with open(file_name, 'r') as config_file:
    _settings: list[str] = config_file.read().split('\n')

for line in _settings:
    setting = line.strip()
    if not setting or setting.startswith("#"):
        continue                         # skip blank lines and comments
    if setting.count('=') > 1:
        raise ValueError(...)            # e.g. "WIDTH=10=extra"
    elif "=" not in setting:
        raise ValueError(...)            # e.g. "WIDTH 10"
    else:
        key, value = setting.split('=')
        parsed_settings[key.upper()] = value.strip(new_punctuation)
```

**Steps:**
1. Read the whole file as a string and split on newlines
2. Skip blank lines and `#` comment lines
3. Validate each line has exactly one `=`
4. Split on `=` - left side becomes the key (upper-cased), right side becomes the value
5. Strip punctuation from the value (except commas, which are needed for tuples like `(0, 0)`)

> `new_punctuation = punctuation.replace(',', '')` - removes all punctuation characters except `,` from values, so `(0, 0)` and `True` parse cleanly.

### The `config.txt` format

```
# This is a comment - ignored by the parser
WIDTH=10
HEIGHT=10
ENTRY=(0, 0)
EXIT=(4, 4)
OUTPUT_FILE=maze.txt
PERFECT=True
SEED=42
```

All keys are case-insensitive (uppercased internally). Values are stripped of surrounding punctuation before being passed to Pydantic.

### Final step

```python
settings: Settings = Settings(**parsed_settings)
return settings
```

The parsed dict is unpacked into `Settings(...)`. Pydantic's validators run at this point - if anything is invalid, a `ValidationError` is raised and caught by `load_settings()`.
