# build-day-summary

[![CI](https://github.com/84emllc/claude-build-day-summary-skill/actions/workflows/ci.yml/badge.svg)](https://github.com/84emllc/claude-build-day-summary-skill/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Agent Skill](https://img.shields.io/badge/type-agent%20skill-8250df.svg)](https://agentskills.io/specification)

An agent skill for generating an end-of-session "build day summary": a dense, factual, narrative record of what shipped, broke, and was learned in a working session, auto-gathered from git/gh and the session's own context with no back-and-forth questions.

Built by [84EM](https://84em.com) for use with Claude Code and other agent harnesses that load [Agent Skills](https://agentskills.io/specification).

## What is in here

| Path | What it is | License |
|------|------------|---------|
| `SKILL.md` | The skill: reading the session record, section selection, the two structural patterns (chronological vs. outcome-first), the separate validation pass, and the hard rules governing tone and factual precision | MIT |
| `references/voice-guide.md` | Tone calibration: right-vs-wrong phrasing examples | MIT |
| `references/section-templates.md` | Exact templates for each summary section | MIT |
| `references/example-summaries.md` | Excerpts from four example summaries for shape reference | MIT |

## Install

**Claude Code (personal skill):** clone and symlink into your skills directory.

```bash
git clone https://github.com/84emllc/claude-build-day-summary-skill.git
ln -sfn "$(pwd)/claude-build-day-summary-skill" ~/.claude/skills/build-day-summary
```

The skill is then discoverable by its `name` and `description` frontmatter. No build step is required.

## Usage

Ask the agent for a build day summary, day summary, or EOD summary at the end of a working session inside a project repo. It reads the session record from disk (the bb thread when the session runs in bb, the Claude Code transcript, and git and `gh` for every repository touched), picks a structure that fits the session's shape, and writes a factual markdown file with no marketing voice, no emojis, and no em dashes. Claims it could not verify are marked as unverified.

Every summary records the Operator's decisions and corrections, and a ledger of actions taken outside the repository (cards, messages, documents, settings, merges). A session that spans several projects produces one summary per project.

**Validation pass.** Before the file is written, a separate agent with fresh context checks the draft against the session record and reports errors and omissions. When the harness can't start a subagent, the agent runs the same check itself as a second pass.

**Output location defaults to the current repo root.** `SKILL.md` writes the summary as `YYYY-MM-DD-HHMMSS-build-day-summary.md` at the root of the repo you run it in. Point the "Write the file" step at wherever you keep session notes instead (a local `docs/` folder, a notes repo, your own knowledge base) if you prefer. A project or private rule can name the destination; if that rule requires confirming it, the agent asks that one question and nothing else.

**Voice-calibration references.** `references/voice-guide.md`, `references/section-templates.md`, and `references/example-summaries.md` (cited throughout `SKILL.md`) calibrate the tone and structure of a summary. The worked examples are illustrative and use invented placeholder names, not any real engagement.

## Licensing

MIT. See [LICENSE](./LICENSE).
