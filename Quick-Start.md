# Quick Start

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

This page gives practical examples you can copy into a terminal.

## Ask Linux Questions

```bash
comai what is /etc in linux?
comai how do I list open ports?
comai how do I find files larger than 1GB?
```

## Explain Commands

```bash
comai explain chmod 755
comai explain "tar -czf backup.tar.gz /etc"
comai how this command work -command "find . -type f -size +100M"
```

## Use a Specific Provider

```bash
comai hi
comai ollama hi
comai gpt hi
```

## Analyze Files

```bash
comai explain this file -f script.sh
comai summarize this config -f nginx.conf
comai gpt review this file -f app.log
comai ollama explain this script -f install.sh
```

## Scan Logs for Problems

```bash
comai do you see any error? -f application.log
comai check this log -f llama-swap.log
comai is this healthy? -f service.log
```

## Pick a Model for One Request

```bash
comai --model=Qwen2.5-7B-Instruct-Q4_K_M hi
comai ollama --model=qwen2.5-coder:7b hi
comai gpt --model=gpt-5.5 hi
```

## Limit Answer Length

```bash
comai --max-tokens=120 explain chmod 755
COMAI_MAX_TOKENS=120 comai explain chmod 755
```

## Use a Custom API Base

```bash
comai --api-base=http://127.0.0.1:11435/v1 hi
comai gpt --api-base=https://api.openai.com/v1 hi
```

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
