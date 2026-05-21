---
name: exec-cloudinary-cli
description: >-
  Uses Cloudinary CLI (cld): configure credentials, verify setup, upload with
  background removal, and download transformed images. Use when the user mentions
  Cloudinary, cld, image upload, or background removal.
---

# Cloudinary CLI

## Install and verify

```bash
pip3 install cloudinary-cli
cld config
```

`cld config` should show the configured `cloud_name` (value comes from the user's environment).

## Credentials (local only)

Set via environment or shell profile—never commit real values:

```bash
export CLOUDINARY_CLOUD_NAME=<CLOUD_NAME>
export CLOUDINARY_API_KEY=<API_KEY>
export CLOUDINARY_API_SECRET=<API_SECRET>
# Or:
export CLOUDINARY_URL=cloudinary://<API_KEY>:<API_SECRET>@<CLOUD_NAME>
```

Ping API (replace placeholders):

```bash
curl -u "<API_KEY>:<API_SECRET>" \
  "https://api.cloudinary.com/v1_1/<CLOUD_NAME>/ping"
```

## Upload with AI background removal

Requires the Cloudinary account to have **AI Background Removal** enabled.

```bash
cld uploader upload <LOCAL_IMAGE_PATH> \
  background_removal=cloudinary_ai \
  folder=<FOLDER> \
  public_id=<PUBLIC_ID>
```

## Download transformed asset

```bash
curl -o <OUTPUT.png> \
  "https://res.cloudinary.com/<CLOUD_NAME>/image/upload/e_background_removal/<PUBLIC_ID>.png"
```

## Batch workflow pattern

1. Upload each source image with `background_removal=cloudinary_ai` and distinct `public_id`.
2. Download PNGs via transformation URL into a local assets directory the user specifies.
3. Do not hardcode cloud names, API keys, or machine-specific paths in this repo.

## Troubleshooting

| Symptom | Check |
|---------|--------|
| `cld` not found | `pip3` user bin on `PATH`; reinstall `cloudinary-cli` |
| Auth errors | `cld config` and env vars |
| Background removal fails | Feature enabled on Cloudinary account; correct `public_id` in URL |
