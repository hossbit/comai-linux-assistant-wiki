# Uninstall

<div align="center">

![Uninstall](https://img.shields.io/badge/Uninstall-safe%20checks-red)
![User Files](https://img.shields.io/badge/User%20space-only-blue)

</div>

ComAI includes an uninstaller that removes ComAI-managed user files and leaves unrelated paths untouched.

## Run Uninstaller

```bash
~/localcomai/scripts/uninstall.sh
```

If you installed to a custom directory:

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
- `~/ai`
- files that do not look managed by ComAI
- command wrappers that point somewhere else

## Verify Removal

```bash
command -v comai
command -v comi
ls ~/.local/bin/comai ~/.local/bin/comi
```

If the commands are gone, `command -v` returns no path.

## Local AI Is Separate

ComAI and localai are separate pieces. Removing ComAI does not delete:

```bash
~/ai
```

This is intentional so your local models and local AI setup are preserved.

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
