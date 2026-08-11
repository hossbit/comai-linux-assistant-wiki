# Providers

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

ComAI supports six provider modes: local OpenAI-compatible APIs, Ollama, LM Studio, OpenAI, Gemini, and OpenRouter.

LocalAI is one optional local provider. ComAI can also use llama.cpp server or any other local API that exposes OpenAI-compatible `/v1/models` and `/v1/chat/completions` endpoints.

## Status And Models

```bash
comai status
comai provider
comai models
```

Target one provider:

```bash
comai status local
comai status ollama
comai status lmstudio
comai status openai
comai status gemini
comai status openrouter
comai models local
comai models ollama
comai models lmstudio
comai models openai
comai models gemini
comai models openrouter
```

`comai models` and `comai status` accept `all` explicitly (`comai models all`) and default to `all` when no provider is given. Cloud provider catalogs, especially OpenRouter, can list hundreds of models — narrow the list with `--filter`:

```bash
comai models openrouter --filter claude
comai models all --filter gemini
```

## Provider Selection

| Command | Provider |
| --- | --- |
| `comai hi` | default provider from config |
| `comai local hi` | local OpenAI-compatible provider |
| `comai --local hi` | local OpenAI-compatible provider |
| `comai ollama hi` | Ollama |
| `comai --ollama hi` | Ollama |
| `comai olm hi` | Ollama |
| `comai --olm hi` | Ollama |
| `comai lmstudio hi` | LM Studio |
| `comai --lmstudio hi` | LM Studio |
| `comai lms hi` | LM Studio |
| `comai --lms hi` | LM Studio |
| `comai gpt hi` | OpenAI |
| `comai chatgpt hi` | OpenAI |
| `comai --gpt hi` | OpenAI |
| `comai gemini hi` | Gemini |
| `comai --gemini hi` | Gemini |
| `comai gem hi` | Gemini |
| `comai --gem hi` | Gemini |
| `comai opr hi` | OpenRouter |
| `comai openrouter hi` | OpenRouter |
| `comai --opr hi` | OpenRouter |
| `COMAI_PROVIDER=lmstudio comai hi` | LM Studio |

## Local Provider

Local mode talks to any OpenAI-compatible local API server.

<div align="center">
  <a href="https://github.com/hossbit/local-ai-server">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/local-ai-server.webp" alt="Local AI Server" width="300">
    <br>
    <strong>hossbit/local-ai-server</strong>
  </a>
  <br>
  OpenAI-compatible Linux local AI backend for the <code>local</code> provider.
</div>

Default:

```yaml
providers:
  local:
    api_base: http://127.0.0.1:11435
    model: Qwen2.5-Coder-7B-Instruct-Q4_K_M
    api_key:
    api_key_cmd:
```

`api_key`/`api_key_cmd` are optional and blank by default — a local server
with no active keys (the default) works completely unauthenticated. Only set
one if your server requires auth (for example after `localai key create`):

```bash
export LOCALAI_API_KEY="your_api_key"
```

Or use a command-based secret lookup:

```bash
comai config set providers.local.api_key_cmd "pass show localai"
```

