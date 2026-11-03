# Local vLLM guide

Run local AI models through a Linux server using vLLM for text and, optionally,
vLLM-Omni for supported multimodal models. This guide covers the usual path;
[advanced setup and the full reference](Local-vLLM-Advanced.md) cover special
hardware and configuration.

## Requirements

- Linux on x86-64 or ARM64. Ubuntu/Debian and Fedora/RHEL-style package managers
  are supported by the installer; actual wheel support depends on the upstream
  version, architecture, distribution, GPU, and driver.
- A supported GPU with a working driver and enough VRAM for your model. NVIDIA
  is the simplest tested path; [AMD and CPU](#amd-and-cpu-installation) need
  additional compatibility checks.
- Enough disk space for packages and model weights. An install can download
  several gigabytes. The installer may request `sudo` for system packages.

The installer chooses a compatible Python for new environments. If none is
available, it downloads an isolated Python 3.12 using uv inside `~/ai-vllm`;
it does not replace your system Python. Existing environments are preserved
unless you explicitly install, repair, update, or uninstall that backend.

## Install

Run the installer:

```bash
curl -fsSL https://hossbit.github.io/localvllm/install.sh | bash
```

Open a new terminal, then follow the prompts:

```bash
localvllm
```

The menu lists your models and lets you download a model, add a folder, start
or stop serving, and open the web interface. Choose **b** to manage vLLM or
Omni; both have the same workflow and separate model lists. If an engine is
missing, the menu offers to install it. Choose a compatible model that fits
your hardware. `localvllm menu` and `localvllm setup` open the same menu.

The default installation directory is `~/ai-vllm`. If `localvllm` is not found,
open a new terminal or add `~/.local/bin` to your `PATH`.

### AMD and CPU installation

The manager detects NVIDIA and AMD hardware, but cannot safely choose the
correct ROCm or CPU wheel for every device. A working AMD ROCm stack and a
matching vLLM wheel are required; the installer stops with a guide if no ROCm
wheel is configured rather than downloading a CUDA wheel. CPU inference is experimental in this project;
it may require a CPU-specific wheel or an upstream source build. The installer
will not install GPU drivers. For GGUF models or a simpler CPU option, see
[Local AI Server](https://github.com/hossbit/local-ai-server).

Follow the [AMD and CPU compatibility steps](Local-vLLM-Advanced.md#amd-and-cpu-installation)
when needed. For ARM64 and Fedora/RHEL, wheel and driver availability still
depend on your specific configuration; see [platform notes](#platform-notes).

### Platform notes

- **ARM64:** vLLM publishes some ARM64 builds and vLLM-Omni provides ARM64 CUDA
  container images, but not every combination has a pip wheel. This manager
  does not install the container images. Verify your model, GPU, and build in
  the [vLLM hardware guide](https://docs.vllm.ai/en/latest/getting_started/installation/).
- **Fedora/RHEL:** the installer supports `dnf` and `yum` for system packages.
  The installed Python environment still needs a compatible upstream wheel and
  driver. Support varies by distro release, glibc, architecture and GPU.
- **AMD:** check the [upstream ROCm instructions](https://docs.vllm.ai/en/stable/getting_started/installation/gpu/)
  and the [Omni platform guide](https://docs.vllm.ai/projects/vllm-omni/en/latest/getting_started/installation/gpu/)
  for the version you are installing. This manager cannot guarantee matching
  wheels across all ROCm stacks.

These platform combinations have not all been tested by this project.

## Serving modes

**Direct is the default.** Your client connects straight to vLLM or Omni,
without a proxy. Select a model in the menu to switch and start it.

**Automatic switching is optional.** llama-swap lists all registered models on
the same API address and starts the requested model on demand. Each backend
loads one model at a time. The menu's mode option handles enabling it, or use:

```bash
localvllm mode swap
localvllm --backend omni mode swap
```

The first enable downloads an official llama-swap release and verifies
its SHA256 checksum. Direct installs do not download it. If a server is running,
changing mode stops the old service and starts the new one on the same port;
a failed change restores the previous mode. Models, tuning and API keys remain
available in both modes. If the server was stopped, start it from the menu.

Return to direct mode:

```bash
localvllm mode direct
localvllm --backend omni mode direct
```

Use `localvllm mode` to see the selected backend's mode. Each backend has its own
mode, port and service. If both use the same GPU, budget memory across both;
independent backends do not coordinate GPU reservations.

### Which model is loaded?

```bash
localvllm models
localvllm --backend omni models
```

In automatic mode, the **STATE** column reports the selected backend's live
router state: `loaded`, `loading`, `unloading`, or `unloaded`. Requests can switch
models, so run the command again to refresh. The saved **Selection for direct
mode** is only the model to use when returning to direct mode. If the router
cannot be queried, the state is `unavailable`; it never substitutes that saved
selection. A stopped service has no loaded models.

In direct mode, **SELECTED** marks the manually chosen model. `version` and
`status --json` distinguish this saved choice from models reported by the running
API. A stopped service has no active models. An unreachable API reports its
runtime state as unavailable. The menu shows live model states in automatic mode.

For scripts, `localvllm models --json` includes `mode`, each model's `state`,
`direct_model`, `active_models`, and `runtime_state_available`. In automatic mode,
`active_model` is populated only when exactly one model is loaded; use
`active_models` for the full list. Direct mode also uses API-reported models for
these fields; `direct_model` always preserves your saved selection.

### Web interface

In automatic mode, open llama-swap's model dashboard:

```bash
localvllm ui --open
localvllm --backend omni ui --open
```

The menu's **w** action opens the same page. On a headless server, `localvllm ui`
prints the URL. You can explicitly request it with `localvllm ui --dashboard`.
The dashboard provides model load/unload, status and logs. In direct mode,
`localvllm ui --open` opens the engine's API documentation instead.

Automatic mode currently binds to loopback. Default dashboard addresses are
`http://127.0.0.1:8000/ui` for vLLM and `http://127.0.0.1:8091/ui` for Omni.
Use an SSH tunnel from another computer; for example,
`ssh -L 8000:127.0.0.1:8000 user@server`, then open the local URL. Upstream
model APIs that are not supported by the proxy remain available in direct mode.

## Example models and downloads

Choose **e** in the menu to select and download an example model. Weight storage
estimates exclude runtime memory and KV caches. Larger context and concurrent
requests need additional memory. Omni models may require several stages and the
[matching upstream deployment recipe](https://docs.vllm.ai/projects/vllm-omni/en/latest/models/supported_models/).
Examples are starting points; custom Hugging Face IDs remain supported.

Before downloading, Local vLLM shows the cache location and available filesystem
space. For gated models, accept the model's access conditions on Hugging Face
and provide an account read token using `HF_TOKEN` or `LOCALVLLM_HF_TOKEN` in your
environment. Downloads reuse cached files; rerun the same command after a failed
connection to resume. Never put a token in an issue or log you share.

## Apply or remove registrations

`pull` and `add` preserve the running service. In automatic mode, apply new
registrations using the menu's **c** action or `localvllm reload` when ready.
Applying a changed launcher may restart the backend and interrupt requests.
A failed application restores the previous launcher and attempts service recovery.

Choose **x** in the menu, or run `localvllm remove MODEL`, to remove a registration.
Stop the backend and select a different direct-mode model first. Weight files
and the shared download cache are preserved. User-managed folders or links must
be moved outside the model library first; they are never deleted by this command.
Use `--backend omni` for Omni. Reload before starting after removing registrations.

## Add models

The simplest way is the menu. For direct commands, download without changing
the running model, then select it:

```bash
localvllm pull org/model-name
localvllm use org/model-name
localvllm start
```

For Omni, use `localvllm --backend omni` with `use`, or choose Omni in setup.
See [model setup and GGUF notes](Local-vLLM-Advanced.md#add-models).

### Where to put your own models

Use one complete folder per model:

```text
~/ai-vllm/models/vllm/my-text-model/
~/ai-vllm/models/omni/my-omni-model/
```

Keep the original `config.json` (or `model_index.json` for diffusion pipelines),
tokenizer files, all weight shards, and every required model component together.
A lone `.safetensors` file is not a complete model. The model must also be
supported by the selected engine. For GGUF, use [Local AI Server](https://github.com/hossbit/local-ai-server).

You can add a folder from any disk without copying it:

```bash
localvllm add /mnt/models/my-text-model
```

After registering, copying or linking your folder:

```bash
localvllm models --paths
localvllm use my-text-model
localvllm start vllm
```

For Omni, select it explicitly:

```bash
localvllm --backend omni use my-omni-model
localvllm start omni
```

You can keep large model collections on another disk without copying them:

```bash
localvllm use /mnt/models/my-text-model
```

Selecting a different model restarts the backend if it is already running.
For a shared library link instead, use `ln -s /mnt/models/my-text-model ~/ai-vllm/models/vllm/my-text-model`.

### Downloaded models and cache

A Hugging Face ID downloads weights on first start. The shared cache is
`~/ai-vllm/hf-cache/`; successful starts add readable model links under the
selected backend's `models/` folder. Links reuse the same weights, saving disk
space. `localvllm models --paths` shows the library path and cache target.
Your manually added folders are never replaced by these links.

Keep cache files while a model link uses them. Moving a linked model to
another machine requires its actual files, not just the link. Custom
`LOCALVLLM_DIR` and `LOCALVLLM_HF_HOME` settings change these paths; the CLI
prints the configured locations.

## Commands

```bash
localvllm                       # Guided model management
localvllm pull ORG/MODEL        # Download/register without selecting it
localvllm add /path/to/model    # Register an existing complete folder
localvllm mode                 # Show the serving mode
localvllm status                # Server status
localvllm models                # Available models
localvllm ui --open             # API docs or model dashboard
localvllm doctor all            # Diagnostics
localvllm backend install omni   # Install or repair Omni
localvllm update                # Active backend, llama-swap and helpers
localvllm update --all          # Every installed backend
```

[Full command reference](Local-vLLM-Advanced.md#commands).

## Versions and updates

```bash
localvllm version
localvllm update             # Active backend, llama-swap and helpers
localvllm update --all       # Every installed backend
```

`version` shows the manager, vLLM, Omni, Python, serving mode and installed
llama-swap version/path. The menu's **u** action updates all installed components.

A normal update checks the active backend, shared manager/helpers, and an
installed project-managed llama-swap binary. The active backend is selected by
`--backend`, or the configured default when omitted. Use `localvllm update --all`
to update every installed backend. It does not install llama-swap
on direct-only installations. Router downloads are verified and staged before
replacement; failures restore the previous files and serving configuration.
A shared router update restarts affected running routers, including Omni.
An external llama-swap selected by `LOCALVLLM_LLAMA_SWAP_BIN` or PATH is reported
but left for its owner/package manager to update.

`localvllm update --scripts-only` refreshes manager files without changing engine
or router versions. `--vllm-only` updates engine packages only. `--no-start`
leaves affected services stopped, including after a failed update. Models are
kept; updates do not download new model-weight revisions automatically.

## Configuration

The two backends have separate models, ports, keys and environments. See
[configuration and per-model options](Local-vLLM-Advanced.md#configuration).

## Troubleshooting

### Omni says no matching distribution or Python 3.14

Older manager versions may select Python 3.14, which current Omni releases do
not support. Refresh the manager, then retry:

```bash
localvllm update --scripts-only
localvllm backend install omni
```

The new installer finds a compatible interpreter or automatically downloads an
isolated Python 3.12. It also matches the vLLM and Omni release lines.
A running service may be restarted by the update. The download requires internet
and disk space. If it still fails, inspect `~/ai-vllm/logs/pip-install-omni.log`
and run `localvllm doctor all`. Other causes include unsupported wheels,
missing GPU drivers, network errors, or insufficient disk space.

For other issues, see the [troubleshooting reference](Local-vLLM-Advanced.md#troubleshooting).

## Security

Servers bind to `127.0.0.1` by default. API keys do not protect every vLLM
endpoint. Use a TLS reverse proxy with authentication for network exposure.
Read the [security reference](Local-vLLM-Advanced.md#security).

## More help

[Advanced guide](Local-vLLM-Advanced.md) ·
[Project README](https://github.com/hossbit/local-ai-server-vllm#readme) ·
[Report an issue](https://github.com/hossbit/local-ai-server-vllm/issues)
