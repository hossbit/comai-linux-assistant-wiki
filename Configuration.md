# Configuration

<div align="center">

![YAML](https://img.shields.io/badge/Config-YAML-blue)
![Env](https://img.shields.io/badge/Overrides-environment-green)

</div>

ComAI reads defaults from its YAML config and allows one-off overrides from command flags or environment variables.

## Config Files

Source default:

```bash
config/comai.yaml
```

Installed config:

```bash
~/localcomai/config/comai.yaml
```

## Example Config

```yaml
provider: local
ai_dir: ~/ai
api_base_url: http://127.0.0.1
api_base_port: 11435
model: Qwen2.5-Coder-7B-Instruct-Q4_K_M
gpt_model: gpt-5.5
ollama_api_base: http://127.0.0.1:11434
ollama_model: qwen2.5-coder:7b
openai_api_base: https://api.openai.com
openai_api_key:
max_tokens: 420
timeout: 120
file_max_bytes: 24000
dir_context_max: 120
error_regex: error|errors|failed|failure|exception|fatal|panic|timeout|warn|warning|traceback
error_intent_regex: error|errors|failed|failure|warning|warnings|problem|problems|issue|issues|wrong|bad|broken|fail|crash|crashed|panic|timeout|traceback|healthy|health|(^|[[:space:]])ok([[:space:]]|$)|okay|check (this )?log|scan (this )?log
```

## Key Reference

| Key | Purpose |
| --- | --- |
| `provider` | Default provider: `local`, `ollama`, or `openai`. |
| `ai_dir` | Directory where localai is installed. |
| `api_base_url` | Local OpenAI-compatible API URL without the port. |
| `api_base_port` | Local OpenAI-compatible API port. |
| `model` | Default local model. |
| `gpt_model` | Default OpenAI model. |
| `ollama_api_base` | Ollama API URL. |
| `ollama_model` | Default Ollama model. |
| `openai_api_base` | OpenAI or compatible API base. |
| `openai_api_key` | Optional API key in config. Environment variable is safer. |
| `max_tokens` | Maximum requested answer length. |
| `timeout` | Request timeout in seconds. |
| `file_max_bytes` | Maximum bytes read from each file passed with `-f`. |
| `dir_context_max` | Maximum current-directory entries used for context. |
| `error_regex` | Pattern used to find likely problem lines in files. |
| `error_intent_regex` | Pattern used to detect log-check style questions. |

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
comai --api-base=http://127.0.0.1:11435/v1 hi
```

## API Key Safety

Prefer:

```bash
export OPENAI_API_KEY="your_api_key"
```

Avoid committing a real key into git. If you store a key in `~/localcomai/config/comai.yaml`, keep that installed config private.

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
