# Providers

<div align="center">

![Local AI](https://img.shields.io/badge/Provider-local%20AI-blue)
![Ollama](https://img.shields.io/badge/Provider-Ollama-black)
![OpenAI](https://img.shields.io/badge/Provider-OpenAI-10A37F)

</div>

ComAI supports three provider modes: local AI, Ollama, and OpenAI-compatible APIs.

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

## Local AI

Local mode is the default. It expects a local OpenAI-compatible API server from `hossbit/localai` under:

```bash
~/ai
```

Start the user service:

```bash
systemctl --user start comai-localai.service
```

Or start it manually:

```bash
~/ai/start.sh
```

Check local models:

```bash
curl -s http://127.0.0.1:11435/v1/models | jq -r '.data[].id'
```

Local mode uses the `model:` value from `~/localcomai/config/comai.yaml`.
That model must exist as a `.gguf` file in `~/ai/models`.

Run a local request:

```bash
comai explain chmod 755
```

## Ollama

Start Ollama first, then run:

```bash
comai ollama hi
comai ollama explain chmod 755
comai ollama summarize this file -f README.md
```

Check installed Ollama models:

```bash
curl -s http://127.0.0.1:11434/api/tags | jq -r '.models[].name'
```

Choose one model for a request:

```bash
comai ollama --model=qwen2.5-coder:7b hi
```

## OpenAI / ChatGPT

Set your API key:

```bash
export OPENAI_API_KEY="your_api_key"
```

Run:

```bash
comai gpt hi
comai gpt explain chmod 755
comai chatgpt summarize this file -f README.md
```

Choose one model for a request:

```bash
comai gpt --model=gpt-5.5 hi
```

## Compatible APIs

ComAI can also point to OpenAI-compatible APIs by changing the API base:

```bash
comai --api-base=http://127.0.0.1:11435/v1 hi
```

For persistent settings, edit:

```bash
~/localcomai/config/comai.yaml
```

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
