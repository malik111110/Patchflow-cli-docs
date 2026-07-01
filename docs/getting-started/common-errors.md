---
title: Common Errors
description: Troubleshooting common PatchFlow CLI errors and issues
---

# Common Errors

This page covers the most common errors and issues you may encounter when using
PatchFlow CLI, along with their solutions.

## No Git Repository Found

### Error

```text
Error: no Git repository found in the current directory or any parent directory
```

### Cause

PatchFlow requires a Git repository for most commands. The current directory is
not inside a Git repository.

### Solution

```bash
# Initialize a Git repository
git init

# Or navigate to an existing repository
cd /path/to/your/repo
```

PatchFlow uses Git to detect changed files, compute diffs, and manage
baselines. Without a Git repository, most commands will not work.

## No Lockfile Found

### Error

```text
Warning: no lockfile found for npm. Dependency resolution may be incomplete.
```

### Cause

PatchFlow looks for lockfiles (e.g., `package-lock.json`, `yarn.lock`,
`requirements.txt`, `go.sum`, `Gemfile.lock`, `composer.lock`, `Cargo.lock`) to
identify dependencies. Without a lockfile, SCA analysis may miss transitive
dependencies.

### Solution

Generate a lockfile for your package manager:

```bash
# npm
npm install --package-lock-only

# Yarn
yarn install

# Python (pip)
pip freeze > requirements.txt

# Go
go mod tidy

# Ruby
bundle install

# PHP
composer install

# Rust
cargo generate-lockfile
```

If you intentionally do not use a lockfile, you can suppress this warning — SCA
will still scan direct dependencies from the manifest file.

## OSV.dev Unavailable

### Error

```text
Error: failed to query OSV.dev: connection timeout
Error: failed to query OSV.dev: connection refused
```

### Cause

PatchFlow cannot reach `api.osv.dev` for vulnerability lookups. This may be due
to network restrictions, firewalls, or OSV.dev being temporarily unavailable.

### Solution

**Option 1: Use offline mode**

Download the OSV database on a connected machine, then use offline mode:

```bash
# On a connected machine
patchflow cache update

# Transfer ~/.patchflow/osv-db/ to the restricted machine

# Run scans with no network calls
patchflow scan run --offline
```

**Option 2: Skip SCA analysis**

If you only need SAST and secret detection:

```bash
patchflow scan run --no-sca
```

Note: `--no-sca` is not a documented flag. To skip SCA, skip reachability and
focus on SAST:

```bash
patchflow scan run --no-reachability
```

**Option 3: Check network connectivity**

```bash
curl -s https://api.osv.dev/v1/query -X POST \
  -H "Content-Type: application/json" \
  -d '{"queries":[{"package":{"name":"lodash","ecosystem":"npm"},"version":"4.17.20"}]}'
```

If this fails, your network is blocking access to OSV.dev. Work with your
network administrator to allow `api.osv.dev`.

## SARIF Upload Failed

### Error

```text
Error: failed to upload SARIF to GitHub: 401 Unauthorized
Error: failed to upload SARIF to GitHub: 403 Forbidden
```

### Cause

The `GITHUB_TOKEN` environment variable is missing, expired, or lacks the
`security_events: write` permission.

### Solution

**For GitHub Actions:**

Ensure the workflow has the correct permissions:

```yaml
permissions:
  contents: read
  security-events: write
```

**For manual uploads:**

```bash
export GITHUB_TOKEN="ghp_your_token_here"
patchflow scan export --format sarif --upload-github
```

The token needs the `repo` scope (for private repositories) or `public_repo`
scope (for public repositories).

### Error

```text
Error: failed to upload SARIF to GitHub: 422 Unprocessable Entity
```

### Cause

The SARIF file is too large (GitHub limit: 10MB for the SARIF payload) or
contains invalid content.

### Solution

Filter findings to reduce SARIF size:

```bash
patchflow scan run --profile standard --format sarif --output patchflow.sarif
```

If the file is still too large, scan specific paths:

```bash
patchflow scan run --path src/ --format sarif --output patchflow.sarif
```

## Authentication Token Missing

### Error

```text
Error: not authenticated. Run 'patchflow login' first.
Error: failed to submit review: 401 Unauthorized
```

### Cause

Backend-connected commands (`review pr --submit`, `review status`) require
authentication. No token is stored, or the token has expired.

### Solution

```bash
# Login with a token
patchflow login --token "$PATCHFLOW_TOKEN"

# Verify authentication
patchflow auth status
```

In CI, provide the token via an environment variable:

```bash
export PATCHFLOW_TOKEN="your_token_here"
patchflow login --token "$PATCHFLOW_TOKEN"
```

Store the token as a secret in your CI system (GitHub Secrets, GitLab CI/CD
variables, Jenkins credentials). Never commit tokens to repository files.

## Scan Timeout

### Error

```text
Error: scan timed out after 10m0s
```

### Cause

The scan is taking longer than the default timeout. This may happen with large
repositories or when using the `deep` profile.

