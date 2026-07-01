# run-ai-run

Curated, public-safe **summaries, guides, runbooks, reviews, and self-contained apps** for working
with Claude Code and agent tooling. Everything here is scrubbed of personal information before it is
published, by an automated gate (gitleaks + path/email redaction) that runs on every push.

> Named after *"Run, Forrest, Run!"* — keep the agent running.

## Reading surface

The site is a bilingual [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) build,
served by GitHub Pages:

- English (default) → https://garyhedgeme.github.io/run-ai-run/
- Русский → https://garyhedgeme.github.io/run-ai-run/ru/

Sections: **Summaries**, **Guides**, **Runbooks**, **Reviews**, plus a **Blog** (articles + short notes).

## Repository layout

| Path | What |
|------|------|
| `docs-en/` | English content (default language, served at `/`) |
| `docs-ru/` | Russian content (served at `/ru/`) |
| `mkdocs.en.yml` / `mkdocs.ru.yml` | one Material config per language (blog-safe multilingual via `extra.alternate`) |
| `requirements.txt` | pinned toolchain — `mkdocs-material[imaging]` |
| `.github/workflows/deploy.yml` | builds both languages into one tree, deploys via GitHub Pages Actions |
| `.github/workflows/publish.yml` | the scrub-gate (no personal info reaches the public mirror) |

Each language is a separate single-language build (HTML5 allows one `lang` per document); the header
language selector interlinks them. This keeps the Material **blog** plugin working, which is
incompatible with the single-build `mkdocs-static-i18n` approach.

## Build locally

```bash
pip install -r requirements.txt
mkdocs build -f mkdocs.en.yml    # -> ./site      (/)
mkdocs build -f mkdocs.ru.yml    # -> ./site/ru   (/ru/)
mkdocs serve -f mkdocs.en.yml    # live preview
```

---

Published from a private engine; only this curated mirror is public.
