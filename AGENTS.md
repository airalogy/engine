# AGENTS.md

## Build & Run
- **Python** (`python/`): Uses `uv` (>=0.11). Install: `cd python && uv sync`. Run tests: `uv run pytest tests/ -v`.
- **Node** (`node/`): Uses `pnpm`. Install: `cd node && pnpm install`. Build: `pnpm run build` (copies executor + runs `tsc`). Run tests: `pnpm test`.
- **Docker sandbox**: `docker build -t airalogy-engine:latest .` (shared by both packages).

## Sandbox Modes
The engine supports two sandbox backends:
- **Remote image** (`image=`): Pulls a Docker image from a registry (e.g., `numbcoder/airalogy-engine:0.1`). Requires network access.
- **Local OCI rootfs** (`rootfs_path=`): Uses a pre-exported rootfs directory. Faster, no network required. Build with:
  ```bash
  docker build -t airalogy-engine:latest .
  docker save airalogy-engine:latest -o airalogy-engine-image.tar
  mkdir airalogy-engine-image && tar -xf airalogy-engine-image.tar -C airalogy-engine-image
  ```
- If neither is provided, falls back to `DEFAULT_IMAGE` (`numbcoder/airalogy-engine:0.1`).

## Testing

### Python
Tests are in `python/tests/` and use pytest with `pytest-asyncio`. A `conftest.py` provides CLI options to switch sandbox mode:
- `--sandbox-mode=rootfs` (default): Uses local OCI rootfs. Override path with `--rootfs-path=<path>`.
- `--sandbox-mode=image`: Uses a remote Docker image. Override with `--sandbox-image=<image>`.

Examples:
```bash
cd python
uv run pytest tests/ -v                                                  # default rootfs mode
uv run pytest tests/ -v --sandbox-mode=rootfs --rootfs-path=../airalogy-engine-0.3
uv run pytest tests/ -v --sandbox-mode=image --sandbox-image=numbcoder/airalogy-engine:0.1
```

### Node
Tests are in `node/tests/` and use vitest. Sandbox mode is configured via environment variables:
- `SANDBOX_MODE=rootfs`: Uses local OCI rootfs. Override path with `ROOTFS_PATH=<path>`.
- `SANDBOX_MODE=image` (default): Uses a remote Docker image. Override with `SANDBOX_IMAGE=<image>`.

Examples:
```bash
cd node
pnpm test                                                                # default image mode
SANDBOX_MODE=rootfs ROOTFS_PATH=../airalogy-engine-0.1 pnpm test        # rootfs mode
SANDBOX_IMAGE=numbcoder/airalogy-engine:0.1 pnpm test                   # custom image
```

## Architecture
Monorepo with two SDK packages (`python/`, `node/`) that wrap the same `protocol_executor.py` to run protocol packages (parse, assign, validate) inside a BoxLite sandbox. Both packages are fully implemented. `example_protocol/` contains a sample protocol directory.

## Key APIs
- Python: async functions `parse_protocol`, `assign_variable`, `validate_variables` in `python/src/airalogy_engine/engine.py`. All accept `image` or `rootfs_path` kwargs and return `dict` with `success`, `message`, `data` keys.
- Node: async functions `parseProtocol`, `assignVariable`, `validateVariables` in `node/src/index.ts`. All accept a `SandboxOptions` object with `image` or `rootfsPath` and return `Promise<ProtocolResult>` with `success`, `message`, `data` keys.

## Code Style
- **Python**: Python 3.13+, async/await, type hints with `dict | None` union syntax, snake_case, double-quote strings, 4-space indent. Uses `boxlite` SDK. Build system: `uv_build`.
- **TypeScript**: ESNext target, nodenext module resolution, strict mode, ESM (`"type": "module"`), camelCase functions, explicit return types. Uses `pnpm` for package management.
- Errors: return error dicts (`success: false`) rather than raising; only raise `ValueError` for invalid inputs at the boundary.
