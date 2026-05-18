# Virtual Environments, Packages & Environment Variables

**Module:** The Matrix — Welcome to the Real World of Data Engineering
**Exercise:** Ex0–Ex2 — Entering the Matrix, Loading Programs, Accessing the Mainframe
**Topic:** `venv`, pip, Poetry, `requirements.txt`, `pyproject.toml`, `os.environ`, `python-dotenv`

---

## The Problem: Dependency Isolation

Every Python project needs specific package versions. Without isolation, installing `requests 2.28` for one project silently breaks another that needs `requests 2.25`. The global Python environment becomes a dependency conflict zone.

A **virtual environment** is a self-contained directory with its own Python interpreter and package installation path. Projects never interfere with each other or the system Python.

---

## Virtual Environments with `venv`

### Creating a Virtual Environment

```bash
python3 -m venv matrix_env     # creates the environment in ./matrix_env/
```

The directory contains:
```
matrix_env/
├── bin/           # python3, pip, activate script (Unix)
├── lib/           # installed packages land here
└── pyvenv.cfg     # records the base Python version
```

### Activating and Deactivating

```bash
source matrix_env/bin/activate     # Unix / macOS
matrix_env\Scripts\activate        # Windows

(matrix_env) $>                    # prompt prefix confirms activation

deactivate                         # return to global environment
```

While active, `python` and `pip` point to the venv — not the system installation.

### Detecting a Virtual Environment from Python

```python
import sys
import os

def in_venv() -> bool:
    # sys.prefix is the venv path when active; sys.base_prefix is always the system Python
    return sys.prefix != sys.base_prefix

# Alternative: check the environment variable set by activate
def in_venv_alt() -> bool:
    return os.environ.get("VIRTUAL_ENV") is not None
```

### Key Paths

```python
import sys
import site

print(sys.executable)          # path to the Python interpreter currently in use
print(sys.prefix)              # root of the active environment (venv or system)
print(site.getsitepackages())  # where packages are installed
```

---

## Package Management with `pip`

`pip` is Python's standard package installer.

```bash
pip install requests            # install latest
pip install requests==2.28.0   # install specific version
pip install -r requirements.txt # install from file
pip list                        # show installed packages
pip freeze                      # output installed packages in requirements format
pip show requests               # details about a specific package
pip uninstall requests          # remove a package
```

### `requirements.txt`

A plain text file listing your dependencies, one per line. The standard format for pip-based projects:

```
pandas==2.1.0
numpy==1.25.0
requests==2.31.0
matplotlib==3.7.2
```

Generate it from the current environment:

```bash
pip freeze > requirements.txt
```

Install from it:

```bash
pip install -r requirements.txt
```

---

## Package Management with Poetry

Poetry is a modern dependency manager and build tool. It handles virtual environments automatically and tracks both direct and transitive dependencies with a lockfile.

```bash
poetry new my_project       # scaffold a new project
poetry init                 # add pyproject.toml to an existing directory
poetry add requests         # install and record a dependency
poetry add --dev pytest     # development-only dependency
poetry install              # install everything from pyproject.toml
poetry run python script.py # run a command inside the Poetry environment
poetry shell                # activate the Poetry environment
```

### `pyproject.toml`

The modern standard for Python project metadata (PEP 517/518). Poetry uses it as its main config file:

```toml
[tool.poetry]
name = "matrix"
version = "0.1.0"
description = "Data pipeline"
python = "^3.10"

[tool.poetry.dependencies]
python = "^3.10"
pandas = "^2.1.0"
numpy = "^1.25.0"
matplotlib = "^3.7.2"

[tool.poetry.group.dev.dependencies]
flake8 = "^6.0.0"
mypy = "^1.0.0"
```

`poetry.lock` — generated automatically — pins every transitive dependency to an exact version. Commit this file to guarantee reproducible installs across machines.

### pip vs Poetry

| Feature | pip + requirements.txt | Poetry |
|---|---|---|
| Virtual env management | Manual | Automatic |
| Lockfile | No (unless pip-tools) | Yes (`poetry.lock`) |
| Transitive deps | Not tracked | Fully tracked |
| Build / publish | Separate tools | Built-in |
| Config file | `requirements.txt` | `pyproject.toml` |

---

## Detecting Installed Packages at Runtime

```python
import importlib.util

def is_available(package: str) -> bool:
    return importlib.util.find_spec(package) is not None

if not is_available("pandas"):
    print("pandas not found. Run: pip install pandas")
```

