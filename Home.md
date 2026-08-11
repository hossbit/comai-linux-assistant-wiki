# ComAI Linux Assistant Wiki

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.webp" alt="ComAI local AI assistant for Linux" width="900">
</div>

ComAI is a Bash-powered AI assistant for Linux terminals. It helps you ask Linux questions, explain commands, inspect files, scan logs, and use local AI providers, Ollama, LM Studio, or OpenAI from one command.

ComAI is the client. LocalAI is one optional backend. See [ComAI And LocalAI](ComAI-and-LocalAI.md) for install order and how the two separate projects fit together. LocalAI-specific install, Fedora, model, and update notes live in [Local AI Service](Local-AI-Service.md).

## ❤️ Support

If you find ComAI useful, you can support its continued development:

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/bmc-button.webp" alt="Buy me a coffee" width="300">
  </a>
</div>

⭐ Starring and sharing the repository also helps a lot.

## Start Here

```bash
comai status
comai explain chmod 755
comai do you see any error? -f application.log
comai ollama hi
comai lmstudio hi
comai gpt hi
comai opr hi
```

## What ComAI Does

| Area | What you can do |
| --- | --- |
| Linux help | Ask command-line and system administration questions. |
| Command explanation | Explain commands and flags before running them. |
| File analysis | Send scripts, configs, notes, and logs with `-f`. |
| Log checks | Find likely errors, warnings, failures, and tracebacks. |
| Local provider | Use any OpenAI-compatible local server. |
| Ollama | Use local Ollama models with `comai ollama ...`. |
| LM Studio | Use LM Studio with `comai lmstudio ...`. |
| OpenAI | Use OpenAI models with `comai gpt ...`. |
| Gemini | Use Gemini models with `comai gemini ...`. |
| OpenRouter | Use OpenRouter models with `comai opr ...`. |

## Main Pages

- [Quick Start](Quick-Start.md)
- [Installation](Installation.md)
- [Providers](Providers.md)
- [Configuration](Configuration.md)
- [ComAI And LocalAI](ComAI-and-LocalAI.md)
- [Local AI Service](Local-AI-Service.md)
- [File and Log Analysis](File-and-Log-Analysis.md)
- [Troubleshooting](Troubleshooting.md)
- [LocalAI Troubleshooting](Troubleshooting-LocalAI.md)
- [Uninstall](Uninstall.md)
- [FAQ](FAQ.md)

## Main Commands

```bash
comai setup       # Configure provider, API, and model
comai ask         # Ask one question
comai chat        # Start an interactive conversation
comai explain     # Explain a command, error, or output
comai analyze     # Analyze logs, files, or piped output
comai status      # Show provider status and connections
comai provider    # Show active and available providers
comai models      # List models from all providers
comai config      # View, get, or edit settings
comai history     # Show previous conversations
comai start       # Start the optional LocalAI helper service
comai stop        # Stop the optional LocalAI helper service
comai restart     # Restart the optional LocalAI helper service
comai update      # Update ComAI
comai version     # Show installed version
comai uninstall   # Remove ComAI
```

## Project Links

- Project repository: `hossbit/comai-linux-assistant`
- Wiki repository: `hossbit/comai-linux-assistant-wiki`
- Main command: `comai`
- Short alias: `comi`
- Default install directory: `~/localcomai`
- Command symlinks: `~/.local/bin/comai` and `~/.local/bin/comi`
- Installed config: `~/localcomai/config/comai.yaml`
- Service/status log: `~/localcomai/logs/comai.log`

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.webp" alt="ComAI local AI assistant for Linux" width="900">
</div>
