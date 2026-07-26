# Necromancer Design Wiki

The single point of truth for **Necromancer** (working title): design, systems, and project state.

Source is plain Markdown in `docs/`. Push to `main` and GitHub Actions rebuilds and
deploys the site in ~1 minute (see `.github/workflows/deploy.yml`).

Live site: https://cap7n.github.io/necromancer-docs/

> **Setup not finished:** the repo URL above is a placeholder following the TowerDrop
> convention. Create the GitHub repo, then fix `repo_url` / `site_url` in `mkdocs.yml`
> and the link above.

## Edit locally (optional)

```
pip install mkdocs-material
mkdocs serve
```

Open http://127.0.0.1:8000. See [How to Edit This Wiki](docs/project/how-to-edit.md).

## Status

The project is at **concept stage**. Almost nothing here is decided — most pages lay out
the option space and are marked as open. The master list of what needs deciding is
[Open Questions](docs/project/open-questions.md).
