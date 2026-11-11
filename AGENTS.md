# AGENTS.md

Instructions for agents and human contributors working in this repository.

## What this repo is

A single agent skill (`SKILL.md`) that defines the structure, voice, and factual-precision rules for an end-of-session build day summary.

## Hard rules

1. **Never add real client names, domains, or repo URLs to this repository, in `SKILL.md`, examples, or any `references/` file.** Example and voice-calibration content must use invented, fully-generic examples (`example-sso`, `example-org`, `the Operator`, and similar), never a real client's work.

2. **The hard-rules list in `SKILL.md` (no em dashes, no emojis, no Claude attribution, never invent facts, no marketing voice, no first-person-as-Claude and always name the actor for decisions, never ask about content, never skip the validation pass, no filler) is the skill's core contract.** Loosening any of these changes what the skill fundamentally does; don't edit them without updating the rest of `SKILL.md` to match.

3. **The two structural patterns (chronological narrative spine vs. outcome-first) are both valid, not a hierarchy.** Don't collapse them into one "correct" structure -- the skill explicitly picks the pattern that fits the day's shape.

## Conventions

- Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`). Semantic Versioning. Keep a Changelog.
- GitHub Actions pinned to commit SHAs, not moving tags.
- Do not commit generated junk.
