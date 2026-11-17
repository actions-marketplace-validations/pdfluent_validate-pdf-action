# Validate PDF/A with PDFluent

A GitHub Action that fails your workflow when a PDF is not PDF/A conformant. It installs the [PDFluent](https://pdfluent.com) command-line tool, checks every PDF you point it at and prints the violated rules. Files stay on the runner: nothing is uploaded and no account is needed.

```yaml
name: PDF/A
on: [push, pull_request]

jobs:
  pdfa:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pdfluent/validate-pdf-action@v1
        with:
          files: docs/**/*.pdf
          profile: a2b
```

A failing file looks like this in the log, and as an error annotation on the file:

```text
FAIL fixtures/invalid-not-pdfa.pdf
     [NOT_COMPLIANT] not compliant with PDF/A-a2b (4 violation(s))
       [6.6.2.1] No XMP metadata stream in catalog
       [6.1.7.1] endstream keyword not preceded by required end-of-line marker
       ...
1 checked, 1 failed
```

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `files` | `**/*.pdf` | Glob patterns, one per line, relative to `working-directory`. `**` matches across directories. |
| `profile` | `a2b` | `a1b`, `a2b` or `a3b`. |
| `version` | `1.4.0` | PDFluent CLI release to install. |
| `working-directory` | `.` | Where the patterns are resolved. |
| `fail-on-invalid` | `true` | Set to `false` to report without failing the step. |
| `allow-empty` | `false` | By default a pattern list that matches no file fails, so a typo cannot turn the check green. |

## Outputs

| Output | Meaning |
|---|---|
| `checked` | PDFs checked. |
| `failed` | PDFs that are not conformant or could not be read. |

The step also writes a table of results to the job summary.

## Notes

- **Runners:** Linux (x64, arm64) and macOS (arm64, x64). Windows runners are not supported yet.
- **Install:** the action downloads the release binary from [pdfluent-sdk releases](https://github.com/pdfluent/pdfluent-sdk/releases) and checks it against the published `SHA256SUMS` before it runs.
- **PDF/UA:** not offered. PDFluent CLI 1.4.0 accepts `--pdfua` but reports that PDF/UA validation is not exposed, so the action does not pretend to check it.
- **Scope:** the check covers PDF/A-1b, 2b and 3b conformance as the PDFluent engine reports it. It is not a replacement for a certification run if a standard or a contract asks for one.
- **Fixtures:** `fixtures/invalid-not-pdfa.pdf` is the plain sample PDF from [pdfluent/examples](https://github.com/pdfluent/examples). `fixtures/valid-pdfa-2b.pdf` is the same file converted with `pdfluent convert --to pdfa`.

## Licence

This action is MIT, see [LICENSE](LICENSE). The PDFluent CLI it installs is released under AGPL-3.0 or a commercial licence; see the [PDFluent licence page](https://pdfluent.com/license).
