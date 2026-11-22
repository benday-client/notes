# Foldkit acceptance record

This record lives in `benday-client/notes`, outside the code repository.
A path such as `docs/acceptance.md` in older text means this file.

Every row names the obligation and its oracle, how the tree enforces it, the
evidence that was actually observed, who closed it, and the gate it belongs to.
A row marked **blocked** is not waived; it is missing evidence.

Oracle source: `benday-client/benday` at
`237e98b3f2e46e3843eece22ab2429ae0cb0708c`, clean `main`.

## Bootstrap and workspace

| requirement and oracle | enforcement | evidence | owner and closure | gate and status |
|---|---|---|---|---|
| one fully empty signed root commit on `main` | no files, signed commit | tree lists no paths; commit `6c9fe89`; GitHub reports `verified=true reason=valid` for ED25519 `SHA256:+iUOnFOmcLGD9YbPscOWfqgzXdgFGAmODrRtAC1mtvQ` | integration owner | foundation: done |
| public remote exists without GitHub-generated content | `gh repo create` with no auto-init, then PR-only delivery | `benday-client/benday-foldkit` public; `main` at merge commit `8474b9c` (parents `6c9fe89` bootstrap and `6c6bc82` head), tree `81a7ecec` equal to the head, 114 files; branch `foundation/workspace` kept | integration owner | foundation: done |
| destination is a new sibling repository, reference untouched | separate checkout | `benday-foldkit` created under `src/lexa`; reference HEAD still `237e98b3` and clean | integration owner | foundation: done |
| eight independent reusable libraries, app-private workspace | workspace manifests, `exports` maps | `packages/{richtext,bsky-model,bsky-client,draft-store,deck,dither,media,effect-query}`; `apps/benday` | integration owner | foundation: done |
| pinned toolchain with one frozen lockfile | catalog + `--frozen-lockfile` | `pnpm-lock.yaml`; installed Node 24.21.0, pnpm 12.16.1, foldkit 0.171.0, effect 4.1.5, @effect/platform-browser 4.1.5, typescript 7.0.2, vite 8.3.5, vitest 5.0.5, playwright 1.64.1, @atproto/api 0.29.6, oauth-client-browser 0.5.11, hls.js 1.7.4, @tanstack/query-core 5.105.0 | integration owner | foundation: done |
| GPL-3.0-only everywhere, no `or later`, no linking exception | manifest licence fields, byte-identical package LICENSE | root `LICENSE` sha256 `fb981668c18a279e285fc4d83fba1e836cc84dd4daa73c9697d3cfd2d8aca6e0`; all eight package copies identical; `test:packages` verifies each tarball | integration owner | foundation: done |
| library packages private but packable, no publication | `private: true`, `files: [dist, LICENSE]`, `sideEffects: false` | manifest checks in `scripts/check-packages.ts`; packed tarballs installed in a temporary project | integration owner | foundation: done |
| import boundaries: no app import, no wildcard internals, explicit `.js` | NodeNext build per package, explicit exports | `pnpm build:packages` green; publint strict and ATTW `esm-only` green on every tarball | integration owner | foundation: done |

## Library contracts

