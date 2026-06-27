# ComAI Linux Assistant Wiki

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>

<div align="center">

![Linux](https://img.shields.io/badge/Linux-Ubuntu%20%7C%20Fedora-orange)
![Bash](https://img.shields.io/badge/Bash-shell-4EAA25)
![Local AI](https://img.shields.io/badge/Local%20AI-llama.cpp-blue)
![Ollama](https://img.shields.io/badge/Ollama-supported-black)
![OpenAI](https://img.shields.io/badge/OpenAI-compatible-10A37F)
![License](https://img.shields.io/badge/license-MIT-green)

</div>

ComAI Linux Assistant is an open-source Linux terminal assistant written in Bash. It helps you ask Linux questions, explain commands, inspect files, scan logs for errors, and talk to local or cloud AI models without leaving your shell.

## What ComAI Does

| Area | What you can do |
| --- | --- |
| Linux help | Ask command-line and system administration questions. |
| Command explanation | Explain shell commands and flags before you run them. |
| File analysis | Send scripts, configs, notes, and logs with `-f`. |
| Log checks | Quickly find likely errors, warnings, failures, and tracebacks. |
| Local AI | Use a local OpenAI-compatible server from `~/ai`. |
| Ollama | Use local Ollama models with `comai ollama ...`. |
| OpenAI | Use OpenAI or compatible APIs with `comai gpt ...`. |

## First Commands

```bash
comai explain chmod 755
comai how do I find files larger than 1GB?
comai do you see any error? -f application.log
comai ollama summarize this file -f README.md
comai gpt explain this nginx config -f nginx.conf
```

## Main Pages

- [Installation](Installation)
- [Quick Start](Quick-Start)
- [Providers](Providers)
- [Configuration](Configuration)
- [File and Log Analysis](File-and-Log-Analysis)
- [Local AI Service](Local-AI-Service)
- [Troubleshooting](Troubleshooting)
- [Uninstall](Uninstall)
- [FAQ](FAQ)
- [Publishing](Publishing)

## Provider Shortcuts

ComAI chooses a provider from the command:

```bash
comai hi              # default provider from config, usually local
comai ollama hi       # Ollama provider
comai gpt hi          # OpenAI provider
comai chatgpt hi      # OpenAI provider
```

## Project Links

- Project repository: `hossbit/comai-linux-assistant`
- Wiki repository: `hossbit/comai-linux-assistant-wiki`
- Main command: `comai`
- Short alias: `comi`
- Default install directory: `~/localcomai`
- Default local AI directory: `~/ai`
- Installed config: `~/localcomai/config/comai.yaml`
