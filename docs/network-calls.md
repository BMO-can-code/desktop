# Network calls inventory

This document lists outbound network calls in the tree. Updated as part of M1 network cleanup.

## Removed at M1 (deleted or removed paths)
| Endpoint/pattern | Status | Notes |
| --- | --- | --- |
| `updates.zen-browser.app` | Removed | No updater until M5. |
| `share.zen-browser.app` | Removed | Share feature disabled/removed. |
| `zen-browser.github.io` (mods registry) | Removed | M1 mods are local files only. |
| `zen-browser.app` (privileged injection) | **Deleted** | Injection actor registration removed from `src/zen/common/sys/ZenActorsManager.sys.mjs` (ZenModsMarketplace actor and matches removed). Verified by `scripts/check-injection.sh`. |
| Live folder hardcoded feeds | Removed/disabled | Feeds must be user-supplied. |

## Kept (documented)
| Service | Status | Notes |
| --- | --- | --- |
| Firefox Safe Browsing | Kept | Security feature; keys may be compiled in. |
| Firefox Sync | Available but default-off | Opt-in only. |
| Remote Settings | Kept | Blocklists/search config. |
| Mozilla telemetry/Normandy/crash upload/sponsored/suggestions | Off/locked | Kept disabled. |

Verification: `scripts/check-removed-hostnames.sh` fails on `zen-browser.app` and `zen-browser.github.io` substrings in source paths. `scripts/check-injection.sh` fails if injection actor registration exists.
