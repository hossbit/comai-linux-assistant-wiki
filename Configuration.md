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

`openai_api_key` is masked when shown.

## Current Config Example

```yaml
provider: local
local_api_base: http://127.0.0.1:11435
local_model: Qwen2.5-Coder-7B-Instruct-Q4_K_M
gpt_model: gpt-5.5
ollama_api_base: http://127.0.0.1:11434
ollama_model: qwen2.5-coder:7b
openai_api_base: https://api.openai.com
openai_api_key:
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
| `provider` | Default provider: `local`, `ollama`, or `openai`. |
| `local_api_base` | OpenAI-compatible local API base. |
| `local_model` | Default local model. |
| `gpt_model` | Default OpenAI model. |
| `ollama_api_base` | Ollama API base. |
| `ollama_model` | Default Ollama model. |
| `openai_api_base` | OpenAI or compatible API base. |
| `openai_api_key` | Optional API key in config. Environment variable is safer. |
| `max_tokens` | Maximum requested answer length. |
| `timeout` | Request timeout in seconds. |
| `log_file` | Service/status log path. Relative paths are under the ComAI install directory. |
| `file_max_bytes` | Maximum bytes read from each file passed with `-f`. |
| `dir_context_max` | Maximum current-directory entries used for context. |
| `error_regex` | Pattern used to find likely problem lines in files. |
| `error_intent_regex` | Pattern used to detect log-check style questions. |

## Compatibility

Older installed configs still work, but new configs should use `local_api_base` and `local_model` for local provider requests.

`ai_dir` is only for the optional LocalAI start/stop helper service.

## Environment Overrides

```bash
COMAI_PROVIDER=ollama comai hi
COMAI_MODEL=qwen2.5-coder:7b comai hi
COMAI_MAX_TOKENS=120 comai explain chmod 755
OPENAI_API_KEY=your_api_key comai gpt hi
```

## Command Overrides

```bash
comai --model=Qwen2.5-7B-Instruct-Q4_K_M hi
comai --max-tokens=120 explain chmod 755
comai --api-base=http://127.0.0.1:11435 hi
```

## Edit Config

```bash
comai config path
comai config edit
comai config set provider ollama
```
