<!--
   - This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/.
   -->

# ADR-0003: Pref namespace

- Status: proposed
- Date: 2026-10-31
- Deciders: project owner

## Decision

Needs a choice. Recommendation: **keep `zen.*` for M1**, add new prefs under
`zen.` for internal consistency, and revisit the rename after the product name
and first-run migration exist.

## Context

The tree uses `zen.` in 189 prefs, plus module paths, CSS ids, l10n ids and
file names. Prefs are user data: anything a user has set, and anything synced
through `services.sync.prefs.sync.*`, refers to the old names.

A rename is mechanical (`sed` across YAML and JS) but wide:

- 189 pref names, ~25 of them with sync twins.
- Reader code often uses string literals; a missed one silently falls back to
  the default instead of failing loudly.
- Existing profiles (ours included) would lose settings unless a migration
  copies old→new.
- Defaults are compiled into `zen-static-prefs.inc` and
  `browser/app/profile/zen.js`; both are generated, so the rename must happen
  before `ffprefs`, not after.

## Options

1. **Keep `zen.*`.** Zero risk, zero migration. Downside: internal naming no
   longer matches the product, and users see `zen.` in `about:config`.
2. **Rename to `<product>.*` with a migration.** Clean surface. Cost: migration
   code, sync twin updates, a full audit of string literals, and a profile
   version bump. Roughly 1-2 days plus QA.
3. **Rename without migration.** Rejected: silently discards user settings and
   sync state.
4. **Dual-read shim** (new name wins, old name read as fallback). Lowest user
   impact, but every pref access needs the shim, and the code becomes
   permanently more complex.

## Consequences of the recommendation

- New features look like `zen.commandbar.enabled`, not
  `<product>.commandbar.enabled`.
- The rebrand remains incomplete in `about:config` after ship; this is
  acceptable because prefs are not user-facing branding in the same way as
  menus and icons.
- If we later rename, the migration is a single well-scoped change and must
  land with a pref-version marker.

## Requirements for any rename

- A migration that copies every old pref to the new name once, then leaves the
  old value in place (do not delete user data).
- Sync twin prefs updated in the same change, or sync silently stops working.
- A grep test in CI that fails on `zen.` in `prefs/**` after the rename lands.
- Pref-gated: `zen.migration.rename-prefs=true` until verified.
