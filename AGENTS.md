# AGENTS.md

## Purpose

This fork makes Papermorph first-class for OpenAI Codex while preserving the upstream Claude layout for compatibility.

## Canonical Codex skill

- The Codex / Agent Skills source of truth is `skills/papermorph/`.
- `.claude/skills/papermorph/` is retained as a legacy upstream snapshot. Do not edit it for normal Codex work.
- When intentionally syncing a newer upstream Papermorph release, port the relevant changes into `skills/papermorph/` and review Codex-specific orchestration before replacing anything.

## Repository rules

- Never commit source PDFs or rendered/extracted source pages. Keep `*.pdf` and `books/*/pages/` private and ignored.
- Generated web books live under `site/`; book state belongs under `books/<book>/`.
- Keep reusable workflow guidance in `skills/papermorph/SKILL.md`; keep detailed supporting guidance in `references/`, repeatable commands in `scripts/`, and templates/runtime assets in `assets/`.
- Prefer small, targeted changes. Do not rewrite the engine when a chapter-local fix is sufficient.

## Codex workflow

1. Read `skills/papermorph/SKILL.md` for Papermorph tasks.
2. Keep intake and the pilot chapter in the primary thread.
3. After pilot approval, use fresh chapter-scoped subagents sequentially when available.
4. The coordinator owns shared state: `BOOK.md`, `chapters.md`, `UNITS`, and inter-chapter navigation.
5. Never run two agents that can edit the same chapter or shared state concurrently.
6. Use separate worktrees for truly independent parallel maintenance tasks.

## Validation

For skill/package changes:
- ensure `skills/papermorph/SKILL.md` has valid YAML front matter with `name` and `description`;
- verify referenced files exist under `skills/papermorph/`;
- run Python syntax checks for changed scripts;
- run targeted browser/review checks when engine, templates, or chapter behavior changes;
- inspect `git diff --check` before committing.

For a generated book, follow the delivery pass in `skills/papermorph/references/review.md`.
