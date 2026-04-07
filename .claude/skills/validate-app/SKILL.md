---
name: validate-app
description: >-
  Run comprehensive validation on a commerce app package before PR submission. Use this skill
  immediately before ANY submission attempt, when users mention "validate", "check app", "verify",
  "ready to submit", or after packaging an app. Also trigger proactively BEFORE calling submit-pr
  to catch errors early and save CI/CD cycles. This is a REQUIRED pre-submission step - don't let
  users submit without validation. Checks directory structure, manifest format, SHA256 hashes,
  impex XML syntax, and runs the complete CONTRIBUTING.md checklist. Use whenever debugging
  validation failures or import errors - it will identify the root cause.
---

# Validate Commerce App Package

Run comprehensive validation checks on a commerce app before submitting a PR.

## Step 1: Identify the app to validate

Gather or infer:
- Domain (one of: `tax`, `payment`, `shipping`, `gift-cards`, `ratings-and-reviews`, `loyalty`, `search`, `address-verification`, `analytics`, `approaching-discounts`, `fraud`)
- App name (e.g., `avalara-tax`)
- Version to validate (or use latest ZIP in directory)

**Folder Structure:** Apps must be at `{domain}/{appName}/` where `{appName}` matches the "id" field. Installation URL: `https://raw.githubusercontent.com/{owner}/{repo}/{tag}/{domain}/{appName}/{zipFileName}`

## Step 2: Validate ZIP file exists

Check that ZIP file exists at `<domain>/<appName>/<appName>-v<version>.zip`

## Step 3: Compute and verify SHA256

1. Compute SHA256 hash:
   ```bash
   shasum -a 256 <domain>/<appName>/<appName>-v<version>.zip
   ```

2. Read the root manifest at `commerce-apps-manifest/manifest.json`

3. Find your app's entry in the appropriate domain array (e.g., `tax`, `payment`, `gift-cards`)

4. Compare computed hash with `sha256` field in the manifest entry

5. **CRITICAL**: Hashes must match exactly

## Step 4: Validate root manifest entry

Check that the app entry in `commerce-apps-manifest/manifest.json` contains all required fields:

**Required fields:**
- `id` - must match app name
- `name` - human-readable display name
- `description` - app description
- `iconName` - icon filename (e.g., `avalara.png`)
- `domain` - must be one of: `tax`, `payment`, `shipping`, `gift-cards`, `ratings-and-reviews`, `loyalty`, `search`, `address-verification`, `analytics`, `approaching-discounts`, `fraud`
- `type` - must be `"app"`
- `provider` - must be `"thirdParty"`
- `version` - semantic version (e.g., `0.2.8`)
- `zip` - must match `<appName>-v<version>.zip`
- `sha256` - must match actual ZIP hash

**Validation steps:**
1. Verify entry exists in correct domain array
2. Verify all required fields are present and non-empty
3. Verify `domain` is a valid value (hyphen-case)
4. Verify version format follows semantic versioning
6. Verify zip filename matches the actual file
7. Verify SHA256 matches computed hash

## Step 5: Validate ZIP contents and detect architecture

Extract and inspect the ZIP structure:

1. List ZIP contents without extracting:
   ```bash
   unzip -l <domain>/<appName>/<appName>-v<version>.zip
   ```

2. Check for common issues:
   - [ ] Single root folder named `commerce-<appName>-app-v<version>/`
   - [ ] No registry path prefixes (no `tax/`, `domain/`, etc.)
   - [ ] No junk files (`.DS_Store`, `__MACOSX`, `Thumbs.db`, hidden files)
   - [ ] No duplicate directory trees
   - [ ] commerce-app.json exists at root

