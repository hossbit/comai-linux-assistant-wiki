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

This page is for users who install both projects. If you only use LocalAI, you can ignore ComAI commands and use [Local AI Service](Local-AI-Service.md) directly.

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

## Check Through ComAI

After LocalAI is running and ComAI is configured, check the integration from the ComAI side:

```bash
comai status local
comai provider
comai models local
comai ask "Reply with OK"
```

These commands verify that ComAI can reach the local OpenAI-compatible API and that the configured model name matches what LocalAI exposes.

## ComAI Local Provider Config

Normal ComAI local provider config:

```yaml
providers:
  local:
    api_base: http://127.0.0.1:11435
    model: Qwen2.5-Coder-7B-Instruct-Q4_K_M
    api_key:
    api_key_cmd:
```

`api_key`/`api_key_cmd` are optional and blank by default — see [Securing LocalAI With API Keys](#securing-localai-with-api-keys).

Optional LocalAI helper directory:

```yaml
ai_dir: ~/ai
```

`ai_dir` only controls the optional ComAI start/stop helper. It does not choose the API endpoint or model for normal local requests.

## Securing LocalAI With API Keys

LocalAI is unauthenticated by default. If you turn on its optional API keys
(see [Local AI Service](Local-AI-Service.md#api-keys)), ComAI's `local`
provider needs to know the key too, or every request fails with `401
Unauthorized`.

Create a key on the LocalAI side:

```bash
localai key create
```

Save the printed secret — it is shown only once — then give it to ComAI. Any
one of these works, checked in this order:

```bash
# 1. Environment variable, highest priority
export LOCALAI_API_KEY="sk-localai-REPLACE_ME"

# 2. A command that prints the key, stored in config
comai config set providers.local.api_key_cmd "pass show localai"

# 3. The key itself, stored in config
comai config set providers.local.api_key "sk-localai-REPLACE_ME"
```

Option 3 stores the secret in plain text in `~/localcomai/config/comai.yaml`
(mode `600`). Options 1 and 2 keep it out of the config file.

Confirm it works:

```bash
comai status local
comai ask "Reply with OK"
```

If you forget this step after creating a key, ComAI reports it clearly
instead of a generic connection failure:

```text
comai: Local provider API at http://127.0.0.1:11435 returned 401 Unauthorized.
comai: Set providers.local.api_key in ~/localcomai/config/comai.yaml, or export LOCALAI_API_KEY.
```

Revoking every active key on the LocalAI side returns both sides to
unauthenticated behavior; there is nothing to undo in ComAI's config, though
you may want to blank the stored `api_key` too.

## Optional ComAI Helper Details

When enabled, `comai-localai.service` calls the LocalAI helper scripts in the configured `ai_dir`:

```bash
systemctl --user start comai-localai.service
systemctl --user stop comai-localai.service
systemctl --user restart comai-localai.service
```

If the service file is not installed, ComAI falls back to:

```bash
~/ai/bin/start.sh
~/ai/bin/stop.sh
```

Older LocalAI installs with `~/ai/start.sh` and `~/ai/stop.sh` are still supported as a fallback.

For a custom LocalAI directory when installing ComAI from source:

```bash
./scripts/install.sh --ai-dir ~/myai
```

Then the helper service uses:

```bash
~/myai/bin/start.sh
~/myai/bin/stop.sh
```

ComAI service and provider events are written under the ComAI install directory:

```bash
~/localcomai/logs/comai.log
```

Watch logs:

```bash
tail -f ~/localcomai/logs/comai.log
```
