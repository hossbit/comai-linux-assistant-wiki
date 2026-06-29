# Troubleshooting

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

## First Check

```bash
comai status
comai provider
comai models
```

Also check the service/status log:

```bash
tail -n 80 ~/localcomai/logs/comai.log
```

## `comai` Command Not Found

```bash
ls -l ~/.local/bin/comai ~/.local/bin/comi
echo "$PATH"
export PATH="$HOME/.local/bin:$PATH"
comai --help
```

## Local Provider Is Not Responding

Start the bundled helper if you use LocalAI:

```bash
comai start
```

Or check your OpenAI-compatible local server manually:

```bash
curl -s http://127.0.0.1:11435/v1/models | jq -r .data[].id
```

Confirm config:

```yaml
local_api_base: http://127.0.0.1:11435
local_model: Qwen2.5-Coder-7B-Instruct-Q4_K_M
```

## Configured Local Model Was Not Found

List local models:

```bash
comai models local
```

Set `local_model` to one of those model IDs, or add the matching model to your local provider.

## Ollama Cannot Be Reached

```bash
comai status ollama
curl -s http://127.0.0.1:11434/api/tags | jq -r .models[].name
```

Confirm config:

```yaml
ollama_api_base: http://127.0.0.1:11434
ollama_model: qwen2.5-coder:7b
```

## OpenAI API Key Missing

Set:

```bash
export OPENAI_API_KEY="your_api_key"
```

Or edit:

```bash
~/localcomai/config/comai.yaml
```

ComAI masks `openai_api_key` when you run:

```bash
comai config show
```

## OpenAI Says `429`

OpenAI rejected the request because of rate limit, quota, billing, project, or credit limits. Check your OpenAI account and project settings.

## File Is Too Large

ComAI reads up to `file_max_bytes` from each file.

```yaml
file_max_bytes: 24000
```

Increase it only if your model and context window can handle larger input.

## Need A Clean Reinstall

```bash
~/localcomai/scripts/uninstall.sh
```

Then install again:

```bash
./scripts/install.sh
```