### Solution

**Option 1: Use a faster profile**

```bash
patchflow scan run --profile quick
```

**Option 2: Scan specific paths**

```bash
patchflow scan run --path src/ --path lib/
```

**Option 3: Use incremental scanning**

```bash
patchflow scan run --incremental
```

Incremental scanning only re-scans files changed since the last scan, using
cached file hashes.

**Option 4: Skip slow analysis stages**

```bash
patchflow scan run --no-reachability --no-licenses
```

## No Findings Reported

### Symptom

```text
Scan complete. No findings detected.
```

### Cause

This may be correct (your code is clean), or it may indicate a configuration
issue.

### Diagnosis

1. **Verify files were scanned:**
   ```bash
   patchflow scan run -v
   ```
   Verbose mode shows which files were processed and which scanners ran.

2. **Check governance profile:**
   ```bash
   patchflow scan run --governance-profile audit
   ```
   The `audit` profile includes experimental rules that are hidden in other
   profiles.

3. **Check if frameworks were detected:**
   ```bash
   patchflow rules list-frameworks
   ```
   If your framework is not detected, force-enable it:
   ```bash
   patchflow scan run --framework express
   ```

4. **Check if findings are suppressed:**
   ```bash
   patchflow scan run --show-suppressed
   ```

5. **Run doctor to verify scanners:**
   ```bash
   patchflow doctor
   ```

## Too Many False Positives

### Symptom

Scan reports many findings that are not real vulnerabilities.

### Solution

1. **Suppress individual false positives:**
   ```bash
   patchflow suppress PY001 --file src/app.py --line 42 --reason "eval is safe, input is sanitized"
   ```

2. **Add custom sanitizers for framework rules:**
   ```yaml
   # .patchflow/rules.yaml
   framework_overrides:
     express:
       custom_sanitizers:
         - func: validateRedirectTarget
   ```

3. **Use a stricter governance profile:**
   ```bash
   patchflow scan run --governance-profile dev
   ```
   The `dev` profile only includes stable rules with low false-positive rates.

4. **Reduce taint analysis depth:**
   ```bash
   patchflow scan run --taint-depth 1
   ```

5. **See [Suppressions](../user-guides/suppressions.md) and
   [Custom Rules](../user-guides/custom-rules.md) for more options.**

## External Tools Not Detected

### Symptom

```text
External tools:
  gosec:    not installed (optional)
  semgrep:  not installed (optional)
```

### Cause

External tools (gosec, semgrep, bandit, gitleaks, trivy) are not on your `PATH`.

### Solution

External tools are optional. PatchFlow's embedded scanners work without them.
To install them for additional coverage:

```bash
# gosec
go install github.com/securego/gosec/v2/cmd/gosec@latest

# semgrep
pip install semgrep

# bandit
pip install bandit

# gitleaks
brew install gitleaks  # macOS
# or: go install github.com/gitleaks/gitleaks/v8@latest

# trivy
brew install trivy  # macOS
```

Verify with `patchflow doctor`.

## Cache Directory Permission Error

### Error

```text
Error: failed to create cache directory: permission denied
```

### Cause

PatchFlow cannot write to the default cache directory
(`~/.cache/patchflow/`).

### Solution

```bash
# Use a custom cache directory
patchflow scan run --cache-dir /tmp/patchflow-cache

# Or set the environment variable
export PATCHFLOW_CACHE_DIR=/tmp/patchflow-cache
patchflow scan run
```

## Framework Not Auto-Detected

### Symptom

Your project uses a framework (e.g., Express, Django) but the framework pack is
not activated during scans.

### Diagnosis

```bash
patchflow rules list-frameworks
```

Check the "Detected" column for your framework.

### Solution

1. **Force-enable the pack:**
   ```bash
   patchflow scan run --framework express
   ```

2. **Check detection signals:**
   PatchFlow auto-detects frameworks based on filesystem signals (config files,
   directory structures, manifest contents). If your project uses a non-standard
   layout, auto-detection may fail.

3. **Enable in YAML:**
   ```yaml
   # .patchflow/rules.yaml
   frameworks:
     auto_detect: true
     enabled:
       - express
       - django
   ```

## PatchFlow Binary Not Found

### Error

```text
Error: patchflow: command not found
```

### Solution

Ensure PatchFlow is installed and on your `PATH`:

```bash
# Check if installed
which patchflow

# If not found, install it
go install github.com/Patchflow-security/patchflow-cli@latest

# Or build from source
cd /path/to/patchflow-cli
go build -o patchflow .
sudo mv patchflow /usr/local/bin/
```

See [Installation](./installation.md) for all installation methods.

## Next Steps

- [Quickstart](./quickstart.md) — Get started with PatchFlow
- [Scan Your Project](../user-guides/scan.md) — All scan flags
- [Suppressions](../user-guides/suppressions.md) — Reduce false positives
- [Cache Management](../user-guides/cache.md) — Offline mode setup
