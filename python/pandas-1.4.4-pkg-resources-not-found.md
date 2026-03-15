[← Back to Known Fixes](../README.md)
# ModuleNotFoundError: No module named 'pkg_resources' — pandas 1.4.4 pip install (February 2026)

## Error
```
ModuleNotFoundError: No module named 'pkg_resources'
ERROR: Could not find a version that satisfies the requirement pandas==1.4.4
ERROR: No matching distribution found for pandas==1.4.4
```

## Environment
- pandas `1.4.4`
- pip `23+`
- setuptools `82.0.0+`
- Python `3.11`
- OS: macOS / Linux / Windows

> ⚠️ Tested on Python 3.11. May not work on newer versions.

## Root cause

When pip compiles a package from source, it creates a temporary isolated environment in `/tmp` and installs the build dependencies defined in the package's `pyproject.toml` ([PEP 517](https://peps.python.org/pep-0517/#get-requires-for-build-wheel)).

The [`pyproject.toml` of pandas 1.4.4](https://github.com/pandas-dev/pandas/blob/v1.4.4/pyproject.toml) requires `setuptools>=51.0.0` — pip always installs the **latest version** of setuptools in that temporary environment, ignoring your venv entirely. Since February 2026, [setuptools `v82.0.0`](https://setuptools.pypa.io/en/latest/history.html) removed `pkg_resources`, breaking the build.

Downgrading setuptools in your venv does **not** fix the issue — pip ignores it for compilation.

## Fix

Install dependencies in the correct order with `--no-build-isolation` to force pip to use your venv directly, and `--no-cache-dir` to avoid stale cache:
```bash
pip install wheel==52.0.0
pip install cython==0.29.32
pip install oldest-supported-numpy
pip install numpy==1.26.4
pip install pandas==1.4.4 --no-build-isolation --no-cache-dir
pip install -r requirements.txt
```

## References
- [PEP 517](https://peps.python.org/pep-0517/#get-requires-for-build-wheel)
- [pandas 1.4.4 pyproject.toml](https://github.com/pandas-dev/pandas/blob/v1.4.4/pyproject.toml)
- [setuptools v82.0.0 changelog](https://setuptools.pypa.io/en/latest/history.html)
- [pip --no-build-isolation](https://pip.pypa.io/en/stable/cli/pip_install/#cmdoption-no-build-isolation)
- [Related issue on CLIP](https://github.com/openai/CLIP/issues/528#issuecomment-3884279862)