| requirement and oracle | enforcement | evidence | owner and closure | gate and status |
|---|---|---|---|---|
| richtext graphemes/bytes equal the pinned SDK, facets use UTF-8 bytes | differential unit test against `RichText`/`UnicodeString` | `packages/richtext/src/graphemes.test.ts` over ascii, combining mark, flag, family emoji, Polish diacritics and lone surrogates; 167 unit tests green | library owner; integration recheck | foundation: done |
| richtext keeps malformed facets local, plain text intact, HTTP(S) only | `segmentText`, `safeHttpUrl` tests | `facets.test.ts`: out-of-range/negative/fractional spans lose only their feature; overlapping spans kept; unsafe destinations null | library owner | foundation: done |
| markdown interprets only labelled links, byte offsets survive multibyte text | `parseMarkdown` tests | `markdown.test.ts` incl. balanced parentheses and multibyte prefix | library owner | foundation: done |
| mention completion keeps caret/CRLF/suffix rules | `suggestionAt`, `insertSuggestion` tests | `suggestions.test.ts` boundary, punctuation, empty query, CRLF | library owner | foundation: done |
| dither reproduces the exact source graph, caller-scoped ids | typed node assertions | `packages/dither/src/index.test.ts`: 9 nodes per variant, grunge 0.9/seed 17/two octaves, k2/k3/k4 values, ramp tables, `probe-dark`/`probe-light`, independent ids | library owner | foundation: done |
| dither data URI equals a browser `btoa` of the same SVG | differential test | `index.test.ts` compares `buildBayerPatternDataUri()` against `btoa(buildBayerPatternSvg())` and round-trips | library owner | foundation: done |
| draft-store revision-and-text cleanup, 20-entry eviction, denied reads never rewritten | injected port + revision tests | `draft-store/src/index.test.ts` | library owner | foundation: done |
| deck key-first restore, axis rules, memory is app-lifetime only | pure tests | `deck/src/index.test.ts`: key fallback, full restore-mode table, non-finite geometry rejected, instance isolation | library owner | foundation: done |
| deck browser geometry/settle helpers use `data-deck-index` and complete cleanup | DOM-scoped helpers | `deck/src/browser.ts` typechecked; no browser measurement yet | library owner; closure pending | foundation: **partial** — geometry not measured in a real browser |
| effect-query owns one native QueryClient and native observers | lifecycle tests | `effect-query/src/index.test.ts`: shared in-flight request, stop publishing after scope close, unmount-before-clear, `removed` emits native hash | library owner | foundation: done |
| bsky-model schemas and mappers keep source fields, moderation and degradation | schema decode + mapper tests | `bsky-model/src/model.test.ts` incl. embeds, facets, hidden pruning, 80-entry thread cap, notification labels | library owner | foundation: done |
| interaction reconciliation: read-start identity, copy-on-write, write confirmation | pure tests | `bsky-model/src/interactions.test.ts` covers 44_999/45_000 ms write-confirmation, identical-entry read consumption, stale-read rejection, identity preservation and count retirement. Canonical read freshness (`sideStale`/`needsViewerRead`) is app-owned; the package exports `VIEWER_STALE_TIME` for it | library owner | foundation: done |
| bsky-client/session ship a reviewed contract only, no fake runtime | type-only modules, no constructors | `packages/bsky-client/src/{index,session}.ts`; package builds to empty JS; consumer compiles against the types | integration owner | foundation: contract only; runtime implementation is gated to identity |
| media playback facts keep counters and phase across retry and failure | reducer tests | `media/src/index.test.ts`: Playing keeps Loading/LoadFailed/Unsupported and only sets `playing`; Ended keeps the tag; counters survive retry; ended is Ready with playback stopped | library owner; closure pending | foundation: **partial** — element/HLS lifecycle still has no browser or synthetic-media evidence |

## Application foundation

