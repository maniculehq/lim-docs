# lim documentation

This directory contains the source for the Limrun documentation site, built with [Mintlify](https://mintlify.com).

## Develop locally

Install the Mintlify CLI:

```bash
npm i -g mint
```

From the root of this directory (where `docs.json` lives), run:

```bash
mint dev
```

The site will be available at `http://localhost:3000`.

## Structure

- `docs.json`: Mintlify site configuration (theme, navigation tabs, branding).
- `introduction.mdx`, `quickstart.mdx`: top-level Getting Started pages.
- `agents/`: CLI and MCP guides for coding agents.
- `ios/`: build-with-xcode, run-simulator, pr-previews, in-app-purchases.
- `android/`: run-emulator.
- `testing/`: appium, playwright.
- `platform/`: embed-simulator, asset-storage.
- `reference.mdx`: SDK reference (auth, errors, resources × CRUD).
- `images/console/`: console screenshots embedded across pages.
- `_internal/`: notes that are NOT shipped (e.g. dogfood findings).
