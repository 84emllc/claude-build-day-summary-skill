# Section Templates

Exact templates for each section that may appear in a build day summary. Use only the sections with real content; skip the rest entirely.

## Title

```text
# Build Day Summary - YYYY-MM-DD
```

Or, when the project name adds clarity (typically when it appears across many of the Operator's recent days):

```text
# Build Day Summary: <Project> - YYYY-MM-DD
```

## Lead paragraph

One paragraph, 2-4 sentences. Captures: how the day started, what shifted, where it ended. Concrete (PR numbers, tag names, test counts allowed; but readable as prose). Do not bullet.

Example shapes that work in the Operator's voice:

> Single-day push on the `<plugin>` plugin. Started as <small thing>, surfaced <real bug>, and ended with <substantial outcome> and a v<x.y.z> release. Nothing client-facing changed; this was internal hardening of work already shipped.
>
> Single-session execution of an approved <N>-day rebuild plan for the <project>. Started with a documentation gap analysis on the legacy plugin; ended with a complete RP plugin ready for the client's Postman testing handoff.
>
> Single-session build of <thing> for <project>. The plugin itself was finished <when>; today's work added <what>, finishing with <verification state>.

## TL;DR (optional)

Tight bulleted list of the day's high-level outcomes. Use when the day produced a release, a merged PR, or has 4+ distinct outcomes worth surfacing up top. Each bullet is one fact, often with a backticked path or a URL.

```markdown
## TL;DR

- New `tests/E2E/` harness: Playwright + Chromium driving `https://example.local` against an `oidc-server-mock` IdP container on `http://localhost:4011`.
- 9 specs covering the documented edge-case matrix.
- One small production code change in `src/OIDC/Client.php`: log the underlying discovery fetch error before throwing `discovery_unavailable`.
- 1 commit on `test/e2e-suite`, PR #5 open at https://github.com/example-org/example-sso/pull/5.
- Plan: `~/.claude/plans/<plan-slug>.md`. KB: `<Client>/Projects/<project>/2026-04-30-000000-e2e-testing-strategy.md`.
```

## Released (optional, alternative to TL;DR)

Use when the day produced a tagged version. Lead with the tag, the PR that merged it, and a one-line characterization of what's in it.

```markdown
## Released

**v1.0.1** tagged and merged to `main` via PR #13 (https://github.com/example-org/example-sso/pull/13). Production-affecting fix plus test infrastructure improvements. Eight commits on the release branch (three fix-and-test, one merge, one version bump, three follow-up improvements after the tag in response to Copilot review and a discovered no-mocks-rule violation).
```

## Production bug (optional)

Use when the day diagnosed and fixed a non-trivial bug. Walk the mechanism step-by-step. Numbered list inside the section is standard.

```markdown
## Production bug fixed

**`<exact thing that was wrong, in a sentence>`.**

Mechanism:

1. <Step 1: what was happening at the call site>
2. <Step 2: what was happening at the next layer>
3. <Step 3: where the actual bug surfaces>
4. <Step 4: visible symptom>

Fix landed in `<file path>`: <one-line description of the change>. <When relevant, contrast with what the framework's own code does.>

The fix is one production file, six lines added. <Anything else worth knowing about the surface area of the change.>
```

## Test layer / Test infrastructure (optional)

Use when tests were a substantial part of the day's work. Sub-headed by test type when there's more than one (e.g., PHPUnit, Playwright, infrastructure).

```markdown
## Test layer

### PHPUnit (in-process)

- New `tests/Integration/<Test>.php` exercises <what>. <What it asserts.>
- <Other test additions.>
- Suite gated on `<env var>` so <existing behavior preserved>. With var set: <counts>. Without: <counts>.

### Playwright E2E

- <Probe or fixture changes.>
- <Spec changes, named individually if material.>

### Test infrastructure

- <Seed/teardown changes.>
- <New helpers, with paths and one-line purposes.>
```

## Environmental obstacles table (optional)

Use when the session lost meaningful time to environment issues. Standard four-column format.

```markdown
## The environmental obstacle course

What followed was a long sequence of "now what" failures, each with a one-line fix:

| # | Symptom | Cause | Fix |
|---|---|---|---|
| 1 | `<exact error string>` | <one-line cause> | `<one-line command or change>` |
| 2 | `<exact error string>` | <one-line cause> | <one-line fix> |
```

If the issues were domain-specific (OIDC, network, permissions, etc.), use a themed section title and group similar issues together. The "OIDC obstacle course" / "environmental obstacle course" pattern in the Operator's summaries is a useful precedent.

## Verification at end of day (optional)

Concrete commands and outputs that prove the end state. Use when the day had a clean stopping point worth documenting.

```markdown
## Verification at end of day

- `<command>` -- <result, with counts>.
- `<command>` -- <result>.
- <Other end-state observations: PR merged, tag pushed, etc.>
```

## Files touched / Files committed (mandatory)

Group by purpose. One file per line, parenthetical purpose suffix when the path alone is not self-explanatory.

```markdown
## Files touched (all on `main` post-merge)

Production:
- `src/<Path>.php` (<purpose>)
- `src/<Path>.php` (<purpose>)

Tests / infrastructure:
- `tests/<Path>` (new)
- `tests/<Path>` (<one-word change description>)

Docs:
- `CHANGELOG.md`
- `<Path>` (<purpose>)
- Version bumps in `<file>`, `<file>`, `<file>` (X.Y.Z -> X.Y.Z+1)
```

If the day spans branches, prefix the section header with the branch name in backticks: `## Files touched on \`feat/whatever\``.

## Open items / Carry-over (optional)

Numbered list. One item per concern. State the item, current status, and any pending dependency.

```markdown
## Open items for the next session

1. **<Concern>.** <One or two sentences on status, blocker, decision needed.>
2. **<Concern>.** <Status.>
3. **<Concern>.** <Status.>
```

For release-shaped sessions, use "Carry-over" instead and focus on what didn't make the tag:

```markdown
## Carry-over

- <Thing that landed post-tag>. <Implication for next release.>
- <Decision deferred>. <What triggers the decision.>
```

## Notable engineering decisions (optional)

Use when 2+ non-obvious decisions were made that a future maintainer would benefit from knowing. Each is a bolded lead-in followed by an explanation.

```markdown
## Notable engineering decisions worth flagging for future maintainers

- **<Decision in a phrase>.** <Why it was made, what the alternative was, what would change it back.>
- **<Decision in a phrase>.** <Same shape.>
```

## Workflow notes (optional)

Meta-observations on tooling, hooks, agent dispatch, collab patterns. Useful when the session validated or invalidated a workflow choice. Bolded lead-ins with explanation.

```markdown
## Workflow notes

- **<Pattern that worked or didn't>.** <One or two sentences of detail.>
- **<Hook or tool behavior worth noting>.** <Detail.>
- **<Memory or assumption that turned out wrong>.** <What was wrong, what the truth is.>
```

## File index (optional)

Two-column reference table. Use for handoff-shaped sessions where the next session (or another person) needs to pick up cold.

```markdown
## File index for the next session

| Path | Purpose |
|---|---|
| `~/.claude/plans/<plan>.md` | <Purpose> |
| `<KB path>` | <Purpose> |
| Plugin root | `<absolute path>` |
| Test runner | `<command>` |
| PR | <URL> |
```
