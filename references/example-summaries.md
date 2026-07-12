# Example Summaries

Three examples of the Operator's prior build day summaries. Read these for shape, density, and section flow. The voice and structure are exemplary.

The three examples show different shapes of day:

- **2026-04-29**: Pattern A, chronological narrative spine. A multi-day plan executed in one session. Heavy "what changed today, in order" with numbered subsections.
- **2026-04-30**: Pattern A with heavy obstacle-course tables. Day was dominated by environmental and OIDC debugging. Tables shine here.
- **2026-05-01**: Pattern B, outcome-first. Day was hardening and a v1.0.1 release. Leads with "Released" and "Production bug fixed" sections.

## Excerpts to study

### Lead paragraphs

> Single-day push on the `example-sso` WordPress plugin (the OIDC Relying Party at portal.example.com). Started as a small documentation tweak, surfaced a real production bug in the SSO role-assignment flow, and ended with a substantially hardened test layer and a v1.0.1 release. Nothing client-facing changed; this was internal hardening of work already shipped.
>
> Single-session execution of an approved 5-day rebuild plan for the SSO plugin. Started with a documentation gap analysis on the legacy IdP plugin; ended with a complete RP plugin ready for the client's Postman testing handoff (committed May 4 in the Apr 27 meeting).
>
> Single-session build of an end-to-end test harness for the SSO plugin. The plugin itself was finished yesterday (2026-04-29); today's work added a browser-driven E2E layer that runs the full OIDC round-trip against a local IdP container, finishing with all 9 specs green and a PR open against `main`.

### Production bug section (mechanism walk-through)

> **`UMRoleManager::assign_role()` left every SSO-provisioned user reading as `subscriber`.**
>
> Mechanism:
>
> 1. `Provisioner::find_or_create()` calls `wp_insert_user(['role' => 'subscriber'])` to create the placeholder, then calls `UMRoleManager::assign_role($user_id, $org->get_um_role())`.
> 2. `assign_role()` calls `UM()->roles()->set_role()`, which internally calls `WP_User::add_role()` -- that **appends** rather than replaces. User now holds `[subscriber, um_<org>]`.
> 3. UM's `get_priority_user_role()` ranks roles by `_um_priority` from each role's `um_role_<key>_meta` option. Both `subscriber` (UM default) and a freshly-registered UM custom role tie at priority 0. PHP's stable sort preserves insertion order, so `subscriber` (added first) wins.
> 4. Anything reading the priority role -- UMLR redirect rules, profile branding, capability checks that walk `um_user('role')` -- saw `subscriber` regardless of which org's UM role had been assigned. The capabilities themselves were correct on the user; only the priority-role read was wrong.
>
> Fix landed in `src/UltimateMember/UMRoleManager.php`: after `set_role()`, explicitly call `$user->remove_role('subscriber')` unless the Organization explicitly maps to subscriber. This matches what UM's own user-registration flow does.

### Environmental obstacles table

> | # | Symptom | Cause | Fix |
> |---|---|---|---|
> | 1 | `permission denied` on `/var/run/docker.sock` | No `docker` group on this Linux install; socket owned by `root:root` | `sudo groupadd docker && sudo usermod -aG docker <user> && sudo chgrp docker /var/run/docker.sock && sudo chmod g+rw /var/run/docker.sock && newgrp docker` |
> | 2 | `IdP did not become ready within 30 seconds`; container running | `start-idp.sh` mapped host `4011 -> 80`, but the new .NET 8 build of oidc-server-mock listens on `8080` | Changed mapping to `4011:8080` |
> | 3 | `Playwright requires Node.js 18 or higher` under sudo | sudo resets PATH to `/usr/local/bin/node` (v17.9.0); the Operator's nvm-installed v23.7.0 was hidden | Skip sudo; rely on docker-group fix above |

### Files-touched grouping

> Production:
>
> - `src/UltimateMember/UMRoleManager.php` (the fix)
> - `src/Admin/HelpPage.php` (Developers tab content)
>
> Tests / infrastructure:
>
> - `tests/Integration/ProvisionerUMActiveTest.php` (new)
> - `tests/Integration/ProvisionerTest.php` (skip-when-UM-loaded)
> - `tests/bootstrap.php` (env-gated UM loader)
> - `tests/E2E/global-setup.ts` (Playwright role seed step)
>
> Docs:
>
> - `CHANGELOG.md`
> - `tests/E2E/README.md`
> - Version bumps in `example-sso.php`, `composer.json`, `src/Core/Bootstrap.php` (1.0.0 -> 1.0.1)

### Carry-over / Open items

> ## Carry-over
>
> - `v1.0.1` tag points at `0630ef5` (the version-bump commit, which contains the production fix and direct-option-write seed). The no-mocks fix and Copilot follow-ups landed in `main` after the tag, as part of the same merge. If the next release-shaped artifact wants to include those, it will need a `v1.0.2` tag at the post-merge tip; or the existing tag could be moved (destructive). No external system depends on `v1.0.1` pointing at a specific SHA; deferred decision.
> - The seed roles `client_lite` and `client_pro` are pure test fixtures; nothing in `src/` references them. They exist in two specs because spec 02 exercises a role upgrade path that requires two distinct slugs.

### Notable engineering decisions

> - **Test-only IdP config tweaks are concessions to plain HTTP, not mistakes.** `Authentication.CookieSameSiteMode=Lax` exists because Chromium drops `SameSite=None` cookies without `Secure`. `AlwaysIncludeUserClaimsInIdToken=true` matches real Azure AD. Both are documented inline in the JSON files and in the KB strategy doc. Production IdP config is a different file owned by the client.
> - **`page.route` does not reliably intercept cross-origin top-level navigations in this Playwright + Chromium build.** Spec 09 was rewritten to extract deterministic data (`state`) at a point where the page is at rest (the IdP login URL's `ReturnUrl` parameter), then act before the navigation completes. This pattern is more robust than trying to intercept in-flight requests for cross-origin nav.

### Workflow notes

> - **Plan-mode discipline** (Phase 1 explore in parallel, web research in parallel, AskUserQuestion for the three decisions that mattered, ExitPlanMode for approval) saved meaningful time. The agent didn't have to backtrack on IdP choice or test scope mid-build.
> - **Hands-on browser testing caught what tests missed.** The cap-pollution bug, the menu duplication, the format-only validator letting `asdfasdf` through, and the missing required-field gates were all surfaced by the Operator clicking around the actual admin UI.
> - **One memory updated**: `~/.claude/projects/.../memory/project_db_prefixes.md` claimed local prefix was `<local_prefix>_` but `wp-config.php` actually has `<prod_prefix>_` (matching production). Discovered when constructing a log query. The memory was rewritten to mark prior info as outdated.

### File index

> | Path | Purpose |
> |---|---|
> | `~/.claude/plans/<plan-slug>.md` | Approved E2E plan |
> | `<Client>/Projects/<project>/2026-04-30-000000-e2e-testing-strategy.md` | KB strategy (final state) |
> | Plugin root | `/path/to/project/app/public/wp-content/plugins/example-sso/` |
> | E2E harness | `tests/E2E/` (Playwright + IdP fixture + seed/teardown) |
> | Test runner from harness | `cd tests/E2E && npx playwright test` |
> | `composer test:e2e` | Wrapper that does the cd + npm test |
> | PR | `https://github.com/example-org/example-sso/pull/5` |
