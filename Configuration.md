# Configuration

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

ComAI reads defaults from YAML and supports one-off overrides from flags or environment variables.

## Config Files

Source default:

```bash
config/comai.yaml
```

Installed config:

```bash
~/localcomai/config/comai.yaml
```

Show installed config safely:

```bash
comai config show
```

`providers.openai.api_key` and legacy `openai_api_key` values are masked when shown. The installed config file is tightened to mode `600` when ComAI reads or writes it.

Read one value:

```bash
comai config get provider
```

## Current Config Example

```yaml
# Default provider. Use "local", "ollama", "lmstudio", or "openai".
provider: local

# Optional LocalAI helper directory. This is only used by the start/stop helper service.
ai_dir: ~/ai

# Provider-specific settings.
providers:
  local:
    api_base: http://127.0.0.1:11435
    model: Qwen2.5-Coder-7B-Instruct-Q4_K_M

  ollama:
    api_base: http://127.0.0.1:11434
    model: qwen2.5-coder:7b

  lmstudio:
    api_base: http://127.0.0.1:1234
    model: local-model

  openai:
    api_base: https://api.openai.com
    model: gpt-5.5
    api_key:
    api_key_cmd:

max_tokens: 420
timeout: 120
log_file: logs/comai.log
file_max_bytes: 24000
dir_context_max: 120
error_regex: error|errors|failed|failure|exception|fatal|panic|timeout|warn|warning|traceback
error_intent_regex: error|errors|failed|failure|warning|warnings|problem|problems|issue|issues|wrong|bad|broken|fail|crash|crashed|panic|timeout|traceback|healthy|health|(^|[[:space:]])ok([[:space:]]|$)|okay|check (this )?log|scan (this )?log
```

## Key Reference

| Key | Purpose |
| --- | --- |
| `provider` | Default provider: `local`, `ollama`, `lmstudio`, or `openai`. |
| `ai_dir` | Optional LocalAI helper directory for `comai start`, `stop`, and `restart`. |
| `providers.local.api_base` | OpenAI-compatible local API base. |
| `providers.local.model` | Default local OpenAI-compatible model. |
| `providers.ollama.api_base` | Ollama API base. |
| `providers.ollama.model` | Default Ollama model. |
| `providers.lmstudio.api_base` | LM Studio local server base URL. |
| `providers.lmstudio.model` | Default LM Studio model. |
| `providers.openai.api_base` | OpenAI API base. |
| `providers.openai.model` | Default OpenAI model. |
| `providers.openai.api_key` | Optional API key in config. Environment variable is safer. |
| `providers.openai.api_key_cmd` | Optional command that prints an API key, such as `pass show openai`. |
| `max_tokens` | Maximum requested answer length. |
| `timeout` | Request timeout in seconds. |
| `log_file` | Service/status log path. Relative paths are under the ComAI install directory. |
| `file_max_bytes` | Maximum bytes read from each file passed with `-f`. |
| `dir_context_max` | Maximum current-directory entries used for context. |
| `error_regex` | Pattern used to find likely problem lines in files. |
| `error_intent_regex` | Pattern used to detect log-check style questions. |

## Compatibility

Older installed configs with flat keys still work. New configs should use the `providers:` sections.

Legacy flat keys include:

```text
local_api_base local_model
ollama_api_base ollama_model
lmstudio_api_base lmstudio_model
openai_api_base gpt_model openai_api_key
```

If both flat keys and provider-section keys exist, the flat key is kept for compatibility.

## Environment Overrides

```bash
COMAI_PROVIDER=ollama comai hi
COMAI_MODEL=qwen2.5-coder:7b comai hi
COMAI_MAX_TOKENS=120 comai explain chmod 755
OPENAI_API_KEY=your_api_key comai gpt hi
COMAI_OPENAI_API_KEY_CMD='pass show openai' comai gpt hi
```

OpenAI key lookup order is `OPENAI_API_KEY`, then `COMAI_OPENAI_API_KEY`, then `providers.openai.api_key_cmd`, then `providers.openai.api_key`.

## Command Overrides

```bash
comai --model=Qwen2.5-7B-Instruct-Q4_K_M hi
comai --max-tokens=120 explain chmod 755
comai --api-base=http://127.0.0.1:11435 hi
```

Useful priority rule:

```text
command option > environment variable > config value > built-in fallback
```

## Edit Config

```bash
comai config path
comai config get provider
comai config edit
comai config set provider ollama
```

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
