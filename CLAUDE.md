# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A one-command installation toolkit for deploying Clash/Mihomo proxy kernels on Linux systems. It handles binary downloads, init system detection, service installation, and shell integration automatically.

## Development Commands

```bash
# Lint shell scripts
shellcheck install.sh uninstall.sh scripts/**/*.sh

# Test installation locally (without subscription)
bash install.sh mihomo

# Test with subscription URL
bash install.sh mihomo "https://example.com/subscription"

# Uninstall completely
bash uninstall.sh
```

## Architecture

### Entry Points
- `install.sh` - Main installer that orchestrates the entire setup flow
- `uninstall.sh` - Cleanup script for complete removal

### Core Scripts
- `scripts/preflight.sh` - Pre-install validation, architecture detection, binary downloads, init system detection, service installation
- `scripts/cmd/clashctl.sh` - Main CLI dispatcher (all `clash*` commands)
- `scripts/cmd/common.sh` - Shared utility functions (port management, config merging, network detection)
- `scripts/cmd/clashctl.fish` - Fish shell wrapper

### Init System Templates (`scripts/init/`)
Templates for systemd, OpenRC, SysVinit, and runit. The installer detects the init system and uses the appropriate template.

### Configuration Flow
```
Base Config (subscription) + Mixin Config (user customizations)
                    ↓ (_merge_config via yq)
            Runtime Config (active)
```

Key config files at runtime (`~/clashctl/resources/`):
- `config.yaml` - Base config from subscription
- `mixin.yaml` - User customizations (ports, TUN, DNS, rules)
- `runtime.yaml` - Merged active configuration
- `profiles.yaml` - Subscription metadata

## Code Conventions

### Function Naming
- `_function()` - Private/internal utilities
- `function command()` - Public CLI commands
- `service_*=()` - Arrays for init system abstraction
- `placeholder_*` - Tokens for template substitution

### Shell Compatibility
Scripts support Bash, Zsh, and Fish. The installer auto-injects sourcing into user's shell config files.

### Error Handling
- `_error_quit "message"` - Fatal error, exits interactive shell
- `_failcat "message"` - Non-fatal warning to stderr

## Key Dependencies

Downloaded to `~/clashctl/bin/`:
- `mihomo` or `clash` - Proxy kernel (architecture-matched)
- `yq` - YAML processor for config merging
- `subconverter` - Subscription format converter

## Environment Variables (`.env`)

```bash
KERNEL_NAME=mihomo          # mihomo or clash
CLASH_BASE_DIR=~/clashctl   # Installation directory
VERSION_MIHOMO=v1.19.17     # Kernel version
VERSION_YQ=v4.49.2          # yq version
```

## Init System Detection

The installer detects via `/proc/1/exe` and container signals:
1. **systemd** - Uses systemctl
2. **OpenRC** - Alpine/Gentoo
3. **SysVinit** - Legacy Debian/RHEL
4. **runit** - Void Linux
5. **nohup** - Containers (docker, kubernetes, podman)

## CLI Commands (after installation)

`clashon`, `clashoff`, `clashstatus`, `clashui`, `clashsub`, `clashmixin`, `clashsecret`, `clashtun`, `clashupgrade`, `clashlog`
