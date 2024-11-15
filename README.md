# Repro package A

For https://github.com/astral-sh/uv/issues/8148.

```bash
git clone git@github.com:pawamoy/repro-uv-8148-a
cd repro-uv-8148-a
uv sync  # works fine
```

Observe that the project is correctly installed in editable mode:

```console
% ls -l .venv/lib/python3.10/site-packages
total 28K
drwxr-xr-x 2 pawamoy users 4.0K Nov 15 16:05 repro_uv_8148_a-1.0.1.dist-info
-rw-r--r-- 2 pawamoy users   35 Nov 15 16:05 repro_uv_8148_a.pth
drwxr-xr-x 2 pawamoy users 4.0K Nov 15 16:05 repro-uv-8148-b
drwxr-xr-x 2 pawamoy users 4.0K Nov 15 16:05 repro_uv_8148_b-0.1.1.dist-info
-rw-r--r-- 1 pawamoy users   18 Nov 15 16:05 _virtualenv.pth
-rw-r--r-- 1 pawamoy users 4.3K Nov 15 16:05 _virtualenv.py
```

Now we clone a fork of the same project, meaning we don't have the Git tags.

```bash
cd ..
git clone git@github.com:pawamoy-forks/repro-uv-8148-a repro-uv-8148-a-fork
cd repro-uv-8148-a-fork
uv sync
```

(we could also delete all tags with `git tag -d $(git tag)` in the original clone)

This fails with the following output:

```console
% uv sync
Using CPython 3.12.7 interpreter at: /home/pawamoy/.basher-packages/pyenv/pyenv/versions/3.12.7/bin/python
Creating virtual environment at: .venv
  × No solution found when resolving dependencies:
  ╰─▶ Because only repro-uv-8148-b==0.1.1 is available and repro-uv-8148-b==0.1.1 depends on your project, we can conclude that all versions of repro-uv-8148-b depend on
      your project.
      And because repro-uv-8148-a:dev depends on repro-uv-8148-b and your project depends on repro-uv-8148-a:dev, we can conclude that your project's requirements are
      unsatisfiable.

      hint: The package `repro-uv-8148-b` depends on the package `repro-uv-8148-a` but the name is shadowed by your project. Consider changing the name of the project.
```

## Attempt: override with local self

Adding the following configuration to `[tool.uv]` to force it to install the current project, ignoring constraints applied to itself by dependencies, will also cause uv to install the project in non-editable mode:

```toml
override-dependencies = ["repro-uv-8148-a @ ${PROJECT_ROOT}"]  # fetched from the current directory
```

Sync again, observe that resolution works (`repro-uv-8148-a` was force-installed in v0.1.dev...), but project is not installed in editable mode anymore:

```console
% rm -rf .venv
% rm uv.lock
% uv sync
% ls -l .venv/lib/python3.10/site-packages
total 28K
drwxr-xr-x 2 pawamoy users 4.0K Nov 15 16:10 repro-uv-8148-a
drwxr-xr-x 2 pawamoy users 4.0K Nov 15 16:10 repro_uv_8148_a-0.1.dev2+g69cfc82.d20241115.dist-info
drwxr-xr-x 2 pawamoy users 4.0K Nov 15 16:10 repro-uv-8148-b
drwxr-xr-x 2 pawamoy users 4.0K Nov 15 16:10 repro_uv_8148_b-0.1.1.dist-info
-rw-r--r-- 1 pawamoy users   18 Nov 15 16:09 _virtualenv.pth
-rw-r--r-- 1 pawamoy users 4.3K Nov 15 16:09 _virtualenv.py
```

## Attempt: override with PyPI self

Adding the following configuration to `[tool.uv]` will cause uv to entirely skip installing the dev-dependency `uv-repro-8148-b`, probably because by installing from PyPI instead of current project we lose the dev-dependency information:

```toml
override-dependencies = ["repro-uv-8148-a>=0"]  # fetched from PyPI
```

```console
% rm -rf .venv
% rm uv.lock
% uv sync
Using CPython 3.10.15
Creating virtual environment at: .venv
Resolved 1 package in 410ms
Installed 1 package in 0.47ms
 + repro-uv-8148-a==1.0.1
```

The project will still not be installed in editable mode:

```console
% ls -l .venv/lib/python3.10/site-packages
total 20K
drwxr-xr-x 2 pawamoy users 4.0K Nov 15 16:12 repro-uv-8148-a
drwxr-xr-x 2 pawamoy users 4.0K Nov 15 16:12 repro_uv_8148_a-1.0.1.dist-info
-rw-r--r-- 1 pawamoy users   18 Nov 15 16:12 _virtualenv.pth
-rw-r--r-- 1 pawamoy users 4.3K Nov 15 16:12 _virtualenv.py
```

## Attempt: declaring sources

Declaring sources like following has no effect: the project is still not installed in editable mode, and when fetched from PyPI, the dev-dependency `uv-repro-8148-b` is still not installed:

```toml
[tool.uv.sources]
repro-uv-8148-a = { workspace = true }
```

```toml
[tool.uv.sources]
repro-uv-8148-a = { path = "${PROJECT_ROOT}", editable = true }
```
