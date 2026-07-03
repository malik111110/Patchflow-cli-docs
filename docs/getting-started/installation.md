---
title: Installation
description: Install PatchFlow CLI on macOS, Linux, Windows, or via Docker/Podman
---

# Installation

PatchFlow CLI is a single, self-contained binary. Choose your environment below.

<Tabs>
  <Tab title="macOS">
    <Tabs>
      <Tab title="Install Script (recommended)">
        ```bash
        curl -fsSL https://github.com/Patchflow-security/patchflow-cli/raw/main/scripts/install.sh | bash
        ```

        The script detects your platform and architecture, downloads the signed release, verifies the checksum, and installs the binary to `~/.local/bin`. If that directory is not on your `PATH`, the script prints the exact `export` command to run.

        Verify:

        ```bash
        patchflow version
        ```
      </Tab>
      <Tab title="Homebrew">
        ```bash
        brew install Patchflow-security/tap/patchflow
        ```

        Verify:

        ```bash
        patchflow version
        ```
      </Tab>
      <Tab title="Go Install">
        Requires Go 1.25 or later:

        ```bash
        go install github.com/Patchflow-security/patchflow-cli@latest
        ```

        The binary is placed in `$GOPATH/bin` (typically `~/go/bin`). Ensure that directory is on your `PATH`.
      </Tab>
      <Tab title="Build from Source">
        ```bash
        git clone https://github.com/Patchflow-security/patchflow-cli.git
        cd patchflow-cli
        go build -o patchflow .
        sudo mv ./patchflow /usr/local/bin/patchflow
        patchflow version
        ```
      </Tab>
    </Tabs>
  </Tab>

  <Tab title="Linux">
    <Tabs>
      <Tab title="Install Script (recommended)">
        ```bash
        curl -fsSL https://github.com/Patchflow-security/patchflow-cli/raw/main/scripts/install.sh | bash
        ```

        For containers or CI, install directly to `/usr/local/bin` so the binary is already on `PATH`:

        ```bash
        curl -fsSL https://github.com/Patchflow-security/patchflow-cli/raw/main/scripts/install.sh | bash -s -- --install-dir /usr/local/bin
        ```

        Verify:

        ```bash
        patchflow version
        ```
      </Tab>
      <Tab title="Homebrew">
        ```bash
        brew install Patchflow-security/tap/patchflow
        ```

        <Note>
          The Patchflow Security tap does not currently publish Linux bottles. On Linux, `brew install` will fall back to building from source and requires a C compiler (`gcc` or `clang`). For Linux and containerized environments, we recommend the install script above.
        </Note>
      </Tab>
      <Tab title="Go Install">
        Requires Go 1.25 or later:

        ```bash
        go install github.com/Patchflow-security/patchflow-cli@latest
        ```

        The binary is placed in `$GOPATH/bin` (typically `~/go/bin`). Ensure that directory is on your `PATH`.
      </Tab>
      <Tab title="Build from Source">
        ```bash
        git clone https://github.com/Patchflow-security/patchflow-cli.git
        cd patchflow-cli
        go build -o patchflow .
        sudo mv ./patchflow /usr/local/bin/patchflow
        patchflow version
        ```
      </Tab>
    </Tabs>
  </Tab>

  <Tab title="Windows">
    <Tabs>
      <Tab title="Scoop (recommended)">
        ```powershell
        scoop bucket add patchflow https://github.com/Patchflow-security/scoop-bucket
        scoop install patchflow
        ```

        Verify:

        ```powershell
        patchflow version
        ```
      </Tab>
      <Tab title="WinGet">
        ```powershell
        winget install Patchflow-security.patchflow
        ```
      </Tab>
      <Tab title="Download Binary">
        Download the latest Windows binary from the
        [releases page](https://github.com/Patchflow-security/patchflow-cli/releases),
        add it to your `PATH`, and run:

        ```powershell
        patchflow version
        ```
      </Tab>
      <Tab title="Build from Source">
        Requires Go 1.25 or later:

        ```powershell
        git clone https://github.com/Patchflow-security/patchflow-cli.git
        cd patchflow-cli
        go build -o patchflow.exe .
        ```
      </Tab>
    </Tabs>
  </Tab>

  <Tab title="Docker / Podman">
    ```bash
    podman pull ghcr.io/patchflow-security/cli:latest
    podman run --rm -v "$PWD:/repo" ghcr.io/patchflow-security/cli:latest scan run --path /repo
    ```

    The container image bundles the PatchFlow binary. Mount your repository at `/repo` and pass scan commands as arguments.

    For a permanent alias in your shell:

    ```bash
    alias patchflow='podman run --rm -v "$PWD:/repo" ghcr.io/patchflow-security/cli:latest'
    ```
  </Tab>
</Tabs>

---

## Verify the Environment

After installation, run `doctor` to verify that PatchFlow can detect your repository and that optional external tools are available:

```bash
patchflow doctor
```

`doctor` checks:

- Git installation and version
- Git repository detection and remote URL
- Embedded SAST scanners (always available)
- External SAST tools (`gosec`, `bandit`, `semgrep`, `gitleaks`) — reported as found or not found

External tools are optional. PatchFlow's embedded scanners work without them.

## Optional External Tools

PatchFlow's embedded scanners require no installation. The following external tools supplement the embedded scanners when installed:

| Tool | Language | Coverage | Install |
| --- | --- | --- | --- |
| `gosec` | Go | Go AST security analysis | `go install github.com/securego/gosec/v2/cmd/gosec@latest` |
| `bandit` | Python | Python security linting | `pip install bandit` |
| `semgrep` | Multi-language | Pattern-based SAST | `pip install semgrep` or `brew install semgrep` |
| `gitleaks` | Secrets | Secret detection | `brew install gitleaks` |
| `trivy` | Containers, IaC | Container image and IaC scanning | `brew install trivy` |
| `checkov` | IaC | Infrastructure-as-code scanning | `pip install checkov` |

External tools run automatically when detected on `PATH`. Use `patchflow doctor` to see which tools are available in your environment.

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

## Next Steps

- [Quickstart](./quickstart) — Run your first scan
- [Core Concepts](./concepts) — Understand SCA, SAST, reachability, risk scoring, and governance
