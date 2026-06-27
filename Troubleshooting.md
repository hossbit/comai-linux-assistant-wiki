# Troubleshooting

<div align="center">

![Help](https://img.shields.io/badge/Troubleshooting-common%20fixes-blue)
![Logs](https://img.shields.io/badge/Logs-check%20first-red)

</div>

## `comai` Command Not Found

Check that the wrapper exists:

```bash
ls -l ~/.local/bin/comai ~/.local/bin/comi
```

Check your `PATH`:

```bash
echo "$PATH"
```

Temporary fix:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Then retry:

```bash
comai --help
```

## Local AI API Is Not Responding

Start local AI:

```bash
systemctl --user start comai-localai.service
```

Or:

```bash
~/ai/start.sh
```

Check models:

```bash
curl -s http://127.0.0.1:11435/v1/models | jq -r '.data[].id'
```

## Ollama Cannot Be Reached

Make sure Ollama is running, then check:

```bash
curl -s http://127.0.0.1:11434/api/tags | jq -r '.models[].name'
```

Confirm config:

```yaml
ollama_api_base: http://127.0.0.1:11434
```

## OpenAI Says `429`

OpenAI rejected the request because of rate limit, quota, billing, project, or credit limits. Check your OpenAI account and project settings.

## OpenAI Works Without Exporting a Key

ComAI may be reading the key from:

```bash
~/localcomai/config/comai.yaml
```

Look for:

```yaml
openai_api_key:
```

For better safety, keep the config key empty and use:

```bash
export OPENAI_API_KEY="your_api_key"
```

## File Is Too Large

ComAI reads up to `file_max_bytes` from each file.

Edit:

```bash
~/localcomai/config/comai.yaml
```

Set:

```yaml
file_max_bytes: 24000
```

Increase it only if your model and context window can handle larger input.

## Installer Cannot Replace Existing Command

If `~/.local/bin/comai` exists and is not managed by ComAI, the installer leaves it alone. Rename or remove your existing command manually if you want ComAI to use that name.

## Need a Clean Reinstall

Uninstall:

```bash
~/localcomai/scripts/uninstall.sh
```

Then install again:

```bash
./scripts/install.sh
```