3. **Detect app architecture** by checking directory structure:
   
   ```bash
   # Check for storefront-next directory
   HAS_UI=$(unzip -l <zip-file> | grep -c "storefront-next/" || echo 0)
   
   # Check for cartridges directory
   HAS_BACKEND=$(unzip -l <zip-file> | grep -c "cartridges/" || echo 0)
   ```
   
   Determine architecture:
   - **UI-only**: Has `storefront-next/`, NO `cartridges/`
   - **Backend-only**: Has `cartridges/`, NO `storefront-next/`
   - **Fullstack**: Has both `storefront-next/` AND `cartridges/`

4. Verify required files based on architecture:
   
   **All apps:**
   - [ ] `commerce-app.json`
   - [ ] `README.md`
   - [ ] `app-configuration/tasksList.json`
   
   **UI-only OR Fullstack:**
   - [ ] `storefront-next/src/extensions/<appName>/target-config.json`
   - [ ] `storefront-next/src/extensions/<appName>/index.ts`
   - [ ] `storefront-next/src/extensions/<appName>/locales/` with at least en-US
   
   **Backend-only OR Fullstack:**
   - [ ] `cartridges/site_cartridges/` with at least one cartridge
   - [ ] At least one cartridge contains `package.json` with `"hooks"` field
   - [ ] `impex/install/` directory
   - [ ] `impex/uninstall/` directory (for cleanup)

## Step 6: Validate commerce-app.json

Extract and read `commerce-<appName>-app-v<version>/commerce-app.json`:

**Required fields:**
- `id` - must match app name
- `name` - display name
- `description`
- `domain` - must match root manifest domain
- `version` - must match root manifest version
- `publisher.name`
- `publisher.url`
- `publisher.support`

**Validation steps:**
1. Verify version in commerce-app.json matches root manifest version
2. Verify id matches app name
3. Verify domain matches
4. Check that publisher fields are valid URLs

## Step 7: Validate storefront files (UI-only and Fullstack apps only)

**Skip this step if Backend-only architecture.**

Extract and validate Storefront Next extension at `storefront-next/src/extensions/<appName>/`:

**Extension structure:**
- [ ] `target-config.json` with valid `components` array (targetId, path, order)
- [ ] `index.ts` barrel file exports all public components/hooks/providers
- [ ] All three required locales: `locales/en-US/`, `en-GB/`, `it-IT/` with translations.json

**TypeScript files (.ts/.tsx):**
- [ ] Apache 2.0 copyright header (first thing in file)
- [ ] `'use client'` directive (after copyright, before imports)
- [ ] Use inline type imports: `import React, { type ReactElement }`
- [ ] Components use default export

**Code quality:**
- [ ] No hardcoded strings (use `useTranslation()`)
- [ ] No hardcoded Tailwind colors (use `@theme` variables)
- [ ] No `console.log` statements
- [ ] No array indices as React keys

## Step 8: Validate impex files (Backend-only and Fullstack apps only)

**Skip this step if UI-only architecture.**

Extract the ZIP and run comprehensive impex validation:

```bash
find commerce-<appName>-app-v<version>/impex/ -name "*.xml" -exec xmllint --noout {} \;
```

For detailed validation rules, read `references/impex-validation.md` which covers:
- Services XML (install/uninstall pairs, credentials, timeouts)
- Site preferences (metadata, attribute naming, data types)
- Custom objects (key attributes, storage scope, retention)
- Cross-file validation (ID matching between install/uninstall)
- Common errors (XML encoding, naming conventions)

**Key checks:**
- [ ] All XML files pass xmllint validation
- [ ] Services use dotted notation, no hardcoded credentials
- [ ] Install/uninstall service IDs match exactly
- [ ] Attribute IDs use camelCase with app-name prefixes
- [ ] SITEID placeholder used in preferences (not actual site ID)

Report specific file paths and line numbers for any failures.

## Step 9: Check for catalog.json

- If this is an **existing app** (catalog.json already exists):
  - [ ] Do NOT modify catalog.json - CI will update it
  - [ ] Verify catalog.json exists but unchanged in PR

