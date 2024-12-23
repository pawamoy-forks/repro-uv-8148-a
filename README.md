# Repro package A

For https://github.com/astral-sh/uv/issues/8148.

```bash
git clone git@github.com:pawamoy/repro-uv-8148-a
cd repro-uv-8148-a
uv sync
```

Observe that the development dependency `repro-uv-8148-b` was not installed in `.venv`.

