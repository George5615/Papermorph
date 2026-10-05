---
name: papermorph
description: Turn reference PDFs into animated, narrated, interactive static web books. Use for book planning, chapter storyboards, SVG/HTML animation, exercises, narration, chapter continuation, visual fixes, and delivery review.
---

# Papermorph

Build an animated interactive web book from a reference PDF. Prefer visual explanations that change over time: rearrange, split, balance, count, compare, reveal, and test. Each chapter is one 1600x900 SVG stage plus controls; beats pair narration clips with timed actions. Quick checks and final practice stay inside the visual scene.

This skill is harness-neutral but is written to work cleanly with OpenAI Codex. When the user explicitly asks for Papermorph, invoke this workflow as `$papermorph` where the Codex host supports explicit skill invocation.

Follow the pipeline forward, entering at the requested stage for existing work. User requests override defaults. Default to code/SVG artwork; record any other asset choices in the book conventions. Re-create explanations, examples, and drawings rather than copying source layout verbatim.

Keep source PDFs, rendered source pages, and extracted text outside the static output folder and out of version control (`.gitignore`: `*.pdf`, `books/*/pages/`). Deliver a self-contained book folder with a cover, contents, chapters, and local preview instructions.

## Project layout

```
books/<book>/
  book.pdf, sections.json      source and page map (private)
  pages/chNN/text.md, *.png    extracted reference material (private)
  BOOK.md                      current scope, conventions, helper index
  chapters.md                  chapter map and status
  chapters/chNN.md             brief storyboard, errata and feedback
content/<book>/chNN/           narration.<lang>.json and TTS cache
tools/<book>/                  targeted tests when needed
site/                          static output; private sources stay outside
  <book>/index.html            cover + contents        (assets/templates/book.html)
  <book>/unit-art.js           unit sketches           (assets/templates/unit-art.js)
  <book>/lib/engine.js, .css   this book's engine copy (assets/engine/)
  <book>/chNN/index.html       a lesson                (assets/templates/chapter.html)
  <book>/chNN/audio/<lang>/    beat MP3s + timings.js
```

`SKILL` means this skill's absolute folder. `<book>` is the book slug and `<lang>` is its primary language code. Existing books keep their paths.

Runtime needs: `uv`, `ffmpeg`/`ffprobe`, Edge TTS network access, and Playwright Chromium (`uv run --with playwright playwright install chromium`). Preview with `python3 -m http.server 8765 -d site`.

## Pipeline

Resolve intake, book map, and pilot decisions with the user; reuse answers and approvals already given.

1. **Intake.** Choose a slug; create `books/<book>/BOOK.md` from the template. Record readers, tone, primary language, scope, and assets/guide. Ask for missing decisions together and give sensible defaults.
2. **Split.** Run `uv run --with pymupdf SKILL/scripts/outline.py books/<book>/book.pdf`; use `--level N -o books/<book>/sections.json` at chapter depth. For a PDF without bookmarks, inspect `split_pages.py ... --pages 1-12 --out books/<book>/pages`, then write the map from its contents (schema: `outline.py --help`). Match the map to the contents; extract chapter text with `split_pages.py ... --sections books/<book>/sections.json --out books/<book>/pages --text-only`.
3. **Book map.** Read contents and chapter openings; inspect more text where scope is unclear. Create `chapters.md` with columns `# | Title | Unit | Minutes | Status` using `planned`, `ready`, or `user-approved`. Put colours and candidate recurring visual models in `BOOK.md`. Confirm the list and visual approach with the user.
4. **Initialize.** Follow [site.md](references/site.md#start-a-book-folder) to copy the book engine, cover/contents, and chapter skeleton before making the pilot.
5. **Pilot.** Make chapter 1 through the chapter loop. Deliver it for user review and record approved conventions in `BOOK.md`. Implement helpers as needed; share one when another chapter will reuse it.
6. **Remaining chapters and finish.** Produce chapters sequentially. Complete cover text, contents, and unit sketches using [site.md](references/site.md). Deliver the static book folder, its local preview entry URL, and unresolved issues for human review.

## Chapter loop

1. **Read.** Read this chapter's `pages/chNN/text.md`; render its pages with `split_pages.py ... --only chNN` when diagrams or missing text require page images. Select the ideas and examples to teach.
2. **Storyboard.** Write `chapters/chNN.md`: beat list, visual changes, narration triggers, and questions. Use [authoring.md](references/authoring.md) for content conventions. Verify examples and answers used in the finished chapter.
3. **Narration.** Write `content/<book>/chNN/narration.<lang>.json`, then run `uv run --with edge-tts SKILL/scripts/tts.py content/<book>/chNN/narration.<lang>.json site/<book>/chNN/audio/<lang>`.
4. **Beats.** Implement the storyboard in `site/<book>/chNN/index.html`. API and replay invariants are in [engine.md](references/engine.md). Use the book's approved look and interactions.
5. **Delivery pass.** Follow [review.md](references/review.md) once for loading, opening blanks, overlap/cropping, text density, audio/beat timing, and colour consistency. Correct clear defects locally, recheck the affected part, then deliver. Report unresolved problems.
6. **Register.** Add the ready chapter to `UNITS`, set the previous chapter's `CHAPTER.next`, update its row in `chapters.md`, and update `chapters/chNN.md`. Promote new shared decisions to `BOOK.md`. The coordinator owns this step when Codex subagents are used.

## Codex orchestration

- Keep intake, book map, and the pilot chapter in the primary Codex thread so user decisions remain visible.
- After pilot approval, prefer one fresh chapter-scoped Codex subagent per chapter when the host supports subagents. Run chapters **sequentially**, not concurrently, because later chapters inherit visual and interaction conventions from earlier chapters.
- Give a worker only the chapter number, relevant `chapters.md` row, source/output paths, this skill path, `BOOK.md`, its chapter note, and current user requests. Do not forward the full conversation when file-based state is enough.
- The worker performs chapter-loop steps 1-5 and returns a compact summary: changed files, delivery result, errata, and any reusable helper or convention.
- The coordinator performs step 6, updates shared state, and only then dispatches the next chapter.
- If parallel Codex agents are ever used for independent maintenance work, isolate their edits in separate worktrees. Never allow two agents to modify `BOOK.md`, `chapters.md`, `UNITS`, or the same chapter simultaneously.
- If subagents are unavailable, use the same chapter loop in the current agent and rely on file-based state instead of retaining large conversational context.

## Context discipline

- Read only the reference needed for the current step.
- Use the engine API; inspect individual implementation functions only when needed.
- Copy assets rather than reading their full source as instructions.
- Keep `BOOK.md` limited to current conventions and the helper index. Chapter status, errata, and historical feedback stay in chapter-specific files.
- Write from the storyboard in one pass. Use contact sheets and short command summaries; retain failure details.
- Run targeted interaction tests when engine behavior or grading changes, as described in [review.md](references/review.md).
- Do not declare a chapter complete until the delivery pass succeeds or unresolved defects are explicitly reported.
