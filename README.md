# Selium User Guide

Selium's user guide, built by [`mdBook`](https://github.com/rust-lang/mdBook).

## Editing

To render the book whilst making changes, mdBook has a built in web server to help:

```bash
mdbook serve --open
```

## Deployment

The rendered book is deployed to GitHub Pages using the `pages.yml` workflow. Any push to `main`
will trigger this deployment.
