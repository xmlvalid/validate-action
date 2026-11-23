# xmlvalid GitHub Action

Validates XML files and e-invoices in your repository with [xmlvalid.com](https://xmlvalid.com)
and marks the failing lines in the pull request: XSD 1.0, DTD and RelaxNG schemas, your own
Schematron rules, and the business rules of EN 16931, Peppol BIS and PINT, XRechnung,
Factur-X / ZUGFeRD and national e-invoicing formats from the
[schema catalog](https://xmlvalid.com/schemas).

```yaml
name: XML
on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: xmlvalid/validate-action@v1
        with:
          files: |
            invoices/**/*.xml
          schema: ubl-2.1-invoice
          api-key: ${{ secrets.XMLVALID_API_KEY }}
```

E-invoices of several formats, each checked against the profile it names (Peppol, XRechnung,
Factur-X…), with an HTML report kept as an artifact:

```yaml
      - uses: xmlvalid/validate-action@v1
        with:
          files: invoices/**/*.xml
          schema: auto
          report: xmlvalid-report.html
          api-key: ${{ secrets.XMLVALID_API_KEY }}
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: xmlvalid-report
          path: xmlvalid-report.html
```

Make a key under [Dashboard → API keys](https://xmlvalid.com/dashboard/api) and store it
as the `XMLVALID_API_KEY` secret. Each file is one API validation (Free: 100 a month,
Pro: 10,000).

| Input | |
|---|---|
| `files` | files or globs, one per line or space-separated |
| `urls` | public URLs, one per line |
| `schema` | a catalog slug (`atom-1.0`), a saved schema (`@orders-v3`), or `auto` for the profile each e-invoice names |
| `schema-file` | an XSD, DTD or RelaxNG file in the repository |
| `rules` | your own Schematron (`.sch`) file in the repository, run after the schema check |
| `rules-version` | pins such as `peppol-bis-3-ubl@3.0.21`, one per line: fail when the business rules are at another release |
| `doctype` | `true` to validate against each document's DOCTYPE |
| `fail-on-warnings` | `true` to fail on warnings |
| `report` | also write an HTML report of every document to this path (keep it with `actions/upload-artifact`) |
| `share` | `true` to also make a public link to the report, open for 30 days (Pro); in the job summary and the `report-url` output |
| `api-key` | required |
| `api-url` | the API base URL, default `https://xmlvalid.com/api/v1` |
| `cli-version` | the `@xmlvalid/cli` version to run, default `1` |

A schema that spans several files (imports, includes) is easiest to save once in the
dashboard and use by its `@…` name.

The step runs [`@xmlvalid/cli`](https://www.npmjs.com/package/@xmlvalid/cli) with
`npx`; the runner needs Node.js 20 or newer (GitHub-hosted runners have it).
