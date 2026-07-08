# ComAI And LocalAI

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

ComAI and LocalAI are separate projects that can work together.

- **ComAI** is the terminal assistant client.
- **LocalAI** is one optional local backend server.

ComAI does not contain a language model and does not require LocalAI. It sends
requests to a provider. That provider can be LocalAI, Ollama, LM Studio,
llama.cpp server, OpenAI, or any OpenAI-compatible API.

```text
ComAI CLI
  -> LocalAI
  -> Ollama
  -> LM Studio
  -> llama.cpp server
  -> OpenAI
  -> any OpenAI-compatible API
```

## Which Project Does What

| Project | Role | Owns |
| --- | --- | --- |
| ComAI | Terminal AI client | `comai`, provider config, prompts, files, logs, history |
| LocalAI | Local model server | GGUF models, llama.cpp, llama-swap, local API, `localai` |

Install ComAI when you want the terminal assistant. Install LocalAI only when
you want to run GGUF models locally through a local OpenAI-compatible API.

## Install Order

Both install orders are valid.

### ComAI First, LocalAI Later

Use this when you want the ComAI client first, or when you already have another
local provider.

```bash
curl -fsSL https://hossbit.github.io/comai/install.sh | bash
```

After installing ComAI, local provider status may fail until a local server is
running. That is normal.

```bash
comai status
```

Later, install LocalAI if you want to run GGUF models locally:

```bash
curl -fsSL https://hossbit.github.io/localai/install.sh | bash
```

The LocalAI installer defaults to the Vulkan llama.cpp backend. For a CPU-only
machine or simple VM test, use:

```bash
curl -fsSL https://hossbit.github.io/localai/install.sh | LLAMA_CPP_BACKEND=cpu bash
```

If a non-CPU backend installs but cannot run on the system, LocalAI retries once
with the CPU backend by default.

Add a chat model to the LocalAI models directory:

```bash
cp Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf ~/ai/models/
localai start
localai check
```

Then point ComAI at the model name without `.gguf`:

```bash
comai config set provider local
comai config edit
```

Use the local provider section:

```yaml
providers:
  local:
    api_base: http://127.0.0.1:11435
    model: Qwen2.5-Coder-7B-Instruct-Q4_K_M
```

Then check it:

```bash
comai status local
```

### LocalAI First, ComAI Later

Use this when you want the local server and model ready before installing the
terminal assistant.

```bash
curl -fsSL https://hossbit.github.io/localai/install.sh | bash
```

Add a model and start the server:

```bash
cp your-chat-model.gguf ~/ai/models/
localai start
localai check
```

Then install ComAI:

```bash
curl -fsSL https://hossbit.github.io/comai/install.sh | bash
```

Set the matching model in ComAI:

```bash
comai config set provider local
comai config edit
```

Update:

```yaml
providers:
  local:
    api_base: http://127.0.0.1:11435
    model: MODEL_NAME_WITHOUT_GGUF
```

Then run:

```bash
comai status local
```

## Optional ComAI Helper Service

ComAI installs an optional `comai-localai.service` helper, but it does not start
it automatically. This helper is only for users who want ComAI commands such as
`comai start`, `comai stop`, and `comai restart` to call a LocalAI install's
`bin/start.sh` and `bin/stop.sh`.

If you use Ollama, LM Studio, llama.cpp server, or another local provider, you
can ignore this helper.

Start the helper only when LocalAI is installed and the configured `ai_dir`
contains `bin/start.sh` and `bin/stop.sh`:

```bash
systemctl --user enable --now comai-localai.service
```

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

CPU installs use:

```text
LOCALAI_N_GPU_LAYERS=0
LOCALAI_CTX_SIZE=4096
LOCALAI_FLASH_ATTN=0
LOCALAI_PARALLEL=1
```

CPU mode is useful for compatibility testing, but larger models may be slow.

## LocalAI Install Options

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

Backend selection:

```bash
LLAMA_CPP_BACKEND=cpu ./install-local-ai.sh
LLAMA_CPP_BACKEND=vulkan ./install-local-ai.sh
LLAMA_CPP_BACKEND=rocm ./install-local-ai.sh
LLAMA_CPP_BACKEND=openvino ./install-local-ai.sh
LLAMA_CPP_BACKEND=sycl-fp16 ./install-local-ai.sh
LLAMA_CPP_BACKEND=sycl-fp32 ./install-local-ai.sh
```

Start LocalAI automatically when you log in:

```bash
systemctl --user enable --now localai
```

## Model Notes

Use a chat or instruct model for ComAI chat requests. For example:

```text
Qwen2.5-Coder-7B-Instruct-Q2_K.gguf
Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf
```

Embedding models, such as many `bge` or `e5` files, are for embeddings and
search-style workflows. They are not the right first choice for chat.

Quantization tradeoffs:

| Quantization | Use |
| --- | --- |
| Q2_K | Smallest, lower quality, useful for limited RAM or quick tests |
| Q4_K_M | Better default balance for chat quality and memory |
| Q6_K/Q8_0 | Higher quality, more memory |

### Add Models

Place GGUF files in:

```text
~/ai/models
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

Check installed models and get hardware-aware suggestions:

```bash
localai models
localai suggest
localai suggest MODEL_NAME
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
| `LOCALAI_FLASH_ATTN` | Adds `--flash-attn` when set to `1`. |
| `LOCALAI_JINJA` | Adds `--jinja` when set to `1`. |
| `LOCALAI_MLOCK` | Adds `--mlock` when set to `1`. |
| `LOCALAI_NO_MMAP` | Adds `--no-mmap` when set to `1`. |
| `LOCALAI_EXTRA_LLAMA_ARGS` | Appends extra single-line llama-server flags. |

## Test Commands

ComAI:

```bash
comai version
comai status
comai provider
comai models local
comai ask "Reply with OK"
```

LocalAI:

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

## Summary

ComAI is the client. LocalAI is one optional backend. They integrate well, but
they remain independent installs with separate commands, updates, and
responsibilities.

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
