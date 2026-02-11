# OpenLambda GitHub Pages — MyST Markdown Site

This repo deploys automatically to GitHub Pages using [Jupyter Book](https://jupyterbook.org)
with [MyST Markdown](https://myst-parser.readthedocs.io/).

---

## File Structure

```
.
├── index.md                        ← Homepage content (MyST Markdown)
├── _config.yml                     ← Jupyter Book site settings
├── _toc.yml                        ← Table of contents (add more pages here)
├── requirements.txt                ← Python deps for the build
├── .gitignore
└── .github/
    └── workflows/
        └── deploy.yml              ← GitHub Actions auto-deploy workflow
```

---

## One-Time Setup (do this once per repo)

### 1. Clone the repo and copy these files in

```bash
git clone https://github.com/open-lambda/open-lambda.github.io.git
cd open-lambda.github.io

# Copy all files from this bundle into the repo root, then:
git add .
git commit -m "Add MyST Markdown Jupyter Book site"
git push
```

### 2. Enable GitHub Pages with GitHub Actions as the source

Go to the repo on GitHub:

1. **Settings** → **Pages** (left sidebar)
2. Under **Source**, select **GitHub Actions** (not "Deploy from a branch")
3. Save

That's it. Every push to `main` will now rebuild and redeploy the site automatically.

---

## Local Preview

```bash
pip install -r requirements.txt
jupyter-book build .
# Open in browser:
open _build/html/index.html
```

---

## Adding More Pages

1. Create a new `.md` file, e.g. `worker.md`
2. Add it to `_toc.yml`:

```yaml
format: jb-book
root: index
chapters:
  - file: worker
```

3. Push — the site rebuilds automatically.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Build fails: `jupyter-book not found` | Make sure `requirements.txt` lists `jupyter-book>=1.0.0` |
| Pages shows 404 | Confirm **Settings → Pages → Source** is set to **GitHub Actions** |
| Old content still showing | Hard-refresh browser (`Ctrl+Shift+R`) or wait ~1 min for CDN |
| Action fails with permissions error | Go to **Settings → Actions → General → Workflow permissions** and enable "Read and write permissions" |