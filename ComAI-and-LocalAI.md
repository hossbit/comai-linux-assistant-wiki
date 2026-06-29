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
git clone https://github.com/hossbit/comai-linux-assistant.git
cd comai-linux-assistant
./scripts/install.sh
```

After installing ComAI, local provider status may fail until a local server is
running. That is normal.

```bash
comai status
```

Later, install LocalAI if you want to run GGUF models locally:

```bash
git clone https://github.com/hossbit/local-ai-server.git
cd local-ai-server
LLAMA_CPP_BACKEND=cpu ./install-local-ai.sh
```

Add a chat model:

```bash
cp Qwen2.5-Coder-7B-Instruct-Q2_K.gguf ~/ai/models/
localai start
localai check
```

Then point ComAI at the model name without `.gguf`:

```bash
comai config set provider local
comai config set local_api_base http://127.0.0.1:11435
comai config set local_model Qwen2.5-Coder-7B-Instruct-Q2_K
comai status
```

### LocalAI First, ComAI Later

Use this when you want the local server and model ready before installing the
terminal assistant.

```bash
git clone https://github.com/hossbit/local-ai-server.git
cd local-ai-server
./install-local-ai.sh
```

Add a model and start the server:

```bash
cp your-chat-model.gguf ~/ai/models/
localai start
localai check
```

Then install ComAI:

```bash
git clone https://github.com/hossbit/comai-linux-assistant.git
cd comai-linux-assistant
./scripts/install.sh
```

Set the matching model in ComAI:

```bash
comai config set provider local
comai config set local_model MODEL_NAME_WITHOUT_GGUF
comai status
```

## Optional ComAI Helper Service

ComAI installs an optional `comai-localai.service` helper, but it does not start
it automatically. This helper is only for users who want ComAI commands such as
`comai start`, `comai stop`, and `comai restart` to call a LocalAI install's
`start.sh` and `stop.sh`.

If you use Ollama, LM Studio, llama.cpp server, or another local provider, you
can ignore this helper.

Start the helper only when LocalAI is installed and the configured `ai_dir`
contains `start.sh` and `stop.sh`:

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
ca-certificates curl iproute jq tar vulkan-loader
```

For a CPU-only Fedora VM, use:

```bash
LLAMA_CPP_BACKEND=cpu ./install-local-ai.sh
```

CPU installs use:

```text
LOCALAI_N_GPU_LAYERS=0
LOCALAI_CTX_SIZE=4096
```

CPU mode is useful for compatibility testing, but larger models may be slow.

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
```

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
