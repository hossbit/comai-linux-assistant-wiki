# Local AI Service

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

Local provider mode can use any OpenAI-compatible local server. The bundled service helper is mainly for the companion LocalAI install under `~/ai`.

## Start, Stop, Restart

Use ComAI commands:

```bash
comai start
comai stop
comai restart
```

When the user service exists, these call:

```bash
systemctl --user start comai-localai.service
systemctl --user stop comai-localai.service
systemctl --user restart comai-localai.service
```

If the service file is not installed, ComAI falls back to:

```bash
~/ai/start.sh
~/ai/stop.sh
```

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
local_api_base: http://127.0.0.1:11435
local_model: Qwen2.5-Coder-7B-Instruct-Q4_K_M
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

Install ComAI with:

```bash
./scripts/install.sh --ai-dir ~/myai
```

Then the helper service uses:

```bash
~/myai/start.sh
~/myai/stop.sh
```
