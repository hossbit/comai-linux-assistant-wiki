# Local AI Service

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

LocalAI is a separate local model server for GGUF models. It installs `llama.cpp`, `llama-swap`, helper scripts, a user service, and an OpenAI-compatible local API.

You can use LocalAI by itself, or point OpenAI-compatible clients at its local API.

## Install LocalAI

Install LocalAI when you want to run local GGUF models through an
OpenAI-compatible API.

Default install:

```bash
curl -fsSL https://hossbit.github.io/localai/install.sh | bash
```

The public installer is served from `https://hossbit.github.io/localai/install.sh`. The LocalAI source repository keeps the project installer at the repository root and does not need a duplicate `site/localai` copy.

CPU-only install for simple VMs, older machines, or systems without a supported
GPU backend:

```bash
curl -fsSL https://hossbit.github.io/localai/install.sh | LLAMA_CPP_BACKEND=cpu bash
```

The LocalAI installer downloads current `llama.cpp` and `llama-swap` releases
by default. Pin explicit versions only when you need a known-good build:

```bash
LLAMA_CPP_VERSION=b9672 LLAMA_SWAP_VERSION=v226 ./install-local-ai.sh
```

Custom install directory:

```bash
curl -fsSL https://hossbit.github.io/localai/install.sh | LOCALAI_DIR="$HOME/my-ai" bash
```

Manual install from source:

```bash
git clone https://github.com/hossbit/local-ai-server.git
cd local-ai-server
chmod +x ./*.sh
./install-local-ai.sh
```

## Backend Selection

The LocalAI installer defaults to the Vulkan llama.cpp backend. Select another
backend when needed:

```bash
LLAMA_CPP_BACKEND=cpu ./install-local-ai.sh
LLAMA_CPP_BACKEND=vulkan ./install-local-ai.sh
LLAMA_CPP_BACKEND=rocm ./install-local-ai.sh
LLAMA_CPP_BACKEND=openvino ./install-local-ai.sh
LLAMA_CPP_BACKEND=sycl-fp16 ./install-local-ai.sh
LLAMA_CPP_BACKEND=sycl-fp32 ./install-local-ai.sh
```

If a non-CPU backend installs but cannot run on the system, LocalAI retries once
with the CPU backend by default.

CPU installs use conservative runtime settings:

```text
LOCALAI_N_GPU_LAYERS=0
LOCALAI_CTX_SIZE=4096
LOCALAI_FLASH_ATTN=0
LOCALAI_PARALLEL=1
```

CPU mode is useful for compatibility testing, but larger models may be slow.

## Fedora Notes

Fedora may need `git` before cloning the projects:

```bash
sudo dnf install -y git
```

LocalAI installs its runtime dependencies with `dnf`, including:

```text
ca-certificates curl iproute jq mesa-vulkan-drivers tar vulkan-loader
```

On RHEL 7/8, enable EPEL first if `jq` is not available from the enabled
repositories.

For a CPU-only Fedora VM, use:

```bash
curl -fsSL https://hossbit.github.io/localai/install.sh | LLAMA_CPP_BACKEND=cpu bash
```

## Add Models

Use a chat or instruct GGUF model for chat requests. For example:

```text
Qwen2.5-Coder-7B-Instruct-Q2_K.gguf
Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf
```

Place GGUF files under the LocalAI models directory:

```text
~/ai/models
```

Example with an existing file:

```bash
cp Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf ~/ai/models/
localai start
localai check
localai models
```

Example with the Hugging Face CLI:

```bash
python3 -m pip install --user huggingface_hub
hf auth login

hf download bartowski/Qwen2.5-Coder-7B-Instruct-GGUF \
  Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf \
  --local-dir ~/ai/models
```

