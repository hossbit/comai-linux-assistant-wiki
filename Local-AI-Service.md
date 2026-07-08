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

Manual API check:

```bash
curl -s http://127.0.0.1:11435/v1/models | jq -r .data[].id
```

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
| `LOCALAI_N_GPU_LAYERS` | Sets `--n-gpu-layers`. |
| `LOCALAI_THREADS` | Sets `-t`. |
| `LOCALAI_CACHE_TYPE_K` / `LOCALAI_CACHE_TYPE_V` | Set KV cache quantization. |
| `LOCALAI_PARALLEL` | Adds `--parallel` when set. |
| `LOCALAI_BATCH_SIZE` | Adds `--batch-size` when set. |
| `LOCALAI_UBATCH_SIZE` | Adds `--ubatch-size` when set. |
| `LOCALAI_FLASH_ATTN` | Adds `--flash-attn on` when set to `1`. |
| `LOCALAI_JINJA` | Adds `--jinja` when set to `1`. |
| `LOCALAI_MLOCK` | Adds `--mlock` when set to `1`. |
| `LOCALAI_NO_MMAP` | Adds `--no-mmap` when set to `1`. |
| `LOCALAI_EXTRA_LLAMA_ARGS` | Appends extra single-line llama-server flags. |

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
```

LocalAI update and uninstall:

```bash
localai update
localai uninstall
```

LocalAI 1.2.3 and newer updates are layout-safe. The updater copies the
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
