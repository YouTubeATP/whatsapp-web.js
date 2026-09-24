# YouTubeATP fork — maintenance notes

This fork exists solely to carry one focused patch: normalizing WhatsApp
Web's `id._serialized` → `id.$1` rename (WA Web 2.3000.x) so
`whatsapp-web.js@1.34.7` keeps working (see upstream
[wwebjs/whatsapp-web.js#201852](https://github.com/wwebjs/whatsapp-web.js/issues/201852)
and [#201862](https://github.com/wwebjs/whatsapp-web.js/issues/201862)).

The patch was cherry-picked narrowly from `MuhdSHiBiLi/whatsapp-web.js`'s
fork, deliberately excluding all of that fork's unrelated changes (the
`AuthStore` removal, the `inject()` rewrite, new call-media features). See
the root commit message for the full list of what was included/excluded.

## Repo layout for updates

- `origin` → `https://github.com/YouTubeATP/whatsapp-web.js` (this fork)
- `upstream` → `https://github.com/wwebjs/whatsapp-web.js` (canonical repo)
- `.fork-base-version` — the upstream tag this fork's `main` is currently
  built from (currently `1.34.7`). Bump this file every time `main` is
  rebased onto a newer upstream tag.

A GitHub Actions workflow (`.github/workflows/check-upstream.yml`) runs
weekly, compares `.fork-base-version` against upstream's latest release
tag, and opens/updates a tracking issue when upstream has moved on. It
never auto-merges anything — the patch is small and hand-curated, and
should stay that way.

## How to pull in a newer upstream release

1. `git fetch upstream --tags` and confirm the new tag, e.g. `v1.35.0`.
2. `git diff v1.34.7 v1.35.0 -- <patched files>` (list below) to see what
   upstream changed in exactly the files this fork touches. Two outcomes:
   - Upstream's own diff already fixes the `_serialized`/`$1` issue in a
     given spot → drop our hunk there, upstream's fix wins.
   - Upstream hasn't touched that code → our hunk still applies, carry it
     forward as-is (or adjust for nearby unrelated upstream changes).
3. Check whether upstream itself shipped a first-class fix for
   [#201852](https://github.com/wwebjs/whatsapp-web.js/issues/201852) /
   [#201862](https://github.com/wwebjs/whatsapp-web.js/issues/201862) — if
   so, this entire fork may become unnecessary; that is the preferred
   outcome, not a failure of this process.
4. Rebuild `main` from the new tag plus the (possibly trimmed) patch,
   following the same discipline as the original patch: squash to one
   clean commit documenting exactly what's included/excluded, bump
   `package.json`'s version to `<new-tag>-youtubeatp.N`, update
   `.fork-base-version`, push.
5. Bump the `whatsapp-web.js` dependency in `TutorTrack/package.json` to
   the new commit SHA's tarball URL
   (`https://github.com/YouTubeATP/whatsapp-web.js/tarball/<sha>`) and
   re-run `npm install`.

## Patched files (relative to `v1.34.7`)

- `src/structures/Base.js` — adds the shared `Base._normalizeId(id)` helper.
- `src/structures/Chat.js`, `Broadcast.js`, `Channel.js`, `Contact.js`,
  `GroupNotification.js` — `this.id` assignment normalized.
- `src/structures/ClientInfo.js` — `this.wid` assignment normalized.
- `src/structures/GroupChat.js` — 9 spots normalized (participant/group id
  lookups and comparisons).
- `src/structures/Message.js` — `this.id`, `this.from`/`this.to`/`this.author`,
  and the invite-message `fromId`/`toId` block normalized. Upstream's
  `downloadMedia()` is deliberately untouched.
- `src/util/Injected/Utils.js` — 7 of the ~11 relevant upstream-fork hunks
  (message lookup by key, `editMessage`, `msg.id.remote`, `createWid`,
  `lastReceivedKey`, 2 group-membership-request normalizations).
- `src/Client.js` — 8 hunks (reaction/poll-vote sender ids, group
  participant/creation ids, blocklist ids, chat-filter ids, LID/phone
  pairs, `serialized.chatId`).

Anything not in this list was intentionally left untouched and should stay
that way unless a future patch has its own, separately-reviewed reason to
touch it.
