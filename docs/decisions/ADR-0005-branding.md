<!--
   - This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/.
   -->

# ADR-0005: Product identity and rebrand

- Status: proposed
- Date: 2026-10-31
- Deciders: project owner

## Decision

Needs the name, app id, binary name, vendor and domains. Until then, M1 may use
a placeholder, but no build is distributed and no Zen mark is shipped.

## Context

The tree is Zen-branded at every layer: build identity, icons, installer art,
strings, endpoints and profile paths. Zen's trademarks are not ours, and the
project brief requires a complete rebrand before ship. The full inventory is in
`docs/branding-inventory.md`.

Two items have lasting consequences beyond cosmetics:

1. **`appId`** determines the profile directory and `application.ini` identity.
   Changing it later orphans profiles; testers would lose their data. Choose
   once, before the first tester build.
2. **`updateHostname`** decides who can push updates to installed builds. It
   must be ours before any binary leaves the machine, or our users receive
   Zen's updates.

## Options for the identity

| Option | App id | Consequence |
| --- | --- | --- |
| Keep `zen`, rebrand only visible strings | `zen` | Profiles continue to work; internal identity stays Zen; looks unfinished in `about:support` |
| New id, e.g. `<product>` | new | Clean identity; requires a one-time profile migration for existing testers |
| New id with a migration that copies the profile | new | Clean identity, testers keep data; ~half a day of work |

Recommendation: new id **with** profile migration, decided before the first
tester build. Keeping `zen` is only acceptable if we never ship.

## Naming decisions required

| Item | Why it blocks |
| --- | --- |
| Product name and short name | Every string, icon wordmark, installer |
| Vendor/publisher string | macOS bundle, Windows publisher, About |
| App id and binary name | Profiles, executable, update identity |
| Website, docs, support domains | Links, update manifest, help |
| Share and mods domains (or removal) | Network posture, ADR-0006 |
| Update host | ADR-0007 |

## Placeholder policy for M1

- A neutral placeholder name is acceptable in development builds.
- Placeholder icons must be visibly distinct from Zen's marks, not recolours.
- Development builds must not contact Zen endpoints. Set
  `updateHostname` to a local/invalid host, and leave `zen.share.base-url`
  empty.
- No placeholder may leak into a release artifact; the release workflow must
  fail if `surfer.json` still contains a `placeholder` marker.

## Consequences

- Rebranding is a multi-day task spread across build identity, assets, strings
  and endpoints; it is not a find/replace.
- Icon generation is automated on macOS (`logo-mac.png` → `firefox.icns` via
  `async-icns`), so the source art is one file per brand.
- The 15 MB welcome video is a packaging-size decision as well as a branding
  one.
- Pref names may stay `zen.*` under ADR-0003, so the rebrand can ship without a
  pref namespace change.

## Acceptance

See `docs/branding-inventory.md` §7. The grep check is mandatory, and so is a
packaged-app inspection of `Info.plist` and the About dialog.
