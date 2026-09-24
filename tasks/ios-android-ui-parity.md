# iOS / Android parity

**Normative source: `docs/design-system.md`.** That document is the specification — type scale,
spacing, radius, colour, and the per-component numbers. This document is the *migration plan*:
what to change, in what order, and what it costs. Values are not duplicated between the two; if
they disagree, `docs/design-system.md` wins.

## The five documents

| Document | Role |
| --- | --- |
| `docs/design-system.md` | Specification — tokens and components |
| `docs/screens.md` | Specification — 129 screens, per-element values |
| `docs/copy-and-strings.md` | Specification — terminology, copy rules, key naming |
| `docs/ui-parity.md` | Work list — per-page UI divergence, page by page |
| `docs/logic-parity.md` | Work list — per-rule logic divergence, business code |

`ui-parity.md` and `logic-parity.md` are generated from the specification plus a cross-platform
audit. They are where you pick up work; the first three are where you check what "correct"
means. When a work-list row is closed, update the specification if the agreed value changed.

## Goal

Make the two native clients render the same screens with the same numbers. Today the feature
set is largely at parity, but padding, corner radius, font size, font weight and colour are
chosen per screen on both sides, so visually identical screens carry different values.

The deliverable is a set of concrete value changes. It is not a rewrite: both clients keep
their own rendering stack (SwiftUI / Jetpack Compose) and their own per-platform conventions
where those are justified.

**Unit convention:** iOS `pt` and Android `dp` are treated as 1:1 — both are
density-independent and resolve to the same physical size at reference density. All deltas
below are expressed in that shared unit.

## Scope

- Submodules: `repos/ham-ios`, `repos/ham-android`.
- No protobuf, backend, RN or cloud-config change.
- Logic divergence is tracked in `docs/logic-parity.md` and is a separate work stream from the
  visual work below. Some entries there need backend or product input before they can close.
- Feature gaps found during the survey (print UI, sport status card, RN bundle drift) are
  **out of scope** and listed at the bottom under *Related work* so they are not lost.

## How the values were obtained

Extracted from source, not from rendered output:

- iOS: `grep` over `Ham/iOS/ui/` for `.padding(`, `cornerRadius(`, `.font(`, `.fontWeight(`.
- Android: same over `android/feature/`, `android/core/ui/`, `android/app/`, plus a read of the
  token layer in `android/core/ui/.../config/Font.kt` and `Color.kt`.

iOS has **no** token layer — every value is a literal at the call site, and text styling that
omits `.font()` silently inherits SwiftUI's 17pt `body`. Android has a partial token layer
(`HamFontSize` / `HamFontStyle`) that is used inconsistently: raw `32.sp` and `16.sp` literals
appear beside it.

### Baseline: the two type scales disagree

| iOS SwiftUI style | pt | Android `HamFontStyle` | sp | Delta |
| --- | --- | --- | --- | --- |
| `.caption` | 12 | `caption` | 12 | match |
| `.caption2` | 11 | `caption2` | 11 | match |
| `.body` (default) | 17 | `body` | 16 | −1 |
| `.headline` | 17 | `headline` | 14 | −3 |
| `.title3` | 20 | `title3` | 16 | −4 |
| `.title2` | 22 | `title2` | 20 | −2 |
| `.title` | 28 | `title` | 24 | −4 |

`caption` and `caption2` align. Everything from `headline` up differs by 2–4pt, and because
iOS omits `.font()` in many places, a diffuse 17-vs-16 difference runs through the whole app.

---

## Phase 1 — Android numeric alignment

Single-value fixes, no structural change, no cross-repo dependency. All targets match the
verified iOS value.

### 1a. Status card container (fixes four cards at once)

`android/feature/status/.../component/CommonStatusCard.kt` vs iOS
`Ham/iOS/ui/status/card/CommonStatusCard.swift`.

| Property | iOS | Android now | Change |
| --- | --- | --- | --- |
| corner radius | 16 (`:80`) | 12.dp (`:49`) | `:49` → `16.dp` |
| header h-padding | 12 (`:71`) | 16.dp (`:68`) | `:68` → `12.dp` |
| content padding | 12 (`:22`, `:37`) | 16.dp (`:53`) | `:53` → `12.dp` |

Also add a `padding: Dp = 12.dp` parameter to the Android `CommonStatusCard` and thread it
into `:53`. The iOS component accepts a padding argument and
`Ham/iOS/ui/status/card/bus/StatusBusCard.swift:20` passes `0`; Android currently has no way
to express that, so the bus card cannot match.

### 1b. Brand colours

`android/core/ui/.../config/Color.kt`. The correct hex values already exist in
`core/ui/src/main/res/values/colors.xml` and are referenced by nothing.

