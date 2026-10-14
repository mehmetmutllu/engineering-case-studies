# Kelime Ezberle — English Vocabulary App for Turkish Speakers

A mobile vocabulary trainer: a lesson path organized by topic and CEFR level, a multi-step memorization flow (learn → several quiz types → test), spaced repetition, streaks, a dictionary and a practice area. Released on the App Store; Android is in closed testing on Google Play.

*App Store: [Kelime Ezberle](https://apps.apple.com/tr/app/id6811626340) · Source code: private.*

## Problem

Turkish learners who want to build English vocabulary mostly find either generic flashcard apps or large platforms where Turkish is an afterthought. The goal was a focused app with Turkish-first explanations, example sentences that actually pin down the target word, and a monetization model (ads + optional subscription) that never blocks the learning flow itself.

## My Role

Sole developer — product, architecture, content pipeline, testing and store releases. Development ran in more than 70 working sessions with an AI coding assistant. I acted as orchestrator: I wrote the plans and briefings, reviewed the output and made the release decisions.

## Architecture

- **Vanilla JS single-page app, no build step.** UI/router, quiz engine and data layer are separate modules. Local state lives in IndexedDB (Dexie) and `localStorage`; a service worker makes the app fully offline.
- **Capacitor 8 native shell** for Android and iOS (targetSdk 36, minSdk 24, iOS 15). Native features come in through plugins: AdMob, RevenueCat, Firebase Authentication, local notifications, text-to-speech, haptics, in-app review.
- **Embedded content package.** The vocabulary ships inside the app as a single JSON package (currently 828 words: 728 topic words + 100 academic words). A PHP generator produces it from the authoring database. Content therefore changes only with a store release, but every lesson works offline and needs no backend.
- **Spaced repetition** (SM-2) runs on the device. Lesson steps are spread across the review schedule instead of being crammed into one session.
- **A PHP REST API + MySQL** exists from the original web version. The shipped mobile app runs in local mode and does not depend on it.

## Monetization

- **Premium subscription (IAP) via RevenueCat**, with monthly and yearly plans and a free trial on iOS. Premium removes ads and unlocks extra practice. Premium state is checked on the device through the RevenueCat SDK. A fake-store query-string mode (`?premium=aktif_aylik|deneme|odeme_sorunu|…`) lets every premium screen be tested in a browser without store accounts.
- **Refund service on Cloudflare Workers.** Deleting an account with an active Google Play subscription has to cancel it and refund the unused days. The Worker:
  1. verifies the Firebase ID token itself (RS256 against Google's public certificates: issuer, audience, expiry)
  2. looks up active subscriptions in RevenueCat
  3. resolves the Play order to a purchase token
  4. revokes the subscription with a prorated refund

  If the refund step fails, the client aborts the deletion and nothing is removed. Secrets live only in Worker secrets.
- **AdMob** with a consent (UMP) flow. Rewarded ads are voluntary (refill energy, unlock practice uses). The only non-voluntary format is a skippable rewarded interstitial between lesson steps, capped per day. Banners were evaluated and rejected. Real ad units are swapped in only by the release build flag. The source always ships with test IDs, so a debug build can never serve live ads.

## Accounts and Cloud Backup

Firebase Authentication (Google sign-in, plus email/password for store review accounts) with progress backup to Firestore through its REST API — no Cloud Functions, on the free plan. A pure merge module combines device and cloud documents without losing progress from either side. It has no network or DOM access, so it is unit-tested in Node. Guests never trigger a network call, and account deletion runs in a fixed order: refund → cloud data → auth user → subscriber record.

## Old Android WebView Compatibility

Test devices running Android 8–10 had WebViews around Chrome 80–86. Those versions **silently drop** CSS they don't understand: `inset`, flex `gap`, `:focus-visible`, `:has()`. The result was modals collapsing into the top-left corner and icons sticking to their labels, with no error anywhere. The rule now:

- Properties newer than Chrome 80 are either avoided or given a fallback.
- A head script feature-detects flex gap and `:focus-visible` and adds `no-flexgap` / `no-focus-visible` classes, which a fallback CSS block uses.
- A dedicated test does two things: a static scan (forbidden syntax, and every flex container that uses `gap` must have a fallback) and a layout comparison of 8 screens with `gap` zeroed out.

## Testing

- **26 Playwright web suites** (the last full run: ~1,576 checks), covering lesson flow, energy system, premium gates, purchase flows, accounts, cloud sync merge, streaks, the refund Worker, distractors and the old-WebView rules. AdMob is replaced by an injected fake plugin, because ads cannot be tested in a browser.
- **Native tests on the Android emulator** against the real APK (54 checks, including a premium section, plus a 66-check streak scenario suite).
- iOS builds run on Codemagic. Each release bumps the iOS and Android version numbers independently.

## Content Quality Pipeline

Content is the product, so it goes through its own measured pipeline:

- **Example sentences come only from Tatoeba** (licensed, attributed). In the latest content pass, 238 sentences were replaced with ones that make the target word unambiguous, and the 100 academic words moved onto the same pipeline (91 Tatoeba sentences, CEFR labels corrected).
- **Translations checked by blind back-translation**, which led to 19 translation fixes.
- **Distractors for fill-in-the-blank questions.** A research pass showed that 14% of blank questions had two correct options and 22% could be solved from word form alone. Now an NLP step proposes about 10 candidates per word (same part of speech, same or lower level, not a synonym or antonym, not a target in the same lesson). An AI agent accepts or rejects each one in its sentence. A second agent with a clean context then independently re-judges a random **25% sample**.
- **Acceptance threshold: 5% disagreement per topic.** 5 topics exceeded it and were redone from scratch. The final measurement put every topic at **≤4%** (≤3.1% for the replaced sentences).
- **Coverage:** words with audited distractors went from 457 to **616 of 728** topic words. The rest fall back to the older generator.

## Release Process

- **App Store:** eight versions shipped between 13 September and 12 October 2026 (1.0 → 1.4.0, including the premium and accounts releases); all are live. The first submission was held under Guideline 2.1 (information needed) and passed after a written response.
- **Google Play:** closed testing. Production access requires the 12-testers × 14-days rule.
- Release builds are produced by a whitelisting build script that copies only allowed files and checks version consistency.

## Development Process

The project keeps a session-memory system: a capped "active context" file (≤200 lines, last 3 sessions), a roadmap file and a `memory/` folder of numbered decisions (K1…K124). Older sessions rotate into an archive. This kept more than 70 sessions coherent without re-explaining the project each time. Every user-visible change is logged in a changelog and approved with a live preview before implementation.

## What I'd Do Differently

- **Measure content quality from day one.** The 14% / 22% distractor problems were in production for weeks before a measured review found them. The audit pipeline should have come before the first 500 words, not after.
- **Set the CSS baseline before writing UI.** The old-WebView fallbacks were retrofitted after device testing. Picking "Chrome 80" as the baseline up front would have avoided a whole class of silent bugs.
- **Release per feature from the start.** Account, social and avatar work were first planned as one bundle and later split into single-purpose releases (premium → streak → accounts). The split should have been the default plan.
- **Retire dead code from the web era earlier.** Unused API paths and a dead second XP calculation survived for a long time because removing them was always out of scope.
