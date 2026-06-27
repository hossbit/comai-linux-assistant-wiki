# Publishing the Wiki

<div align="center">

![GitHub Wiki](https://img.shields.io/badge/GitHub-Wiki-181717)
![Markdown](https://img.shields.io/badge/Markdown-ready-blue)

</div>

This `wiki/` directory is ready for the `hossbit/comai-linux-assistant` project and can be published in either of two ways:

- as a standalone repository: `hossbit/comai-linux-assistant-wiki`
- as the built-in GitHub Wiki repository: `hossbit/comai-linux-assistant.wiki.git`

The standalone repository is useful when you want normal pull requests, issues, releases, or CI for documentation. The built-in GitHub Wiki is useful when you want GitHub's Wiki tab to render these pages automatically.

## Standalone Wiki Repository

Create and push to:

```bash
gh repo create hossbit/comai-linux-assistant-wiki --public --source wiki --remote comai-wiki --push
```

After that, the wiki repository will contain:

```text
Home.md
README.md
_Sidebar.md
Installation.md
Quick-Start.md
Providers.md
Configuration.md
File-and-Log-Analysis.md
Local-AI-Service.md
Troubleshooting.md
Uninstall.md
FAQ.md
Publishing.md
```

## Built-In GitHub Wiki

On GitHub:

1. Open the repository.
2. Go to `Settings`.
3. Enable `Wikis` if it is disabled.
4. Open the `Wiki` tab once so GitHub creates the wiki.

Clone the built-in wiki repo:

```bash
git clone https://github.com/hossbit/comai-linux-assistant.wiki.git
```

From this project checkout:

```bash
cp wiki/*.md ../comai-linux-assistant.wiki/
```

```bash
cd ../comai-linux-assistant.wiki
git add .
git commit -m "Add ComAI wiki"
git push
```

## Page Names

GitHub Wiki page URLs use the Markdown file names:

```text
Home.md
Quick-Start.md
Local-AI-Service.md
```

The `_Sidebar.md` file becomes the wiki sidebar automatically.

<div align="center">
  <img src="https://raw.githubusercontent.com/hossbit/mirassets/main/images/comai-hero2.png" alt="ComAI local AI assistant for Linux" width="900">
</div>
