---
name: exec-obsidian-cli
description: >-
  Drives Obsidian from the terminal via the `obsidian://` URI scheme or the
  `obsidian-cli` npm package: open vaults, open/search notes, create notes,
  and trigger commands. Use when the user mentions Obsidian, obsidian-cli,
  obsidian:// URI, opening a vault from CLI, or scripting note creation.
---

# Obsidian CLI

> Requirement: the Obsidian desktop app must be installed **and running** for any of these commands to take effect. The CLI does not run headlessly—it sends URIs/commands that the running app handles. If the app is closed, launch it first (`open -a Obsidian` on macOS) and wait for the workspace to load before issuing further commands.

## Install (option A: npm CLI)

```bash
npm install -g obsidian-cli
obsidian --help
```

The `obsidian-cli` package wraps the URI scheme; the Obsidian app still needs to be running.

## Install (option B: URI scheme only)

No install needed—any shell that can call `open` (macOS), `xdg-open` (Linux), or `start` (Windows) can drive Obsidian via `obsidian://` URIs while the app is running.

## Common operations

Use placeholders such as `<VAULT_NAME>`, `<NOTE_PATH>`, `<QUERY>`—never hardcode personal vault names or absolute note paths into this repo.

### Open a vault

```bash
# Via obsidian-cli
obsidian open --vault <VAULT_NAME>

# Via URI scheme (macOS)
open "obsidian://open?vault=<VAULT_NAME>"
```

### Open a specific note

```bash
obsidian open --vault <VAULT_NAME> --path "<NOTE_PATH>"

# URI equivalent
open "obsidian://open?vault=<VAULT_NAME>&file=<NOTE_PATH_URL_ENCODED>"
```

### Search inside a vault

```bash
obsidian search --vault <VAULT_NAME> --query "<QUERY>"

# URI equivalent
open "obsidian://search?vault=<VAULT_NAME>&query=<QUERY_URL_ENCODED>"
```

### Create or append to a note

```bash
# New note (fails if file exists, unless --overwrite/--append is supported)
obsidian new --vault <VAULT_NAME> --path "<NOTE_PATH>" --content "<MARKDOWN>"

# URI: create note with content
open "obsidian://new?vault=<VAULT_NAME>&file=<NOTE_PATH>&content=<CONTENT_URL_ENCODED>"

# URI: append to existing note
open "obsidian://new?vault=<VAULT_NAME>&file=<NOTE_PATH>&append=true&content=<CONTENT_URL_ENCODED>"
```

### Trigger a workspace command

```bash
# Run a registered command id (e.g. editor:toggle-source)
open "obsidian://advanced-uri?vault=<VAULT_NAME>&commandid=<COMMAND_ID>"
```

The `advanced-uri` form requires the Advanced URI community plugin; without it, stick to the built-in actions above.

## URL encoding

Spaces, slashes inside paths, and Markdown content must be URL-encoded when passed through `obsidian://`:

```bash
python3 -c "import urllib.parse,sys; print(urllib.parse.quote(sys.argv[1]))" "<RAW_STRING>"
```

`obsidian-cli` handles encoding for you; raw `open` / `xdg-open` does not.

## Agent workflow

1. Confirm Obsidian is running (`pgrep -x Obsidian` on macOS, or check via `ps`); if not, launch it and wait for the vault to open.
2. Confirm `<VAULT_NAME>` with the user—do not guess.
3. Prefer `obsidian-cli` for scripts; fall back to URI scheme for one-off shell calls.
4. For batch note creation, generate the content locally, then invoke `obsidian new` per file to keep encoding consistent.

## Troubleshooting

| Symptom | Check |
|---------|--------|
| Nothing happens after running command | Obsidian app not running; launch it and retry |
| `vault not found` | Vault name must match exactly (case sensitive); list vaults via the Obsidian app's vault switcher |
| URI special chars break | URL-encode the value; quote the whole URI in the shell |
| `command not found: obsidian` | `npm install -g obsidian-cli`, ensure global npm bin is on `PATH` |
| URI scheme not handled | Open Obsidian once manually so the OS registers the handler |
