# Local AI Service

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

Local provider mode can use any OpenAI-compatible local server. The optional service helper is mainly for the separate LocalAI project when it is installed under the configured `ai_dir`.

ComAI does not require LocalAI. If you use Ollama, LM Studio, llama.cpp server, or another compatible local API, you can ignore the helper service and configure the provider's `api_base` instead.

## Start, Stop, Restart

Use ComAI commands:

```bash
comai start
comai stop
comai restart
```

The ComAI installer creates `comai-localai.service` but does not start it automatically. Start it only after LocalAI is installed and the configured `ai_dir` contains `bin/start.sh` and `bin/stop.sh`.

```bash
systemctl --user enable --now comai-localai.service
```

When the user service exists, these call:

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

## Check Status

```bash
comai status local
comai provider
comai models local
```

Manual API check:

```bash
curl -s http://127.0.0.1:11435/v1/models | jq -r .data[].id
```

## Config

Normal local provider config:

```yaml
providers:
  local:
    api_base: http://127.0.0.1:11435
    model: Qwen2.5-Coder-7B-Instruct-Q4_K_M
```

Optional LocalAI helper directory:

```yaml
ai_dir: ~/ai
```

`ai_dir` only controls the start/stop helper. It does not choose the API endpoint or model for normal local requests.

## Logs

Service and provider events are written under the ComAI install directory:

```bash
~/localcomai/logs/comai.log
```

Watch logs:

```bash
tail -f ~/localcomai/logs/comai.log
```

## Custom LocalAI Directory

Install ComAI from a source checkout with:

```bash
./scripts/install.sh --ai-dir ~/myai
```

Then the helper service uses:

```bash
~/myai/bin/start.sh
~/myai/bin/stop.sh
```

For the full two-project explanation, see [ComAI And LocalAI](ComAI-and-LocalAI.md).

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
