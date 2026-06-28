# FAQ

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

## Is ComAI only for Linux?

ComAI is designed for Linux shells and Linux command-line workflows. The installer and service integration expect common Linux tools and user systemd.

## Is ComAI written in Python or Node?

No. ComAI is written in Bash and uses standard command-line tools such as `curl`, `jq`, `sed`, `awk`, and `grep`.

## Does ComAI require internet access?

Not always. Local mode and Ollama mode can run locally when your models are available on the machine. OpenAI mode requires network access to the configured OpenAI-compatible API.

## Which command should I use, `comai` or `comi`?

Both point to the same tool. Use `comai` for clarity and `comi` as a short alias.

## Where is the config file?

Installed config:

```bash
~/localcomai/config/comai.yaml
```

Source default:

```bash
config/comai.yaml
```

## Can I use Ollama without localai?

Yes. Start Ollama and run:

```bash
comai ollama hi
```

## Can I use OpenAI without localai?

Yes. Set an API key and run:

```bash
export OPENAI_API_KEY="your_api_key"
comai gpt hi
```

## How do I use a different model once?

```bash
comai --model=MODEL_NAME hi
comai ollama --model=qwen2.5-coder:7b hi
comai gpt --model=gpt-5.5 hi
```

## Does ComAI change files it analyzes?

No. Passing a file with `-f` reads it as context. ComAI does not edit that file.

## How do I publish this wiki to GitHub?

For a standalone documentation repository:

```bash
gh repo create hossbit/comai-linux-assistant-wiki --public --source wiki --remote comai-wiki --push
```

For the built-in GitHub Wiki, use the separate wiki git repository:

```bash
git clone https://github.com/hossbit/comai-linux-assistant.wiki.git
```

Copy the files from this local `wiki/` directory into that cloned wiki repo, then commit and push.

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
