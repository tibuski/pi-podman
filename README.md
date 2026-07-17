# pi-podman

Containerized [Pi Coding Agent](https://www.npmjs.com/package/@earendil-works/pi-coding-agent) powered by Debian Sid and Node.js 26.

A ready-to-use Podman image that bundles the Pi coding agent with essential development tools in an isolated, reproducible environment.

## Quick Start

### Prerequisites

- [Podman](https://podman.io/)

### One-liner

```bash
./scripts/pi-podman
```

Or install it to your PATH:

```bash
sudo ln -s "$(pwd)/scripts/pi-podman" /usr/local/bin/pi
```

Then just run `pi` — it auto-builds the image if missing, creates a new container on first run, and re-attaches on subsequent runs.

### Manual Build

```bash
podman build -t pi-podman -f Dockerfile.pi .
```

### Manual Run

```bash
podman run -it --name pi-podman \
  -v "$(pwd):/workspace:Z" \
  -v "$HOME/.config/pi.dev:/root/.pi/agent:Z" \
  pi-podman
```

The first volume mount uses `$(pwd)` to bind your current directory as the agent's workspace. Replace `$(pwd)` with the absolute path to any project directory you want the agent to work in. The second volume mount persists the agent's configuration across runs.

By giving the container a fixed name (`--name pi-podman`) and omitting `--rm`, you're building a persistent development environment. The first run creates the container; each subsequent `podman start -ai pi-podman` resumes that same container with everything you've installed still in place — tools, libraries, CLI helpers like `gh`, everything.

Running `podman run --name pi-podman` a second time will fail (name already taken), which is a good guardrail: it forces you to `podman start -ai pi-podman` and pick up where you left off. If you ever want a fresh start, just remove the old container (`podman rm pi-podman`) and run again.

If you prefer a clean, ephemeral container, add `--rm` and drop `--name`:

```bash
podman run --rm -it \
  -v "$(pwd):/workspace:Z" \
  -v "$HOME/.config/pi.dev:/root/.pi/agent:Z" \
  pi-podman
```

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

`bash`, `ca-certificates`, `curl`, `fd-find`, `git`, `gnupg`, `jq`, `libatomic1`, `make`, `openssh-client`, `procps`, `ripgrep`, `tree`, `unzip`, `wget`, `xz-utils`, `zip`

### Installing Additional Tools

The image is based on Debian Sid, so the agent can install missing development tools or libraries on the fly. Simply ask the agent to run:

```bash
apt-get update && apt-get install -y <package-name>
```