- If this is a **brand new app** (no catalog.json):
  - [ ] Verify catalog.json is created with INIT values:
    ```json
    {
      "latest": {
        "version": "INIT",
        "tag": "INIT"
      },
      "versions": []
    }
    ```

## Step 10: Validate translations in manifest

**CRITICAL:** Verify app translations were added to commerce-apps-manifest directory.

### Translation Validation

1. Check if app entry exists in en-US.json (minimum requirement):
   ```bash
   jq '."<appName>"' commerce-apps-manifest/translations/en-US.json
   ```

2. Verify translation structure:
   - [ ] App entry exists in `commerce-apps-manifest/translations/en-US.json`
   - [ ] Entry has `name` field matching app display name
   - [ ] Entry has `description` field matching app description
   - [ ] Translation structure is valid JSON: `{"<appName>": {"name": "...", "description": "..."}}`

3. Check other locale files (best practice):
   ```bash
   for locale in de fr es it ja ko nl pl pt zh_CN zh_TW ar_MA; do
     jq '."<appName>"' commerce-apps-manifest/translations/${locale}.json
   done
   ```
   
   - [ ] App entry present in all 13 locale files (or at least en-US.json)
   - [ ] All locale entries have identical structure (name and description fields)

### Icon Validation

Verify the app ZIP contains an icon that the CI workflow will extract:

1. Check root manifest for iconName:
   ```bash
   jq '.[] | .[] | select(.id == "<appName>") | .iconName' commerce-apps-manifest/manifest.json
   ```

2. Verify icon exists and matches:
   ```bash
   # Get iconName from manifest
   ICON_NAME=$(jq -r '.[] | .[] | select(.id == "<appName>") | .iconName' commerce-apps-manifest/manifest.json)
   
   # Check ZIP contains matching icon
   unzip -l <domain>/<appName>/<appName>-v<version>.zip | grep "icons/$ICON_NAME"
   ```
   
   - [ ] Icon file exists in ZIP's `icons/` directory
   - [ ] Icon filename matches the `iconName` field in root manifest exactly
   - [ ] Icon is PNG (512x512px recommended) or SVG format

The CI workflow extracts icons from the ZIP by filename, so any mismatch will break icon references.


**Common issues:**
- Translations not added to en-US.json
- Icon filename doesn't match `iconName` in manifest
- Icon missing from ZIP's `icons/` directory

## Step 11: Run final PR checklist

**All architectures:**
- [ ] App is located at `<domain>/<appName>/` where `<appName>` matches the "id" field in manifest.json
- [ ] ZIP name: `<appName>-v<version>.zip`, no junk files, single root folder
- [ ] SHA256 hash matches root manifest entry
- [ ] commerce-app.json version matches root manifest
- [ ] catalog.json included for new apps only (INIT values)
- [ ] tasksList.json exists with merchant post-installation tasks
- [ ] Icon exists in ZIP at `commerce-<appName>-app-v<version>/icons/` (CI will extract automatically)
- [ ] Translations in `commerce-apps-manifest/translations/en-US.json` (minimum)

**UI-only or Fullstack:**
- [ ] target-config.json with valid targetIds and paths
- [ ] Apache 2.0 headers + 'use client' directives in all .ts/.tsx files
- [ ] Three locales: en-US, en-GB, it-IT with translations.json
- [ ] No hardcoded strings, colors, or console.log statements

**Backend-only or Fullstack:**
- [ ] All impex XML files pass xmllint validation
- [ ] Services: no hardcoded credentials, install/uninstall pairs match
- [ ] Site preferences: camelCase IDs with app prefix, SITEID placeholder
- [ ] Cartridge package.json has "hooks" field, hooks.json uses explicit paths

## Report validation results

Provide a clear summary:
- **✅ PASS** - All validations passed, ready for PR
- **❌ FAIL** - List specific issues found with file paths and line numbers
- Provide fix recommendations for each issue
