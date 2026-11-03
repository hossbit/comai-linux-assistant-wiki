# Local vLLM advanced reference

This page contains advanced options and the complete reference. Start with the
[Local vLLM guide](Local-vLLM.md) for the simplest installation and repair steps.
[Project repository](https://github.com/hossbit/local-ai-server-vllm).

## Find what you need

- [Backends](#two-isolated-backends) and [requirements](#requirements)
- [Installation](#install), [AMD and CPU setup](#amd-and-cpu-installation), and [model setup](#add-models)
- [Commands](#commands) and [configuration](#configuration)
- [API usage](#use-the-server), [API keys](#api-keys), and [security](#security)
- [Updates](#update), [uninstall](#uninstall), and [troubleshooting](#troubleshooting)

## Two isolated backends

| | `vllm` backend | `omni` backend |
| --- | --- | --- |
| Engine package | `vllm` | `vllm-omni` |
| Server flag | plain `vllm serve` | `vllm serve ... --omni` |
| Model root | `~/ai-vllm/models/vllm` | `~/ai-vllm/models/omni` |
| State/config | `conf/backends/vllm` | `conf/backends/omni` |
| Virtualenv | `venv/vllm` | `venv/omni` |
| Default port | `8000` | `8091` |
| systemd service | `localvllm.service` | `localvllm-omni.service` |
| API keys | independent | independent |
| Typical models | text, embeddings, reranking | text, image, audio, video, action |

Both can run at the same time if the GPU has enough VRAM for both. Every
command can target one backend:

```bash
localvllm --backend vllm models
localvllm --backend omni models
localvllm start all
localvllm status all
```

`localvllm` without `--backend` uses the configured default backend (`vllm`
unless changed with `localvllm backend default omni`).

## What it provides

- Two isolated vLLM environments with no shared Python packages
- OpenAI-compatible chat, completion, embedding, and (where supported) reranking APIs
- Optional vLLM-Omni serving through the same `vllm serve` entrypoint
- Per-backend model registries and per-model override files
- Automatic model discovery from each backend's model directory
- A systemd user service per backend
- A single `localvllm` CLI for start/stop, models, keys, doctor, update, uninstall
- Hardware-aware setting suggestions and a small live benchmark
- Optional vLLM sleep mode for releasing VRAM without stopping the process
- Atomic launcher generation and transactional `use`; guarded, locked key changes
- A global operation lock that prevents concurrent install/update/start/stop races
- Staged virtualenv upgrades with verification, `pip check`, and rollback
- Rotated per-backend logs instead of truncating the previous failure
- JSON output for `status`, `models`, and `doctor`; Prometheus metrics via `localvllm metrics`
- Explicit security refusal for insecure public listeners

## Requirements

- Linux x86-64 or ARM64 (Ubuntu, Debian, Fedora, RHEL, or compatible; hardware support depends on the vLLM wheel)
- Python 3.11–3.13 for new environments; Python 3.12 is preferred and installed privately if unavailable
- An NVIDIA GPU with a working driver for normal GPU serving
- ROCm is possible with a matching vLLM wheel; CPU is experimental
- ROCm/CPU may require hardware-specific packages and pip options; see
  [AMD and CPU installation](#amd-and-cpu-installation)
- `sudo` access during installation for system packages
- Enough disk for two virtualenvs and the model weights
- Enough VRAM for the model(s) you select

The installer creates a virtualenv per backend and installs vLLM with `pip`.
For a new environment it prefers Python 3.12 and uses an isolated uv-managed
Python if a compatible interpreter is missing. It does not replace your system
Python or install a GPU driver or CUDA Toolkit.

## Install

From this checkout:

```bash
./install-localvllm.sh --all
```

Install only the plain vLLM backend:

```bash
./install-localvllm.sh --backend vllm
```

Install only vLLM-Omni:

```bash
./install-localvllm.sh --backend omni
```

Custom directory:

```bash
./install-localvllm.sh --dir ~/ai-vllm --all
```

One-line install from GitHub:

```bash
curl -fsSL https://hossbit.github.io/localvllm/install.sh | bash -s -- --all
```

The default install directory is `~/ai-vllm`.

After installation:

```bash
localvllm backend list
localvllm doctor all
```

If the command is not found in the current terminal, run
`source ~/.bashrc` (or open a new terminal); the installer adds
`~/.local/bin` to PATH when needed.

### AMD and CPU installation

These paths are advanced and depend on upstream vLLM wheels, hardware, Python,
and operating-system compatibility. The installer creates a virtual environment
and runs pip; it does **not** install GPU drivers, choose a compatible ROCm
wheel, or perform the upstream CPU source-build procedure. Check the relevant
upstream instructions before installing. Use the plain `vllm` backend first.

#### AMD GPU (ROCm)

1. Check that your GPU and ROCm stack are supported by the [vLLM GPU installation guide](https://docs.vllm.ai/en/stable/getting_started/installation/gpu/)
   (select AMD ROCm). Install a compatible driver/runtime as required by that
   guide. `rocminfo` or `rocm-smi` can help verify your setup.
2. Check the wheel's Python and system requirements. Choose an appropriate
   Python interpreter and wheel index from the upstream guide. For example,
   current upstream ROCm wheels require Python 3.12; availability and the
   appropriate index depend on the vLLM and ROCm release.
3. Pin a matching wheel version and its index. This example uses upstream's
   vLLM 0.18.0 / ROCm 7.0 wheel; use it only if it matches your system:

   ```bash
   curl -fsSL https://hossbit.github.io/localvllm/install.sh | \
     LOCALVLLM_PYTHON=python3.12 VLLM_TARGET_DEVICE=rocm \
     VLLM_VERSION="0.18.0+rocm700" \
     VLLM_PIP_ARGS="--extra-index-url https://wheels.vllm.ai/rocm/0.18.0/rocm700" \
     bash -s -- --backend vllm
   ```

   Choose a different `VLLM_VERSION` and index together if needed. This is an
   example, not a compatibility guarantee: confirm the wheel supports your GPU
   and installed stack. An extra index without an exact version does not
   guarantee pip will select a ROCm build. If Python or a compatible wheel is unavailable,
   pip may install a CUDA build or fail; do not assume GPU support succeeded.
   Inspect `~/ai-vllm/logs/pip-install-vllm.log` and run `localvllm doctor vllm`.

To try Omni, check the [vLLM-Omni installation guide](https://docs.vllm.ai/projects/vllm-omni/en/latest/getting_started/installation/)
for the matching vLLM, Omni, ROCm, and Python versions. For a compatible stack,
set the appropriate `VLLM_PIP_ARGS` and, if needed, `VLLM_OMNI_PIP_ARGS` before
running the installer with `--backend omni`. This project installs both
packages in one pip transaction; the indices and versions must work together.

#### CPU only

CPU inference is experimental in this manager and can be slow. `VLLM_ALLOW_CPU=1`
only bypasses the manager's no-GPU check; `VLLM_TARGET_DEVICE=cpu` selects the
CPU target. Neither setting guarantees that pip installs a CPU-compatible
vLLM build. Follow the [vLLM CPU installation guide](https://docs.vllm.ai/en/latest/getting_started/installation/cpu/)
for your architecture to determine whether compatible packages are available
or a source build is required.

If you have identified suitable pip options for your system, set them before
installing (replace the placeholder with those actual options):

```bash
export VLLM_ALLOW_CPU=1 VLLM_TARGET_DEVICE=cpu
export VLLM_PIP_ARGS="<CPU-compatible pip options from the vLLM guide>"
curl -fsSL https://hossbit.github.io/localvllm/install.sh | bash -s -- --backend vllm
```

The placeholder is not a usable index; do not run it unchanged. If upstream
requires a source build or a direct wheel that this installer's `pip install
vllm` flow cannot select, use the upstream installation workflow instead; this
manager does not automate that build. vLLM-Omni on CPU is not a supported
production target here. For a simpler CPU/GGUF option, see [Local AI Server](https://github.com/hossbit/local-ai-server).

After a successful install, start a model that fits your system's memory and
check `localvllm doctor vllm`; inspect the pip log if installation fails.

## Add models

### Local Hugging Face models

For the `vllm` backend, put a complete Hugging Face model directory under:

```text
~/ai-vllm/models/vllm/
`-- Qwen3-8B/
    |-- config.json
    |-- tokenizer.json
    |-- tokenizer_config.json
    `-- model-00001-of-00004.safetensors ...
```

For the `omni` backend, use its own model root:

```text
~/ai-vllm/models/omni/
`-- Qwen3-Omni-30B-A3B-Instruct/
    |-- config.json
    `-- ...
```

vLLM-Omni models often have their own deployment recipes and stage
configuration. Keep any extra deployment files with the model directory and
pass model-specific flags through `EXTRA_ARGS` when needed.

List and select:

```bash
localvllm models
localvllm use Qwen3-8B
localvllm start

localvllm --backend omni models
localvllm --backend omni use Qwen3-Omni-30B-A3B-Instruct
localvllm start omni
```

### Hugging Face repo IDs

Selecting a repo id creates a small models.d entry and lets vLLM download it on
first start:

```bash
localvllm use Qwen/Qwen3-8B
localvllm --backend omni use Qwen/Qwen3-Omni-30B-A3B-Instruct
```

`/v1/models` uses the selected model ID as the served model name by default.
Override it per model with `SERVED_MODEL_NAME=...`.

For gated repositories, set a token in `conf/localvllm.conf`:

```bash
LOCALVLLM_HF_TOKEN="hf_..."
```

or export `HF_TOKEN` when starting the service.

### Local paths outside the model root

`localvllm use /absolute/path/to/model` works too. The path is written to the
backend's `models.d` entry, so it survives reloads.

### GGUF models

For a straightforward GGUF setup, use [Local AI Server](https://github.com/hossbit/local-ai-server),
which runs GGUF models with llama.cpp.

vLLM also supports GGUF experimentally, but its support is under-optimized and
may not work with every model or feature. Current releases require the separate
`vllm-gguf-plugin`, which this project's installer does not install. A compatible
base-model tokenizer may also be needed. See the official [vLLM GGUF guide](https://docs.vllm.ai/en/latest/features/quantization/gguf/)
for version-specific requirements.

vLLM-Omni supports GGUF for certain diffusion transformer weights, with a base
Hugging Face checkpoint and the GGUF plugin. This is model- and hardware-specific;
see the [vLLM-Omni GGUF guide](https://docs.vllm.ai/projects/vllm-omni/en/stable/user_guide/quantization/gguf/).
Discovering a `.gguf` file here does not guarantee that it can be served.

## Commands

| Command | Purpose |
| --- | --- |
| `localvllm start [BACKEND\|all]` | Start one or both services. |
| `localvllm stop [BACKEND\|all]` | Stop services and release GPU memory. |
| `localvllm restart [BACKEND\|all]` | Restart services. |
| `localvllm reload [BACKEND\|all]` | Rescan models; restart only if a launcher changed. |
| `localvllm status [--json] [BACKEND\|all]` | Show service, process, model, port, and API status. |
| `localvllm check [--chat] [BACKEND\|all]` | Check `/health`, `/v1/models`, optionally chat. |
| `localvllm logs [BACKEND]` | Follow one service's current log (previous logs are rotated). |
| `localvllm metrics [--watch] [BACKEND]` | Show vLLM Prometheus metrics: queue, KV cache, throughput. |
| `localvllm models [--json] [BACKEND]` | List that backend's models. |
| `localvllm use MODEL` | Select the model for the current backend. |
| `localvllm load [MODEL]` | Start the service, or wake it from sleep mode. |
| `localvllm unload` | Sleep the model when sleep mode is on, else stop the service. |
| `localvllm ui [--open]` | Print vLLM API docs and OpenAI base URL. |
| `localvllm suggest [MODEL]` | Suggest TP, memory, context, and batching settings. |
| `localvllm suggest --benchmark MODEL` | Measure the running server. |
| `localvllm key ...` | Manage the current backend's active API key. |
| `localvllm backend list` | Show both backends, versions, ports, services, models. |
| `localvllm backend install NAME` | Install/repair one backend. |
| `localvllm backend default NAME` | Change the default backend. |
| `localvllm doctor [--json] [BACKEND\|all]` | Diagnose Python, vLLM, GPU, model, port, keys, pip consistency. |
| `localvllm update [--backend NAME] [--all]` | Update packages and helper scripts. |
| `localvllm version [BACKEND\|all]` | Show versions and active models. |
| `localvllm uninstall` | Remove both backends; models are kept by default. |

Machine-readable output:

```bash
localvllm status --json all
localvllm models --json
localvllm doctor --json all
```

### Backend management

```bash
localvllm backend list
localvllm backend install omni
localvllm backend default omni
```

`backend list` shows an asterisk next to the default backend. Installing a
backend does not change the default. `backend install` is intended for an
existing installation and skips system-package installation; for a fresh
machine, run `install-localvllm.sh` instead.

### Running both at once

```bash
localvllm --backend vllm use Qwen/Qwen3-8B
localvllm --backend omni use Qwen/Qwen3-Omni-30B-A3B-Instruct
localvllm start all
localvllm check all
localvllm status all
```

Both services listen on loopback by default, on different ports. Check
`nvidia-smi` before starting both: two models can easily exceed one GPU's VRAM.

## Direct and automatic serving

Use `localvllm [--backend vllm|omni] mode direct|swap` to change one backend.
Direct remains the default; the engine's native API and tuning are retained.
Automatic mode uses a separate llama-swap instance for each backend, on that
backend's original port. Requests select models by the IDs from `/v1/models`.
Generated upstream workers bind to loopback on dynamically assigned ports,
using the same model overrides and isolated Python environment as direct mode.

Automatic mode uses the backend's existing active API key at the proxy. Private
loopback workers do not duplicate authentication; they should not be exposed.
The dashboard and administration APIs should remain private. Automatic mode
currently refuses non-loopback listeners. Request capture buffers are disabled
by default in the generated configuration.

`localvllm load MODEL` loads a routed model; `localvllm unload` releases routed
models while leaving the proxy ready. `localvllm stop` stops the full backend.
`localvllm ui --dashboard --open` opens its dashboard. In direct mode, existing
load/unload and API-documentation behavior is preserved.

llama-swap is installed on first enable from an official release with SHA256
verification. `LOCALVLLM_LLAMA_SWAP_VERSION=latest` is the default; set a tag
such as `v269` to pin it. `LOCALVLLM_LLAMA_SWAP_BIN` can select an existing executable.
Generated router state lives under `conf/backends/BACKEND/router/`; do not edit
those generated files. Mode state is in `conf/backends/BACKEND/serving-mode`.
Use model override files for tuning. Adding models or changing their settings
refreshes the router configuration and may restart its workers.

Omni worker generation preserves `--omni`, stage/deployment arguments and native
engine tuning mode. Common chat, embedding, image and audio proxy paths are
covered by local HTTP integration tests. Use direct mode for upstream endpoints
or protocols not supported by the selected llama-swap version. Multi-node
orchestration remains an upstream deployment concern.

## Configuration

### Global defaults

Edit `~/ai-vllm/conf/localvllm.conf` for settings shared by both backends.
Per-backend ports and service names also live there:

```bash
LOCALVLLM_VLLM_PORT=8000
LOCALVLLM_OMNI_PORT=8091
LOCALVLLM_LISTEN_HOST=127.0.0.1
```

### Backend profiles

Backend-specific defaults live in:

```text
~/ai-vllm/conf/backends.d/vllm.conf
~/ai-vllm/conf/backends.d/omni.conf
```

They are sourced after the global config whenever that backend is activated.
Use them for differences between the two engines, for example:

```bash
# conf/backends.d/omni.conf
VLLM_OMNI=1
VLLM_MAX_MODEL_LEN=""
VLLM_TENSOR_PARALLEL_SIZE=1
VLLM_GPU_MEMORY_UTILIZATION=0.90
```

The backend identity is enforced: the `omni` backend always passes `--omni`,
and the `vllm` backend never does. A single omni-backend model can opt out with
`OMNI=0` in its override if it is really a text-only model.

### Per-model overrides

Create:

```text
~/ai-vllm/conf/backends/vllm/models.d/<model-id>.conf
~/ai-vllm/conf/backends/omni/models.d/<model-id>.conf
```

Example for a large text model:

```bash
TENSOR_PARALLEL_SIZE=2
GPU_MEMORY_UTILIZATION=0.92
MAX_MODEL_LEN=32768
MAX_NUM_SEQS=128
DTYPE=bfloat16
KV_CACHE_DTYPE=fp8
SERVED_MODEL_NAME="my-model"
REVISION="main"          # optional HF revision/branch/tag
CODE_REVISION="main"     # optional code revision for remote-code models
EXTRA_ARGS=(--enable-prefix-caching)
```

Example for an embedding model:

```bash
MODEL_TYPE=embedding
TASK=embed
MAX_MODEL_LEN=8192
MAX_NUM_BATCHED_TOKENS=2048
```

Example for a reranking model:

```bash
MODEL_TYPE=reranking
TASK=score
```

`EXTRA_ARGS` can be a plain string (`EXTRA_ARGS="--foo bar"`) or a Bash array
when a value contains spaces:

```bash
EXTRA_ARGS=(--rope-scaling '{"type":"yarn","factor":2.0}')
```

Use `EXTRA_ENV` for environment variables:

```bash
EXTRA_ENV=(VLLM_ATTENTION_BACKEND=FLASH_ATTN)
```

After editing, run:

```bash
localvllm --backend vllm reload
```

### Multi-GPU

Local vLLM can launch large models on a single machine with multiple GPUs.
Model size, context length, and concurrency depend on available memory and
upstream engine support. The manager does not impose a model-size limit;
`MAX_MODEL_LEN=8192` is an overridable default, not a maximum.

Select your model, then edit the override file reported by `localvllm models`.
For a text model spread over four GPUs, a starting configuration is:

```bash
TENSOR_PARALLEL_SIZE=4
CUDA_VISIBLE_DEVICES=0,1,2,3
MAX_MODEL_LEN=32768
MAX_NUM_SEQS=256
GPU_MEMORY_UTILIZATION=0.90
```

Choose a tensor-parallel size supported by the model and GPU topology. These
values are examples, not a guarantee that a particular model fits. Increase
context or concurrency only after checking memory use and benchmarking.
`CUDA_VISIBLE_DEVICES` in a model override controls that model; the global
alternative is `LOCALVLLM_CUDA_VISIBLE_DEVICES`.

Additional native vLLM options go in the same file as a Bash array. For example,
when the hardware and model support pipeline parallelism:

```bash
EXTRA_ARGS=(--pipeline-parallel-size 2)
```

Tensor and pipeline parallelism multiply the GPU count. With tensor parallelism
4 and pipeline parallelism 2, select eight GPUs. See the
[upstream parallelism and scaling guide](https://docs.vllm.ai/en/latest/serving/parallelism_scaling/)
for topology, model constraints, and multi-node deployment. This manager runs
services on one host; it does not provision a distributed cluster.

Apply and measure the selected text model:

```bash
localvllm --backend vllm reload
localvllm --backend vllm suggest --benchmark MODEL_ID
```

For Omni, use its own model override and the upstream deployment recipe for
that model. Omni stages may need different GPU placement or parallelism; do
not assume a text-model recipe applies to every Omni model. When running both
backends, assign separate GPUs or budget shared GPU memory explicitly.

Multi-GPU launcher generation is covered by local regression tests. Actual
multi-GPU performance has not been validated on this project's test laptop.

### Native engine tuning and Omni deployment files

Set `ENGINE_DEFAULTS=1` in a model override to let vLLM or an Omni deployment
file own resource tuning. The manager then omits its tensor-parallel, memory,
context, dtype, quantization, batching, offload, eager-mode and runner flags.
Network binding, model identity, authentication, explicit model options and
`EXTRA_ARGS` still apply. Do not combine short tuning settings with this mode;
put desired native flags in `EXTRA_ARGS` or the upstream deployment file.

For an Omni recipe compatible with your installed version:

```bash
ENGINE_DEFAULTS=1
EXTRA_ARGS=(--deploy-config /srv/models/deploy.yaml --strategy-config /srv/models/strategy.yaml)
```

See [Omni deployment configuration](https://docs.vllm.ai/projects/vllm-omni/en/latest/configuration/stage_configs/)
and [parallel strategies](https://docs.vllm.ai/projects/vllm-omni/en/latest/configuration/composable_parallel/).
CLI values can override YAML sizing, so this mode avoids the manager's TP=1
or context defaults overriding a stage recipe. Supported flags depend on the
installed engine version.

### Server hardware and deployment scope

| Hardware or deployment | Setup |
| --- | --- |
| NVIDIA workstation/server GPUs | Use a supported CUDA wheel and driver; single-host multi-GPU options pass through to vLLM. |
| Grace-Hopper / Grace-Blackwell ARM64 | Use matching ARM64 wheels or an upstream build/container; x86 wheels cannot be reused. |
| AMD Instinct / ROCm | Select matching ROCm vLLM, PyTorch and Omni builds; the default CUDA wheel is unsuitable. |
| Intel XPU, NPU and other upstream plugins | Require their platform-specific environment and installation recipe; automatic setup and diagnostics here are not validated for these platforms. |
| Multi-node / Kubernetes | Use upstream orchestration; this manager does not provision clusters or networking. |

For Hopper/Blackwell, upstream recommends precompiled FlashInfer kernels in
the same environment, refreshed after engine upgrades. When supported by the
installed release, run its `vllm download-kernels` command. Blackwell requires
CUDA 12.8 or newer. Wheel, driver, architecture and model support must all
match; see the [vLLM hardware installation guide](https://docs.vllm.ai/en/latest/getting_started/installation/gpu/)
and [Omni installation guide](https://docs.vllm.ai/projects/vllm-omni/en/latest/getting_started/installation/gpu/).

GPU suggestions respect the configured NVIDIA device subset, including a
model's `CUDA_VISIBLE_DEVICES`. MIG partition memory is reported as unavailable
rather than incorrectly using an entire physical GPU's capacity. Suggestions
are starting points, not topology-aware placement or capacity guarantees.

### Sleep mode (fast unload/wake)

vLLM can release GPU memory while keeping the process alive:

```bash
ENABLE_SLEEP=1
```

Then:

```bash
localvllm unload   # POST /sleep?level=1
localvllm load     # POST /wake_up
```

Sleep mode sets `VLLM_SERVER_DEV_MODE=1` in the generated launcher. That exposes
vLLM development endpoints; keep the listener on loopback and do not expose it
to an untrusted network.

### Model type and task

For names that do not make the model's purpose obvious:

```bash
MODEL_TYPE=chat|completion|embedding|reranking
TASK=auto|generate|embed|score|...
```

If `TASK` is empty, `embedding` models get `--task embed` and `reranking`
models get `--task score`. Set `TASK=auto` to let vLLM choose.

## Use the server

Each backend has its own base URL:

```text
vllm: http://127.0.0.1:8000/v1
omni: http://127.0.0.1:8091/v1
```

```bash
curl http://127.0.0.1:8000/v1/models

curl http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen3-8B",
    "messages": [{"role": "user", "content": "Reply with OK"}],
    "max_tokens": 16
  }'
```

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://127.0.0.1:8000/v1",
    api_key="not-needed",
)

response = client.chat.completions.create(
    model="Qwen3-8B",
    messages=[{"role": "user", "content": "Hello"}],
)
print(response.choices[0].message.content)
```

For the omni backend, use port `8091` and its served model name. vLLM-Omni's
supported request shapes depend on the model (chat, image, audio, video); see
the [vLLM-Omni documentation](https://docs.vllm.ai/projects/vllm-omni/).
`localvllm check` (without `--chat`) and `localvllm models` work for every
backend. The chat round-trip and benchmark helpers are text/embedding/rerank
oriented; for image, audio, or video generation use the endpoint documented by
that model's recipe.

### Web UI

vLLM does not ship a chat UI. `localvllm ui` prints:

- API docs: `http://127.0.0.1:8000/docs`
- OpenAI base URL: `http://127.0.0.1:8000/v1`

Point Open WebUI, LibreChat, or any OpenAI-compatible client at the base URL.

### Metrics

vLLM exposes Prometheus metrics. The CLI filters the useful vLLM series:

```bash
localvllm metrics vllm
localvllm metrics --watch omni
```

This shows request queue depth, running requests, KV-cache usage, and
throughput. Use it together with `nvidia-smi` when checking whether both
backends can coexist on one GPU.

## API keys

Each backend has its own key registry under
`conf/backends/<backend>/api-keys.tsv` (mode `0600`). vLLM supports one active
API key per server, so `create` refuses to add a second active key:

```bash
localvllm key create work-laptop
localvllm key list
localvllm key rotate <id>
localvllm key revoke <id>

localvllm --backend omni key create omni-laptop
```

The secret is printed once. Once a key is active, every `/v1` request needs:

```text
Authorization: Bearer sk-localvllm-...
```

Important vLLM limitation: the API key protects `/v1`, `/v2`, `/inference`, and
`/cohere`, but **not** `/health`, `/docs`, or development endpoints such as
`/sleep`. Keep `LOCALVLLM_LISTEN_HOST=127.0.0.1` unless you add TLS and a
reverse proxy with its own authentication.

`LOCALVLLM_REQUIRE_API_KEY=1` makes config generation refuse to produce an
unauthenticated launcher.

## Suggest and benchmark

```bash
localvllm suggest
localvllm suggest Qwen3-8B
localvllm suggest --benchmark Qwen3-8B --runs 3 --tokens 128
localvllm suggest --benchmark Qwen3-8B --json
```

Suggestions use the actual model directory size and detected RAM/VRAM. They
are starting points, not exact predictions: context length, KV cache type,
batching, CUDA graphs, and other workloads all change memory use.

The benchmark measures the running server with one warm-up and repeated
requests, and reports median wall time plus tokens/second from vLLM's `usage`
counts.

## Update

```bash
localvllm update                  # active backend, llama-swap and helpers
localvllm update --all            # every installed backend
localvllm update --backend omni
localvllm update --scripts-only
localvllm update --vllm-only
```

vLLM and vLLM-Omni use separate virtualenvs, so upgrading one does not modify
the other. When pinning versions, keep vLLM-Omni on the matching vLLM release
line:

```bash
VLLM_VERSION=0.32.0 VLLM_OMNI_VERSION=0.32.0 localvllm backend install omni
```

Normal updates also check an installed managed llama-swap. Its download is
staged, checked against the official SHA256 checksum, and executed to verify
the expected version before promotion. Replacement is atomic and participates
in the manager/environment rollback transaction. External executables are never
overwritten. Missing optional routers are not installed by update.

Shared manager changes include all installed backends in the service rollback
scope. Shared router changes restart routed backends while leaving unrelated
direct services running when their manager and packages did not change. Backend
selection still controls which Python environments are upgraded. Launcher
regeneration failure is fatal even for a stopped backend, so a broken generated
configuration cannot silently commit. `--no-start` is respected during rollback.

Updates are staged into a copy of the backend virtualenv, verified with
`pip check` and an import test, then swapped in. If the new environment or the
restarted service fails, the previous environment is restored and restarted.
Running services are never pip-installed in place.

While developing from this checkout, point updates at it instead of the
repository URL:

```bash
LOCALVLLM_LOCAL_SOURCE_DIR="$PWD" localvllm update --scripts-only
```

The default update source is
`https://github.com/hossbit/local-ai-server-vllm.git`; override
`LOCALVLLM_REPO_URL` for a fork or mirror.

## Uninstall

```bash
localvllm uninstall                 # both backends; models are kept
localvllm uninstall --backend omni  # only the omni backend
localvllm uninstall --remove-models --remove-hf-cache --force
localvllm uninstall --keep-venv
```

## Troubleshooting

### `nvidia-smi` works but vLLM cannot use the GPU

Check the driver and the installed torch/CUDA build:

```bash
localvllm doctor all
venv/vllm/bin/python -c 'import torch; print(torch.cuda.is_available())'
```

### Out of memory during startup

Lower one or more of:

```bash
GPU_MEMORY_UTILIZATION=0.85
MAX_MODEL_LEN=4096
MAX_NUM_SEQS=64
```

For a model that still does not fit, use more GPUs:

```bash
TENSOR_PARALLEL_SIZE=2
```

or CPU offload:

```bash
CPU_OFFLOAD_GB=8
```

### Port already in use

The active port is stored per backend in
`conf/backends/<backend>/port`. Edit that file, or delete it and set the
default in `conf/localvllm.conf`:

```bash
LOCALVLLM_VLLM_PORT=8001
LOCALVLLM_OMNI_PORT=8092
```

Then run `localvllm reload` (or `localvllm restart <backend>`). The port is
read before the service starts, so changing the global default alone does not
move an already-installed backend.

### Model not found

```bash
localvllm models
localvllm use MODEL
```

For the omni backend, remember it reads `models/omni`, not `models/vllm`.

### vLLM-Omni installation or version mismatch

vLLM-Omni releases are aligned with vLLM minor versions. Check the
[vLLM-Omni installation guide](https://docs.vllm.ai/projects/vllm-omni/en/latest/getting_started/installation/)
and pin both versions when needed. The pip log is at
`~/ai-vllm/logs/pip-install-omni.log`.

### `use` succeeds but the launcher will not generate

The model must be a Hugging Face directory with `config.json` or a supported
model file. GGUF has additional requirements; see [GGUF models](#gguf-models). Run `localvllm models` to see what was discovered. For remote models,
the first start downloads weights and can take a long time.

### Inspect logs

```bash
localvllm logs vllm
localvllm logs omni
localvllm --backend omni doctor
```

## Security

- Both services bind to `127.0.0.1` by default.
- vLLM's API key does not protect `/health`, `/docs`, or development endpoints.
- `start.sh` refuses a non-loopback listener without an active API key and
  `LOCALVLLM_ALLOW_INSECURE_PUBLIC=1`.
- Sleep/dev mode is refused on non-loopback listeners because `/sleep` and
  `/wake_up` are not protected by vLLM's API key.
- For LAN/WAN access, put a TLS-terminating reverse proxy in front and add
  authentication at the proxy.
- API keys are stored under `conf/backends/<backend>/` with mode `0600`; the
  generated launcher is mode `0700`.
- Request logging is disabled by default (`VLLM_DISABLE_LOG_REQUESTS=1`) so
  prompts are not written to logs unless you explicitly enable it.


## Further reading

- [vLLM documentation](https://docs.vllm.ai/)
- [vLLM-Omni documentation](https://docs.vllm.ai/projects/vllm-omni/)
- [Project issues](https://github.com/hossbit/local-ai-server-vllm/issues)
