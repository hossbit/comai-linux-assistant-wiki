# LocalAI Troubleshooting

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

This page covers the standalone [LocalAI](https://github.com/hossbit/local-ai-server)
server itself. If you're debugging ComAI's connection *to* a local provider
(including LocalAI), see [Troubleshooting](Troubleshooting.md#local-provider-is-not-responding) instead.

## First Check

```bash
localai status
localai check
localai models
```

Watch the logs:

```bash
localai logs
# or
tail -n 80 ~/ai/logs/llama-swap.log
```

## `localai` Command Not Found

```bash
ls -l ~/.local/bin/localai
echo "$PATH"
export PATH="$HOME/.local/bin:$PATH"
localai --help
```

## Service Won't Start Or Won't Respond

```bash
localai start
localai status
systemctl --user status localai
```

Manual API check:

```bash
curl -s http://127.0.0.1:11435/v1/models | jq -r .data[].id
```

`localai check --chat` also round-trips a real chat completion against the
first non-embedding model, confirming it actually loads and generates:

```bash
localai check --chat
```

## Configured Model Was Not Found

List installed models:

```bash
localai models
```

Models are discovered from `~/ai/models`. After adding or removing GGUF
files, reload so LocalAI regenerates its config:

```bash
localai reload
localai models
```

## Split GGUF Warning

If `localai check`, `localai models`, or `localai start` says a file looks like
a split GGUF fragment but does not match canonical llama.cpp split naming, the
file was skipped instead of being registered as a broken model.

Use canonical shard names and keep all shards in one directory:

```text
name-00001-of-00003.gguf
name-00002-of-00003.gguf
name-00003-of-00003.gguf
```

For many split models, use one folder per model:

```text
~/ai/models/model-name/
  model-name-00001-of-00003.gguf
  model-name-00002-of-00003.gguf
  model-name-00003-of-00003.gguf
```

Or merge shards into one GGUF:

```bash
llama-gguf-split --merge FIRST_SHARD.gguf OUTPUT.gguf
```

Then rebuild/restart:

```bash
localai reload
localai models
```

Use `localai suggest` to check model size against detected RAM/VRAM. See
[Split GGUF Models](Local-AI-Service.md#split-gguf-models) for the full
walkthrough.

## API Requires A Key You Don't Have

If a client gets `401 Unauthorized`, at least one API key is active on the
server. List and manage keys:

```bash
localai key list
localai key create work-laptop
localai key rotate KEY_ID
localai key revoke KEY_ID
```

`create` and `rotate` print the full secret exactly once — save it
immediately, it cannot be shown again. Revoking the last active key returns
the server to unauthenticated behavior, unless `LOCALAI_REQUIRE_API_KEY=1` is
set in `localai.conf`. See [API Keys](Local-AI-Service.md#api-keys) for the
full walkthrough, or
[Securing LocalAI With API Keys](ComAI-and-LocalAI.md#securing-localai-with-api-keys)
if you're connecting through ComAI.

## Need A Clean Reinstall

```bash
localai uninstall
```

Then install again:

```bash
curl -fsSL https://hossbit.github.io/localai/install.sh | bash
```

If `localai` is not on your `PATH`, run the installed uninstaller directly:

```bash
~/ai/bin/uninstall-local-ai.sh
```

If you are working from a source checkout, reinstall from that checkout with:

```bash
./install-local-ai.sh
```

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/localai-hero.webp" alt="LocalAI local LLM server" width="900">
</div>
