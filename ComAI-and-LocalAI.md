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

For LocalAI install options, backend selection, service commands, and model
management, see [Local AI Service](Local-AI-Service.md).

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
comai status local
```

Later, install and start LocalAI if you want to run GGUF models locally. See
[Local AI Service](Local-AI-Service.md) for the LocalAI install and model setup.

Then point ComAI at the LocalAI model name:

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
comai provider
```

### LocalAI First, ComAI Later

Use this when you want the local server and model ready before installing the
terminal assistant.

Install LocalAI, add a model, and confirm the API works first. See
[Local AI Service](Local-AI-Service.md) for those steps.

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
comai models local
```

## Optional ComAI Helper Service

ComAI installs an optional `comai-localai.service` helper, but it does not start
it automatically. This helper is only for users who want ComAI commands such as
`comai start`, `comai stop`, and `comai restart` to call a LocalAI install's
`bin/start.sh` and `bin/stop.sh`.

If you use Ollama, LM Studio, llama.cpp server, or another local provider, you
can ignore this helper and configure the provider's `api_base` directly.

Start the helper only when LocalAI is installed and the configured `ai_dir`
contains `bin/start.sh` and `bin/stop.sh`:

```bash
systemctl --user enable --now comai-localai.service
```

For helper service details, see [Local AI Service](Local-AI-Service.md).