| Token | iOS | Android now | Change |
| --- | --- | --- | --- |
| `ham_brand_sport` | #34C759 / dark #30D158 | #4CAF50 static (`:79`) | `:79` → `R.color.ham_green` (`colors.xml:75`) |
| `ham_brand_score` | #FF9500 / dark #FF9F0A | #FF9800 static (`:82`) | `:82` → `R.color.ham_orange` (`colors.xml:73`) |

Both tokens are already `@Composable get()`, so adding `colorResource` needs no signature
change. This also fixes the weather card, which reads `Color.ham_orange`
(`feature/status/.../weather/WeatherCardView.kt:101`).

**Check before changing:** `Color.ham_green` and `Color.ham_orange` have other consumers
(e.g. `ToastManager.kt:114`). If those must keep the Material value, point
`ham_brand_sport` / `ham_brand_score` at the resources directly instead of re-pointing the
shared tokens.

### 1c. Course timetable

| Property | iOS | Android now | Change |
| --- | --- | --- | --- |
| period-number column width | 42 (`CourseViewBodyView.swift:16`) | 32.dp (`CourseMainView.kt:71`) | `→ 42.dp` |
| weekday header height | 36 (`CourseViewBodyView.swift:25`) | 48.dp (`CourseMainView.kt:70`) | `→ 36.dp` |
| today cell fill | solid `#E6F1FF` (`CourseViewBodyWeekdayView.swift:82`) | `ham_blue @0.15f` (`CourseMainViewWeekdayCell.kt:88`) | `→ Color.ham_lightBlue` |
| empty cell fill | solid `#EDEEEF` (`CourseViewBodyCourseItemView.swift:143`) | `ham_gray @0.15f` (`CourseMainViewBodyCell.kt:187`) | `→ Color.ham_lightGray` |
| grid cell radius | 10 (`CourseViewBodyCourseItemView.swift:43`) | 12.dp (`CourseMainViewBodyCell.kt:183`) | `→ 10.dp` |
| grid cell padding | 2 (`:52`) | 4.dp (`:196`) | `→ 2.dp` |

The column-width and header-height changes shift the entire grid by 10dp and 12dp
respectively — verify against a populated timetable, not an empty one.

### 1d. Score screen

| Property | iOS | Android now | Change |
| --- | --- | --- | --- |
| function-card background | gray @0.1 translucent (`ScoreMainViewFunctionCard.swift:64`) | `Color.ham_lightGray` solid (`ScoreMainViewFunctionCard.kt:92`) | `→ Color.ham_gray.copy(alpha = 0.1f)` |
| instructor font in score row | 12 caption (`.swift:30`) | 16sp, same span as course name (`ScoreMainDetailScoreCard.kt:269`) | `→ HamFontStyle.caption.fontSize` |
| colour bar height / radius | 35 / 6 (`.swift:24`, `:22`) | 28.dp / 3 (`:262`, `:250`) | `→ 35.dp` / `6.dp` |
| semester row spacing | 15 (`.swift:87`) | 8.dp (`:142`) | iOS `→ 8` (see note) |
| function-card radius | 16 (`.swift:63`) | 12.dp (`:91`) | `→ 16.dp` |

Note: the semester row spacing is the one item in this phase fixed on the **iOS** side.
Android's 8dp matches its own spacing grid; iOS's 15 is the outlier.

### 1e. Sport screen

| Property | iOS | Android now | Change |
| --- | --- | --- | --- |
| card order | banner → current order → quick order (`SportMainView.swift:17-31`) | banner → **quick** → **current** (`SportMainView.kt:34-35`) | swap `:34` and `:35` |
| function-card row height | 150 (`.swift:132`) | 160.dp (`SportMainViewFunctionButtonCard.kt:53`) | `→ 150.dp` |
| large icon size | 64 (`.swift:20`) | 72.dp (`:70`) | `→ 64.dp` |
| small-card trailing icon | 32 @0.75 (`.swift:79`) | 56.dp @0.65 (`:121`) | `→ 32.dp` / `0.75f` |

### 1f. Library screen