See [Securing LocalAI With API Keys](ComAI-and-LocalAI.md#securing-localai-with-api-keys) for the full walkthrough, including the 401 error message if you forget.

Examples:

```yaml
# LocalAI
providers:
  local:
    api_base: http://127.0.0.1:11435

# llama.cpp server
providers:
  local:
    api_base: http://127.0.0.1:8080
```

See [ComAI And LocalAI](ComAI-and-LocalAI.md) if you want to use the separate LocalAI project with ComAI.

Local mode sends chat requests to:

```text
${api_base}/v1/chat/completions
```

## Ollama

Start Ollama first, then run:

```bash
comai ollama hi
comai ollama explain chmod 755
comai ollama summarize this file -f README.md
```

`olm` is a short alias for `ollama`: `comai olm hi`.

Default config:

```yaml
providers:
  ollama:
    api_base: http://127.0.0.1:11434
    model: qwen2.5-coder:7b
```

## LM Studio

Start the LM Studio local server:

```bash
lms server start --port 1234
```

Check models:

```bash
curl -s http://127.0.0.1:1234/v1/models | jq -r .data[].id
```

Default config:

```yaml
providers:
  lmstudio:
    api_base: http://127.0.0.1:1234
    model: local-model
```

Run:

```bash
comai lmstudio hi
comai lmstudio --model=qwen/qwen3.5-9b hi
```

`lms` is a short alias for `lmstudio`: `comai lms hi`. This is a ComAI provider alias, unrelated to the `lms` command from LM Studio's own CLI used above to start the server.

Some reasoning models may return `reasoning_content` before normal `content`. If ComAI reports an empty LM Studio response, try a non-reasoning chat model, increase `--max-tokens`, or disable thinking in LM Studio for that model.

## OpenAI

Set your key:

```bash
export OPENAI_API_KEY="your_api_key"
```

Or use a command-based secret lookup:

```bash
comai config set api_key_cmd "pass show openai"
```

That stores:

```yaml
providers:
  openai:
    api_key_cmd: pass show openai
```

You can also set `providers.openai.api_key` in installed config, but environment variables or `api_key_cmd` are safer.

Run:

```bash
comai gpt hi
comai gpt explain chmod 755
comai chatgpt summarize this file -f README.md
```

Default config:

```yaml
providers:
  openai:
    api_base: https://api.openai.com
    model: gpt-4o-mini
    api_key:
    api_key_cmd:
```

Choose one model for one request:

```bash
comai gpt --model=gpt-4o-mini hi
```

Model self-identification is not reliable. Use `comai models openai` and your command/config to know which model was requested.

## Gemini

Set your key:

```bash
export GEMINI_API_KEY="your_api_key"
```

Or use a command-based secret lookup:

```bash
comai config set gemini.api_key_cmd "pass show gemini"
```

That stores:

```yaml
providers:
  gemini:
    api_key_cmd: pass show gemini
```

You can also set `providers.gemini.api_key` in installed config, but environment variables or `api_key_cmd` are safer.

Run:

```bash
comai gemini hi
comai gemini explain chmod 755
comai gemini summarize this file -f README.md
```

`gem` is a short alias for `gemini`: `comai gem hi`.

Default config:

```yaml
providers:
  gemini:
    api_base: https://generativelanguage.googleapis.com
    model: gemini-2.5-flash
    api_key:
    api_key_cmd:
```

Choose one model for one request:

```bash
comai gemini --model=gemini-2.5-pro hi
```

## OpenRouter

OpenRouter is a unified API that routes to many hosted models, including OpenAI, Anthropic, Google, and open-weight models, through one API key.

Set your key:

```bash
export OPENROUTER_API_KEY="your_api_key"
```

Or use a command-based secret lookup:

```bash
comai config set providers.openrouter.api_key_cmd "pass show openrouter"
```

That stores:

```yaml
providers:
  openrouter:
    api_key_cmd: pass show openrouter
```

You can also set `providers.openrouter.api_key` in installed config, but environment variables or `api_key_cmd` are safer.

Run:

```bash
comai opr hi
comai openrouter hi
comai opr explain chmod 755
comai opr summarize this file -f README.md
```

Default config:

```yaml
providers:
  openrouter:
    api_base: https://openrouter.ai/api
    model: openrouter/auto
    api_key:
    api_key_cmd:
```

Choose one model for one request:

```bash
comai opr --model=anthropic/claude-sonnet-4.5 hi
```

The default model, `openrouter/auto`, lets OpenRouter pick a different underlying model for each request. Model self-identification is not reliable, and `auto` makes it worse since the answering model can change between requests. Use `comai models openrouter --filter TEXT` to search the catalog and pin a specific model in config for consistent results.

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.webp" alt="ComAI local AI assistant for Linux" width="900">
</div>
