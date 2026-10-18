# sndb-docs

Documentation for [sndb](https://github.com/loopy5418/sndb), published at <https://loopy5418.github.io/sndb-docs/>.

Built with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/).

## Local preview

```sh
python -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Then open <http://127.0.0.1:8000/sndb-docs/>.

## Deploying

Every push to `main` builds the site and deploys it to GitHub Pages (`.github/workflows/deploy.yml`). Pull requests are built with `--strict` but not deployed.

One-time setup: in the repository's **Settings → Pages**, set **Source** to **GitHub Actions**.
