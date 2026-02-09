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

## Remote Deployment (Ethernet-only Device)

When deploying on a device (e.g. RK3588) connected to a MacBook via Ethernet without direct internet:

### 1. MacBook Internet Sharing

```bash
# Enable IP forwarding
sudo sysctl -w net.inet.ip.forwarding=1

# Enable NAT (en0 = MacBook Wi-Fi interface)
echo "nat on en0 from 10.0.0.0/24 to any -> (en0)" | sudo pfctl -ef -
```

On the remote device, configure gateway and DNS:

```bash
sudo ip route add default via 10.0.0.1 dev eth0
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

### 2. Subscription Update (if device can't download directly)

Download on MacBook and transfer:

```bash
curl -o /tmp/clash_config.yaml "<subscription_url>"
scp /tmp/clash_config.yaml fan@10.0.0.2:/userdata/fan/clashctl/resources/config.yaml
```

Then restart on the device: `clashoff && clashon`

### 3. API Operations

The API port and secret are in `runtime.yaml`, not necessarily the default `9090`:

```bash
# Find actual port and secret
grep 'external-controller' ~/clashctl/resources/runtime.yaml
clashsecret

# List proxy nodes
curl -s -H "Authorization: Bearer <secret>" http://127.0.0.1:<port>/proxies | ~/clashctl/bin/yq '.proxies | keys' -P

# Switch proxy group node
curl -X PUT -H "Authorization: Bearer <secret>" -H "Content-Type: application/json" \
  http://127.0.0.1:<port>/proxies/主代理 \
  -d '{"name": "JP自动选择"}'

# Verify exit country
curl -s https://ipinfo.io/country
```

### Notes
- Anthropic supported countries: US, JP, GB, KR, DE, FR, AU, etc. Full list at https://anthropic.com/supported-countries
- JP nodes typically have lower latency than US from East Asia
- `yq` is installed at `~/clashctl/bin/yq`, not in system PATH
