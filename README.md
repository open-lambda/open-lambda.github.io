# OpenLambda Documentation Site

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Deploy](https://github.com/open-lambda/open-lambda.github.io/actions/workflows/deploy.yml/badge.svg)](https://github.com/open-lambda/open-lambda.github.io/actions/workflows/deploy.yml)

Source for the **OpenLambda** documentation and blog, published at
**<https://open-lambda.github.io/>**.

The site is built with [Jupyter Book](https://jupyterbook.org) (v1, Sphinx-based) using
[MyST Markdown](https://myst-parser.readthedocs.io/) and the `sphinx_book_theme`, with
[ABlog](https://ablog.readthedocs.io/) powering the blog. Every push to `master` is built
and deployed to GitHub Pages automatically.

> [!NOTE]
> This repo hosts the **website**, not the OpenLambda runtime. For the serverless platform
> itself, see [open-lambda/open-lambda](https://github.com/open-lambda/open-lambda).

---

## Repository layout

```
.
├── index.md                  ← Homepage
├── worker.md                 ← Worker documentation
├── applications/             ← Application case studies
│   ├── index.md
│   └── ag-forecasting-api.md
├── blog/                     ← Blog (ABlog)
│   ├── index.md
│   └── post/                 ← One file per post: YYYY-MM-DD-slug.md
├── _config.yml               ← Jupyter Book / Sphinx settings
├── _toc.yml                  ← Table of contents (register new pages here)
├── _static/                  ← Logo, favicon, and other assets
├── requirements.txt          ← Pinned Python build dependencies
├── CONTRIBUTING.md           ← How to add pages, posts, and authors
├── LICENSE                   ← Apache 2.0
└── .github/workflows/
    └── deploy.yml            ← Build + deploy to GitHub Pages
```

---

## Local preview

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter-book build .
open _build/html/index.html        # macOS; use xdg-open on Linux
```

> [!IMPORTANT]
> Dependencies are pinned to the Jupyter Book **v1** stack. Do **not** bump `jupyter-book`
> to 2.x — v2 is a MyST-CLI rewrite that is incompatible with this repo's `_config.yml`,
> `_toc.yml`, and `sphinx_book_theme` setup.

A successful build ends with `build succeeded.` and writes HTML to `_build/html/`.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to add a documentation page, write a blog
post (front matter, file naming, excerpts), and register a blog author. In short:

- **New page** → add the `.md` file and register it in [`_toc.yml`](_toc.yml).
- **New blog post** → add `blog/post/YYYY-MM-DD-slug.md` with `blogpost: true` front
  matter (ABlog collects it automatically — no `_toc.yml` entry needed).

Please run `jupyter-book build .` locally and confirm it succeeds without warnings before
opening a pull request.

---

## Deployment

[`.github/workflows/deploy.yml`](.github/workflows/deploy.yml) builds the book and publishes
`_build/html` to GitHub Pages on every push to `master`. Changes to docs-only files
(`README.md`, `CONTRIBUTING.md`, `LICENSE`) do not trigger a deploy.

---

## License

Licensed under the **Apache License 2.0** — see [LICENSE](LICENSE) for the full text.
