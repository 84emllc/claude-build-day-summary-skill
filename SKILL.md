---
name: build-day-summary
description: Generate an end-of-session build day summary for the Operator. Use whenever the Operator asks for a "build day summary", "build summary", "day summary", "wrap up the day", "summarize today's work", "EOD summary", or any equivalent end-of-session writeup. Produces a dense, narrative, factually precise markdown file capturing what shipped, what broke, what was learned, and what carries over. Auto-gathers from git, gh, and the current Claude Code session; never asks the Operator for input. Default invocation context is Claude Code at the end of a working session inside a project repo.
---

# Build Day Summary

A skill for generating the Operator's end-of-session build day summary in their established voice and structure. The summary is the durable, candid record of what actually happened that day: what shipped, what broke, what was learned, what carries over. It is written for the Operator's future self and any maintainer who needs to pick up the work cold.

"The Operator" is the single human this skill serves: the engineer whose build day is being summarized, who drives the working session and edits the resulting draft. It is always the same person. Some older summaries called this person "the user"; name them "the Operator" instead. Reserve "user" for a WordPress user account, an OS user, or a genuine end-user of the software.

## Operating principles

The summary is **factual, dense, and narrative**. Every claim is traceable to a commit, a file, an error message, or a session event. No marketing voice. No hedging. No Claude attribution. No emojis. No em dashes (use commas, periods, colons, parens, or restructure).

When in doubt, **prefer specific over general**. Include exact commit SHAs (short form, 7 chars), exact file paths, exact error strings in code formatting, exact test counts, exact PR numbers and URLs. Vague phrasing like "made some improvements to testing" is wrong; "90 tests / 316 assertions / 1 skipped, gated on `WP_TESTS_LOAD_UM=1`" is right.

The tone is **candid internal**, not external-facing. If something was a slog, say so. If a fix masked a real bug for hours, say that. If an agent dispatch left an uncommitted method on disk, name the agent dispatch and the method. The audience is the Operator and future maintainers, who benefit from honesty about what actually went wrong.

**Never ask the Operator for input.** This skill runs at end of session and produces a best-effort draft. The Operator will edit it. If a section's content is unclear, leave it out rather than asking.

## Workflow

### 1. Gather inputs (auto, no questions)

Run these in the current working directory. Capture stderr too, in case `gh` isn't authed or the repo isn't on GitHub.

```bash
# Identity
pwd
git rev-parse --show-toplevel 2>/dev/null
git rev-parse --abbrev-ref HEAD 2>/dev/null

# Commit history for today (and yesterday, in case the session crossed midnight)
git log --since="36 hours ago" --pretty=format:'%h %s%n%b%n---' --name-status

# Working tree state
git status --short
git diff --stat HEAD 2>/dev/null

# Branches
git branch -vv

# PRs (best-effort; may fail silently if no gh or not a github remote)
gh pr list --state all --limit 20 2>/dev/null
gh pr view --json number,title,url,state,headRefName 2>/dev/null

# Tags created recently
git tag --sort=-creatordate | head -5
```

Also pull from the **current Claude Code session context**:

- The Operator's prompts and corrections (these often capture the "why" and the "what was tried first")
- Tool results showing errors that were debugged
- Files read and edited
- Any plan-mode plans or KB documents referenced

If a `~/.claude/plans/<plan-name>.md` was approved this session, note its path.
If a knowledge-base or project document was written to this session, note its path too.

### 2. Decide which sections apply

Mandatory (always include):

