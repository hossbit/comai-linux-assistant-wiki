# Local AI Service

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

Local mode expects an OpenAI-compatible API server running from:

```bash
~/ai
```

The recommended companion project is `hossbit/localai`.

If you install LocalAI into a custom directory, pass that path when installing
ComAI:

```bash
./scripts/install.sh --ai-dir ~/myai
```

## Start Local AI

Using the user service:

```bash
systemctl --user start comai-localai.service
```

Manual start:

```bash
~/ai/start.sh
```

For a custom LocalAI directory:

```bash
~/myai/start.sh
```

## Stop Local AI

```bash
systemctl --user stop comai-localai.service
```

## Enable on Login

```bash
systemctl --user enable comai-localai.service
```

## Check Service Status

```bash
systemctl --user status comai-localai.service
```

## Check API Health

```bash
curl -s http://127.0.0.1:11435/v1/models | jq -r '.data[].id'
```

If this command prints no model IDs, add one or more `.gguf` model files to:

```bash
~/ai/models
```

For a custom LocalAI directory, use that directory's `models` folder instead.

## Config Values

```yaml
provider: local
ai_dir: ~/ai
api_base_url: http://127.0.0.1
api_base_port: 11435
model: Qwen2.5-Coder-7B-Instruct-Q4_K_M
```

## Common Error

If you see:

```text
Local AI API is not responding
```

Start the service:

```bash
systemctl --user start comai-localai.service
```

Or use another provider:

```bash
comai ollama hi
comai gpt hi
```

If you see:

```text
LocalAI is running, but the configured model was not found
```

Add the matching `.gguf` file to `~/ai/models`, or edit `model:` in:

```bash
~/localcomai/config/comai.yaml
```

For a custom LocalAI directory, add the model to that directory's `models`
folder.

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