For version information once imported:

```python
import pandas
print(pandas.__version__)
```

---

## Environment Variables

Environment variables are key-value pairs set in the OS shell. They configure applications without hardcoding values — credentials, modes, endpoints, log levels.

```bash
export API_KEY=abc123          # set in the current shell session
echo $API_KEY                  # read in the shell
API_KEY=abc123 python3 app.py  # set only for one command
```

### Reading with `os`

```python
import os

# Returns the value or None if not set
api_key = os.environ.get("API_KEY")

# Returns the value or a default
mode = os.environ.get("MATRIX_MODE", "development")

# Raises KeyError if missing — use when the variable is mandatory
db_url = os.environ["DATABASE_URL"]
```

---

## `.env` Files with `python-dotenv`

Storing environment variables in the shell works for one session. A `.env` file persists them for a project during development.

```
# .env
MATRIX_MODE=development
DATABASE_URL=postgresql://localhost/matrix_dev
API_KEY=dev_key_12345
LOG_LEVEL=DEBUG
ZION_ENDPOINT=http://localhost:8080
```

Load it at the start of your program:

```python
from dotenv import load_dotenv
import os

load_dotenv()   # reads .env from the current directory into os.environ

mode = os.environ.get("MATRIX_MODE", "development")
api_key = os.environ.get("API_KEY")
```

`load_dotenv()` does **not** override variables that are already set in the environment — a shell export always wins over the `.env` file. This makes it safe to override dev settings in production without changing the `.env`.

---

## `.env.example` and `.gitignore`

**Never commit real secrets to version control.** Once pushed, a secret is permanently in the git history — even if deleted later.

### `.env.example`

A template with placeholder values — safe to commit:

```
# .env.example — copy to .env and fill in real values
MATRIX_MODE=development
DATABASE_URL=postgresql://user:password@host/dbname
API_KEY=your_api_key_here
LOG_LEVEL=DEBUG
ZION_ENDPOINT=https://your-endpoint.com
```

### `.gitignore`

Tells git to never track the real `.env`:

```
# .gitignore
.env
matrix_env/
__pycache__/
*.pyc
```

The workflow:
1. Commit `.env.example` → tells collaborators which variables are needed
2. Each person creates their own `.env` locally → never committed
3. Production uses real shell environment variables → no `.env` file on the server

---

## Development vs Production Configuration

A common pattern — branch on a single environment variable:

```python
import os
from dotenv import load_dotenv

load_dotenv()

mode = os.environ.get("MATRIX_MODE", "development")

if mode == "production":
    db_url = os.environ["DATABASE_URL"]     # must be set — raise if missing
    log_level = "WARNING"
else:
    db_url = os.environ.get("DATABASE_URL", "sqlite:///dev.db")
    log_level = os.environ.get("LOG_LEVEL", "DEBUG")
```

---

## Key Takeaways

- Always use a virtual environment per project — `python3 -m venv name` then `source name/bin/activate`
- `pip` + `requirements.txt` is the standard baseline; Poetry adds automatic venv management and a proper lockfile
- `pyproject.toml` is the modern single config file for Python projects
- Read environment variables with `os.environ.get()` — never hardcode secrets
- `python-dotenv` loads a `.env` file into `os.environ` for local development; shell variables always override it
- Commit `.env.example` (with placeholders), never `.env` (with real values); add `.env` to `.gitignore`

---

## Common Mistakes

```bash
# Installing packages globally instead of in a venv
pip install pandas              # pollutes the global environment
# Always activate the venv first

# Committing the venv directory
git add matrix_env/             # hundreds of MB of vendor code — never do this
# Add it to .gitignore

# Committing real credentials
echo "API_KEY=real_secret" > .env
git add .env && git commit      # secret is now in git history forever
```

```python
# Using os.environ[] for optional config — crashes if variable is missing
api_key = os.environ["API_KEY"]   # KeyError if not set

# Use get() with a sensible default for optional variables
api_key = os.environ.get("API_KEY", "")

# Use [] (raise) only for mandatory variables where a missing value is a hard error
db_url = os.environ["DATABASE_URL"]
```

---

*Tags: #resource/python #python/environments #python/packages #python/configuration*

---

**Index:** [[Python Modules Index]]
**Previous:** [[Ex7 - Design Patterns]] | **Next:** [[Ex9 - Pydantic Models & Validation]]
**See also:** [[Ex6 - Modules, Packages & Imports]]
