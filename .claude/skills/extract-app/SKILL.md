---
name: extract-app
description: >-
  Extract a commerce app ZIP for development or modification. Use when users want to "extract",
  "unzip", "look at", "inspect", "explore", or "see what's inside" any app ZIP.
---

# Extract Commerce App

Extract a commerce app ZIP to its app directory.

## Usage

```bash
/extract-app <app-name>
/extract-app <domain>/<app-name>
```

## Step 1: Locate ZIP

```bash
# App name only → search all domains
find . -name "*<app-name>-v*.zip" -type f | head -1

# Domain/app → direct lookup
ls <domain>/<app-name>/*-v*.zip
```

## Step 2: Extract

```bash
cd <domain>/<app-name>/
unzip -t <app-name>-v<version>.zip
unzip -q <app-name>-v<version>.zip
```

Creates: `commerce-<app-name>-app-v<version>/`

If directory already exists, ask user before overwriting.

## Step 3: Confirm

```
✅ Extracted to: <domain>/<app-name>/commerce-<app-name>-app-v<version>/

Use /package-app to repackage when done.
⚠️  Delete extracted directory before committing.
```
