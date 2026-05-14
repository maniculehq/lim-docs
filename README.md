# lim documentation

This directory contains the source for the lim documentation site, built with [Mintlify](https://mintlify.com).

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

- `docs.json` — site configuration (theme, navigation, branding)
- `index.mdx` — landing page
- `quickstart.mdx` — getting-started guide
- `guides/` — task-oriented walkthroughs
- `api-reference/` — API reference pages
