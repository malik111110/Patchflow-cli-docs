# PatchFlow CLI Documentation

Documentation for the [PatchFlow CLI](https://github.com/Patchflow-security/patchflow-cli) —
a local-first security scanner for modern engineering teams.

## Structure

This repository contains structured Markdown documentation only. No build
system or static site generator is required — all documentation is plain
Markdown that can be read directly on GitHub or in any Markdown viewer.

```text
docs/
├── docs.json                     Mintlify configuration (navigation, theme, branding)
├── index.md                      Home page and overview
├── getting-started/
│   ├── installation.md           Installation methods
│   ├── quickstart.md             First scan, report, and baseline
│   ├── concepts.md               Core concepts (SCA, SAST, reachability, risk)
│   ├── why-patchflow.md          Why PatchFlow vs stacking Trivy + Semgrep + Gitleaks
│   ├── local-vs-backend.md       What data leaves your machine (trust/privacy)
│   ├── common-errors.md          Troubleshooting common errors
│   └── changelog.md              Version history and release notes
├── user-guides/
│   ├── scan.md                   Scan command reference
│   ├── dependencies.md           deps command
│   ├── reachability.md           Reachability analysis
│   ├── framework-packs.md        Framework rule packs
│   ├── custom-rules.md           Custom rules and YAML
│   ├── reports.md                Report generation
│   ├── baselines.md              Baseline management
│   ├── pr-review.md              PR risk review
│   ├── fixes.md                  Fix generation and application
│   ├── explain.md                Finding explanation
│   ├── suppressions.md           Suppression directives
│   ├── container-scanning.md     Container image scanning
│   ├── configuration.md          Configuration and profiles
│   ├── authentication.md         Authentication and tokens
│   └── cache.md                  Cache management and offline mode
├── workflows/
│   ├── recommended.md            Workflow hub (solo dev vs team)
│   ├── solo-dev.md               Solo developer workflow
│   ├── teams.md                  Team workflow with phased adoption
│   └── ci-adoption.md            CI adoption strategy
├── integrations/
│   ├── github-actions.md         GitHub Actions integration
│   ├── gitlab.md                 GitLab CI integration
│   ├── sarif.md                  SARIF uploads
│   ├── pre-commit.md             pre-commit hook
│   ├── jenkins.md                Jenkins integration
│   └── azure-devops.md           Azure DevOps integration
├── developers/
│   ├── overview.md               Developer overview and packages
│   ├── architecture.md           Internal architecture
│   ├── rule-packs.md             Framework pack development
│   ├── adding-packs.md           Step-by-step pack creation
│   ├── ci-cd.md                  CI for the CLI
│   └── benchmarks.md             Benchmark suite
└── reference/
    ├── commands.md               Curated command map
    ├── generated-commands.md     Full command help output
    ├── global-flags.md           Global CLI flags
    ├── configuration.md          Config file reference
    ├── yaml-policy.md            YAML policy format
    ├── rules-governance.md       Rule governance reference
    └── scanners.md               Scanner reference
```

## Regenerating the Command Reference

The `reference/generated-commands.md` file is auto-generated from the CLI
binary's help output. To regenerate it after changing CLI commands:

```bash
cd /Users/digitalcenter/patchflow-cli
go build -o patchflow .

# Run the generation script
bash /path/to/gen_cmds.sh
```

The generation script runs `patchflow <command> --help` for every command and
subcommand, collecting the output into a single Markdown file.

## Contributing

1. Edit the relevant Markdown file(s) in `docs/`.
2. If CLI commands changed, regenerate `reference/generated-commands.md`.
3. Verify all internal links point to existing files.
4. Submit a pull request.

## License

See the PatchFlow CLI repository for license information.
