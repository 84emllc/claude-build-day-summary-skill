---
name: build-day-summary
description: Generate an end-of-session build day summary for the Operator. Use whenever the Operator asks for a "build day summary", "build summary", "day summary", "wrap up the day", "summarize today's work", "EOD summary", or any equivalent end-of-session writeup. Produces a dense, narrative, factually precise markdown file capturing what shipped, what broke, what was decided, what was done outside the repo, and what carries over. Finds every session on the project across all bb threads and Claude Code transcript folders, plus git and gh, lists any source it could not check, then has a separate agent check the draft and the coverage. Also audits a date range for days with no summary and backfills them. Asks the Operator nothing about content. Default invocation context is Claude Code at the end of a working session inside a project repo.
---

# Build Day Summary

A skill for generating the Operator's end-of-session build day summary in their established voice and structure. The summary is the durable, candid record of what actually happened: what shipped, what broke, what the Operator decided or corrected, what was done outside the repo, and what carries over. It is written for the Operator's future self and any maintainer who needs to pick up the work cold.

"The Operator" is the single human this skill serves: the engineer whose build day is being summarized, who drives the working session and edits the resulting draft. It is always the same person. Some older summaries called this person "the user"; name them "the Operator" instead. Reserve "user" for a WordPress user account, an OS user, or a genuine end-user of the software.

## Operating principles

The summary is **factual, dense, and narrative**. Every claim is traceable to a commit, a file, an error message, or an event in the session record. No marketing voice. No Claude attribution. No emojis. No em dashes (use commas, periods, colons, parens, or restructure).

**No vague hedging, and no unverified facts stated as facts.** A claim either comes from a source read for this summary or is marked `(unverified)`. "Probably fixed" is wrong. "Fixed in `a1b2c3d`" or "Fix applied; not re-tested on staging (unverified)" is right.

When in doubt, **prefer specific over general**. Include exact commit SHAs (short form, 7 chars), exact file paths, exact error strings in code formatting, exact test counts, exact PR numbers and URLs. Vague phrasing like "made some improvements to testing" is wrong; "90 tests / 316 assertions / 1 skipped, gated on `WP_TESTS_LOAD_UM=1`" is right.

The tone is **candid internal**, not external-facing. If something was a slog, say so. If a fix masked a real bug for hours, say that. If the agent got something wrong and the Operator corrected it, record the correction. The audience is the Operator and future maintainers, who benefit from honesty about what actually went wrong.

**Ask the Operator nothing about content.** This skill runs at end of session and produces a best-effort draft. The Operator will edit it. If a section's content is unclear, leave it out or mark it unverified rather than asking. The one permitted question is the output destination, and only when a project or private rule requires the agent to confirm it (see step 7).

## Workflow

### 1. Find every session in the span, then read each one

Facts come from the session record, not from memory. Context may have been compacted, and work on one project is often spread across several sessions, working directories, and threads. Find all of it before reading any of it.

**Set the span.** For an end-of-session summary, the span runs from the first activity on the project in the record to now; a session can run across several days, and the summary covers all of it. For an audit or backfill (see "Audit: find missed days"), the span is the date range the Operator named.

**Find the sessions.** Search every source below, not only the current session's. Match on what identifies the project: its repository names, local paths, project-board ID, knowledge-base client name, and project name.

1. **bb threads, in every bb project.** Run `bb thread list --json --include-hidden` and again with `--archived`, with no `--project` filter: work for one project is often done in a thread filed under another. Keep threads whose activity overlaps the span, then read each one with `bb thread log <id> --all --format verbose` (or `--json` when you need a timestamp on each turn) and keep those that touch the project. For the current thread, `bb thread show --self` also gives the title, project, and pull request status. `bb thread history <id>` returns only the newest 50 prompts, so don't use it here. Check bb even when the current session isn't running in bb (`BB_THREAD_ID` unset): the Operator may have worked on the project from bb at another time.
2. **Claude Code transcripts, in every project folder.** Search all JSONL files under `~/.claude/projects/`, not only the folder for the current working directory. Match on tool calls whose inputs name the project's paths, repositories, board, or client. Don't match on plain text: instruction and memory files that load into every session can name a project, and would make every transcript match. Transcripts hold every tool call and result, which bb's timeline condenses. Claude Code deletes them after `cleanupPeriodDays` (30 days by default), so older sessions may survive only in bb.
3. **git and gh** for each repository touched, using `git -C <repo>` and `gh --repo <owner/name>`.
4. **Dated files the work produced** (documents, exports, notes), when no session record survives. They show that a day had work, not what was decided.

