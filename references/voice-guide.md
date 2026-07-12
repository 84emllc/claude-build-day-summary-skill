# Voice Guide

This file calibrates the tone of a build day summary against the Operator's own prior writing. Read this when you're not sure if a sentence sounds right.

"The Operator" is the single human this skill serves: the engineer whose build day is being summarized, who drives the working session and edits the resulting draft. It is always the same person. Some older summaries called this person "the user"; in new summaries, name them "the Operator". Reserve "user" for a WordPress user account, an OS user, or a genuine end-user of the software, never for the Operator.

## Core voice

Third person, factual, candid, dense. The reader is the Operator's future self or a maintainer picking up the work cold. Both audiences want exact information, not narrative flourish.

## Phrasing rules

### Use specific over general

| Wrong | Right |
|---|---|
| Made several improvements to testing | New `tests/Integration/ProvisionerUMActiveTest.php` exercises the active-UM provisioning path: registers a UM custom role, calls `Provisioner::find_or_create()`, asserts the configured UM role lands and `subscriber` is absent |
| Fixed a bug in role assignment | `UMRoleManager::assign_role()` left every SSO-provisioned user reading as `subscriber` because `set_role()` calls `add_role()` internally, which appends rather than replaces |
| Added some tests | 90 tests / 316 assertions / 1 skipped with `WP_TESTS_LOAD_UM=1`; 90 / 309 / 2 without |
| Cleaned up the build | Single commit on `test/e2e-suite`: `c289b42 test: add Playwright E2E suite with local OIDC IdP container`. 29 files changed, 1192 insertions, 1 deletion |

### Name the actor when it matters

| Wrong | Right |
|---|---|
| Discovered the issue | The Operator's own testing surfaced two more issues |
| Was decided | Through ~30 rounds of one-by-one clarifying questions, settled on these decisions |
| It was determined that | The agent dispatched left a PHP method added to disk but uncommitted |

When the actor is the Operator, name them. When it's an automated/agent action, name that. When it's a passive technical fact ("the file was renamed"), passive is fine.

### Lead with the mechanism, not the verdict

When describing a bug fix, walk the mechanism step by step before stating the fix. This is the Operator's signature pattern:

> 1. `Provisioner::find_or_create()` calls `wp_insert_user(['role' => 'subscriber'])` to create the placeholder, then calls `UMRoleManager::assign_role($user_id, $org->get_um_role())`.
> 2. `assign_role()` calls `UM()->roles()->set_role()`, which internally calls `WP_User::add_role()` -- that **appends** rather than replaces. User now holds `[subscriber, um_<org>]`.
> 3. UM's `get_priority_user_role()` ranks roles by `_um_priority`...
> 4. Anything reading the priority role saw `subscriber` regardless of which org's UM role had been assigned.
>
> Fix landed in `src/UltimateMember/UMRoleManager.php`: after `set_role()`, explicitly call `$user->remove_role('subscriber')`...

The mechanism explanation is what makes the summary useful six months later.

### Code, paths, and identifiers in backticks

Always backtick:

- File paths: `tests/E2E/scripts/seed.sh`
- Function/method names: `Provisioner::find_or_create()`
- Variable names: `$wpdb`
- Branch names: `feat/strict-domain-input`
- Commit SHAs: `c289b42`
- Slugs and constants: `example_sso_org`, `WP_TESTS_LOAD_UM=1`
- Error strings: `permission denied on /var/run/docker.sock`
- CLI commands: `npx playwright install chromium`

### Be candid about what went wrong

This is not a marketing doc. If something masked a bug for hours, say so. If an agent left work uncommitted, name the dispatch. Examples from prior summaries:

> Teardown now sweeps fixture-domain users at the end so a partial run can no longer leave a stale user with an orphan capability key that would short-circuit `Provisioner::find_or_create` on the existing-user branch and trap subsequent runs. (This was the actual mechanism that masked the production bug for hours during the build day -- the stale user appeared to invalidate any fix until the sweep was added.)
>
> One agent dispatch left a PHP method (`is_valid_domain_shape`) added to disk but uncommitted, while committing the JS that depended on it. Caught by checking `git status` against the runtime behavior the Operator observed.
>
> The original seed used `wp eval-file /dev/stdin` with a heredoc piped in. wp-cli's eval-file under Local's bundled .NET 8 PHP build silently fails to read `/dev/stdin` (no error, no output, no side effect). Even the seed's success message printed because we never asserted on the `ok\n` echo coming back.

### Acknowledge scope and tradeoffs

When something was deliberately deferred, scoped out, or left as a known quirk, say so. Don't pretend the day's work was fully complete if it wasn't.

> `v1.0.1` tag points at `0630ef5` (the version-bump commit). The no-mocks fix and Copilot follow-ups landed in `main` after the tag, as part of the same merge. If the next release-shaped artifact wants to include those, it will need a `v1.0.2` tag at the post-merge tip; or the existing tag could be moved (destructive). No external system depends on `v1.0.1` pointing at a specific SHA; deferred decision.

### No emojis, anywhere

Not in headers, not in lists, not in commit messages quoted in the summary, not in section dividers.

### No Claude attribution

Do not write "Claude generated", "with assistance from Claude", "Co-authored-by: Claude", "Generated by Claude Code". This is the Operator's record of their work.

## Section opener patterns

The lead sentence of each numbered section typically establishes the trigger: a request from the Operator, an error encountered, a discovery, a planned next step. Examples:

- "Started in a fresh session on the `feat/strict-domain-input` branch with PR #4 already merged."
- "The Operator asked for an internal E2E testing strategy with a real IdP, before handing off to the client."
- "What followed was a long sequence of 'now what' failures, each with a one-line fix:"
- "Pulled the 2026-04-27 meeting notes from the KB and discovered the actual scope:"
- "After the bag-and-tag and PR push, the Operator flagged this as a violation of the strict no-mocks rule for tests."

## Final-state / verification phrasing

When ending a section or the summary with a verification, use commands and exact outputs:

> - `bin/run-tests.sh` -- 90 tests, 309 assertions, 2 skipped (the active-UM tests skip cleanly).
> - `WP_TESTS_LOAD_UM=1 bin/run-tests.sh` -- 90 tests, 316 assertions, 1 skipped (the inactive-UM sanity test).
> - `composer test:e2e` -- 9/9 specs green end-to-end, total ~1m 50s.

Or for end-of-day state, a code block summarizing key paths:

```text
Plugin:    /path/to/project/app/public/wp-content/plugins/example-sso/
Branch:    test/e2e-suite (1 commit ahead of main)
PR:        https://github.com/example-org/example-sso/pull/5
Tests:     78 PHPUnit + 9 Playwright + 30 Postman = 117 verified assertions
```
