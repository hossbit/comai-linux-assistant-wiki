# File and Log Analysis

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

ComAI can include file content in the prompt with `-f`, `--file`, or `--files`.

## Analyze Files

```bash
comai explain this file -f script.sh
comai summarize this config -f nginx.conf
comai review this markdown -f README.md
```

Provider-specific examples:

```bash
comai ollama summarize this file -f README.md
comai gpt explain this nginx configuration -f nginx.conf
```

## Multiple Files

```bash
comai compare these files -f old.conf -f new.conf
comai gpt review these scripts --file install.sh --file uninstall.sh
```

## Mentioned Files

ComAI also tries to detect file names mentioned in your request:

```bash
comai explain README.md
comai explain scripts/install.sh
```

Using `-f` is still the clearest option when accuracy matters.

## Scan Logs

For local provider requests, ComAI can answer simple log and file checks before calling an AI model.

```bash
comai do you see any error? -f application.log
comai scan this log -f llama-swap.log
comai is this healthy? -f service.log
```

By default, ComAI looks for words such as:

```text
error errors failed failure exception fatal panic timeout warn warning traceback
```

## ComAI Service Logs

ComAI writes its own service/status log here:

```bash
~/localcomai/logs/comai.log
```

This log stores provider status, service start/stop events, request status, selected provider/model, and request/response sizes. It does not write API keys.

Watch it:

```bash
tail -f ~/localcomai/logs/comai.log
```

## File Size Limit

```yaml
file_max_bytes: 24000
```

Edit this in:

```bash
~/localcomai/config/comai.yaml
```

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
