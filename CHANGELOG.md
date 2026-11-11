# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.3.0] - 2026-11-11

### Added

- "Audit: find missed days": compares existing summaries against every work day found in a date range, reports days with no surviving session record, and backfills approved days, one summary per run of consecutive days, named so each files under the right month.
- `sources_not_checked` frontmatter key and a matching line in the report to the Operator, so a source that was not checked is visible.
- Hard rule: never skip a source silently.
- The validation pass now also checks coverage by repeating a quick session search for threads, transcripts, or repositories the draft missed.

### Changed

- Step 1 is now "Find every session in the span, then read each one". It searches bb threads in every bb project, archived and hidden included, whether or not the current session runs in bb, and Claude Code transcripts in every project folder, matched on tool calls that name the project rather than on plain text. Dated files the work produced count as evidence of a work day when no session record survives.
- Step 1 notes that Claude Code deletes transcripts after `cleanupPeriodDays` (30 days by default), so older sessions may survive only in bb.

## [0.2.0] - 2026-11-11

### Added

- Step 1, "Read the session record": the bb thread (when the session runs in bb), the Claude Code transcript, and git and `gh` for every repository touched, read from disk so a compacted context does not lose facts. The summary covers the session's real span, which can run across several days.
- Separate validation pass: a fresh-context subagent checks the draft against the session record and reports errors and omissions before the file is written. When no subagent is available, the agent runs the same check itself.
- Mandatory "Operator decisions and corrections" section, with short exact quotes.
- Mandatory "Actions outside the repo" ledger with an "Approved by" column.
- YAML frontmatter (date, span, project, repositories, sources) and a "Covers:" line under the title.
- "Open items and pending questions" section with a status on each item, including questions put to the Operator that were not answered.
- One summary per project when a session touches several projects.
- A client- and scope-heavy example in `references/example-summaries.md`.
- Voice-guide rules for naming the actor on every decision and outward action, and for marking unverified claims.

### Changed

- "No hedging" is now "no vague hedging": a claim that cannot be sourced is omitted or marked `(unverified)`.
- "Never ask for input" is now "never ask about content". The one permitted question is the output destination, when a project or private rule requires confirming it.
- Git commands run with `git -C <repo>` for each repository touched, across the session's span, in place of a fixed 36-hour window in the current directory.

## [0.1.0] - 2026-07-11

### Added

- Initial `build-day-summary` agent skill: `SKILL.md` (input-gathering procedure, section-selection rules, the two structural patterns, and the hard rules governing tone and factual precision) and the `references/` tone-and-structure calibration files (`voice-guide.md`, `section-templates.md`, `example-summaries.md`).

[0.3.0]: https://github.com/84emllc/claude-build-day-summary-skill/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/84emllc/claude-build-day-summary-skill/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/84emllc/claude-build-day-summary-skill/releases/tag/v0.1.0
