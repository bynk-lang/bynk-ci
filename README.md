# bynk-ci

The standard Bynk quality gate in one step: format check, type check, and tests.
Installs the toolchain via [`setup-bynk`](https://github.com/bynk-lang/setup-bynk)
and annotates `bynkc` diagnostics inline on the pull request via a problem
matcher.

## Usage

```yaml
name: CI
on: [push, pull_request]

jobs:
  bynk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: bynk-lang/bynk-ci@v1
        with:
          version: 0.107.0
          source: src
```

Run only some checks:

```yaml
- uses: bynk-lang/bynk-ci@v1
  with:
    format: "true"
    check: "true"
    test: "false"
```

## Inputs

| Input               | Default        | Description |
| ------------------- | -------------- | ----------- |
| `version`           | `latest`       | Bynk version (forwarded to setup-bynk). |
| `repository`        | `accuser/bynk` | Release repository. |
| `working-directory` | `.`            | Directory to run in. |
| `source`            | `src`          | Path passed to `bynkc` (file or directory). |
| `format`            | `true`         | Run `bynkc fmt --check`. |
| `check`             | `true`         | Run `bynkc check --format short`. |
| `test`              | `true`         | Run `bynkc test`. |
| `github-token`      | `${{ github.token }}` | Token for setup-bynk. |

## How annotations work

`bynkc check --format short` emits one diagnostic per line as
`path:line:col: severity[category]: message`. The bundled
[`bynk-problem-matcher.json`](bynk-problem-matcher.json) parses that form so
errors and warnings appear directly on the changed lines in the PR.

## License

Licensed under either of [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE) at
your option.