| Property | iOS | Android now | Change |
| --- | --- | --- | --- |
| gap between cards | ~8, implicit (`LibraryMainView.swift:16`) | `spacedBy(16.dp)` (`LibraryMainView.kt:43`) | `→ 8.dp` |
| banner height | 200 (`LibraryMainViewBannerCard.swift:191`) | 180.dp (`.kt:50`) | `→ 200.dp` |
| banner background icon | 36 @0.25 (`.swift:112`) | 72 @0.20 (`.kt:59`) | `→ 36.dp` / `0.25f` |
| booking status-bar v-padding | 8 (`.swift:25`) | 12.dp (`ReservedCard.kt:70`) | `→ 8.dp` |
| quick-book location font | 12 caption (`.swift:50`) | 16sp body (`LibraryMainViewQuickBookCard.kt:75`) | `→ HamFontStyle.caption` |
| quick-book seat number | 28 Bold (`.swift:23`) | raw `32.sp` (`.kt:74`) | `→ HamFontStyle.title` and align with 1g |
| quick-book button radius / fill | 10 / blue@0.10 (`.swift:116`, `:117`) | 12.dp / blue@0.25 (`.kt:122`, `:121`) | `→ 10.dp` / `0.10f` |
| function-card row height | 150 (`.swift:132`) | 142.dp (`FunctionCard.kt:54`) | `→ 150.dp` |
| analytics bar height | 30 (`.swift:95`) | 20.dp (`.kt:200`) | `→ 30.dp` |
| analytics group-name font | 17, `.font` omitted (`.swift:90`) | 12sp caption (`.kt:121`) | **iOS** `→ .font(.caption)` |

### 1g. Toast

`android/core/ui/.../ToastManager.kt` vs `Ham/iOS/ui/common/toast/ToastView.swift`.

| Property | iOS | Android now | Change |
| --- | --- | --- | --- |
| subtitle font | 12 caption (`.swift:95`) | 16sp body — same size as title (`:191`) | `→ HamFontStyle.caption` |
| icon size | 36 (`.swift:87`) | 32.dp (`:174`) | `→ 36.dp` |
| icon→text gap | 5 (`.swift:85`) | 8.dp (`:170`) | `→ 5.dp` |
| duration | 3.3s (`.swift:37`) | 2.0s (`:150`, `:202`) | `→ 3300` |
| error background | #FF3B30 | `R.color.red` #F44336 (`:123`) | `→ R.color.ham_red` |
| corner radius | 0, plain rectangle (`.swift:105`) | 12.dp (`:166`) | **keep Android** — see note |
| normal background | `secondarySystemBackground` (adaptive) | hardcoded `Color(0xFFEDEEEF)` (`:105`) | `→ Color.ham_lightGray` so it dark-modes |

Note: iOS's 0 radius is an omission, not a decision. Android's 12.dp stays.

### 1h. Delete dead `dimens.xml`

`android/core/ui/src/main/res/values/dimens.xml` (plus copies in `app/` and `values-land/`).
`grep -rn "R.dimen\|dimenResource"` over all `.kt` returns **zero** hits — the Compose layer
never reads it. Worse, it conflicts with the `HamFontSize` scale that *is* live:

| Token | dimens.xml (dead) | HamFontSize (live) |
| --- | --- | --- |
| title | 32sp | 24sp |
| title2 | 26sp | 20sp |
| title3 | 18sp | 16sp |
| headline | 18sp | 14sp |
| caption2 | 10sp | 11sp |

Only `body`/`normal` (16) and `caption` (12) agree. Leaving it in place is a trap: anyone who
"fixes" a font size from this file moves it away from parity.

---

## Phase 2 — Normalise the type scale

**Decided: iOS is the baseline.** Android converges on the iOS type scale. The normative
values live in `docs/design-system.md` — this section only covers the migration.

`HamFontSize` in `android/core/ui/.../config/Font.kt` takes these values, and iOS gains a
matching `Font+Ham.swift` so both sides read from named tokens:

| Token | iOS (baseline) | Android now | Change |
| --- | --- | --- | --- |
| `largeTitle` | 34 | 28 | +6 |
| `title` | 28 | 24 | +4 |
| `title2` | 22 | 20 | +2 |
| `title3` | 20 | 16 | +4 |
| `headline` / `headlineBold` | 17 semibold / bold | 14 | **+3** |
| `body` / `bodyBold` | 17 | 16 | +1 |
| `callout` | 16 | — | add |
| `subheadline` | 15 | — | add |
| `footnote` | 13 | — | add |
| `caption` / `captionBold` | 12 | 12 | — |
| `caption2` | 11 | 11 | — |

**This is not a find-and-replace.** `headline` grows 21% (14 → 17sp) and Android layouts were
tuned around the smaller value, so text-heavy screens will reflow. Land it as its own change
with visual review on every affected screen — see *Open questions* #3 on whether to stage it
separately from the Phase 1 numeric fixes.

Two follow-ups that come with the change:

- **iOS must stop relying on the implicit default.** Omitting `.font()` yields SwiftUI's 17pt
  `body`, which happens to match this scale, but makes the value invisible and unsearchable.
  Write the token explicitly.
- **Android must stop relying on MaterialTheme inheritance.** Once `body` is 17 rather than
  Material's 16, call sites that omit a style still get 16. Pass the token explicitly.

Deliverable: every text style in both apps resolves through a named token, and no text renders
at a size that cannot be traced to one.

---

## Phase 3 — Shared spacing and radius tokens

