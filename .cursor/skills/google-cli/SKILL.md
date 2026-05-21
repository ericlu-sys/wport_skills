---
name: google-cli
description: >-
  Operates Google Cloud CLI (gcloud): check active account, switch accounts,
  ADC scopes, and read Google Sheets via curl or Sheets API. Use when the user
  mentions gcloud, Google Sheets, spreadsheets, ADC, or Google API 403/scope errors.
---

# Google CLI

## Before any API or deploy work

1. Confirm active account:

```bash
gcloud auth list
gcloud config get-value account
```

The row with `*` is the active account.

2. Switch default account if needed:

```bash
gcloud config set account <ACCOUNT_EMAIL>
gcloud auth list
```

3. For a single command with a specific account:

```bash
gcloud --account=<ACCOUNT_EMAIL> auth print-access-token
```

## Application Default Credentials (ADC)

If Sheets or Drive returns `ACCESS_TOKEN_SCOPE_INSUFFICIENT`, refresh ADC with the scopes that match the project's needs. Treat `<SCOPES>` as a configurable variable—do not assume read-only.

Decide scopes per project:

| Need | Sheets scope | Drive scope |
|------|--------------|-------------|
| Read only | `.../auth/spreadsheets.readonly` | `.../auth/drive.readonly` |
| Read + write | `.../auth/spreadsheets` | `.../auth/drive` (or `drive.file`) |

Pattern:

```bash
# Adjust scopes per project config (e.g. project_config.json) before running.
gcloud auth application-default login --scopes=<SCOPES>
```

Examples:

```bash
# Read-only Sheets + Drive
gcloud auth application-default login \
  --scopes=https://www.googleapis.com/auth/spreadsheets.readonly,https://www.googleapis.com/auth/drive.readonly,https://www.googleapis.com/auth/cloud-platform

# Read + write Sheets + Drive
gcloud auth application-default login \
  --scopes=https://www.googleapis.com/auth/spreadsheets,https://www.googleapis.com/auth/drive,https://www.googleapis.com/auth/cloud-platform
```

Verify:

```bash
gcloud auth application-default print-access-token
```

## Read Google Sheets

### Option A: CSV export (when sheet is readable with current permissions)

```bash
curl -L -s "https://docs.google.com/spreadsheets/d/<SPREADSHEET_ID>/export?format=csv&gid=<GID>"
```

### Option B: Sheets API (requires correct scopes)

```bash
TOKEN="$(gcloud auth print-access-token)"
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://sheets.googleapis.com/v4/spreadsheets/<SPREADSHEET_ID>/values/<SHEET_NAME>!A1:AA200"
```

## Credentials on disk

- OAuth client secrets and service account JSON live outside this repo; use env vars or paths the user provides locally.
- Never paste credential file contents or filenames into chat or commits.

## Troubleshooting

| Symptom | Check |
|---------|--------|
| Wrong Google identity | `gcloud auth list`, then `gcloud config set account` |
| 403 / insufficient scope | Re-run ADC login with scopes matching the operation (read vs write) |
| Token works but API fails | Confirm API is enabled in the correct GCP project |
| `gcloud: command not found` (Apple Silicon Mac) | `export CLOUDSDK_PYTHON="/opt/homebrew/bin/python3"` and retry; add the export to your shell rc to persist |
