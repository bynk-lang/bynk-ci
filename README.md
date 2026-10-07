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
      - uses: actions/checkout@v7
      - uses: bynk-lang/bynk-ci@v1
        with:
          version: 0.307.0
          source: .
```

`source: .` is the project root: `bynkc` then checks and tests the project's
`[paths] include` trees (by default `src/` and `tests/`). With `source: src`,
`bynkc test` doesn't see a `tests/` directory.

### Minimum Bynk versions

| Feature | Bynk |
| --- | --- |
| A directory `source` for the format check | 0.307.0 |
| `bynkc test` on Windows | 0.309.0 |
| Annotations on the right file when `working-directory` or `source` isn't `.` | 0.309.3 |

Before 0.309.3, `bynkc check` named each file relative to the directory it was
given, so with the default `source: src` an annotation pointed at
`greeting.bynk` instead of `src/greeting.bynk`, and GitHub couldn't place it on
the changed line. The check still failed the step; only the annotation was lost.

### A directory `source` before 0.307.0

The format check runs `bynkc fmt --check <source>`. `bynkc fmt` accepts a
directory from **Bynk 0.307.0**
([accuser/bynk#1753](https://github.com/accuser/bynk/issues/1753)); before
that it took files only, and a directory `source` failed the format step with
"Is a directory". On an older version, either pass a single file as `source` or
turn the format check off with `format: "false"`.

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
| `source`            | `src`          | Path passed to `bynkc` (file or directory). A directory needs Bynk 0.307.0 or later for the format check. |
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