Some model repositories require a Hugging Face account and read token. See
[Hugging Face access tokens](https://huggingface.co/docs/hub/security-tokens).

The model ID exposed by LocalAI is the filename without `.gguf`:

```text
Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf
```

becomes:

```text
Qwen2.5-Coder-7B-Instruct-Q4_K_M
```

Embedding models, such as many `bge` or `e5` files, are for embeddings and
search-style workflows. They are not the right first choice for chat.

Quantization tradeoffs:

| Quantization | Use |
| --- | --- |
| Q2_K | Smallest, lower quality, useful for limited RAM or quick tests |
| Q4_K_M | Better default balance for chat quality and memory |
| Q6_K/Q8_0 | Higher quality, more memory |

Use `localai suggest` after adding large models to get advisory runtime settings
based on your installed models, RAM, backend, and detected GPU memory. The
advisor uses the actual GGUF file size as the base estimate, not an exact
parameter-count formula. Runtime memory also depends on context length, KV
cache type, batch size, backend buffers, and operating-system headroom.

### Split GGUF Models

Split GGUF models are supported when shards use llama.cpp canonical naming:

```text
name-00001-of-00003.gguf
name-00002-of-00003.gguf
name-00003-of-00003.gguf
```

For a small number of split models, a flat layout works:

```text
~/ai/models/DeepSeek-V4-Flash-UD-IQ1_M-00001-of-00003.gguf
~/ai/models/DeepSeek-V4-Flash-UD-IQ1_M-00002-of-00003.gguf
~/ai/models/DeepSeek-V4-Flash-UD-IQ1_M-00003-of-00003.gguf
```

For many split models, use one folder per model:

```text
~/ai/models/deepseek-v4-flash/
  DeepSeek-V4-Flash-UD-IQ1_M-00001-of-00003.gguf
  DeepSeek-V4-Flash-UD-IQ1_M-00002-of-00003.gguf
  DeepSeek-V4-Flash-UD-IQ1_M-00003-of-00003.gguf
```

LocalAI registers only the first shard and passes it to `llama-server`;
llama.cpp loads the remaining shards from the same directory. If a folder
contains one model, the folder name becomes the model ID.

Files that look like split fragments but do not follow canonical naming are
skipped with a warning. Rename shards to `name-00001-of-000NN.gguf`, keep all
shards in one directory, or merge them:

```bash
llama-gguf-split --merge FIRST_SHARD.gguf OUTPUT.gguf
```

## Start, Stop, Restart

Use the LocalAI command:

```bash
localai start
localai stop
localai restart
```

Or use the LocalAI user service directly:

```bash
systemctl --user start localai
systemctl --user stop localai
systemctl --user restart localai
```

Start LocalAI automatically when you log in:

```bash
systemctl --user enable --now localai
```

## Check Status

```bash
localai status
localai check
localai models
```

`localai check` verifies the process, port, and API model list. Add `--chat`
to also round-trip a real chat completion against the first non-embedding
model, which confirms the model loads and generates:

```bash
localai check --chat
```

Manual API check:

```bash
curl -s http://127.0.0.1:11435/v1/models | jq -r .data[].id
```

## API Keys

By default LocalAI has no authentication, matching llama-swap's own default —
fine as long as it only listens on `127.0.0.1` (the default). If you plan to
reach it from another machine on your LAN, create a key first:

```bash
localai key create work-laptop
localai key list
localai key revoke KEY_ID
localai key rotate KEY_ID
```

`create` and `rotate` print the full secret exactly once, right after it
becomes active — save it immediately, it cannot be shown again. `list` only
ever shows a masked fingerprint.

Once at least one key is active, every request needs it:

```bash
curl http://127.0.0.1:11435/v1/models \
  -H "Authorization: Bearer sk-localai-REPLACE_ME"
```

Revoking the last active key returns to unauthenticated behavior, unless
`LOCALAI_REQUIRE_API_KEY=1` is set (see the table below), which makes config
generation refuse to produce an unauthenticated config at all.

Creating, revoking, or rotating a key restarts the running service, the same
way changing a model does. Keys are stored in `conf/api-keys.tsv`, outside
Git, mode `600`. Active keys are also rendered into `conf/keys.d/keys.yaml`
(mode `600`), each with a `# name` comment taken from its `api-keys.tsv`
label, and merged into the running config via llama-swap's `--config-dir` —
`conf/config.yaml` itself never contains a plaintext key. `keys.yaml` is
removed entirely once no keys are active. You still only manage keys through
`localai key ...`, never by hand-editing either file.

Using ComAI's `local` provider against a key-protected LocalAI? See
[Securing LocalAI With API Keys](ComAI-and-LocalAI.md#securing-localai-with-api-keys).

API keys authenticate requests; they don't encrypt them. For LAN/WAN access,
put a TLS-terminating reverse proxy in front and restrict it with a
firewall.

## LocalAI Configuration

Installed LocalAI defaults live in:

```text
~/ai/conf/localai.conf
```

Environment variables can override runtime settings for one start:

```bash
LOCALAI_CTX_SIZE=8192 LOCALAI_N_GPU_LAYERS=20 localai start
LOCALAI_FLASH_ATTN=1 LOCALAI_PARALLEL=2 localai start
```

Useful tuning variables:

| Variable | Effect |
| --- | --- |
| `LOCALAI_CTX_SIZE` | Sets `--ctx-size`. |
| `LOCALAI_N_GPU_LAYERS` | Sets `--n-gpu-layers` (overridden per model when auto-tune is on; see below). |
| `LOCALAI_THREADS` | Sets `-t`. |
| `LOCALAI_CACHE_TYPE_K` / `LOCALAI_CACHE_TYPE_V` | Set KV cache quantization (overridden per model when auto-tune is on). |
| `LOCALAI_PARALLEL` | Adds `--parallel` when set. |
| `LOCALAI_BATCH_SIZE` | Adds `--batch-size` when set. |
| `LOCALAI_UBATCH_SIZE` | Adds `--ubatch-size` when set. |
| `LOCALAI_FLASH_ATTN` | Adds `--flash-attn on` when set to `1` (overridden per model when auto-tune is on). |
| `LOCALAI_JINJA` | Adds `--jinja` when set to `1`. |
| `LOCALAI_MLOCK` | Adds `--mlock` when set to `1`. |
| `LOCALAI_NO_MMAP` | Adds `--no-mmap` when set to `1`. |
| `LOCALAI_EXTRA_LLAMA_ARGS` | Appends extra single-line llama-server flags. |
| `LOCALAI_SPLIT_MODE` | Sets `--split-mode` (`none`, `layer`, or `tensor`) for multi-GPU installs. See [Multi-GPU](#multi-gpu). |
| `LOCALAI_TENSOR_SPLIT` | Sets `--tensor-split`, e.g. `3,1` to give GPU 0 three times GPU 1's share. |
| `LOCALAI_MAIN_GPU` | Sets `--main-gpu` (device index), used with `--split-mode none`. |
| `LOCALAI_DEVICE` | Sets `--device`, a comma-separated device list (e.g. `CUDA0,CUDA1`) to restrict which GPUs llama-server uses. |
| `LOCALAI_AUTO_TUNE` | `1` (default on non-CPU backends) auto-computes per-model GPU layers/cache/flash-attn. Set `0` to force the flat values above onto every model. |
| `LOCALAI_SPEC_TYPE` | Speculative-decoding mode. Defaults to `ngram-simple` on non-CPU backends, `""` on CPU. See [Speculative Decoding](#speculative-decoding). |
| `LOCALAI_SPEC_DRAFT_N_MAX` | Max tokens to draft per step for speculative decoding. Default `16`. |
| `LOCALAI_METRICS_ENABLED` | `1` (default) exposes llama-swap's `/metrics` endpoint. See [Metrics](#metrics). |
| `LOCALAI_PRELOAD_MODELS` | Comma/space-separated model IDs to warm on `start`/`restart`. See [Preloading Models](#preloading-models). |
| `LOCALAI_EMBEDDING_TTL` | Default `ttl` (seconds) applied to detected embedding models. Default `120`. |
| `LOCALAI_MODELS_OVERRIDE_SUBDIR` | Subdirectory name for per-model override files. Default `models.d`. See [Per-Model Overrides](#per-model-overrides). |
| `LOCALAI_API_KEY_FILE` | Key registry filename under `conf/`. Default `api-keys.tsv`. See [API Keys](#api-keys). |
| `LOCALAI_REQUIRE_API_KEY` | `1` refuses to generate a config with zero active keys, so auth can never be silently disabled. Default `0`. See [API Keys](#api-keys). |

## Auto-Tuning

On any non-CPU backend, `rebuild-config.sh` (run automatically by
`start`/`restart`/`reload`) auto-tunes each model instead of applying one flat
setting to all of them:

- `--n-gpu-layers auto` — lets llama-server fit layers to free device memory
  at load time. This is deliberately not a fixed number: llama-server's own
  auto-fit logic is disabled whenever `-ngl` is set to an explicit number, so
  a hardcoded "does it fit" guess is actually less safe than `auto` and can
  OOM a model that would otherwise have partially offloaded successfully.
- `--cache-type-k q8_0 --cache-type-v q8_0` — halves KV cache VRAM versus
  `f16` at negligible quality cost.
- `--flash-attn on`.

Set `LOCALAI_AUTO_TUNE=0` to disable this and use the flat
`LOCALAI_N_GPU_LAYERS`/`LOCALAI_CACHE_TYPE_K`/`LOCALAI_CACHE_TYPE_V`/`LOCALAI_FLASH_ATTN`
values from the table above for every model instead.

## Speculative Decoding

Non-CPU backends default to `LOCALAI_SPEC_TYPE=ngram-simple` with
`LOCALAI_SPEC_DRAFT_N_MAX=16`. This drafts tokens from patterns already seen
in the response (useful for code, which repeats structure) and verifies every
draft against the target model, so it never changes output quality, only
speed. It needs no separate draft model and no extra VRAM.

Set `LOCALAI_SPEC_TYPE=""` to disable it globally, or override it per model
(see below). Other supported values (draft-model and n-gram variants) come
from llama.cpp's own [speculative decoding
docs](https://github.com/ggml-org/llama.cpp/blob/master/docs/speculative.md).

## Multi-GPU

All blank by default, which leaves llama-server on its own defaults (splits
each model across every visible GPU by layer). Set these to change how a
system with more than one GPU is used:

```bash
LOCALAI_SPLIT_MODE=layer localai start    # default: split layers across GPUs
LOCALAI_SPLIT_MODE=tensor localai start   # split tensors/KV cache across GPUs (needs fast interconnect, --flash-attn)
LOCALAI_SPLIT_MODE=none LOCALAI_MAIN_GPU=1 localai start  # pin to one GPU
LOCALAI_TENSOR_SPLIT=3,1 localai start    # give GPU 0 three times GPU 1's share
LOCALAI_DEVICE=CUDA0,CUDA1 localai start  # restrict which GPUs are visible
```

`localai suggest` reports the number of GPUs it detects and reminds you these
variables exist when there's more than one. `tensor` split mode isn't
available for every model architecture (MoE and state-space models fall back
to `layer`); KV cache quantization also only applies in `layer`/`none` mode.
See llama.cpp's own [multi-GPU
docs](https://github.com/ggml-org/llama.cpp/blob/master/docs/multi-gpu.md)
for the full tradeoffs. Override any of these per model the same way as other
tuning variables (see [Per-Model Overrides](#per-model-overrides)).

## Metrics

`LOCALAI_METRICS_ENABLED=1` (default) exposes llama-swap's built-in
`/metrics` endpoint — system and GPU stats in Prometheus format, no
llama-server rebuild or extra flags required:

```bash
curl -s http://127.0.0.1:11435/metrics
```

If you have [API keys](#api-keys) active, `/metrics` requires the same
`Authorization: Bearer` header as every other endpoint — llama-swap doesn't
carve out an exception for it:

```bash
curl -s http://127.0.0.1:11435/metrics \
  -H "Authorization: Bearer sk-localai-REPLACE_ME"
```

llama-swap has no HTTP Basic Auth support (only Bearer `apiKeys`), so point
Prometheus at a key with `bearer_token` (or `bearer_token_file` to avoid
putting the secret in the scrape config itself):

```yaml
scrape_configs:
  - job_name: localai
    bearer_token: sk-localai-REPLACE_ME
    static_configs:
      - targets: ["127.0.0.1:11435"]
```

llama-swap ships a [ready-made Grafana
dashboard](https://github.com/mostlygeek/llama-swap/blob/main/docs/grafana/example-dashboard.json)
for these metrics.

## Preloading Models

Set `LOCALAI_PRELOAD_MODELS="model-a,model-b"` (comma or space separated
model IDs, as shown by `localai models`) to warm those models on
`localai start`/`restart` instead of paying cold-start latency on the first
request.

## Multimodal Models

Keep a vision/audio model in its own folder alongside a projector file named
`mmproj*.gguf`:

```text
~/ai/models/gemma-vision/
|-- gemma-vision.gguf
`-- mmproj-gemma.gguf
```

LocalAI detects the projector file and adds `--mmproj` automatically. The
projector file itself is never registered as its own model entry. Flat
top-level models (not in their own folder) need an explicit `MMPROJ=` in a
per-model override instead (see below).

## Per-Model Overrides

Global settings and auto-tuning apply to every model the same way. To
override settings for one model, drop a file at
`conf/models.d/<model-id>.conf`, using the same `<model-id>` shown by
`localai models`. It's sourced as bash after auto-tuning runs, so it can set:

```bash
# conf/models.d/Qwen2.5-Coder-7B-Instruct-Q4_K_M.conf
CTX_SIZE=32768
N_GPU_LAYERS=999        # or "auto"/"all"
CACHE_TYPE_K=q8_0
CACHE_TYPE_V=q8_0
FLASH_ATTN=1
SPEC_TYPE=ngram-simple  # or "" to disable for this model
SPEC_DRAFT_N_MAX=24
TTL=0                   # seconds; 0 = never auto-unload
ALIASES="gpt-4o-mini, coder"
MMPROJ=/path/to/mmproj.gguf
SET_TEMPERATURE=0.2
SET_TOP_P=0.9
EXTRA_ARGS="--no-mmproj-offload"
SPLIT_MODE=none        # or "layer"/"tensor"; blank = llama-server's own default
TENSOR_SPLIT=3,1
MAIN_GPU=1
DEVICE=CUDA1            # pin this one model to a specific GPU
```

`ALIASES` becomes llama-swap `aliases:`; `SET_TEMPERATURE`/`SET_TOP_P` become
a `filters.setParams` block. Embedding models get `LOCALAI_EMBEDDING_TTL`
(120s default) automatically unless a `models.d` file sets `TTL` explicitly.

## Logs

LocalAI service logs are written under the LocalAI install directory:

```bash
~/ai/logs/llama-swap.log
```

Watch logs:

```bash
localai logs
```

## Test Commands

```bash
localai version
localai models
localai start
localai status
localai check
localai check --chat
localai load MODEL_NAME
localai unload all
localai key create
localai key list
```

LocalAI update and uninstall:

```bash
localai update
localai uninstall
```

LocalAI 1.2.4 and newer updates are layout-safe. The updater copies the
installed `lib/` tree recursively, so future internal structure changes do not
require uninstalling an older release first. If an older update leaves split CLI
modules missing, `localai` attempts a one-time repair with the installed direct
updater before failing.

`localai uninstall` removes the LocalAI helper/runtime files, including the
installed `bin`, `conf`, `lib`, and `logs` directories. It keeps
`~/ai/models` by default so downloaded GGUF models are not deleted. Use
`localai uninstall --remove-models` only when you also want to delete the model
files.

Direct API:

```bash
curl http://127.0.0.1:11435/v1/models
```

Chat API:

```bash
curl http://127.0.0.1:11435/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "MODEL_NAME",
    "messages": [{"role": "user", "content": "Reply with OK"}],
    "max_tokens": 8
  }'
```
