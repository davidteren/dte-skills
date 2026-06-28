---
name: dte-pwa
description: Make a Rails app installable and offline-capable — web app manifest, service worker, install prompt, offline/cache states, and (optionally) push — built on Rails 8's native PWA support, not a hand-rolled service worker. Composes Rails 8 native PWA (app/views/pwa/manifest.json.erb + service-worker.js + the routes), the ui.sh family + frontend-design for install/offline UI, ie-experience-reviewer for the offline UX, and a Lighthouse PWA audit. Use when the user says "make this a PWA", "make it installable", "add a web app manifest / service worker", "work offline", "add to home screen", or "installable web app". Loops over a list of apps/surfaces. Not for general front-end work (use dte-ux) or native mobile (Hotwire Native is a different toolkit).
---

# dte-pwa — installable, offline-capable Rails app (on Rails 8 native PWA)

Read `../../references/conventions.md` first (tool detection, looping, output shape). This skill **composes
Rails 8's built-in PWA scaffolding** — it does not hand-roll a service worker. Rails 8 ships
`app/views/pwa/manifest.json.erb` + `service-worker.js` and commented routes; this skill fills them in
correctly and wires the install/offline UX.

## Input
- A Rails app (or a specific surface to make installable/offline).
- Or a **list** of apps/surfaces → handle each (serial by default).
- Read the project `CLAUDE.md`/`AGENTS.md`. **Detect the Rails version first** — 8.x has native PWA; older
  needs the manifest + SW added by hand (note it).

## Procedure (per app)
1. **Assess what already exists — and honor it (do this BEFORE changing anything).** Read the current
   `manifest.json.erb`, `service-worker.js`, the `pwa` routes, and the SW **registration** call. **If a
   service worker already exists with a documented design choice** — e.g. a comment explaining why it
   deliberately does *not* intercept fetches / cache offline — **respect it; do not overwrite it.** A
   deliberate **installable-only** SW is a valid, finished state. **Offline caching is opt-in, never
   assumed.** Many apps are already correctly installable and need **zero changes** — say so and stop, don't
   manufacture work. Then enable the native scaffolding only where it's missing: uncomment the `pwa` routes
   in `config/routes.rb`; confirm the Rails 8 files exist (pre-8 → create them from the Rails 8 template).
2. **Fill the manifest** — `name`/`short_name`, `display: standalone`, `start_url`, `theme_color` +
   `background_color` (from the design tokens), and a full **icon set** (incl. maskable). Use the project's
   real brand, not placeholders.
3. **Write the service worker deliberately** — **only if offline behaviour is actually wanted and step 1
   found no deliberate decision against it.** A sane cache strategy (precache the app shell + offline
   fallback; network-first for data). Keep it minimal; `// ponytail:`-mark the cache ceiling. Don't cache
   authenticated/sensitive responses — **and never let a `respondWith` swallow file-download / attachment
   navigations** (a classic regression: the worker re-fetches the attachment instead of letting the browser
   stream it, so the download never starts). Scope any `respondWith` so attachment responses reach the
   browser untouched.
4. **Install + offline UI.** Use the **ui.sh** family + **frontend-design** for the install prompt/affordance
   and the **offline / cached / stale states** (an app that white-screens offline isn't a PWA). Real
   loading/empty/error/offline states, accessible.
5. **Verify it's actually a PWA (runnable signal).** Run a **`lighthouse_audit`** (chrome-devtools) — checks
   installability, manifest, SW registration, offline. Run **`ie-experience-reviewer`** over the install +
   offline UX. Prefer the Lighthouse PWA score over prose; state if no browser was available.
6. **Gate.** App boots, manifest validates, SW registers without console errors, Lighthouse PWA criteria pass.

## For a list of apps/surfaces
One sub-agent per app through 1–6, then a roll-up noting each app's PWA score + the gaps (missing icons,
no offline fallback, SW scope).

## Degrade gracefully
- **Pre-Rails-8** → add the manifest + SW by hand from the Rails 8 template; note native scaffolding absent.
- No chrome-devtools/browser → skip Lighthouse, fall back to a manual PWA checklist (manifest fields, SW
  registration, offline route) and flag the score as unmeasured.

## Done when
The app has a valid manifest + a deliberate service worker (both wired via Rails routes), accessible
install + offline UI with real states, and a **Lighthouse PWA pass** (or an explicit manual-checklist note),
with a gate-green boot.
