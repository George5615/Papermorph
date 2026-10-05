<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="img/readme/logo_light.png">
    <source media="(prefers-color-scheme: light)" srcset="img/readme/logo.png">
    <img src="img/readme/logo.png" alt="Papermorph" width="600">
  </picture>
</p>

<p align="center">
  <strong>A Codex-ready Agent Skill that turns PDFs into animated interactive web books.</strong>
</p>

Papermorph turns reference PDFs into web books you can explore, listen to, and interact with. This fork makes the workflow first-class for **OpenAI Codex** while retaining the original Claude skill layout for upstream compatibility.

<p align="center">
  <a href="https://papermorph.diamonddoge.org/">Explore the original live bookshelf -></a>
  ·
  <a href="skills/papermorph/SKILL.md">Read the Codex Skill</a>
</p>

<p align="center">
  <a href="img/readme/papermorph-preview.mp4">
    <img src="img/readme/papermorph-preview.gif" alt="Real demo: bookshelf, animated lessons, and interactive quizzes" width="900">
  </a>
  <br>
  <a href="img/readme/papermorph-preview.mp4">Watch the original 75-second demo with narration</a>
</p>

```text
PDF -> Book plan -> Storyboards -> Narration -> Animation & quizzes -> Web book
```

## Install in Codex

The repository is packaged as a Codex-compatible plugin and includes its own marketplace entry.

Add the marketplace:

```bash
codex plugin marketplace add George5615/Papermorph
```

Then start Codex:

```bash
codex
```

Open the plugin browser:

```text
/plugins
```

Install **Papermorph**, start a new Codex chat, and invoke the skill:

```text
$papermorph Turn /path/to/book.pdf into an animated interactive web book.

Target readers: [your audience].
Start with one chapter for review.
```

Natural-language requests also work; the skill description lets Codex discover Papermorph for PDF-to-interactive-book tasks.

## Codex architecture

```text
AGENTS.md
plugin.json
.codex-plugin/
  plugin.json
.agents/
  plugins/
    marketplace.json
skills/
  papermorph/
    SKILL.md
    references/
    scripts/
    assets/
```

- `skills/papermorph/` is the canonical Codex / Agent Skills implementation.
- `AGENTS.md` contains repository-wide Codex rules and subagent coordination.
- `plugin.json` is the canonical portable Agent Plugins manifest.
- `.codex-plugin/plugin.json` is retained as a Codex compatibility manifest.
- `.agents/plugins/marketplace.json` makes the Git repository usable as a Codex marketplace source.
- `.claude/skills/papermorph/` is preserved as a legacy upstream snapshot rather than deleted.

The Codex workflow keeps intake and the pilot chapter in the main thread. After the pilot is approved, Codex may use fresh chapter-scoped subagents **sequentially**; the coordinator alone updates shared book state.

## Requirements

- Python 3
- `uv`
- `ffmpeg` / `ffprobe`
- Playwright Chromium
- network access for Edge TTS

The detailed workflow and commands are in [skills/papermorph/SKILL.md](skills/papermorph/SKILL.md).

## Try the examples locally

```bash
git clone https://github.com/George5615/Papermorph.git
cd Papermorph
python3 -m http.server 8765 -d site
```

Open [localhost:8765](http://localhost:8765/).

## Upstream

Original project: [DozenTwelve/Papermorph](https://github.com/DozenTwelve/Papermorph).

The original artwork, engine, templates, examples, and workflow are retained under the upstream MIT license. This fork changes the agent packaging and orchestration for Codex.

[MIT License](LICENSE)
