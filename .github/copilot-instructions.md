# Dalamud Assets – Copilot Instructions

This is a static asset repository for [Dalamud](https://github.com/goatcorp/Dalamud), a plugin framework for Final Fantasy XIV. There is no build step — all files are served directly from GitHub raw URLs.

## Repository Structure

- **`asset.json`** — Asset manifest consumed by Dalamud at runtime. Lists every downloadable file with its raw GitHub URL, local `FileName`, and optional SHA-1 `Hash` (uppercase hex).
- **`UIRes/bannedplugin.json`** — The plugin blocklist. Dalamud blocks any plugin whose name and version match an entry.
- **`UIRes/loc/dalamud/`** — Localization files per language (e.g. `dalamud_de.json`). These are managed through Crowdin and should **not** be edited manually.
- **`loc_for_upload/Dalamud_Localizable.json`** — Source strings exported from the Dalamud project for Crowdin upload. Do not edit manually.
- **`UIRes/`** — Fonts (OFL/Apache licensed) and UI images used by Dalamud.
- **`package.zip`** — Auto-generated on every push to `master` by the `version-increment` workflow. Do not edit or commit manually.

## CI Workflows

| Workflow | Trigger | What it does |
|---|---|---|
| `validate-json.yml` | Pull request | Validates `UIRes/bannedplugin.json` is valid JSON and has no duplicate `Name` entries |
| `version-increment.yml` | Push to `master` | Increments `Version` in `asset.json`, rebuilds `package.zip`, commits & pushes, then POSTs to a refresh endpoint |
| `dalamud-loc.yml` | Daily / manual | Exports strings from the Dalamud source, uploads to Crowdin, downloads translations, opens a PR |

## Conventions

### Blocking a plugin (`UIRes/bannedplugin.json`)

```json
{
  "Name": "PluginInternalName",
  "AssemblyVersion": "1.2.3.4",
  "Reason": "Optional human-readable reason"
}
```

- **`Name`**: The plugin's `InternalName` from its manifest. For plugins from a **custom (third-party) repository**, SHA-256 hash the `InternalName` and use the hash in **ALL CAPS** instead.
- **`AssemblyVersion`**: The ban applies to **all versions ≤ this value**. Setting it to `99999.99999.99999` blocks all versions permanently.
- **`Reason`**: Optional but encouraged. Omit if there is nothing meaningful to say.
- No duplicate `Name` entries are allowed — CI will fail the PR.

### Adding/updating an asset (`asset.json`)

- `Url` must point to the raw GitHub URL for the file on the `master` branch.
- `Hash` is a **SHA-1 uppercase hex** digest. It is optional but recommended for binary assets (fonts, images). Localization JSON files do not include a hash.
- After merging, the `Version` integer is auto-incremented by CI — do not increment it manually.

### Localization files

- Source language is English. Translations live in `UIRes/loc/dalamud/dalamud_<two_letter_code>.json`.
- The Crowdin workflow uses the language code mapping: `%two_letters_code%` → filename suffix (e.g. `de`, `fr`, `ja`, `zh`, `tw`, `ko`, `ru`).
- To add a new language, add a new entry to `asset.json` and ensure the corresponding Crowdin language is configured.

## Common Tasks

**Block a plugin:**
1. Edit `UIRes/bannedplugin.json`, add a new entry (keep the array sorted is not required but helpful).
2. Open a PR — CI will validate for duplicates and JSON validity.
3. After merge, instruct affected users to relaunch.

**Add a new asset:**
1. Add the file to `UIRes/`.
2. Compute the SHA-1 hash: `shasum -a 1 UIRes/<filename>` (output is lowercase — convert to uppercase).
3. Add an entry to `asset.json` with `Url`, `FileName`, and `Hash`.
