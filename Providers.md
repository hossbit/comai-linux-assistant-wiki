# Providers

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

ComAI supports three provider modes: local OpenAI-compatible APIs, Ollama, and OpenAI.

LocalAI is one optional local provider. ComAI can also use LM Studio,
llama.cpp server, or any other local API that exposes OpenAI-compatible
`/v1/models` and `/v1/chat/completions` endpoints.

## Status And Models

```bash
comai check
comai status
comai provider
comai models
```

Target one provider:

```bash
comai status local
comai status ollama
comai status openai
comai models local
comai models ollama
comai models openai
```

## Provider Selection

| Command | Provider |
| --- | --- |
| `comai hi` | default provider from config |
| `comai ollama hi` | Ollama |
| `comai --ollama hi` | Ollama |
| `comai gpt hi` | OpenAI |
| `comai chatgpt hi` | OpenAI |
| `comai --gpt hi` | OpenAI |
| `COMAI_PROVIDER=ollama comai hi` | Ollama |

## Local Provider

Local mode talks to any OpenAI-compatible local API server.

Default:

```yaml
local_api_base: http://127.0.0.1:11435
local_model: Qwen2.5-Coder-7B-Instruct-Q4_K_M
```

Examples:

```yaml
# LocalAI
local_api_base: http://127.0.0.1:11435

# llama.cpp server
local_api_base: http://127.0.0.1:8080

# LM Studio
local_api_base: http://127.0.0.1:1234
```

See [ComAI And LocalAI](ComAI-and-LocalAI) if you want to use the separate
LocalAI project with ComAI.

Local mode sends chat requests to:

```text
${local_api_base}/v1/chat/completions
```

## Ollama

Start Ollama first, then run:

```bash
comai ollama hi
comai ollama explain chmod 755
comai ollama summarize this file -f README.md
```

Default config:

```yaml
ollama_api_base: http://127.0.0.1:11434
ollama_model: qwen2.5-coder:7b
```

## OpenAI

Set your key:

```bash
export OPENAI_API_KEY="your_api_key"
```

Or set `openai_api_key` in installed config.

Run:

```bash
comai gpt hi
comai gpt explain chmod 755
comai chatgpt summarize this file -f README.md
```

Default config:

```yaml
openai_api_base: https://api.openai.com
gpt_model: gpt-5.5
```

Choose one model for one request:

```bash
comai gpt --model=gpt-5.1-chat-latest hi
```

Model self-identification is not reliable. Use `comai models openai` and your command/config to know which model was requested.

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
