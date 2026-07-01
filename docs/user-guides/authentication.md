---
title: Authentication
description: Login, token management, and when authentication is required
---

# Authentication

Authentication is only required for backend-connected review commands
(`review pr --submit`, `review status`). All local analysis — scanning, SAST,
SCA, secrets, reachability, reports, baselines, fixes — works without
authentication.

## Login

### Token Authentication

```bash
patchflow login --token "$PATCHFLOW_TOKEN"
```

Validates the token and persists it to secure storage. In CI, provide the token
via an environment variable. Never store tokens in repository configuration
files.

### GitHub OAuth Device Flow

```bash
patchflow login --device --client-id <github-oauth-app-client-id>
```

Initiates the GitHub OAuth device flow:

1. PatchFlow requests a device code from GitHub.
2. You are shown a verification URL and user code.
3. Open the URL in a browser and enter the user code.
4. PatchFlow polls GitHub for an access token.
5. The access token is stored in secure storage.

## Check Authentication Status

```bash
patchflow auth status
```

Shows:

- Whether you are authenticated
- Masked token (`****<last4chars>` or `none`)
- Storage type (`keychain`, `file`, `config`, or `none`)

```bash
patchflow auth status --json
```

Outputs the authentication status as structured JSON.

## Logout

```bash
patchflow logout
```

Removes stored credentials from secure storage and clears any token from the
config file. This operation is idempotent — logging out when not authenticated
does not produce an error.

## Token Storage

PatchFlow uses secure storage for tokens:

### Keychain (default)

Uses the OS keyring/secret service via `github.com/zalando/go-keyring`:

- **macOS**: Keychain
- **Linux**: Secret Service (GNOME Keyring, KWallet)
- **Windows**: Credential Manager

Service name: `PatchFlow`
Account: `api-token`

### File Storage (fallback)

If keychain operations fail, PatchFlow falls back to file storage with restricted
permissions (0600).

### Migration

Tokens previously stored in `config.yaml` are automatically migrated to secure
storage on first login. The config file token is cleared after successful
migration.

## CI Authentication

In CI, use environment variables for authentication:

```bash
# GitHub Actions
- run: patchflow login --token "$PATCHFLOW_TOKEN"
  env:
    PATCHFLOW_TOKEN: ${{ secrets.PATCHFLOW_TOKEN }}
- run: patchflow review pr --submit
```

```bash
# GitLab CI
script:
  - patchflow login --token "$PATCHFLOW_TOKEN"
  - patchflow review pr --submit
variables:
  PATCHFLOW_TOKEN: $PATCHFLOW_TOKEN
```

Store `PATCHFLOW_TOKEN` as a protected, masked CI/CD variable. Never commit
tokens to repository files.

## Which Commands Require Authentication

| Command | Authentication Required | Purpose |
| --- | --- | --- |
| `patchflow scan run` | No | Local analysis |
| `patchflow scan export` | No | Local export (unless `--upload-github`) |
| `patchflow scan image` | No | Local container scan |
| `patchflow deps *` | No | Local dependency analysis |
| `patchflow reachability` | No | Local reachability analysis |
| `patchflow report` | No | Local report generation |
| `patchflow baseline *` | No | Local baseline management |
| `patchflow pr-review` | No | Local PR risk review |
| `patchflow explain` | No | Local rule/finding explanation |
| `patchflow fix *` | No | Local fix generation |
| `patchflow suppress` | No | Local suppression directives |
| `patchflow rules *` | No | Local rule inspection |
| `patchflow config *` | No | Local configuration |
| `patchflow cache *` | No | Local cache management |
| `patchflow doctor` | No | Local environment check |
| `patchflow review context` | No | Local git context |
| `patchflow review diff` | No | Local diff inspection |
| `patchflow review pr --submit` | **Yes** | Submit review to PatchFlow backend |
| `patchflow review status` | **Yes** | Check backend review job status |

## Next Steps

- [Configuration](./configuration) — Config file and profiles
- [PR Review](./pr-review) — Local PR risk review
- [GitHub Actions](../integrations/github-actions) — CI integration