Neither side has one. iOS has 14 distinct corner-radius values and 11 distinct padding values,
Android has 15 and 20+, all hardcoded per screen. Phase 1 can align today's values; without a
token layer the next new screen drifts again.

Proposal: agree a small scale (radius 4 / 8 / 10 / 12 / 16; spacing 4 / 8 / 12 / 16 / 24),
express it once per platform, and migrate the shared components (`HamCardView`,
`CommonStatusCard`, `HamButton`, `HamSheet`) first. Feature screens can follow incrementally.

Cross-repo by nature — needs a decision from both maintainers before any code moves.

---

## Related work (out of scope here, tracked separately)

Found during the survey, not part of the token alignment:

- ~~**Print UI is Android-only.**~~ **Corrected: print is on both.** The earlier survey was
  wrong. iOS has `Route.swift:33-34` (`.libraryPrint`, `.printPrepare`) wired to
  `LibraryPrintView` and `PrintPrepareRouteView`, plus `PrintPrepareView.swift` and
  `PrintFileSourcePicker.swift` — routes *and* views, not just the data layer. The remaining
  print work is numeric, not a build-out: the library print card's arrow is `12` on iOS and
  `20.dp` on Android, and the share-hint copy differs. See `screens.md` §12.
- **Motion is unspecified on both platforms.** Neither client has a duration scale, and 352 iOS
  `withAnimation {}` sites plus ~91 Android `animate*AsState` sites specify no duration at all,
  so they inherit different platform defaults. Specified in `design-system.md` §2.9; the work
  list is `ui-parity.md` §15.
- **Accessibility is absent on both platforms**, failing in the same direction rather than
  diverging — so it is a shared build-out, not a parity fix. iOS has 2 labelled icons of 236 and
  Android 7 of 283. `design-system.md` §2.10, work list `ui-parity.md` §16.
- **Loading, empty, error, offline and session-expired states are unrendered** on both. Neither
  shared error component offers a retry, and neither app can tell the user it is offline.
  `design-system.md` §3.10, work list `ui-parity.md` §17.
- ~~**Sport status card is iOS-only.**~~ **Retracted — Android has one too.**
  `StatusViewCardType` lists `Sport` (`StatusViewCardScoreManager.kt:40`),
  `StatusView.kt:250-252` composes it, and `StatusSportCard.kt` is 212 lines of real UI.
  Same for the schedule card: `ScheduleCard.kt` is called at `StatusView.kt:247` — its own
  header comment ("until now this was an empty container") is stale and misled the earlier
  survey. Both platforms render all 7 status cards; the work is the container, not the cards.
- **Android weather card shows no data-source attribution** while iOS links Apple's required
  WeatherKit attribution. Android fetches CMA data with a spoofed browser User-Agent and
  displays no credit — a compliance question more than a parity one.
- **RN bundle drift.** Android is at `ham-rn@4f3d241`, iOS at `0939555` — six commits behind,
  one of which is a bug fix (`keep course studentId optional`).
- **Language and widget settings are Android-only**; iOS follows the system locale and has no
  widget settings screen (WidgetKit owns refresh scheduling, so this may be correct as-is).

---

## Open questions

1. ~~Phase 2 direction: Option A or Option B?~~ **Resolved — iOS is the baseline (Option B).**
   See `docs/design-system.md` for the resulting values.
2. Are the 17-vs-16 body-text differences worth a full sweep, or is 1pt inside tolerance and
   only the 2–4pt title differences worth fixing? → **Resolved by the baseline decision:**
   all of them, including body. Staging is still open (see #3).
3. **NEW — stage the type change?** The `headline` 14→17sp bump is the largest reflow in the
   migration. Land the non-type fixes first and type separately, or one PR with full visual
   review?
4. Chevron glyphs differ 3× (iOS 8pt vs Android's default 24dp Material icon) across roughly a
   dozen screens. `docs/design-system.md` §10 proposes 12; needs confirmation.
5. Confirm the secondary-text grey: iOS `Color.gray` #808080 vs Android
   `Color.ham_text_secondary` #888888. Driven by `Color.kt:29` being hardcoded `Gray` instead
   of a resource reference.
6. **NEW — toast radius.** The spec says 8, but the measured iOS value is 0 (a plain
   rectangle, presumably an omission). Confirm this deliberate departure from the baseline.

## Verification

Per submodule, as usual:

- Android: `./gradlew :app:assembleGithubDebug` plus visual check on the four affected
  screens (status, course, score, sport, library) in light and dark mode.
- iOS: build the `Ham` scheme and check the same screens.
- Both: confirm the status card, course grid and score rows against a populated account —
  empty states hide several of these differences entirely.
- No automated UI test exists on either side for these values; this is a visual diff, so
  before/after screenshots per screen are the acceptance evidence.