- Title line
- Lead paragraph (the day's arc, one paragraph)
- "What changed today, in order" (or equivalent narrative spine; see structure-selection rules below)
- Files touched / Files committed

Optional (include only if there is real content):

- TL;DR or Released (high-level outcome bullets; include when the day produced a tagged release, a merged PR, or has 4+ distinct outcomes worth listing up top)
- Production bug section (include when a non-trivial bug was diagnosed and fixed)
- Test layer / Test infrastructure (include when tests were the main artifact)
- Environmental obstacles table (include when the session lost meaningful time to environment issues; use the `# | Symptom | Cause | Fix` table format)
- Domain-specific obstacle sections (e.g., the "OIDC obstacle course" pattern: group thematically related debugging into a named subsection)
- Verification at end of day (include when there's a clean end-state to capture: command + result)
- Open items / Carry-over (include when work spans into next session)
- Notable engineering decisions worth flagging (include when 2+ non-obvious decisions were made that a future maintainer should know)
- Workflow / process notes (include when meta-observations about tooling, hooks, agent dispatch, or collab patterns are worth capturing)
- File index for the next session (include for handoff-shaped sessions)

Skip empty sections entirely. Do not write "N/A" or "None this session."

### 3. Choose a structure

Two structural patterns appear in the Operator's prior summaries, both valid:

**Pattern A: Narrative spine** (used when the day was a chronological build):

- Title
- Lead paragraph
- TL;DR (bulleted)
- "What changed today, in order" with numbered subsections
- Final state / verification
- Open items
- Notable decisions
- Workflow notes
- File index

**Pattern B: Outcome-first** (used when the day was hardening / fixes / multiple PRs):

- Title
- Lead paragraph
- Released (the outcome up front)
- Production bug (mechanism, fix)
- Test layer / Test infrastructure
- Themed subsections for other work
- Verification
- Files touched
- Carry-over

Pick whichever fits the actual shape of the day. Pattern A when the work built up something new across the day. Pattern B when the work was a tighter set of fixes/hardening shipped against existing code.

### 4. Write the summary

Use `references/voice-guide.md` for tone calibration if needed. Use `references/section-templates.md` for the exact shape of each section.

**Title formats the Operator has used:**

- Older summaries use a literal em dash between project and date. **Do not use em dashes in new output.** Substitute a colon or hyphen.
- `# Build Day Summary - YYYY-MM-DD` (safe, hyphen)

Use the second format for new summaries: `# Build Day Summary - YYYY-MM-DD`. Or if the project name adds clarity: `# Build Day Summary: <Project> - YYYY-MM-DD`.

**Lead paragraph shape:** one paragraph, 2-4 sentences, capturing the day's arc. Start state implied or stated, then what shifted, then end state. Concrete (PR numbers, tag names, test counts ok here). Do not bullet.

Example shapes that work:

- "Single-day push on the `<plugin>` plugin. Started as <small thing>, surfaced <real bug>, and ended with <substantial outcome> and a v<x.y.z> release."
- "Single-session execution of an approved <N>-day rebuild plan for the <project>. Started with <starting state>; ended with <ending state>."
- "Single-session build of <thing> for <project>. The plugin itself was finished <when>; today's work added <what>, finishing with <verification state>."

**Numbered narrative sections** (Pattern A): use `### 1. <Short title>`, `### 2. <Short title>`. Each section starts with what triggered it (a request, an error, a discovery), then describes what was done, then names the artifact it produced (commit SHA, branch, file path).

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

### 5. Write the file

Generate the filename from the session date and current time:

```bash
date +'%Y-%m-%d-%H%M%S-build-day-summary.md'
```

Default output location: the root of the current repository, using the filename generated above, unless the Operator specifies otherwise.

If the Operator keeps build day summaries somewhere specific (a `docs/` directory, a separate notes repo, or a knowledge base), write there instead when they have said so. When unsure, default to the repository root and tell the Operator where you wrote it.

After writing, tell the Operator:

- The filename created
- A one-sentence description of what's in it
- Any sections you deliberately skipped (so they can flag if you misjudged)

Do not narrate the gathering process or the section-selection process. The output is the file.

## Hard rules

- **Never use em dashes.** Substitute commas, periods, colons, parens, or restructure. This applies even when quoting older Operator docs that contain them.
- **Never use emojis.**
- **Never include Claude attribution** ("generated by Claude", co-author trailers, etc.). The summary is the Operator's record of their own work.
- **Never invent commit SHAs, PR numbers, file paths, or test counts.** If you can't get the number from git/gh/session context, omit the claim.
- **Never use marketing voice.** "Significantly improved" is wrong; "added 22 assertions covering the active-UM provisioning path" is right.
- **Never write "I" or "we" as if Claude was the actor.** The actor is the Operator, or "the agent dispatched", or unnamed (passive voice is fine for technical actions: "was deleted", "was renamed").
- **Never ask the Operator for input.** Best-effort draft, every time.
- **Never write filler.** If a section has nothing real, omit it.

## Reference files

- `references/voice-guide.md`: tone calibration with examples of right vs. wrong phrasing
- `references/section-templates.md`: exact templates for each optional section
- `references/example-summaries.md`: three full example summaries (the Operator's own prior work) for shape reference
