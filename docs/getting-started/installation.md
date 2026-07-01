---
title: Installation
description: Install PatchFlow CLI via Homebrew, Scoop, Go, or build from source
---

# Installation

PatchFlow CLI is a single Go binary. Choose the installation method that fits
your environment.

## Homebrew (macOS / Linux)

```bash
brew install Patchflow-security/tap/patchflow
```

Verify the installation:

```bash
patchflow version
```

## Scoop (Windows)

```powershell
scoop bucket add patchflow https://github.com/Patchflow-security/scoop-bucket
scoop install patchflow
```

## Install Script (macOS / Linux)

```bash
curl -fsSL https://github.com/Patchflow-security/patchflow-cli/raw/main/scripts/install.sh | bash
```

The script detects your platform and architecture, downloads the appropriate
binary, and places it on your `PATH`.

## Go Install

Requires Go 1.25 or later:

```bash
go install github.com/Patchflow-security/patchflow-cli@latest
```

The binary is placed in `$GOPATH/bin` (typically `~/go/bin`). Ensure that
directory is on your `PATH`.

## Build from Source

```bash
git clone https://github.com/Patchflow-security/patchflow-cli.git
cd patchflow-cli
go build -o patchflow .
```

Move the binary to a directory on your `PATH`:

```bash
sudo mv ./patchflow /usr/local/bin/patchflow
patchflow version
```

### Build Requirements

- Go 1.25 or later
- Git (for repository detection and diff analysis)
- Internet access for OSV.dev queries (unless using offline mode with a local
  OSV database — see [Cache Management](../user-guides/cache.md))

## Docker / Podman

```bash
podman pull ghcr.io/patchflow-security/cli:latest
podman run --rm -v "$PWD:/repo" ghcr.io/patchflow-security/cli:latest scan run --path /repo
```

The container image bundles the PatchFlow binary. Mount your repository at
`/repo` and pass scan commands as arguments.

## Download Binary

Download the latest binary for your platform from the
[releases page](https://github.com/Patchflow-security/patchflow-cli/releases).

Pre-built binaries are available for:

- Linux (amd64, arm64)
- macOS (amd64, arm64)
- Windows (amd64)

## Shell Completion

Generate completion scripts for your shell:

```bash
# Bash
source <(patchflow completion bash)
# Add to ~/.bashrc for persistence

# Zsh
source <(patchflow completion zsh)
# Add to ~/.zshrc for persistence (ensure compinit is enabled)

# Fish
patchflow completion fish | source
# Add to ~/.config/fish/config.fish for persistence

# PowerShell
patchflow completion powershell | Out-String | Invoke-Expression
# Add to your PowerShell profile for persistence
```

## Verify the Environment

After installation, run `doctor` to verify that PatchFlow can detect your
repository and that optional external tools are available:

```bash
patchflow doctor
```

`doctor` checks:

- Git installation and version
- Git repository detection and remote URL
- Embedded SAST scanners (always available)
- External SAST tools (gosec, bandit, semgrep, gitleaks) — reported as found or
  not found

External tools are optional. PatchFlow's embedded scanners work without them.

## Optional External Tools

PatchFlow's embedded scanners require no installation. The following external
tools supplement the embedded scanners when installed:

| Tool | Language | Coverage | Install |
| --- | --- | --- | --- |
| `gosec` | Go | Go AST security analysis | `go install github.com/securego/gosec/v2/cmd/gosec@latest` |
| `bandit` | Python | Python security linting | `pip install bandit` |
| `semgrep` | Multi-language | Pattern-based SAST | `pip install semgrep` or `brew install semgrep` |
| `gitleaks` | Secrets | Secret detection | `brew install gitleaks` |
| `trivy` | Containers, IaC | Container image and IaC scanning | `brew install trivy` |
| `checkov` | IaC | Infrastructure-as-code scanning | `pip install checkov` |

External tools run automatically when detected on `PATH`. Use `patchflow doctor`
to see which tools are available in your environment.

## Next Steps

- [Quickstart](./quickstart.md) — Run your first scan
- [Core Concepts](./concepts.md) — Understand SCA, SAST, reachability, risk
  scoring, and governance
