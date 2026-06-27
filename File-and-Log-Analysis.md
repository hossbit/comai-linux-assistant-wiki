# File and Log Analysis

<div align="center">

![Files](https://img.shields.io/badge/Files--f%20supported-blue)
![Logs](https://img.shields.io/badge/Logs-error%20scan-red)
![Config](https://img.shields.io/badge/Configs-review-green)

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
comai check scripts/install.sh
```

Using `-f` is still the clearest option when accuracy matters.

## Scan Logs

For local provider requests, ComAI can answer simple log and file checks before calling an AI model.

```bash
comai do you see any error? -f application.log
comai check this log -f llama-swap.log
comai is this healthy? -f service.log
```

By default, ComAI looks for words such as:

```text
error errors failed failure exception fatal panic timeout warn warning traceback
```

## File Size Limit

The default per-file read limit is controlled by:

```yaml
file_max_bytes: 24000
```

You can edit this in:

```bash
~/localcomai/config/comai.yaml
```

## Practical Recipes

Explain a shell script:

```bash
comai explain this script step by step -f scripts/install.sh
```

Review a service log:

```bash
comai what is failing here? -f service.log
```

Summarize a config:

```bash
comai gpt summarize the important settings -f config/comai.yaml
```

Compare behavior:

```bash
comai compare installer and uninstaller behavior -f scripts/install.sh -f scripts/uninstall.sh
```

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
