# airalogy-engine

[![PyPI version](https://img.shields.io/pypi/v/airalogy-engine?label=PyPI)](https://pypi.org/project/airalogy-engine/)
[![npm version](https://img.shields.io/npm/v/%40airalogy%2Fairalogy-engine?label=npm)](https://www.npmjs.com/package/@airalogy/airalogy-engine)

Airalogy protocol execution sandbox. Run protocol packages (`parse`, `assign`, `validate`) inside a secure [BoxLite](https://github.com/boxlite-ai/boxlite) sandbox.

## Project Structure

```text
airalogy-engine/
├── python/                      # Python (PyPI) package
├── node/                        # Node.js (npm) package (TypeScript)
├── example_protocol/            # Example protocol package
├── Dockerfile                   # Sandbox Docker image
├── protocol_requirements.txt    # Dependencies for the Docker image
└── README.md
```

## Packages

This repository contains implementations for multiple languages. Please refer to the specific package documentation for installation and usage instructions:

- **[Python Package (PyPI)](./python/README.md)**: The core implementation in Python.
- **[Node.js Package (npm)](./node/README.md)**: The TypeScript/Node.js implementation.

## Sandbox Image

Both Python and Node.js packages use the same underlying sandbox image for protocol execution. The engine supports two modes: **remote Docker image** and **local OCI rootfs**.

### Option 1: Remote Docker Image

Pull a pre-built image from a registry and pass `image=` to the engine functions:

```python
result = await parse_protocol(protocol_path, image="numbcoder/airalogy-engine:0.1")
```

### Option 2: Local OCI Rootfs (Recommended)

Build the image locally and export it as an OCI rootfs directory. This avoids remote pulls and is significantly faster for repeated runs.

```bash
# Build the Docker image
docker build -t airalogy-engine:latest .

# Export to OCI rootfs directory
docker save airalogy-engine:latest -o airalogy-engine-image.tar
mkdir airalogy-engine-image
tar -xf airalogy-engine-image.tar -C airalogy-engine-image
```

Then pass `rootfs_path=` to the engine functions:

```python
result = await parse_protocol(protocol_path, rootfs_path="./airalogy-engine-image")
```

> If neither `image` nor `rootfs_path` is provided, the engine falls back to the default remote image `numbcoder/airalogy-engine:0.1`.

## Running Tests

### Python

Tests use pytest with `pytest-asyncio` and support both sandbox modes via CLI options:

```bash
cd python

# Default: use remote Docker image
uv run pytest tests/ -v

# Explicitly use rootfs mode with a custom path
uv run pytest tests/ -v --sandbox-mode=rootfs --rootfs-path=../airalogy-engine-0.3

# Use remote Docker image mode
uv run pytest tests/ -v --sandbox-mode=image --sandbox-image=numbcoder/airalogy-engine:0.1
```

### Node.js

Tests use vitest and support both sandbox modes via environment variables:

```bash
cd node

# Default: use remote Docker image
pnpm test

# Use local OCI rootfs
SANDBOX_MODE=rootfs ROOTFS_PATH=../airalogy-engine-0.1 pnpm test

# Custom remote image
SANDBOX_IMAGE=numbcoder/airalogy-engine:0.1 pnpm test
```
