# go-ruby-opentype/docs

Documentation for [go-ruby-opentype](https://github.com/go-ruby-opentype/opentype),
built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and
versioned with [mike](https://github.com/jimporter/mike). Served at
<https://go-ruby-opentype.github.io/docs/>.

## Build locally

```sh
pip install -r requirements.txt
mkdocs serve
```

## Deploy

Pushing to `main` runs `.github/workflows/docs.yml`, which deploys the `0.1`
version (aliased `latest`) via `mike` to the `gh-pages` branch.
