# markdown-notion-sync

A 200-line bash script that syncs a local Markdown directory tree to a Notion
page hierarchy. Folders become Notion pages; files become child pages; re-runs
update in place.

```
markdown-notion-sync <parent-page-id> <root-dir> [--state <file>]
```

## What it does

```
docs/                          ┐
├── intro.md                   │
├── product/                   ├──→  Notion: each folder is a page,
│   ├── README.md              │      each file is a child page,
│   ├── spec.md                │      nesting preserved
│   └── metrics.md             │
└── engineering/               │
    └── runbook.md             ┘
```

- Each `.md` file becomes a child page under its parent directory's page.
- Each subdirectory becomes a folder page. If the folder has a `README.md`,
  that file's content + frontmatter become the folder page's body, title, and
  icon.
- A state file (`.markdown-notion-sync.json` at the root by default) maps relative
  paths to Notion page IDs so re-runs update in place — no duplicates.

## Frontmatter

Optional YAML at the top of any `.md` file. Only two keys are read; everything
else is ignored.

```yaml
---
title: Custom title          # overrides H1 / filename
icon: 🧪                      # emoji icon
---
```

## Title resolution

For each file or folder, in order:

1. Frontmatter `title:`
2. First `# H1` in the body (and that H1 is stripped from the body)
3. Filename stem (`metrics.md` → `metrics`) or directory name

When the title comes from the H1, the H1 line is removed from the body so it
doesn't render twice (Notion already shows the title at the top of the page).
When it comes from frontmatter, the H1 is left alone as a body heading.

## Auth

Two options, in order of preference:

1. `ntn login` — uses the OS keychain.
2. `export NOTION_API_TOKEN=secret_…` from an internal integration created at
   <https://www.notion.so/profile/integrations>.

Either way, **the parent page must be explicitly shared with the integration**
that auth represents (`···` menu → Connections → add your integration).

## Requirements

- [`ntn`](https://developers.notion.com/docs/cli) ≥ 0.13
- `jq`
- `awk` (any POSIX awk)
- bash 4+ (for associative arrays in your own customisations — the script
  itself doesn't require it)

## Example

```bash
$ markdown-notion-sync 36171f7a-db20-80d1-91d1-e434429dde8b ./docs
+ intro.md  →  36171f7a-db20-8198-9bf5-d998f505a1e8
+ roadmap.md  →  36171f7a-db20-81a3-bf22-c21cf4f60d44
+ engineering/  →  36171f7a-db20-81dd-8cd4-df68fbe5b5a3
+ engineering/runbook.md  →  36171f7a-db20-814d-838c-e50983f9a2a2
+ product/  →  36171f7a-db20-819c-a8bf-d442c04758a5
+ product/metrics.md  →  36171f7a-db20-81a1-a3c6-e159105f352b
+ product/spec.md  →  36171f7a-db20-81d3-a2af-d7d76984fc3b

$ markdown-notion-sync 36171f7a-db20-80d1-91d1-e434429dde8b ./docs   # re-run
↻ intro.md
↻ roadmap.md
↻ engineering/
↻ engineering/runbook.md
↻ product/
↻ product/metrics.md
↻ product/spec.md
```

## Known limitations

- **Renames orphan pages.** Renaming `intro.md` → `overview.md` locally
  creates a new Notion page and leaves the old one untouched. Clean up by hand
  (or by deleting the state-file entry and the orphan page).
- **Deletions don't propagate.** Removing a file locally won't trash the
  corresponding page; same as renames.
- **Folder-with-README bodies are fully replaced each run.** If you edit a
  folder page's notes directly in Notion, those edits will be overwritten on
  the next sync. Folder pages *without* a README are left alone after their
  initial creation — Notion's automatic child-page listing takes over.
- **Frontmatter is parsed simply.** Only single-line scalar values for `title`
  and `icon`. No multi-line strings, no other keys.
- **Markdown files only.** `.md` only; images and other assets are not synced.
- **Hidden files and directories are skipped** (anything starting with `.`).
- **One Notion workspace per state file.** The state file holds IDs from a
  specific workspace. Don't try to sync the same root to two workspaces with
  one state file.

## License

MIT or whatever you want. It's 200 lines of bash, copy what's useful.
