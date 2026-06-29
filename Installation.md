# Installation

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

ComAI installs as a user-space command-line tool. The default app directory is `~/localcomai`, and command wrappers are created in `~/.local/bin`.

## Requirements

Required commands:

```text
bash curl jq find sort head sed awk grep wc tr readlink date systemctl
```

Optional commands:

```text
file numfmt git
```

## Install ComAI

```bash
git clone https://github.com/hossbit/comai-linux-assistant.git
cd comai-linux-assistant
chmod +x scripts/install.sh
./scripts/install.sh
```

The installer:

- asks where to install ComAI
- uses `~/localcomai` by default
- creates `comai` and `comi` command wrappers
- creates or updates `~/localcomai/config/comai.yaml`
- creates `~/localcomai/logs/`
- preserves existing config values when upgrading
- installs the optional LocalAI user service helper

## Custom Install Directory

```bash
./scripts/install.sh --dir ~/aiass
```

If the bundled LocalAI helper should control a LocalAI install outside `~/ai`, pass:

```bash
./scripts/install.sh --dir ~/aiass --ai-dir ~/myai
```

`--ai-dir` is only for the optional LocalAI helper service. Normal local provider requests use `local_api_base` in config.

## PATH Setup

ComAI command wrappers are installed in:

```bash
~/.local/bin
```

If needed:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

## Verify Install

```bash
comai --help
comai status
comai explain chmod 755
```

If local AI is not running yet, try another provider:

```bash
comai ollama hi
comai gpt hi
```

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
