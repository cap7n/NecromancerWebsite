# How to Edit This Wiki

Every page is a plain Markdown file in the `docs/` folder of the wiki repo. Push to `main` and the
site rebuilds itself in ~1 minute. No local tools required.

!!! warning "Repo not created yet"
    `repo_url` and `site_url` in `mkdocs.yml` are placeholders following the TowerDrop convention.
    Create the GitHub repo, push, enable Pages, then fix those two values and the links below.

## Quickest way (browser)

1. Click the **pencil icon** (top right of any page): it opens the file on GitHub.
2. Edit, then **Commit changes**.
3. Wait a minute, refresh the site.

## Local way (git)

```bash
git clone https://github.com/cap7n/necromancer-docs.git
```

Edit files in `docs/`, then commit and push. Done.

## Adding a new page

1. Create `docs/<section>/<name>.md`
2. Add it to the `nav:` list in `mkdocs.yml`
3. Push

## Optional: live preview on your machine

Only if you want to see changes before pushing (needs Python):

```bash
pip install mkdocs-material
```

```bash
mkdocs serve
```

Then open `http://127.0.0.1:8000`. Auto-reloads as you edit.

## Formatting cheatsheet

- `# Title`, `## Section`: headings
- `**bold**`, `*italic*`, `` `code` ``
- `[link text](../game/combat.md)`: link to another page (relative path)
- Tables: `| a | b |` rows with `|---|---|` under the header
- Task lists: `- [ ]` and `- [x]` render as checkboxes
- Callout boxes:

```
!!! note "Title"
    Indented content becomes a highlighted box.
```

`note`, `tip`, `warning`, `bug`, `danger` all work as box types.

- Status pills: `<span class="pill todo">TODO</span>` — types are
  `done` `wip` `todo` `idea` `check` `risk` `parked`.
- Option blocks (for A/B/C design choices):

```
<div class="opt rec" markdown>
### Option A — the recommended one
Text.
</div>
```

Drop `rec` for non-recommended options.

## House rules

1. **The wiki records decisions, it doesn't replace making them.** Undecided things get a
   `!!! warning "OPEN"` box or an entry in [Open Questions](open-questions.md).
2. When a decision is made, add it to the [Decision Log](decisions.md) **with the why**, and delete
   it from Open Questions.
3. Prefer editing an existing page over creating a new one.
4. **Be honest about status.** At concept stage almost everything is a proposal. Writing proposals
   as though they were decisions is the fastest way to make this wiki useless.
