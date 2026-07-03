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

`comai` and `comi` should be symlinks to the installed ComAI command.

## Local Provider Is Not Responding

If you use the separate LocalAI project through ComAI's optional helper, start the helper:

```bash
comai start
```

Or check your OpenAI-compatible local server manually:

```bash
curl -s http://127.0.0.1:11435/v1/models | jq -r .data[].id
```

Confirm config:

```yaml
providers:
  local:
    api_base: http://127.0.0.1:11435
    model: Qwen2.5-Coder-7B-Instruct-Q4_K_M
```

## Configured Local Model Was Not Found

List local models:

```bash
comai models local
```

Set `providers.local.model` to one of those model IDs, or add the matching model to your local provider.

## Ollama Cannot Be Reached

```bash
comai status ollama
curl -s http://127.0.0.1:11434/api/tags | jq -r .models[].name
```

Confirm config:

```yaml
providers:
  ollama:
    api_base: http://127.0.0.1:11434
    model: qwen2.5-coder:7b
```

## LM Studio Cannot Be Reached

Start the server:

```bash
lms server start --port 1234
```

Check it:

```bash
comai status lmstudio
curl -s http://127.0.0.1:1234/v1/models | jq -r .data[].id
```

Confirm config:

```yaml
providers:
  lmstudio:
    api_base: http://127.0.0.1:1234
    model: local-model
```

If ComAI reports an empty LM Studio response with a reasoning model, try a non-reasoning chat model, increase `--max-tokens`, or disable thinking in LM Studio for that model.

## OpenAI API Key Missing

Set:

```bash
export OPENAI_API_KEY="your_api_key"
```

Or edit:

```bash
~/localcomai/config/comai.yaml
```

ComAI masks `providers.openai.api_key` and legacy `openai_api_key` when you run:

```bash
comai config show
```

The installed config is also tightened to mode `600`. For better secret handling, use `OPENAI_API_KEY` or `providers.openai.api_key_cmd`, for example:

```yaml
providers:
  openai:
    api_key_cmd: pass show openai
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
comai uninstall
```

Then install again:

```bash
curl -fsSL https://hossbit.github.io/comai/install.sh | bash
```

If `comai` is not on your `PATH`, run the installed uninstaller directly:

```bash
~/localcomai/scripts/uninstall.sh
```

If you are working from a source checkout, reinstall from that checkout with:

```bash
./scripts/install.sh
```

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
