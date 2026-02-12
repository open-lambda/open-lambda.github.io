# OpenLambda GitHub Pages — MyST Markdown Site

This repo deploys automatically to GitHub Pages using [Jupyter Book](https://jupyterbook.org)
with [MyST Markdown](https://myst-parser.readthedocs.io/) as the technical documentation of Open Lambda.

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

## Local Preview

```bash
pip install -r requirements.txt
jupyter-book build .
# Open in browser:
open _build/html/index.html
```
