# bynk-ci

The standard Bynk quality gate in one step: format check, type check, and tests.
Installs the toolchain via [`setup-bynk`](https://github.com/bynk-lang/setup-bynk)
and annotates `bynk` diagnostics inline on the pull request via a problem
matcher.

**It needs Bynk 0.512.9 or later** (setup-bynk v2 installs `bynk` and `bynk-lsp`, not `bynkc`). Pin an earlier bynk-ci release for older Bynk versions.

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
          version: 0.403.3
          source: .
```

`source: .` is the project root: `bynk` then checks and tests the project's
`[paths] include` trees (by default `src/` and `tests/`). With `source: src`,
`bynk test` doesn't see a `tests/` directory.

### Minimum Bynk versions

| Feature | Bynk |
| --- | --- |
| A directory `source` for the format check | 0.307.0 |
| `bynk test` on Windows | 0.309.0 |
| Annotations on the right file when `working-directory` or `source` isn't `.` | 0.309.3 |
| The format check on a Windows (CRLF) checkout without a `.gitattributes` | 0.309.4 |

Before 0.309.3, `bynk check` named each file relative to the directory it was
given, so with the default `source: src` an annotation pointed at
`greeting.bynk` instead of `src/greeting.bynk`, and GitHub couldn't place it on
the changed line. The check still failed the step; only the annotation was lost.

Before 0.309.4, `bynk fmt --check` treated CRLF line endings as a formatting
difference, and Git for Windows checks files out with CRLF by default
(`core.autocrlf`). On an older version, keep `.bynk` files LF on every checkout
with a `.gitattributes` line, as this repository does:

```gitattributes
*.bynk text eol=lf
```

`bynk new` writes that file for new projects from 0.309.4.

### A directory `source` before 0.307.0

The format check runs `bynk fmt --check <source>`. `bynk fmt` accepts a
directory from **Bynk 0.307.0**
([bynk-lang/bynk#1753](https://github.com/bynk-lang/bynk/issues/1753)); before
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
| `version-file`      | `""`           | A file holding an exact version, such as `BYNK_VERSION`, relative to the workspace. Use this or `version`, not both. |
| `repository`        | `bynk-lang/bynk` | Release repository. |
| `working-directory` | `.`            | Directory to run in. |
| `source`            | `src`          | Path passed to `bynk` (file or directory). A directory needs Bynk 0.307.0 or later for the format check. |
| `format`            | `true`         | Run `bynk fmt --check`. |
| `check`             | `true`         | Run `bynk check --format short`. |
| `test`              | `true`         | Run `bynk test`. |
| `github-token`      | `${{ github.token }}` | Token for setup-bynk. |

## How annotations work

`bynk check --format short` emits one diagnostic per line as
`path:line:col: severity[category]: message`. The bundled
[`bynk-problem-matcher.json`](bynk-problem-matcher.json) parses that form so
errors and warnings appear directly on the changed lines in the PR.

## License

Licensed under either of [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE) at
your option.
