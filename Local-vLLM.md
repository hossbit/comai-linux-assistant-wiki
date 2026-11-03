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
localvllm setup
```

Choose **1** for text or **2** for a supported multimodal model. The setup
installs the chosen engine if needed, asks for a Hugging Face model ID or a
local model name, and offers to start the server. First startup may download
model weights; choose a model that fits your available memory and GPU.

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

## Add models

The simplest way is `localvllm setup`. You can also run:

```bash
localvllm use org/model-name
localvllm start
```

For Omni, use `localvllm --backend omni` with `use`, or choose Omni in setup.
See [model setup and GGUF notes](Local-vLLM-Advanced.md#add-models).

## Commands

```bash
localvllm setup                 # Guided setup
localvllm status                # Server status
localvllm models                # Available models
localvllm ui                    # API documentation and URL
localvllm doctor all            # Diagnostics
localvllm backend install omni   # Install or repair Omni
localvllm update                # Update installed backends
```

[Full command reference](Local-vLLM-Advanced.md#commands).

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
