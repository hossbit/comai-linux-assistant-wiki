# Installation

<div align="center">

![Linux](https://img.shields.io/badge/Linux-supported-orange)
![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25)
![Install](https://img.shields.io/badge/Install-user%20space-blue)

</div>

ComAI installs as a user-space command-line tool. The default app directory is `~/localcomai`, and command wrappers are created in `~/.local/bin`.

## Requirements

Required commands:

```text
bash curl jq find sort head sed awk grep wc tr readlink systemctl
```

Optional commands:

```text
file numfmt
```

## Install ComAI

Clone the project and run the installer:

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
- preserves existing config values when upgrading
- installs a user systemd service file for local AI startup support

## Custom Install Directory

Use `COMAI_INSTALL_DIR` for a non-default location:

```bash
COMAI_INSTALL_DIR="$HOME/apps/comai" ./scripts/install.sh
```

## PATH Setup

ComAI command wrappers are installed in:

```bash
~/.local/bin
```

If that directory is not in your shell `PATH`, the installer adds it to your shell config. Restart your terminal, or run:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

## Verify Install

```bash
comai --help
comi --help
```

Try a quick question:

```bash
comai explain chmod 755
```

If local AI is not running yet, use Ollama or OpenAI mode first:

```bash
comai ollama hi
comai gpt hi
```

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
