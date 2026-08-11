# Uninstall

<div align="center">
  <a href="https://buymeacoffee.com/mirhh">
    <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/support.gif" alt="Buy me a coffee" width="300">
  </a>
</div>

ComAI includes an uninstaller that removes ComAI-managed user files and leaves unrelated paths untouched.

## Run Uninstaller

Recommended:

```bash
comai uninstall
```

If `comai` is not on your `PATH`, run the installed uninstaller directly:

```bash
~/localcomai/scripts/uninstall.sh
```

If you installed to a custom directory:

```bash
~/aiass/scripts/uninstall.sh
```

Environment variables are also supported:

```bash
COMAI_INSTALL_DIR="$HOME/apps/comai" "$HOME/apps/comai/scripts/uninstall.sh"
```

## What It Removes

The uninstaller removes items only when they look managed by ComAI:

- `~/.local/bin/comai`
- `~/.local/bin/comi`
- `~/.config/systemd/user/comai-localai.service`
- the installed ComAI app directory, usually `~/localcomai`

## What It Leaves Untouched

The uninstaller does not remove:

- the source checkout
- your LocalAI install, usually `~/ai`
- files that do not look managed by ComAI
- command symlinks that point somewhere else

## Verify Removal

```bash
command -v comai
command -v comi
ls ~/.local/bin/comai ~/.local/bin/comi
```

If the commands are gone, `command -v` returns no path.

## LocalAI Is Separate

ComAI and LocalAI are separate pieces. Removing ComAI does not delete:

```bash
~/ai
```

If your LocalAI install is somewhere else, the uninstaller reports that path in
its final summary. This is intentional so your local models and local AI setup
are preserved.

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.webp" alt="ComAI local AI assistant for Linux" width="900">
</div>
