# Astrore Docs

Astrore's standalone product website and developer documentation.

The website is intentionally kept separate from the Astrore desktop and Agent project. It is a
static site with no Node.js, Python, or Rust build requirement.

## Local preview

From this directory:

```powershell
npx vite site --host 127.0.0.1 --port 1422
```

You can also serve the `site` directory with any static HTTP server.

## Deployment

Push this repository to GitHub with the default branch named `main`, then enable GitHub Pages with
GitHub Actions as the source. Changes under `site/` are deployed automatically.
