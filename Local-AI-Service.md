# Local AI Service

<div align="center">

![Service](https://img.shields.io/badge/systemd-user%20service-blue)
![Local](https://img.shields.io/badge/Local%20AI-OpenAI%20compatible-green)

</div>

Local mode expects an OpenAI-compatible API server running from:

```bash
~/ai
```

The recommended companion project is `hossbit/localai`.

## Start Local AI

Using the user service:

```bash
systemctl --user start comai-localai.service
```

Manual start:

```bash
~/ai/start.sh
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

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
