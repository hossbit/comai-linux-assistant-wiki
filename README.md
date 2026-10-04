# ComAI documentation hub

**A terminal assistant for Linux questions, commands, files and logs.**

## Choose your next step

| Your goal | Guide |
| --- | --- |
| Start using ComAI | [Quick start](Quick-Start.md) → [Installation](Installation.md) |
| Connect a model | [Providers](Providers.md) → [Configuration](Configuration.md) |
| Inspect a file or log | [File and log analysis](File-and-Log-Analysis.md) |
| Understand a failure | [Health and diagnostics](Health-and-Diagnostics.md) |
| Update or return to a previous version | [Updates and rollback](Updates-and-Rollback.md) |
| Use the optional LocalAI helper | [ComAI and LocalAI](ComAI-and-LocalAI.md) |
| Remove the application | [Uninstall](Uninstall.md) |

## First session

```bash
comai version
comai doctor --json
comai explain chmod 755
comai context --tail-context -f application.log
comai analyze --tail-context -f application.log
```

`context` previews local file and directory input without making a model
request. `analyze` can send that input to your configured provider. Cloud file
context requires confirmation or explicit opt-in.

## Commands at a glance

| Task | Command |
| --- | --- |
| Ask / converse | `comai ask "QUESTION"` / `comai chat` |
| Inspect input before sending | `comai context -f FILE` |
| Explain / analyze | `comai explain COMMAND` / `comai analyze -f FILE` |
| Check active provider and model | `comai doctor --json` |
| Check provider connections / models | `comai status` / `comai models` |
| Configure | `comai setup` / `comai config show` |
| Preview / apply update | `comai update --check` / `comai update` |
| Restore previous application | `comai update --rollback` |

## Reference and support

[FAQ](FAQ.md) · [Troubleshooting](Troubleshooting.md) ·
[LocalAI troubleshooting](Troubleshooting-LocalAI.md) ·
[Local maintainer validation](Local-Validation.md)

[Application repository](https://github.com/hossbit/comai-linux-assistant) ·
[Releases](https://github.com/hossbit/comai-linux-assistant/releases) ·
[Support development](https://buymeacoffee.com/mirhh)

These Markdown pages are maintained in this documentation repository. Start at [Home](Home.md).
