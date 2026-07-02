---
title: "VS Code Extension"
description: "PatchFlow Security extension for VS Code, Cursor, and Windsurf — scan, review findings, and fix security issues without leaving your editor"
---

# PatchFlow VS Code Extension

The PatchFlow VS Code extension brings local-first security scanning directly
into your editor. Run scans, browse findings in a tree view, see inline
diagnostics, view HTML reports, and explain rules — all without leaving VS Code,
Cursor, or Windsurf.

## Compatibility

The extension works in any VS Code-compatible editor:

| Editor | Supported | Notes |
| --- | --- | --- |
| VS Code | Yes (1.90.0+) | Full feature support |
| Cursor | Yes | Full feature support |
| Windsurf | Yes | Full feature support |
| VSCodium | Yes | Install VSIX manually |
| Theia-based IDEs | Yes | Install VSIX manually |

## Installation

### From the Marketplace

1. Open VS Code
2. Go to the Extensions view (`Ctrl+Shift+X` / `Cmd+Shift+X`)
3. Search for "PatchFlow Security"
4. Click Install

### From VSIX

```bash
code --install-extension patchflow-vscode-0.2.0.vsix
```

Or in Cursor / Windsurf:

```bash
cursor --install-extension patchflow-vscode-0.2.0.vsix
windsurf --install-extension patchflow-vscode-0.2.0.vsix
```

### Prerequisites