**Record coverage.** List every source you checked and every source you could not check, with the reason (bb not installed, transcripts past retention, a command that failed). Both lists go in the frontmatter and in the report to the Operator. An unchecked source must be visible, never silent.

When sources disagree, the transcript and git win over memory. Dates and times come from timestamps in the record, converted to the Operator's time zone.

From the record, collect:

- The Operator's prompts, decisions, and corrections, with short exact quotes (these capture the "why" and the "what was tried first")
- Every action outside the repository: messages sent, cards or tickets created, moved, or commented on, KB or document writes, settings or permission changes, deployments, PR merges, anything another person can see
- Questions asked of the Operator, and whether each was answered
- Tool results showing errors that were debugged
- Files read and edited
- Any plan-mode plans or KB documents referenced or written

### 2. Gather git and gh state

For each repository touched in the session (not only the current directory), with `<start>` set to the session's start from step 1:

```bash
# Identity
git -C <repo> rev-parse --show-toplevel 2>/dev/null
git -C <repo> rev-parse --abbrev-ref HEAD 2>/dev/null

# Commit history across the session's span
git -C <repo> log --all --since="<start>" --pretty=format:'%h %ad %s%n%b%n---' --date=iso --name-status

# Working tree state
git -C <repo> status --short
git -C <repo> diff --stat HEAD 2>/dev/null

# Branches
git -C <repo> branch -vv

# Tags created recently
git -C <repo> tag --sort=-creatordate | head -5

# PRs (best-effort; may fail if gh is not authed or the remote is not GitHub)
gh pr list --repo <owner/name> --state all --limit 20 2>/dev/null
```

Capture stderr too. If a command fails, note the gap rather than filling it from memory.

### 3. Split by project

A summary covers one project. When the session touched several projects, write one summary per project, each holding only that project's work. Never combine projects into one summary. Work that belongs to no project (tooling, harness configuration, personal skills) goes with the project it was done for, or in its own summary when it stands alone.

### 4. Decide which sections apply

Mandatory (always include):

- Title line and "Covers:" line
- Lead paragraph (the arc of the session, one paragraph)
- "What changed, in order" (or equivalent narrative spine; see structure-selection rules below)
- Operator decisions and corrections
- Actions outside the repo
- Files touched / Files committed

When a mandatory section genuinely has nothing in it, write one line saying so ("No actions outside the repository."). For these sections, an explicit "none" is information: it tells the reader nothing was missed.

Optional (include only if there is real content):

