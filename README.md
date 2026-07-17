# pidman.dev

Containerized [Pi Coding Agent](https://www.npmjs.com/package/@earendil-works/pi-coding-agent) powered by Debian Sid and Node.js 26.

A ready-to-use Podman image that bundles the Pi coding agent with essential development tools in an isolated, reproducible environment.

## Quick Start

### Prerequisites

- [Podman](https://podman.io/)

### Build

```bash
podman build -t pidman.dev -f Dockerfile.pi .
```

### Run

```bash
podman run --rm -it \
  -v "$(pwd):/workspace:Z" \
  -v "$HOME/.config/pi.dev:/root/.pi/agent:Z" \
  pidman.dev
```

The first volume mount uses `$(pwd)` to bind your current directory as the agent's workspace. Replace `$(pwd)` with the absolute path to any project directory you want the agent to work in. The second volume mount persists the agent's configuration across runs.

> [!WARNING]
> **Always use rootless Podman** — do **not** run with `sudo podman` or the `--privileged` flag.
>
> In rootless mode, Podman maps the container's root user to your host UID via user namespaces, so files created under `/workspace` remain owned by you. Running with `sudo` or `--privileged` bypasses this mapping: the agent runs as real root, and any file it creates on your mounted volumes will be owned by root on the host.

## Image Contents

| Component       | Version / Details          |
| --------------- | -------------------------- |
| Base image      | Debian Sid (slim)          |
| Node.js         | 26.5.0                     |
| Pi Coding Agent | Latest from npm            |

### Included Tools

`bash`, `ca-certificates`, `curl`, `git`, `gnupg`, `jq`, `libatomic1`, `make`, `openssh-client`, `procps`, `ripgrep`, `tree`, `unzip`, `wget`, `xz-utils`, `zip`

### Installing Additional Tools

The image is based on Debian Sid, so the agent can install missing development tools or libraries on the fly. Simply ask the agent to run:

```bash
apt-get update && apt-get install -y <package-name>
```