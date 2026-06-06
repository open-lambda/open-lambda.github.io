# Contributing to the OpenLambda Site

This site is a [Jupyter Book](https://jupyterbook.org) (v1, Sphinx-based) written in
[MyST Markdown](https://myst-parser.readthedocs.io/). Pushing to `master` triggers the
GitHub Actions workflow in [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml),
which builds the book and deploys it to GitHub Pages.

## Local preview

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter-book build .
open _build/html/index.html
```

```{note}
Keep dependencies pinned to the Jupyter Book **v1** stack in `requirements.txt`. Do **not**
bump `jupyter-book` to 2.x — v2 is a MyST-CLI rewrite that is incompatible with this repo's
`_config.yml` / `_toc.yml` and `sphinx_book_theme` setup.
```

## Adding a page

1. Create a Markdown file (e.g. `applications/my-app.md`).
2. Register it in [`_toc.yml`](_toc.yml) so it appears in the navigation.
3. Add SEO metadata at the top of the file:

   ```yaml
   ---
   myst:
     html_meta:
       description: "One-sentence summary used as the meta description."
       keywords: "OpenLambda, serverless, ..."
   ---
   ```

## Adding a blog post

1. Create a file under `blog/post/` named `YYYY-MM-DD-short-slug.md`. The date prefix is the
   convention used for ordering — it does not need to match the `date:` field exactly, but
   keep them consistent.
2. Use this front matter (do **not** add blog posts to `_toc.yml` — ABlog collects them
   automatically from the `blog/post/` directory):

   ```yaml
   ---
   blogpost: true
   date: 2026-05-18
   author: jane-doe, john-smith
   category: Case Studies          # e.g. Releases, Case Studies
   tags: openlambda, serverless    # comma-separated
   language: en
   myst:
     html_meta:
       description: "One-sentence summary for search engines and link previews."
       keywords: "OpenLambda, ..."
   ---

   # Post Title
   ```

3. The first `# Heading` is the post title; the first paragraph(s) become the excerpt shown
   on the blog index.

## Registering an author

`author:` values are slugs. Map each slug to a display name and profile URL under
`blog_authors` in [`_config.yml`](_config.yml) so posts render a friendly name and link:

```yaml
blog_authors:
  jane-doe:
    - "Jane Doe"
    - "https://example.edu/jane"
```

An unregistered slug still works, but renders as the raw slug with no link.

## Before opening a PR

- Run `jupyter-book build .` locally and confirm it ends with `build succeeded` and no
  warnings.
- Check that internal links resolve and external links are correct.