| requirement and oracle | enforcement | evidence | owner and closure | gate and status |
|---|---|---|---|---|
| anonymous landing loads through Foldkit in three engines at both profiles | Playwright, real production build | `verification/browser/landing.spec.ts`: 18 tests green in chromium/firefox/webkit at 1280×800 and 390×844; heading, body, closed-beta label and disabled action painted | integration owner | foundation: done |
| landing copy, classes and skip target match the source | source `LandingView.tsx`, `layout.css` | same spec asserts exact copy, `.b-landing`, `a.b-skip` → focused `main#main` without a history entry | integration owner | foundation: done |
| first paint decides the stored scheme before the app bundle runs | inline bootstrap generated from one module source | build inlines `firstPaint.js`; browser test aborts the module asset and still observes `data-theme=light` and `theme-color #f4f2ef`; unit probe runs the inlined script against stub globals | integration owner | foundation: done |
| OAuth metadata and deep-link shell contract | Vite emitter + hosting file | browser test fetches `/oauth-client-metadata.json` (`client_id`, scope `atproto transition:generic`, redirect `https://benday.qex.at/feed`) and `/feed?x=1` returns the shell | integration owner | foundation: done |
| build identity describes the tested tree | `build-identity.json` carries `sourceSha` and a `sourceTree` hash over the tracked diff and untracked non-ignored files; a checkout without git fails the build | `scripts/serve-dist.ts` recomputes both and refuses a mismatch; two consecutive builds of one tree emit a byte-identical identity document | integration owner | foundation: done |
| development privacy: no preservation branch, relay or websocket | config + probe | Vite's injected client is replaced by a stub served from `configureServer` (style injection kept, no transport); the probe scans the HTML, every app module and every `optimizeDeps` entry, loads the page in Chromium and sees zero websockets, no `/devtools`/`/mcp`/`/__dev` request, a styled body, and no relay line; the child runs in its own process group and is killed whole; production bundle contains zero `import.meta.hot` | integration owner | foundation: done |
| DevTools off and no credential recording | `devTools: false`, plugin is `foldkitViewIdentity()` only | entry config; optimizer/bundle inspection above | integration owner | foundation: done |
| landing visual baseline from the frozen oracle | `scripts/capture-reference.ts` pinned to the SHA | 6 captures (3 engines × desktop 1280×800 / mobile 390×844): surface `landing`, heading 64px / 36px, `.b-landing` present; manifest records browser versions and geometry | integration owner | foundation: **captured**, comparison review pending |
| numeric performance budgets and paired benchmark | none yet | no budget was approved; the surfaces that need measuring do not exist yet | user decision required | foundation: **blocked** — benchmark harness not implemented, budgets unapproved |
| policy: suppressions, `any`, assertions, focused/skipped, ambient declarations, authored `.d.ts`, misplaced tests | `check:policy` + Oxlint + report reader | 54 authored files across `.ts/.mts/.cts/.tsx/.js/.mjs/.cjs/.jsx`; ambient AST classes, suppression tokens, SPDX, declaration allowlist, tsconfig coverage and runner include placement enforced; `test:policy` green on 15 fixtures; the vitest reader now requires `success` and zero failed test files | integration owner | foundation: done |
| every authored file inside a real tsconfig program | `check:policy` listFilesOnly coverage | 54 authored files covered by root/app/package programs | integration owner | foundation: done |
| dependency declarations are not audited with our config | shared `skipLibCheck: true` | dependency `.d.ts` are their authors' contract; every authored source keeps the full strict flag set, and the dependency-free packages' emitted declarations are compiled by an external NodeNext consumer with `skipLibCheck: false` (`strict-pure` in `scripts/check-packages.ts`) | integration owner | foundation: done |
| external consumer proof for all eight packages | `scripts/check-packages.ts` | build without app, `pnpm pack`, publint strict, ATTW esm-only, temporary NodeNext project with tarball overrides, no workspace/NODE_PATH, strict compile, pure runtime imports with no browser globals; a second consumer compiles the four dependency-free packages with `skipLibCheck: false` | integration owner | foundation: done |
| unit/browser runs cannot silently select nothing | `scripts/check-test-results.ts` | `pnpm test` and `pnpm test:browser` both end in the guard; 171 unit tests; 21 browser tests across three named projects, including fragment focus on a query route | integration owner | foundation: done |
| CI runs the fatal gate on PRs, no deploy | `.github/workflows/check.yml` + repository ruleset `main` (`27863175`) | run [45080595173](https://github.com/benday-client/benday-foldkit/actions/runs/45080595173) passed `pnpm check` on `e3752db` in 110s with all three engines; the ruleset targets `~DEFAULT_BRANCH` with `deletion`, `non_fast_forward`, `pull_request` (0 approvals, thread resolution, no bypass actors) and strict required check `check`; legacy branch protection deleted (API 404); squash and rebase disabled at repository level; no deployment step | integration owner | foundation: done |

## Reuse provenance

Source SHA for every copied file: `237e98b3f2e46e3843eece22ab2429ae0cb0708c`.

| source portion | destination | fidelity reason | licence / notice | status |
|---|---|---|---|---|
| `src/assets/fonts/DepartureMono-Regular.woff2` + `DepartureMono-OFL.txt` | `apps/benday/src/assets/fonts/` | byte-identical face keeps metrics and `font-display: swap` | SIL OFL 1.1, copyright 2022–2024 Helena Zhang; OFL text travels beside the font | copied |
| `src/styles/fonts.css` | `apps/benday/src/styles/fonts.css` | the `@font-face` declaration must match the vendored bytes | project source, GPL-3.0-only | copied |
| five fontsource face imports (`instrument-serif` 400/400-italic, `instrument-sans` 400/400-italic/600) | `apps/benday/src/styles/appStyles.ts` | same glyph sets at pinned 5.3.0 packages | packages ship their own OFL licences and notices | imported |
| `src/styles/tokens.css` | `apps/benday/src/styles/tokens.css` | palette, spacing, type and motion tokens define every painted surface | project source, GPL-3.0-only | copied |
| `src/styles/texture.css` | `apps/benday/src/styles/texture.css` | fringe and scanline recipe is the defining texture | project source, GPL-3.0-only | copied |
| `public/benday.svg` | `apps/benday/public/benday.svg` | brand geometry used as the favicon | project source, GPL-3.0-only | copied |
| `public/beta/invites.json` | `apps/benday/public/beta/invites.json` | the shipped social-gate list is the real gate input | project source, GPL-3.0-only | copied, sha256 `9a05a5cd81eff1739e9c3ebd580821de8932d4162886e229b7690b0444dd1086` |
| `src/lib/dither.ts`, `src/components/DitherDefs.tsx` graph | re-expressed in `packages/dither` | exact thresholds, seed and arithmetic must match the old paint | source algorithm, GPL-3.0-only | re-expressed, tested |
| `src/lib/{graphemes,facets,markdown}.ts`, pure suggestion helpers | re-expressed in `packages/richtext` | byte-offset and grapheme rules are compatibility-critical | source algorithm, GPL-3.0-only | re-expressed, differentially tested |
| `src/lib/types.ts`, `src/lib/model.ts`, `src/lib/notifications.ts`, `src/lib/actionStore.ts` | re-expressed in `packages/bsky-model` | field shapes and moderation rules are the data contract | source algorithm, GPL-3.0-only | re-expressed, tested |
| `src/lib/drafts.ts`, `src/components/Deck.tsx`, `src/components/VideoPlayer.tsx` | re-expressed in `draft-store`, `deck`, `media` | persistence and geometry semantics must not drift | source algorithm, GPL-3.0-only | re-expressed, tested |

Files deliberately **not** copied: `base.css`, `layout.css`, `components.css`,
`cards.css`, `dialogs.css`, `account.css`, `views.css`, `search.css`,
`notifications.css`, `compose.css`, `media.css`, `landing.css`, `admin.css`
(independently written; `base`, `layout` and `landing` exist as a foundation
subset), `dither.css`, `theme-effects.css` (reading phase), the publish
workflow, and all React hooks/components/stores/tests.

## Adversarial review dispositions (2026-11-22 night)

Three defect reviewers (kernels, protocol, app/gates) and three prose reviewers ran against this tree. Fixed before merge:

- skip link: fragment focus now works on query routes, ids are percent-decoded, and a same-page `#` no longer pushes a history entry, with a browser test on `/feed?x=1`
- dev privacy: Vite's injected client is replaced by a local stub, the probe observes zero websockets and a styled body, scans every app module and optimizer entry, starts only on a free port, and kills the whole process group
- build identity: bound to `sourceTree` (tracked diff plus untracked content), fails closed without git, recomputed and compared before serving
- first paint: a denied `localStorage` global no longer aborts the inline decoder
- reports: vitest `success` and failed test files are fatal; test files outside the runner include patterns fail policy; `.mts/.cts/.tsx/.jsx/.mjs/.cjs` are scanned; `new Function` is banned
- media: `Playing`/`Ended` keep the Loading/LoadFailed/Unsupported tag, play cannot resolve true after release
- deck: snapport is a padded viewport span, non-finite index/offset are rejected, negative drift has a test
- protocol: `ViewerState` is the source's two-URI shape; effect-query destroys an observer whose acquisition throws
- prose: headers that overclaimed a check and metaphor comments were replaced with the mechanism

Still open from review: real-browser media lifecycle and deck geometry evidence; the capture server does not bind the served dist to the source SHA; the package proof does not yet reject an undeclared direct dependency.

## Decision: keep the pinned AT Protocol SDK

`@atcute/*` was evaluated at the user's request. Evidence: every package is 0BSD, ESM-only and plain async with no Effect bindings; it covers XRPC, lexicons, OAuth browser sessions and password sessions, but not feed/thread/profile/notification mapping, viewer-state hydration, labeler configuration, moderation application or typed-text facet detection. Its OAuth browser client stores sessions and the DPoP private JWK in localStorage under `atcute-oauth:*`, with no migration from the oracle's IndexedDB database `@atproto-oauth-client`; adopting it would force every existing session through re-authentication and break the continuity anchor. The pinned `@atproto/api 0.29.6` and `@atproto/oauth-client-browser 0.5.11` stay.

## Open blockers and limits

1. **Merged, protected and migrated to a ruleset.** PR #2 merged at
   `2026-11-22T23:03:17Z` as `8474b9c`; PR #3 (post-merge record, concise
   README, split development and library docs) merged as `563e229`. Head
   commits are GitHub-verified (`reason=valid`). The three content commits were
   re-signed with the account's registered signing key `lexa-github` because
   GitHub did not know `contact@qex.at` as a signing key; trees, parents,
   authors and messages are unchanged. PR #1 was auto-closed during the
   re-signing force push and stays closed as superseded. Ruleset `main`
   (`27863175`) replaced legacy branch protection and enforces deletion and
   force-push blocks, pull-request-only delivery with thread resolution and no
   bypass actors, and the strict `check` context.
2. **Foundation gate is not fully met.** Missing: reviewed old/new comparison
   (captures exist, review does not), benchmark harness and approved numeric
   budgets, and deck geometry plus media lifecycle browser evidence.
3. **Dependency declarations are out of scope by decision.** The shared base
   sets `skipLibCheck: true`; the pinned SDK declarations are not re-audited
   under our strict flags. Our own emitted declarations are still checked from
   the outside for the dependency-free packages. The practical consequence:
   a type error that exists only inside an SDK `.d.ts` will not fail our build.
4. **Landing action is visibly disabled.** Session acquisition is the identity
   phase; the action is disabled rather than pretending to sign in. This is a
   foundation-only surface difference from the source.
5. **WebKit host workaround.** WebKit runs only with the two compatibility
   libraries copied into the Playwright bundle on this host; CI uses
   `--with-deps`. The workaround is environmental, not part of the product.
