<!--
   - This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/.
   -->

# ADR-0006: Network posture and third-party services

- Status: proposed
- Date: 2026-10-31
- Deciders: project owner

## Decision

Needs a choice. Recommendation: **no Zen-operated endpoint may remain in a
shipped build**; features that need one are disabled by default, and any
replacement service is opt-in, documented and self-hostable.

## Context

The tree already turns off telemetry, studies, sponsored content, crash upload
and search suggestions, and locks most of them. What remains is not Mozilla
telemetry but Zen's own product services: share uploads, a theme CDN, updates,
and a privileged injection list for the Zen website. There is also a Mozilla
Sync dependency and the usual Google Safe Browsing / location keys compiled in
at build time. The full list, with code paths and prefs, is
`docs/network-calls.md`.

## Position by call

| Call | Recommendation | Default in our build |
| --- | --- | --- |
| Share (`share.zen-browser.app`) | Do not ship a Zen endpoint. Either remove the feature from v1 or make the base URL empty and the UI hidden until a domain exists. | off |
| Zen Mods (`zen-browser.github.io`) | Do not ship Zen's registry. Remote JS from a third-party CDN is the highest-risk surface in the tree. Remove from v1; revisit with a signed, first-party registry. | off |
| Site injection (`zen.injections.match-urls`) | Remove. A locked, privileged injection pointing at a vendor domain is not acceptable in a privacy product. | removed |
| Update host (`updates.zen-browser.app`) | Our own host or no updater. See ADR-0007. | invalid until decided |
| GitHub/RSS live folders | Keep, but off by default and clearly labelled as contacting third parties. They run with the user's cookies. | off |
| Firefox Sync | Keep Mozilla-hosted for M1. See ADR-0008. | as engine |
| Safe Browsing / location | Keep if keys are present; a browser without Safe Browsing is worse. Document it. | build-dependent |
| Remote Settings | Keep; it is the update path for blocklists and search config. | on |
| Telemetry, Normandy, crash upload, sponsored content, suggestions | Keep off and locked. | off |

## Why not simply keep everything off by default?

Because a pref default is not a guarantee. If the code path exists and the UI
exposes it, a user can trigger it, and a future default change can turn it on
silently. For the five Zen-operated calls, the safer answer is removal or a
compiled-out code path, not a default.

## Consequences

- Removing share and mods reduces v1 scope; the feature matrix marks both as
  inherited, not promised.
- Removing injection means the Zen website loses its custom chrome; irrelevant
  after rebrand, but it must be removed from the patch rather than left
  dormant.
- Live folders stay as an opt-in power feature with an explicit warning.
- We inherit Mozilla's Sync and Safe Browsing dependencies and must say so
  plainly in the privacy documentation. "No telemetry" is true; "no network
  calls" is not.
- Any new endpoint needs an ADR amendment before code lands.

## Requirements for any new service

1. Opt-in, with a pref defaulting to `false`.
2. Documented in `docs/network-calls.md` in the same change.
3. Self-hostable, or a written justification for why not.
4. No credentials in the client; no silent background upload.
5. A test or a proxy capture proving the call does not happen when disabled.