- TL;DR or Released (high-level outcome bullets; include when the session produced a tagged release, a merged PR, or has 4+ distinct outcomes worth listing up top)
- Production bug section (include when a non-trivial bug was diagnosed and fixed)
- Test layer / Test infrastructure (include when tests were the main artifact)
- Environmental obstacles table (include when the session lost meaningful time to environment issues; use the `# | Symptom | Cause | Fix` table format)
- Domain-specific obstacle sections (e.g., the "OIDC obstacle course" pattern: group thematically related debugging into a named subsection)
- Verification at end of session (include when there's a clean end state to capture: command + result)
- Open items and pending questions (include when work or an unanswered question carries into the next session; each item has a status)
- Notable engineering decisions worth flagging (include when 2+ non-obvious technical decisions were made that a future maintainer should know)
- Workflow / process notes (include when meta-observations about tooling, hooks, agent dispatch, or collab patterns are worth capturing)
- File index for the next session (include for handoff-shaped sessions)

Skip empty optional sections entirely. Do not write "N/A" or "None this session" for them.

### 5. Choose a structure

Two structural patterns appear in the Operator's prior summaries, both valid:

**Pattern A: Narrative spine** (used when the session was a chronological build):

- Title and Covers line
- Lead paragraph
- TL;DR (bulleted)
- "What changed, in order" with numbered subsections
- Operator decisions and corrections
- Actions outside the repo
- Final state / verification
- Open items and pending questions
- Notable decisions
- Workflow notes
- File index

**Pattern B: Outcome-first** (used when the session was hardening / fixes / multiple PRs):

- Title and Covers line
- Lead paragraph
- Released (the outcome up front)
- Production bug (mechanism, fix)
- Test layer / Test infrastructure
- Themed subsections for other work
- Operator decisions and corrections
- Actions outside the repo
- Verification
- Files touched
- Carry-over

Pick whichever fits the actual shape of the session. Pattern A when the work built up something new across the session. Pattern B when the work was a tighter set of fixes/hardening shipped against existing code.

### 6. Write and validate the draft

Use `references/voice-guide.md` for tone calibration if needed. Use `references/section-templates.md` for the exact shape of each section.

**Frontmatter.** Start the file with YAML frontmatter so summaries can be searched and filed:

```yaml
---
date: YYYY-MM-DD
span: YYYY-MM-DD to YYYY-MM-DD
project: <project name>
repositories:
  - <owner/name>
sources:
  - bb thread <thread id>
  - transcript <session id>
sources_not_checked:
  - <source>: <reason>
---
```

Omit a key whose value is unknown. List under `sources` only what was read, and under `sources_not_checked` every source from step 1 that was not, with the reason. Omit `sources_not_checked` only when every source was checked.

**Title formats the Operator has used:**

- Older summaries use a literal em dash between project and date. **Do not use em dashes in new output.** Substitute a colon or hyphen.
- `# Build Day Summary - YYYY-MM-DD` (safe, hyphen)

Use the second format for new summaries: `# Build Day Summary - YYYY-MM-DD`. Or if the project name adds clarity: `# Build Day Summary: <Project> - YYYY-MM-DD`.

**Covers line.** Directly under the title, one line stating the span and what the summary includes and leaves out:

```text
Covers: 2026-04-29 to 2026-04-30, the example-sso plugin and its project board. Work on other projects in the same session is in their own summaries.
```

**Lead paragraph shape:** one paragraph, 2-4 sentences, capturing the session's arc. Start state implied or stated, then what shifted, then end state. Concrete (PR numbers, tag names, test counts ok here). Do not bullet.

Example shapes that work:

- "Single-day push on the `<plugin>` plugin. Started as <small thing>, surfaced <real bug>, and ended with <substantial outcome> and a v<x.y.z> release."
- "Single-session execution of an approved <N>-day rebuild plan for the <project>. Started with <starting state>; ended with <ending state>."
- "Single-session build of <thing> for <project>. The plugin itself was finished <when>; today's work added <what>, finishing with <verification state>."

**Numbered narrative sections** (Pattern A): use `### 1. <Short title>`, `### 2. <Short title>`. Each section starts with what triggered it (a request, an error, a discovery), then describes what was done, then names the artifact it produced (commit SHA, branch, file path).

**Name the actor** for every decision, reversal, and outward action: the Operator, the agent, a named subagent or reviewer, or a named third party. "The card was archived" is wrong; "The agent archived the card after the Operator ruled it out of scope" is right.

**Tables for environmental issues**: use the four-column format the Operator uses:

```text
| # | Symptom | Cause | Fix |
|---|---|---|---|
| 1 | <exact error string in backticks> | <one-line cause> | <one-line fix, code in backticks> |
```

**Files touched section**: group by purpose (Production / Tests / Docs / Infrastructure). Use bullet lists with one file per line and a parenthetical purpose suffix when the filename alone isn't self-explanatory:

```text
Production:
- `src/UltimateMember/UMRoleManager.php` (the fix)
- `src/Admin/HelpPage.php` (Developers tab content)

Tests / infrastructure:
- `tests/Integration/ProvisionerUMActiveTest.php` (new)
- `tests/Integration/ProvisionerTest.php` (skip-when-UM-loaded)
```

**Check the draft yourself.** Before validation, re-read every count, SHA, PR number, date, quote, and actor against the source it came from. Remove or mark `(unverified)` anything you cannot source.

**Separate validation pass.** Write the draft to a scratch file outside any repository. When the harness can start a subagent, dispatch one with fresh context and a read-only brief:

- Inputs: the draft path, the span, the project identifiers used in step 1, the bb thread IDs and transcript paths found, and the list of repositories.
- Task: check every factual claim in the draft against those sources, and check the record for anything important the draft leaves out, especially Operator decisions and corrections, actions outside the repo, unanswered questions, and reversals. Also check coverage: repeat a quick version of the step 1 search and report any bb thread, transcript, or repository in the span that the draft's sources miss.
- Output: a list of errors (claim, what the source says, where) and omissions. The subagent edits nothing.

Apply the corrections the sources confirm. Where the subagent and the draft disagree, settle it by reading the source, not by picking a side. When no subagent is available, run the same check yourself as a second pass against the sources, reading them fresh rather than from memory.

### 7. Write the file

Generate the filename from the session date and current time:

```bash
date +'%Y-%m-%d-%H%M%S-build-day-summary.md'
```

Default output location: the root of the current repository, using the filename generated above.

If the Operator keeps build day summaries somewhere specific (a `docs/` directory, a separate notes repo, or a knowledge base) and a project or private rule says so, write there instead. If that rule requires confirming the destination with the Operator, ask that one question and nothing else. When no rule applies, default to the repository root and tell the Operator where you wrote it.

After writing, tell the Operator:

- The filename created
- A one-sentence description of what's in it
- What the validation pass changed
- The sources not checked, with reasons
- Any sections you deliberately skipped (so they can flag if you misjudged)

Do not narrate the gathering process or the section-selection process. The output is the file.

## Audit: find missed days

When the Operator asks which days have no summary, or asks for a backfill:

1. List the existing summaries and the days each one covers, from its frontmatter `span` or its title.
2. Run the step 1 search over the requested range and list every day with work on the project, with the evidence for each: thread IDs, transcript files, commits, dated files. Count the Operator's prompts and the project tool calls per day so a passing mention isn't counted as a work day.
3. Compare the two lists. Report each missed day with its evidence, and say plainly which days have work but no surviving session record (only dated files or commits), since those can't be summarized accurately.
4. Backfill only the days the Operator approves. Write one summary per run of consecutive work days, run the full workflow for each (including the validation pass), and name each file with the last day of its stretch and the time of its last activity, so it files under the right month. Add "Backfilled <date> from <sources>." to the Covers line.

## Hard rules

- **Never use em dashes.** Substitute commas, periods, colons, parens, or restructure. This applies even when quoting older Operator docs that contain them.
- **Never use emojis.**
- **Never include Claude attribution** ("generated by Claude", co-author trailers, etc.). The summary is the Operator's record of their own work.
- **Never invent commit SHAs, PR numbers, file paths, test counts, dates, or quotes.** If you can't get it from the session record, git, or gh, omit the claim or mark it `(unverified)`.
- **Never use marketing voice.** "Significantly improved" is wrong; "added 22 assertions covering the active-UM provisioning path" is right.
- **Never write "I" or "we" as if Claude was the actor.** The actor is the Operator, the agent, a named subagent, or unnamed (passive voice is fine for technical actions: "was deleted", "was renamed"). Decisions, reversals, and outward actions always name the actor.
- **Never ask the Operator about content.** Best-effort draft, every time. The only permitted question is the destination, when a rule requires it.
- **Never skip the validation pass.** Use a separate agent when the harness has one; otherwise run the second pass yourself.
- **Never skip a source silently.** Search every source in step 1, across all bb projects and all transcript folders. Anything not checked is listed with its reason.
- **Never write filler.** If an optional section has nothing real, omit it.

## Reference files

- `references/voice-guide.md`: tone calibration with examples of right vs. wrong phrasing
- `references/section-templates.md`: exact templates for each section
- `references/example-summaries.md`: excerpts from four example summaries for shape reference
