---
name: cloudflare-wrangler
description: >-
  Runs Cloudflare Wrangler via npx: login, whoami, Pages vs Workers deploy,
  and CI token setup patterns. Use when the user mentions Wrangler, Cloudflare
  Workers, Cloudflare Pages, or deploy/auth errors with wrangler.
---

# Cloudflare Wrangler

## Run Wrangler

Prefer project-local execution (no global install required):

```bash
npx wrangler <command>
```

## Auth

OAuth tokens are stored locally (typical path: `~/.wrangler/config/default.toml`). Tokens refresh automatically; re-login when auth fails:

```bash
npx wrangler login
npx wrangler whoami
```

For CI, create an API token in the Cloudflare dashboard (profile → API tokens) with permissions matching the resource (e.g. Workers Edit). Store as `<CLOUDFLARE_API_TOKEN>` and set `<CLOUDFLARE_ACCOUNT_ID>` in the CI secret store—never commit values.

## Common commands

```bash
# Pages: list projects
npx wrangler pages project list

# Pages: list deployments
npx wrangler pages deployment list --project-name <PROJECT_NAME>

# Pages: create project
npx wrangler pages project create <PROJECT_NAME>

# Pages: deploy static assets
npx wrangler pages deploy <DIST_DIR> --project-name <PROJECT_NAME> --branch <BRANCH>

# Worker: deploy (when wrangler.toml defines a Worker, not Pages)
npx wrangler deploy
```

## Workers vs Pages

| Config signal | Deploy command |
|---------------|----------------|
| `[assets]` or Worker service in `wrangler.toml` | `npx wrangler deploy` |
| Cloudflare Pages project | `npx wrangler pages deploy ...` |

Using `pages deploy` against a Worker-only project causes failures.

## CI checklist (generic)

- [ ] `CLOUDFLARE_API_TOKEN` secret exists in the repo/org secret store
- [ ] `CLOUDFLARE_ACCOUNT_ID` matches the target account
- [ ] Workflow command matches resource type (Worker vs Pages)

## Agent workflow

1. `npx wrangler whoami` before deploy or list operations.
2. Read `wrangler.toml` to decide Worker vs Pages.
3. On auth errors, suggest `npx wrangler login` locally; in CI, verify token permissions and account ID secrets.