- **PatchFlow CLI** installed and on your `PATH` (or configure the path in
  settings — see [Configuration](#configuration))
- **VS Code 1.90.0** or later
- A **Git repository** as your workspace folder

Install the CLI if you have not already:

```bash
go install github.com/Patchflow-security/patchflow-cli@latest
```

See [CLI Installation](../getting-started/installation) for all installation
methods.

## Features

### 1. Three Scan Modes

| Mode | Command | What It Scans |
| --- | --- | --- |
| Scan Workspace | `patchflow.scanWorkspace` | All files in the workspace |
| Scan Changed Files | `patchflow.scanChanged` | Only files changed since the base branch |
| Scan Current File | `patchflow.scanCurrentFile` | The file currently open in the editor |

### 2. Findings Tree View

After a scan, findings appear in the **PatchFlow** view in the Explorer sidebar.
Findings are grouped by severity, then by file:

```text
PatchFlow
  Critical (2)
    src/auth.py
      28: SQL injection in login query
      45: Hardcoded database password
  High (5)
    src/config.js
      12: AWS access key detected
      ...
  Medium (8)
    ...
  Low (3)
    ...
```

Click any finding to jump to the exact line in the source file.

### 3. HTML Report Panel

The extension opens an HTML report in a side panel after each scan. The report
includes:

- **Summary cards**: Total, Critical, High, Medium, Low, Info counts
- **Risk score**: 0-100 with risk level
- **Findings table**: Severity, rule ID, type, file, line, title
- **Recommendations**: Actionable fix suggestions

The panel uses VS Code theme colors, so it matches your light or dark theme
automatically.

### 4. Inline Diagnostics

Findings are published as native VS Code diagnostics (the Problems tab). Each
diagnostic shows:

- Severity icon (error, warning, info) mapped from PatchFlow severity
- Message with rule ID and description
- Source label: `patchflow`
- Click to navigate to the finding location

Severity mapping:

| PatchFlow Severity | VS Code Diagnostic Severity |
| --- | --- |
| Critical, High | Error |
| Medium | Warning |
| Low | Information |
| Info | Hint |

### 5. Browser-Based Authentication

For backend-connected features (PR review submission, team features), the
extension uses a secure OAuth-style browser login flow:

1. Click **Login** in the PatchFlow view toolbar
2. Your browser opens to the PatchFlow website
3. Sign in (or create an account) on the website
4. The website redirects back to VS Code with a token
5. The token is stored in VS Code SecretStorage (encrypted)
6. The token is passed to the CLI via `patchflow login --token`

Security features:

- **Localhost-only callback**: The callback server only accepts connections from
  `127.0.0.1`
- **State parameter**: Random crypto token prevents CSRF attacks
- **SecretStorage**: Token is stored encrypted, never in plaintext config
- **Token expiration**: Tokens expire after 7 days

### 6. Rule Explanation

Right-click a finding and select **Explain Rule** (or use the command palette)
to see detailed information about a rule:

- What the rule detects
- Why it is dangerous
- How to fix it
- Example vulnerable and fixed code
- CWE and OWASP mapping

The explanation runs `patchflow explain --rule <ruleId>` and displays the output
in the PatchFlow output channel.

### 7. Status Bar Integration

The status bar shows the current PatchFlow state:

| State | Status Bar Text | Tooltip |
| --- | --- | --- |
| Idle | `$(shield) PatchFlow` | "PatchFlow Security" |
| Scanning | `$(sync~spin) PatchFlow: scanning...` | Scan in progress |
| Has findings | `$(shield) 2C/5H (12 total)` | Severity breakdown + risk score |

Click the status bar item to run a workspace scan.

## Commands

All commands are available in the Command Palette (`Ctrl+Shift+P` /
`Cmd+Shift+P`).

| Command | Description | Keyboard Shortcut |
| --- | --- | --- |
| `PatchFlow: Scan Workspace` | Run a full workspace scan | `Ctrl+Shift+F5` / `Cmd+Shift+F5` |
| `PatchFlow: Scan Changed Files` | Scan only files changed vs base branch | - |
| `PatchFlow: Scan Current File` | Scan the file open in the editor | - |
| `PatchFlow: Show Last Report` | Reopen the HTML report panel | - |
| `PatchFlow: Refresh Findings` | Refresh the findings tree view | - |
| `PatchFlow: Explain Rule` | Explain a security rule | - |
| `PatchFlow: Open Settings` | Open VS Code settings filtered to PatchFlow | - |
| `PatchFlow: Login with Browser` | Authenticate via browser OAuth flow | - |
| `PatchFlow: Logout` | Clear stored authentication token | - |
| `PatchFlow: Auth Status` | Show current authentication status | - |

### Context Menu

Right-click in the editor or file explorer to access:

- **Scan Current File** — Scan the file you right-clicked on

### View Toolbar

The PatchFlow view in the Explorer sidebar has toolbar buttons:

- **Scan Workspace** (shield icon)
- **Refresh Findings** (refresh icon)
- **Login** (sign-in icon) — shown when not authenticated
- **Logout** (sign-out icon) — shown when authenticated

## Configuration

All settings are under the `patchflow.*` namespace. Access them via
`File > Preferences > Settings` and search for "patchflow", or use the
`PatchFlow: Open Settings` command.

### Settings Reference

| Setting | Type | Default | Description |
| --- | --- | --- | --- |
| `patchflow.cliPath` | string | `""` | Path to the `patchflow` binary. Leave empty to auto-detect from PATH. |
| `patchflow.websiteUrl` | string | `https://app.patchflow.dev` | PatchFlow website URL for browser login. |
| `patchflow.apiBaseUrl` | string | `https://api.patchflow.dev` | PatchFlow API base URL for backend calls. |
| `patchflow.profile` | enum | `standard` | Scan profile: `quick`, `standard`, or `deep`. |
| `patchflow.failOn` | enum | `""` | Severity threshold to fail scans: `""`, `low`, `medium`, `high`, `critical`. |
| `patchflow.governanceProfile` | enum | `""` | Rule maturity filter: `""`, `dev`, `pr`, `ci`, `audit`. |
| `patchflow.suggestFixes` | boolean | `false` | Generate fix proposals in scan output. |
| `patchflow.framework` | array | `[]` | Framework packs to force-enable (e.g., `["express", "django"]`). |
| `patchflow.disableFramework` | array | `[]` | Framework packs to disable (e.g., `["rails"]`). |
| `patchflow.includeTests` | boolean | `false` | Include test files in SAST analysis. |
| `patchflow.taintDepth` | number | `3` | Taint analysis call-hop depth (1-5). |
| `patchflow.offline` | boolean | `false` | Offline mode — use local OSV database only, no network calls. |
| `patchflow.showDiagnostics` | boolean | `true` | Publish findings as VS Code diagnostics (Problems tab). |
| `patchflow.clearDiagnosticsOnScan` | boolean | `true` | Clear previous diagnostics before each new scan. |

### Configuration Examples

#### Quick Scans During Development

```json
{
  "patchflow.profile": "quick",
  "patchflow.governanceProfile": "dev",
  "patchflow.showDiagnostics": true,
  "patchflow.clearDiagnosticsOnScan": true
}
```

#### Deep Audit With Fix Suggestions

```json
{
  "patchflow.profile": "deep",
  "patchflow.governanceProfile": "audit",
  "patchflow.suggestFixes": true,
  "patchflow.includeTests": false,
  "patchflow.taintDepth": 5
}
```

#### Framework-Specific Scanning

```json
{
  "patchflow.framework": ["express", "react"],
  "patchflow.disableFramework": ["rails"]
}
```

#### Offline / Air-Gapped Mode

```json
{
  "patchflow.offline": true,
  "patchflow.profile": "standard"
}
```

#### Custom CLI Path

```json
{
  "patchflow.cliPath": "/usr/local/bin/patchflow"
}
```

#### Self-Hosted PatchFlow Backend

```json
{
  "patchflow.websiteUrl": "https://patchflow.mycompany.com",
  "patchflow.apiBaseUrl": "https://api.patchflow.mycompany.com"
}
```

## Authentication

### When Authentication Is Required

Most PatchFlow features work **without authentication**. Local scanning, SAST,
SCA, secret detection, reachability, reports, baselines, and fix suggestions
all run locally with no login needed.

Authentication is required for:

- **PR review submission** (`patchflow review pr --submit`) — sends data to the
  PatchFlow backend for AI-assisted review
- **Team features** — shared baselines, team dashboards, policy sync

### Login Flow

1. Click **Login** in the PatchFlow view toolbar, or run
   `PatchFlow: Login with Browser`
2. Your default browser opens to the PatchFlow website
3. Sign in or create an account
4. The website generates a 7-day JWT token and redirects back to VS Code
5. The token is stored in VS Code SecretStorage (encrypted)
6. The extension passes the token to the CLI via `patchflow login --token`
7. A success notification shows your masked token (e.g., `****abcd`)

### Check Auth Status

Run `PatchFlow: Auth Status` to see if you are authenticated and view your
masked token.

### Logout

Run `PatchFlow: Logout` to clear the stored token. The extension also runs
`patchflow logout` on the CLI to clear any CLI-side token storage.

### Token Security

- Tokens are stored in **VS Code SecretStorage**, which uses the OS keychain
  (macOS Keychain, Linux Secret Service, Windows Credential Manager)
- Tokens are **never written to plaintext config files**
- The callback server only accepts connections from **localhost/127.0.0.1**
- The OAuth **state parameter** prevents CSRF attacks
- Tokens expire after **7 days** and must be re-authenticated

## Scanning

### Scan Workspace

Run a full scan of all files in the workspace:

- Command Palette: `PatchFlow: Scan Workspace`
- Keyboard: `Ctrl+Shift+F5` / `Cmd+Shift+F5`
- Status bar: Click the shield icon
- View toolbar: Click the shield icon

This runs `patchflow scan run --format json --quiet` with your configured
settings and displays results in the tree view, diagnostics, and report panel.

### Scan Changed Files

Scan only files changed since the base branch (uses `--changed-only`):

- Command Palette: `PatchFlow: Scan Changed Files`
- View toolbar: Not available (use command palette)

This is faster than a full workspace scan and is ideal for pre-commit checks.

### Scan Current File

Scan only the file currently open in the editor:

- Command Palette: `PatchFlow: Scan Current File`
- Editor context menu: Right-click > Scan Current File
- Explorer context menu: Right-click a file > Scan Current File

This runs `patchflow scan run --path <file> --format json --quiet` for fast
feedback on a single file.

### Scan Output

After each scan, the extension:

1. **Updates the findings tree view** — grouped by severity then file
2. **Publishes diagnostics** — findings appear in the Problems tab
3. **Opens the HTML report panel** — summary cards, findings table, recommendations
4. **Updates the status bar** — shows finding counts and risk score
5. **Shows a notification** — summary of findings or success message

### Scan Progress

During a scan:

- The status bar shows `$(sync~spin) PatchFlow: scanning...`
- The output channel (`PatchFlow`) shows the CLI command, progress, and duration
- Concurrent scans are prevented — you cannot start a new scan while one is
  running

## Viewing Findings

### Findings Tree View

The PatchFlow view in the Explorer sidebar shows findings in a hierarchical
structure:

```text
Severity Group (count)
  File Path (count)
    Line: Finding Title
```

- **Severity groups** are ordered: Critical, High, Medium, Low, Info
- **File groups** show the basename with the full path in the tooltip
- **Finding nodes** show `line: title` with rule ID and analyzer in the tooltip
- **Click a finding** to open the file and select the exact line

### HTML Report Panel

The report panel opens automatically after each scan. It shows:

- **Metadata**: Profile, mode, CLI version, scan duration, risk score
- **Summary cards**: Color-coded counts for each severity level
- **Findings table**: Sortable by severity, rule, type, file, line
- **Recommendations**: Actionable fix suggestions from PatchFlow

To reopen the report later, run `PatchFlow: Show Last Report`.

### Diagnostics (Problems Tab)

Findings appear in the Problems tab (`Ctrl+Shift+M` / `Cmd+Shift+M`):

- Severity icons match PatchFlow severity levels
- Messages include rule ID and description
- Source is labeled as `patchflow`
- Click to navigate to the finding location
- Hover for full description and recommendation

To disable diagnostics, set `patchflow.showDiagnostics` to `false`.

### Output Channel

The `PatchFlow` output channel (`Ctrl+Shift+U` / `Cmd+Shift+U`) shows:

- CLI path resolved for the scan
- Full CLI command executed
- Scan duration
- Number of findings detected
- Any errors or warnings from the CLI

## Explaining Rules

To understand why a rule fired and how to fix it:

1. Right-click a finding in the tree view
2. Select **Explain Rule**
3. Or run `PatchFlow: Explain Rule` from the command palette and enter a rule ID

The extension runs `patchflow explain --rule <ruleId>` and shows the output in
the PatchFlow output channel. The explanation includes:

- Rule description and why it is dangerous
- CWE and OWASP mapping
- Example vulnerable code
- Example fixed code
- Suppression syntax

## Keyboard Shortcuts

| Shortcut | Action |
| --- | --- |
| `Ctrl+Shift+F5` / `Cmd+Shift+F5` | Scan Workspace |

## Troubleshooting

### CLI Not Found

**Symptom:** Error message "PatchFlow CLI not found"

**Solutions:**

1. Install the CLI: `go install github.com/Patchflow-security/patchflow-cli@latest`
2. Or set `patchflow.cliPath` to the full path of the binary
3. Verify the CLI works: `patchflow version`

### Scan Returns No Findings

**Symptom:** Scan completes but shows 0 findings

**Solutions:**

1. Check the scan profile — `quick` uses fewer rules than `deep`
2. Check the governance profile — `dev` only includes stable rules
3. Check if frameworks are detected — run `patchflow rules list-frameworks` in
   a terminal
4. Force-enable framework packs in settings: `"patchflow.framework": ["express"]`
5. Check the output channel for CLI errors

### Diagnostics Not Showing

**Symptom:** Findings appear in the tree view but not in the Problems tab

**Solutions:**

1. Check `patchflow.showDiagnostics` is `true`
2. Check `patchflow.clearDiagnosticsOnScan` — if `true`, diagnostics are cleared
   at the start of each scan and re-added when the scan completes
3. Open the Problems tab (`Ctrl+Shift+M` / `Cmd+Shift+M`)

### Login Fails

**Symptom:** Browser opens but login does not complete

**Solutions:**

1. Check `patchflow.websiteUrl` points to the correct PatchFlow website
2. Check your browser did not block the redirect to `localhost`
3. Check the output channel for callback server errors
4. Try again — the login flow has a 5-minute timeout

### Scan Is Slow

**Solutions:**

1. Use a faster profile: `"patchflow.profile": "quick"`
2. Scan changed files only instead of the full workspace
3. Reduce taint depth: `"patchflow.taintDepth": 1`
4. Disable reachability: not currently exposed in extension settings — use CLI
   directly for this

### Extension Does Not Activate

**Symptom:** Extension commands do not appear in the command palette

**Solutions:**

1. Ensure you have a workspace folder open (not just a loose file)
2. Ensure the workspace is a Git repository
3. Reload the window: `Developer: Reload Window`
4. Check the extension is enabled in the Extensions view

## Development

### Building from Source

```bash
cd /Users/digitalcenter/patchflow-vscode-ext
npm install
npm run compile
```

### Debugging

1. Open the project in VS Code
2. Press `F5` to launch the Extension Development Host
3. The extension activates in the new window

### Packaging

```bash
npm run package
```

This produces `patchflow-vscode-0.2.0.vsix` in the project root.

### Testing

```bash
npm test
```

Tests run in a VS Code test environment and cover the summarize function,
severity counting, and type counting.

## Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                    VS Code Extension                         │
├─────────────────────────────────────────────────────────────┤
│  extension.ts (activation)                                   │
│    ├── scan.ts (ScanRunner) ── runs CLI, parses JSON         │
│    ├── cli.ts (CLI resolution, args, parsing)                │
│    ├── findingsTree.ts (TreeDataProvider)                    │
│    ├── reportPanel.ts (Webview HTML report)                  │
│    ├── diagnostics.ts (VS Code Problems integration)         │
│    ├── auth.ts (OAuth callback server, SecretStorage)        │
│    ├── authCommands.ts (login/logout/status handlers)        │
│    ├── commands.ts (command palette handlers)                │
│    ├── config.ts (settings access, output channel, status)   │
│    ├── store.ts (in-memory results with change events)       │
│    └── types.ts (TypeScript mirrors of CLI JSON output)      │
├─────────────────────────────────────────────────────────────┤
│                    PatchFlow CLI                             │
│  (invoked as child process, JSON output parsed by extension) │
└─────────────────────────────────────────────────────────────┘
```

The extension is a thin UI layer over the PatchFlow CLI. All scanning logic
lives in the CLI binary. The extension:

1. Resolves the CLI path (configured or from PATH)
2. Builds scan arguments from VS Code settings
3. Spawns the CLI as a child process
4. Parses the JSON output
5. Displays results in tree view, diagnostics, and report panel

## Version History

### v0.2.0 (2026-07-02)

- Added browser-based login via PatchFlow website
- Added login/logout/authStatus commands
- Added token storage in VS Code SecretStorage
- Added `websiteUrl` and `apiBaseUrl` settings
- Added context-aware login/logout buttons in view toolbar

### v0.1.0 (2026-07-02)

- Initial release
- Scan commands: workspace, changed files, current file
- Findings tree view with severity/file grouping
- HTML report panel with summary and recommendations
- VS Code diagnostics integration (Problems tab)
- 13 configurable settings
- Status bar integration with finding counts
- Rule explanation via CLI

## Next Steps

- [Installation](../getting-started/installation) — Install the PatchFlow CLI
- [Quickstart](../getting-started/quickstart) — Get started with the CLI
- [Scan Your Project](../user-guides/scan) — Full CLI scan reference
- [GitHub Repository](https://github.com/Patchflow-security/patchflow-vscode-ext) — Extension source code
