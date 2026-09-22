# scitex-sh

<p align="center">
  <a href="https://scitex.ai">
    <img src="docs/scitex-logo-blue-cropped.png" alt="SciTeX" width="400">
  </a>
</p>

<p align="center"><b>Safe subprocess wrapper — list-only (no shell-string parsing), structured result, timeouts, streaming.</b></p>

<p align="center">
  <a href="https://scitex-sh.readthedocs.io/">Full Documentation</a> · <code>uv pip install scitex-sh[all]</code>
</p>

<!-- scitex-badges:start -->
<p align="center">
  <a href="https://pypi.org/project/scitex-sh/"><img src="https://img.shields.io/pypi/v/scitex-sh?label=pypi" alt="pypi"></a>
  <a href="https://pypi.org/project/scitex-sh/"><img src="https://img.shields.io/pypi/pyversions/scitex-sh?label=python" alt="python"></a>
  <a href="https://github.com/scitex-ai/scitex-sh/actions/workflows/rtd-sphinx-build-on-ubuntu-latest.yml"><img src="https://img.shields.io/github/actions/workflow/status/scitex-ai/scitex-sh/rtd-sphinx-build-on-ubuntu-latest.yml?branch=develop&label=docs" alt="docs"></a>
  <a href="https://scitex-sh.readthedocs.io/en/latest/"><img src="https://img.shields.io/readthedocs/scitex-sh?label=docs" alt="docs-rtd"></a>
</p>
<p align="center">
  <a href="https://github.com/scitex-ai/scitex-sh/actions/workflows/ci.yml"><img src="https://img.shields.io/github/actions/workflow/status/scitex-ai/scitex-sh/ci.yml?branch=develop&label=tests" alt="tests"></a>
  <a href="https://github.com/scitex-ai/scitex-sh/actions/workflows/ci.yml"><img src="https://img.shields.io/github/actions/workflow/status/scitex-ai/scitex-sh/ci.yml?branch=develop&label=install-check" alt="install-check"></a>
  <a href="https://codecov.io/gh/scitex-ai/scitex-sh"><img src="https://img.shields.io/codecov/c/github/scitex-ai/scitex-sh/develop?label=cov" alt="cov"></a>
</p>
<!-- scitex-badges:end -->

---

## Quick Start

```python
import scitex_sh as sh

res = sh.sh(["git", "status"])
print(res["stdout"])
```

## Demo

```mermaid
flowchart LR
    A["sh.sh(['git', 'status'])"] --> R1[("{stdout, stderr,\nreturncode, success}")]
    B["sh.sh(['ls', '-la'], return_as='str')"] --> R2[("stdout str")]
    C["sh.sh(['./long.sh'], stream_output=True)"] --> R3[("live tty stream")]
    D["sh.sh(['sleep', '10'], timeout=2)"] --> R4[("TimeoutExpired")]
    E["sh.sh('rm -rf /')\n(string input)"] --> R5[("ValueError: list-only")]
```

<p align="center"><sub><b>Figure 2.</b> Interface matrix. One call shape per use case; string commands are rejected before anything runs.</sub></p>

## Installation

```bash
uv pip install "scitex-sh[all]"
```

<details>
<summary><b>Per-module extras</b></summary>

<br>

| Extra | Pulls in |
|---|---|
| `all` | `dev` + `docs` (recommended) |
| `dev` | pytest, pytest-cov, ruff |
| `docs` | Sphinx + RTD theme + myst-parser (docs build only) |

```bash
uv pip install -e ".[dev]"               # editable install for contributors
```

</details>

## Architecture

```
src/scitex_sh/
├── __init__.py         # public re-exports: sh, sh_run, quote
├── _execute.py         # core sh() — list-only argv, dict result, streaming, timeouts
├── _security.py        # reject shell=True / string-cmd inputs (refuse silent injection)
├── _shell_legacy.py    # opt-in legacy shell-string path (kept off by default)
└── _types.py           # ShResult TypedDict
```

```mermaid
flowchart LR
    user["sh.sh([cmd, arg, arg])"] --> guard["_security: reject str / shell=True"]
    guard --> sub["subprocess.run / Popen"]
    sub -->|"capture"| dict[("{stdout, stderr, returncode, success}")]
    sub -.->|"stream_output=True"| tty[("live stdout/stderr")]
    sub -.->|"timeout=N"| killed[("TimeoutExpired → returncode != 0")]
```

<p align="center"><sub><b>Figure 1.</b> Execution flow. List-only argv passes validation, then runs buffered or streamed; every path returns the same result dict.</sub></p>

## 1 Interfaces

<details open>
<summary><strong>Python API</strong></summary>

<br>

```python
import scitex_sh as sh

# Dict result (stdout / stderr / returncode / success)
res = sh.sh(["git", "status"])

# String result
out = sh.sh(["ls", "-la"], return_as="str")

# Streamed output
sh.sh(["./long-running.sh"], stream_output=True)

# Timeout
sh.sh(["sleep", "10"], timeout=2)

# Lower-level
res = sh.sh_run(["echo", "hi"])
sh.quote("hello world")            # 'hello world' (POSIX-quoted)
```

</details>

## Status

Standalone fork of `scitex.sh`. Minimal deps (scitex-logging for output,
scitex-dev for test tooling). The umbrella package's
`scitex.sh` import path is preserved via a `sys.modules`-alias bridge. The
`scitex.str.color_text` dep used for terminal output is replaced with a tiny
inline ANSI helper that respects `NO_COLOR` and TTY detection.

## Part of SciTeX

`scitex-sh` is part of [**SciTeX**](https://scitex.ai). Install via
the umbrella with `pip install scitex[sh]` to use as
`scitex.sh` (Python) or `scitex sh ...` (CLI).

>Four Freedoms for Research
>
>0. The freedom to **run** your research anywhere — your machine, your terms.
>1. The freedom to **study** how every step works — from raw data to final manuscript.
>2. The freedom to **redistribute** your workflows, not just your papers.
>3. The freedom to **modify** any module and share improvements with the community.
>
>AGPL-3.0 — because we believe research infrastructure deserves the same freedoms as the software it runs on.

## License

AGPL-3.0-only (see [LICENSE](./LICENSE)).

---

<p align="center">
  <a href="https://scitex.ai" target="_blank"><img src="docs/scitex-icon-navy-inverted.png" alt="SciTeX" width="40"/></a>
</p>
