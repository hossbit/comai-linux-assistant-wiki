# Quick Start

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

Practical commands you can copy into a terminal.

## Check Setup

```bash
comai status
comai provider
comai models
```

`status` shows local, Ollama, LM Studio, and OpenAI provider connections. The active provider is marked with `(active)`.

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

## Use Providers

```bash
comai hi             # default provider from config
comai ollama hi      # Ollama
comai lmstudio hi    # LM Studio
comai gpt hi         # OpenAI
```

## Analyze Files

```bash
comai explain this file -f script.sh
comai summarize this config -f nginx.conf
comai gpt review this file -f app.log
comai ollama explain this script -f install.sh
```

## Scan Logs

```bash
comai do you see any error? -f application.log
comai scan this log -f llama-swap.log
comai is this healthy? -f service.log
```

## Pick A Model Once

```bash
comai --model=Qwen2.5-7B-Instruct-Q4_K_M hi
comai ollama --model=qwen2.5-coder:7b hi
comai lmstudio --model=qwen/qwen3.5-9b hi
comai gpt --model=gpt-5.5 hi
```

## Override Request Limits Once

```bash
comai --max-tokens=120 explain chmod 755
comai --api-base=http://127.0.0.1:1234 lmstudio hi
```

## Interactive Chat

```bash
comai chat
```

Type `/exit` or `/quit` to leave chat.

## Logs

ComAI writes service/status events to:

```bash
~/localcomai/logs/comai.log
```

Watch the log:

```bash
tail -f ~/localcomai/logs/comai.log
```

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
