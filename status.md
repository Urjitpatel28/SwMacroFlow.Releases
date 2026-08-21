# status.json

Remote kill switch for SwMacroFlow. Every launch (except unattended scheduled runs), the app fetches
this file from:

```
https://raw.githubusercontent.com/Urjitpatel28/SwMacroFlow.Releases/main/status.json
```

Editing and committing this file to `main` changes behavior for every installed copy of the app on
its next launch — no rebuild, no redeploy.

## Fields

| Field     | Type    | Required | Meaning |
|-----------|---------|----------|---------|
| `enabled` | boolean | yes      | `true` = app runs normally. `false` = app is blocked. |
| `message` | string  | no       | Shown to the user when `enabled` is `false`. See below. |

## `enabled`

- `true` — normal operation, nothing shown to the user.
- `false` — on next launch, the app shows a full-screen blocking overlay and is otherwise unusable
  for that session. The user has to relaunch after this is flipped back to `true` to get in again.
- Missing, or the file itself unreachable/malformed — treated as "no information", **not** as
  disabled. A typo in this file, a GitHub outage, or a user's firewall must never lock people out by
  accident. See the offline/grace-period behavior below.

## `message`

Free text shown inside the blocking overlay when `enabled` is `false` — the only thing a blocked
user actually sees, since the app gives them no other way to find out what happened. Use it to say
*why* the app stopped working and what to do next. It is the one lever this mechanism gives you for
talking to users after the fact, so it's worth writing something specific rather than leaving it
blank. Typical uses:

- **Retiring the app**: `"SwMacroFlow is no longer maintained. Thanks for using it."`
- **Switching to paid**: `"SwMacroFlow now requires a license. Contact you@example.com to continue using it."`
- **Selling the app**: `"SwMacroFlow is now under new ownership. See https://example.com for details."`
- **Emergency pull** (e.g. a serious bug): `"SwMacroFlow has been temporarily disabled while an issue is investigated. Check back soon."`

Left empty or omitted, the app falls back to a generic line: *"This copy of SwMacroFlow has been
disabled. Contact the developer for details."* — worth overriding for anything other than an
emergency pull, since that default gives a blocked user nothing to act on.

There's no version-targeting or per-user field yet — `enabled`/`message` apply to every install that
can reach this file. Nothing about the parser stops that expanding later without a format change.

## Offline / unreachable behavior

Each install caches the last status it actually confirmed. If it can't reach this file (offline, DNS
blocked, GitHub outage):

- No cache yet (fresh install that's never checked in) → runs normally.
- Cache says enabled → runs normally.
- Cache says disabled, and it's been less than 14 days since the last successful check → still
  blocked (the kill switch survives a short outage or someone disconnecting their network).
- Cache says disabled, and it's been 14+ days since the last successful check → runs normally
  (never permanently locks out someone who's legitimately lost connectivity long-term).

## Editing

This is a plain file in this repo — edit it directly on GitHub, or clone, edit, commit, and push to
`main`. Changes take effect the next time each installed copy launches and can reach this URL;
there's no push mechanism, so already-running sessions aren't affected until they're relaunched.
