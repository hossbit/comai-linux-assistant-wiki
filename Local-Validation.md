# Local maintainer validation

[Documentation home](Home.md)

Test suites, fixtures and validation runners are maintained only on the
maintainer workstation. They are ignored by Git and are not included in fresh
clones or future release source archives. GitHub does not run a published test
workflow for these projects.

If your workstation already has the local suites:

```bash
bash scripts/test-local.sh
bash scripts/test-parity.sh /path/to/other/edition
```

Checks cover Bash/ShellCheck, provider/privacy contracts, aggregate input
budgets and local Git update/rollback scenarios. ComAI also has local Bats CLI
tests; ComAIX has local RAG tests and Python lint/type checks. These commands
are maintainer checks, not installation requirements.
