# Ham design system

Cross-client visual specification for the native apps. Every number here is measured from
source, not invented. Where the two clients disagreed, **iOS is the baseline**.

Status: **proposed**. Not implemented on either side yet.

## Contents

- [1. How to read this](#1-how-to-read-this)
- [2. Colour](#2-colour)
- [3. Type](#3-type)
- [4. Spacing](#4-spacing)
- [5. Radius](#5-radius)
- [6. Elevation](#6-elevation)
- [7. Iconography](#7-iconography)
- [8. Components](#8-components)
- [9. Platform-sanctioned divergences](#9-platform-sanctioned-divergences)
- [10. Undecided](#10-undecided)
- [11. Review checklist](#11-review-checklist)
- [Appendix A — Known defects](#appendix-a--known-defects)

---

## 1. How to read this

### Conformance

**Must** means a reviewer should block a PR that violates it without a stated reason.

- New UI **must** take padding, radius, font, and colour from a named token here.
- Literal numbers in view code are permitted only where no token applies, and **should** be
  treated with suspicion in review.
- Every colour **must** have a light and a dark value and **must** be an adaptive resource
  (asset-colour set on iOS, `values` + `values-night` on Android). Never a hardcoded
  `Color(0xFF…)` in view code.

### Units

**pt** (iOS) and **dp** (Android) are treated as 1:1 — both are density-independent and
resolve to the same physical size at reference density. Type sizes are **pt** on iOS and
**sp** on Android, likewise treated 1:1.

### Marking

Each normative value is tagged:

- **[T]** transcribed — both platforms already agree, we are just writing it down.
- **[M]** majority or tokenised — one side has a token or a clear majority, the other migrates.
- **[C]** chosen — neither codebase has a coherent value; this document picks one.

### Why iOS is the baseline

iOS is the only client whose values are internally coherent for the surfaces that matter most.
It has one card container (`HamCardView`, padding 16 / radius 16) used by 71 call sites, and
its brand colours are Apple System Colours that already carry correct dark-mode pairs.

Android's drift was traced to wiring, not intent: `ham_brand_sport` and `ham_brand_score` are
aliases pointing at Material semantic resources (`R.color.green`, `R.color.warning`) while the
correct hexes sit unused in `colors.xml`. The wrong values won by accident.

**This choice has a cost.** Adopting the iOS type scale raises Android `body` 16 → 17sp and
`headline` 14 → 17sp (+21%). Android layouts were tuned around the smaller values. See
[§3.6](#36-migration-risk).

---

## 2. Colour

Measured from `Ham/shared/Assets.xcassets/color/*.colorset`,
`Ham/shared/utils/extension/Color+Ham.swift`, and `android/core/ui/.../config/Color.kt`.

### 2.1 Brand

Each module has its own brand colour. Used for card header bands, tinted fills, icons, and
status-card accents.

| Token | Light | Dark | Android now | Change |
| --- | --- | --- | --- | --- |
| `brand.course` | #1B5E20 | #1B5E20 | same | — |
| `brand.schedule` | #01579B | #01579B | same | — |
| `brand.library` | #007AFF | #0A84FF | same | — |
| `brand.sport` | #34C759 | #30D158 | #4CAF50 static | **change** |
| `brand.score` | #FF9500 | #FF9F0A | #FF9800 static | **change** |
| `brand.coursescore` | #283593 | #283593 | same | — |
| `brand.bus` | #A2845E | #AC8E68 | same | — |
| `brand.pay` | #BF360C | #BF360C | same | — |

**[M]** The two changes are one-line fixes in `Color.kt` — `R.color.ham_green` and
`R.color.ham_orange` already hold the correct hexes and are referenced by nothing. Fixing them
also fixes the weather card, which reads `Color.ham_orange`.

### 2.2 Surface

| Token | Light | Dark | Used for |
| --- | --- | --- | --- |
| `surface.primary` | #F9F9F9 | #000000 | Screen background |
| `surface.secondary` | #FFFFFF | #0F0F0F | Cards |
| `surface.tertiary` | #EDEEEF | #0F0E0F | Chips, inactive cells, borders |
| `surface.tint` | #E6F1FF | #010D18 | Today / selected cells |

**[T]** All four are identical on both platforms today.

Cards are separated from the page by the `surface.secondary` / `surface.primary` fill delta
alone — there is no shadow or border anywhere. See [§6](#6-elevation).

### 2.3 Text

| Token | Light | Dark | Android now | Change |
| --- | --- | --- | --- | --- |
| `text.primary` | #000000 | #FFFFFF | same | — |
| `text.secondary` | #8E8E93 | #98989D | #888888 no dark variant | **change** |
| `text.tertiary` | — | — | — | new, absorbs opacity-on-text drift |
| `text.placeholder` | — | — | — | new |
| `text.link` | #007AFF | #0A84FF | `ham_blue` #007AFF | — |
| `text.danger` | #FF3B30 | #FF453A | `ham_red` #F44336 | **change** |

**[M]** Define these as **semantic roles, never as hex literals in the spec.** iOS's
`Color.primary` and `Color.gray` are dynamic; freezing them to #000000/#8E8E93 in a document
would break iOS dark mode. Quote the hexes only as the *Android* implementation of each role.

Two real bugs behind this table, both Android:

- `Color.kt:29` — `ham_text_secondary` is hardcoded Compose `Gray`, not a resource reference.
  It cannot adapt to dark mode, and `values-night/colors.xml` keeps `gray` at #888888 anyway.
- Three reds are in play for the same string: iOS `Color.red` (#FF3B30), Android `ham_red`
  (#F44336), and raw `Color.Red` (#FF0000) used at `SyncLogoutView.kt:49` for 退出登录 while
  `UserCenterMainView.kt:183` uses #F44336 for the identical string.

### 2.4 Feedback

| Token | Light | Dark |
| --- | --- | --- |
| `feedback.info` | #007AFF | #0A84FF |
| `feedback.success` | #34C759 | #30D158 |
| `feedback.warning` | #FFCC00 | #FFD60A |
| `feedback.error` | #FF3B30 | #FF453A |

**[C]** Android's toast has only three types (normal / success / error); iOS has five. Add
`warning` and `info`.

### 2.5 Tint recipes

Brand colour is never used at full strength behind content. Three recipes cover every case:

| Recipe | Formula | Used for |
| --- | --- | --- |
| `tint.subtle` | `surface.secondary` + brand @ 0.15 | Large tiles, function buttons, status-card header |
| `tint.chip` | brand @ 0.10, brand-coloured text | Filter chips, small pills |
| `tint.active` | brand @ 1.0, white text | Filled primary buttons |

**[M]** The 0.15 recipe is the house style — it appears as `color.opacity(0.15)` on iOS and
`color.copy(alpha = 0.15f)` on Android in the SSO authorize button, which is a near-exact
cross-platform match. It is currently **not** applied consistently: the same semantic role uses
alphas of 0.10, 0.13, 0.15, 0.20, and 0.25 across both platforms.

Two rules that fall out:

1. **Always put the tint over an opaque base.** Android's `MyViewFunctionButtonView` omits the
   base layer, so those buttons are translucent against whatever is behind them.
2. **Tinted button text is the brand colour, not the label colour.** iOS's sport 预定 button
   uses `.ham_text_t1Color` where Android uses `ham_green`. Take Android's: brand.

---

## 3. Type

### 3.1 Scale

| Token | Size | Weight | iOS | Android now | Change |
| --- | --- | --- | --- | --- | --- |
| `largeTitle` | 34 | Bold | `.largeTitle.bold()` | 28 | +6 |
| `title` | 28 | Bold | `.title.bold()` | 24 | +4 |
| `title2` | 22 | Bold | `.title2.bold()` | 20 | +2 |
| `title3` | 20 | Bold | `.title3.bold()` | 16 | +4 |
| `headline` | 17 | Semibold | `.headline` | 14 | +3 |
| `headlineBold` | 17 | Bold | `.headline.bold()` | 14 | +3 |
| `body` | 17 | Regular | `.body` | 16 | +1 |
| `bodyBold` | 17 | Bold | `.body.bold()` | 16 | +1 |
| `callout` | 16 | Regular | `.callout` | — | add |
| `subheadline` | 15 | Regular | `.subheadline` | — | add |
| `footnote` | 13 | Regular | `.footnote` | — | add |
| `caption` | 12 | Regular | `.caption` | 12 | — |
| `captionBold` | 12 | Bold | `.caption.bold()` | 12 | — |
| `caption2` | 11 | Regular | `.caption2` | 11 | — |

**[M]** `caption` (12) and `caption2` (11) already agree, and they are the most-used styles in
both apps — 279 uses on iOS, 266 on Android. They are untouched.

**Retire on Android:** `largeTitle` (0 call sites). `title2` and `title3` are currently both
16sp, identical to `body` — three names, one size. Give them the real iOS values or delete them.

**iOS must stop relying on the implicit default.** Omitting `.font()` yields SwiftUI's 17pt
`body`, which happens to match this scale, but it makes the value invisible at the call site
and unsearchable. Always name the token.

**Android must stop relying on MaterialTheme inheritance.** Once `body` is 17 rather than
Material's 16, call sites that omit a style still get 16. Always pass the token.

### 3.2 Weight convention

**[C] Use Bold for emphasis. Do not use Semibold.**

Evidence: iOS uses bold 202 times vs semibold 26 (7.8 : 1). Android has exactly two weights in
product code — `FontWeight.Bold` and `FontWeight.Normal`; `SemiBold` and `Medium` are absent
(`FontWeight.Medium` appears once, inside a `MaterialTheme.typography` object that nothing
references). Adopting Bold costs ~26 iOS sites and 0 Android sites; adopting Semibold costs
~202 iOS sites and requires inventing a weight Android does not have.

The decisive point is that iOS's semibold is drift, not a rule — one card contains two sibling
headings, same size, one bold and one semibold (`ScoreMainViewMyScoreDataCard.swift:24` and
`:48`). Whichever the author typed.

A practical reason: at 11–12pt, semibold thins CJK strokes, and this is a Chinese-language app
whose caption tier is 11–12.

Also fix the token definitions: `HamFontStyle.headline`, `caption`, and `caption2` declare no
`fontWeight` at all. Give all tokens an explicit weight, and an explicit colour — the missing
colours are why `ReservedCard.kt:74` renders an uncoloured hero number.

### 3.3 Line height

**[C] Neither platform sets line height today.** This is the largest genuine gap in the system.

iOS has zero `lineSpacing` / `lineHeight` / `NSParagraphStyle` declarations in app code.
Android has three, all inside `MaterialTheme.typography`, which is referenced zero times — so
they are unreachable. None of the eleven `HamFontStyle` tokens declares a line height.

The two platforms "agree" only by accident, and the defaults are different numbers: iOS text
styles carry roughly 1.20–1.35× leading; Compose defaults to font metrics with
`includeFontPadding` on. Negligible at 11–12sp, several dp at 24–28sp.

Proposed: add `lineHeight` to every token.

| Token | Size | Line height | Ratio |
| --- | --- | --- | --- |
| `largeTitle` | 34 | 40 | 1.18 |
| `title` | 28 | 34 | 1.21 |
| `title2` / `title3` | 22 / 20 | 28 / 28 | 1.27 / 1.40 |
| `headline` / `body` | 17 / 17 | 24 / 24 | 1.41 |
| `callout` / `subheadline` | 16 / 15 | 22 / 22 | 1.38 / 1.47 |
| `footnote` | 13 | 18 | 1.38 |
| `caption` / `caption2` | 12 / 11 | 16 / 16 | 1.33 / 1.45 |

On iOS there is no native line-height modifier; this needs a helper producing a `UIFont` with
custom leading via `UIFontMetrics` / `NSParagraphStyle`.

### 3.4 Truncation

**[T]** Both teams independently converged on the same convention: iOS has 68 `lineLimit` uses,
Android 66 `maxLines`, with the same distribution.

| Role | Lines |
| --- | --- |
| Screen title, row title | 1 |
| Row meta, two-line card title | 2 |
| Grid cell label, card body | 3 |
| Long-form preview | 5 |
| Body prose, hints | unbounded |

Two rules:

1. **`maxLines = n` must always be paired with `TextOverflow.Ellipsis` on Android.** 23 of 66
   sites (35%) set `maxLines` without it and hard-clip with no visual ellipsis. iOS cannot have
   this bug — `lineLimit(n)` implies tail truncation.
2. Write **unbounded**, not `lineLimit(0)`. The latter is a SwiftUI idiom that reads as "zero
   lines" but means "unbounded".

### 3.5 Numeric display

**[C]** iOS has a real rounded-digit convention, scoped to four modules and applied without
exception inside them: coursescore (4/4 hero numbers), sport (14/14), bus (9/9), weather (1/1).
Android has none — `fontFeatureSettings` and custom `Font(...)` are both absent from product
code.

Promote it to a system role:

| Token | Size | Weight | Design | Used for |
| --- | --- | --- | --- | --- |
| `numeric.regular` | 12 | Regular | default | Course period rail, clock times |
| `numeric.display` | 28 | Bold | rounded | Seat numbers, GPA, rank, rate |
| `numeric.hero` | 36 | Regular | rounded | Temperature |

Add **tabular figures** to `numeric.regular`. Neither platform uses monospaced digits anywhere
(0 hits for `monospacedDigit()` on iOS, 0 for `FontFeature.tabularNumbers` on Android), so the
12pt course-period rail and the sport clock times jitter horizontally as digits change width.

### 3.6 Migration risk

**[M]** The type change is the expensive part of this document.

| Change | Impact |
| --- | --- |
| `headline` 14 → 17sp | +21%. Largest reflow. Text-heavy Android screens will move. |
| `body` 16 → 17sp | +6%, but 240 Android call sites. |
| `title3` 16 → 20sp | +25%, but only 2 call sites. |
| `title2` 20 → 22sp | +10%. |
| `title` 24 → 28sp | +17%. Only 36 call sites, all hero values. |
| `largeTitle` 28 → 34sp | 0 call sites — just delete it. |

Land the non-type changes first, then type as a separate pass with full visual review.

### 3.7 Text roles

The table below is the one to use day to day. `LL` = line limit.

| # | Role | Size | Weight | Colour | LL | Mark |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Screen / large title | 34 | Bold | `text.primary` | 1 | **[M]** |
| 2 | Section header | 12 | Regular | `text.secondary` | 1 | **[M]** |
| 3 | Card title | 17 | Bold | `text.primary` | 2 | **[M]** |
| 4 | Card subtitle | 12 | Regular | `text.secondary` | 1 | **[T]** |
| 5 | Primary value / big number | 28 | Bold, rounded | `text.primary` | 1 | **[M]** |
| 6 | List row title | 17 | Bold | `text.primary` | 1 | **[M]** |
| 7 | List row subtitle | 12 | Regular | `text.secondary` | 1 | **[T]** |
| 8 | Body text | 17 | Regular | `text.primary` | unbounded | **[M]** |
| 9 | Caption / meta | 12 | Regular | `text.secondary` | 1 | **[M]** |
| 10 | Overline / eyebrow | 12 | Regular | `text.secondary` | 1 | **[T]** |
| 11 | Link | 12 | Regular | `text.link`, underlined | 1 | **[C]** |
| 12 | Destructive | 17 | Regular | `text.danger` | 1 | **[C]** |
| 13 | Placeholder / empty state | 12 | Regular | `text.placeholder` | unbounded | **[C]** |
| 14 | Badge / chip label | 12 | Bold | on `tint.chip` | 1 | **[M]** |
| 15 | Numeric, small | 12 | Regular, tabular | `text.primary` | 1 | **[T]** |

Notes on the rows that are not simple transcriptions:

- **Row 2 (section header)** — the role exists only on iOS, in one screen
  (`CourseSettingView*Section.swift`, 3 instances). Android has no such component; the same
  screen uses 16sp bold card titles instead. Android needs a new `HamSectionHeader` composable.
  Do **not** repurpose `HamCardView(title=)` — 78 call sites depend on it.
- **Row 9 (caption colour)** — size agrees at 12; **colour polarity is inverted.** iOS defaults
  captions to secondary grey (47 uses secondary vs 14 primary). Android defaults them to
  primary (90 vs 45), and **68% of Android captions pass no colour at all**, so their rendered
  colour is decided by the enclosing container. Choose secondary.
- **Row 11 (link)** — links never use a brand colour; those are module accents. iOS underlines,
  Android does not. Take iOS. Android already defines `<color name="link">#007AFF</color>` in
  `colors.xml:14` and references it from zero Compose files.
- **Row 13 (placeholder)** — no token exists on either side. iOS has three ad-hoc helpers with
  three different colours (including one literal `.black`); Android has one genuinely reusable
  `HamTextField` that tokenises size and colour. Take Android's structure, iOS's size.
- **Row 3/6 (title weight)** — iOS uses semibold in the shared card component but bold
  everywhere else. Per [§3.2](#32-weight-convention), standardise on Bold.

---

## 4. Spacing

### 4.1 Scale

4pt grid.

| Token | Value | Used for |
| --- | --- | --- |
| `space.1` | 2 | Hairline gaps, course grid gutters |
| `space.2` | 4 | Icon-to-label, tight stack gaps |
| `space.3` | 8 | Default stack gap, internal padding, card gap |
| `space.4` | 12 | Status-card padding, vertical padding inside tiles |
| `space.5` | 16 | Screen margin, card padding |
| `space.6` | 24 | Section separation, screen top offsets |
| `space.7` | 32 | Hero offsets |

| Current outlier | Collapse to |
| --- | --- |
| 6 | 8 (`space.3`) |
| 10 | 8 (`space.3`) |
| 15 | 16 (`space.5`) |
| 20 | 24 (`space.6`) |

### 4.2 Page layout

| Property | Value | Mark | Notes |
| --- | --- | --- | --- |
| Screen horizontal margin | **16** | **[T]** | All 12 non-timetable measurements on both platforms are exactly 16. No screen uses anything else. |
| Gap between cards | **8** | **[M]** | Android authors it (status, sport, score, settings). iOS never authors it — its `VStack`s omit spacing, so the rendered value is SwiftUI's default, not a design decision. Adopting 8 codifies what iOS already renders. |
| Screen background | `surface.primary` | **[T]** | Identical on both. |
| Content top offset | header + status bar | — | See below. |
| Bottom spacing (tab-root screens) | navigation-bar inset **+ 80** | **[M]** | Android computes it (`bottomWithTabBarHeight()`, `Spacer.kt:20`). iOS hardcodes 96 / 100 / 128 / 90, and sport, score and course-settings have **no trailing spacer at all** — content can run under the tab bar. |
| Bottom spacing (pushed/child screens) | navigation-bar inset **+ 24** | **[M]** | `HomeContainer.kt:83`, `ScoreMainView.kt:186`. |

**[M] Android has a shared page scaffold and iOS does not.** This is the single biggest
structural difference between the two codebases.

- Android has four composables in `core/ui/container/nav/`. `HamHomeContainer`
  (`HomeContainer.kt:50`) is used by library and sport; `HamNavigationView`
  (`NavigationView.kt:373`) is the base; `HamNavigationLazyScrollView` (`:333`) and
  `HamNavigationScrollView` (`:300`) wrap list and scroll screens. They own the page background
  (`ham_bg_b1`), the top offset (`TOOLBAR_HEIGHT 42.dp + statusBar`, or `headerHeight 36.dp`),
  and the bottom spacer.
- iOS has none. `Ham/iOS/ui/common/container/MainContainer.swift` is a **7-line empty stub** —
  imports only, no types. Every iOS screen hand-rolls `ScrollView` + `.padding(.horizontal)` +
  a hardcoded bottom `Spacer`.

The cheapest path: promote `MyViewNavContainer` out of `ui/my/component/nav/` into
`ui/common/container/`, parameterise its hardcoded `56` / `72` / `100` constants, and make it
the iOS counterpart to `HamHomeContainer`. It already mirrors the Android structure.

**Do not converge the top chrome.** iOS defers to `UINavigationController`; Android renders an
in-Compose header. Specify the result — content begins below a 42dp (or 36dp) header plus the
status bar — and let each platform meet it natively.

**The timetable is an explicit full-bleed exception**, not a competing default: iOS uses a 10pt
header inset and Android a 4dp grid inset because the grid must reach the screen edges. Record
it as such so nobody "fixes" it.

---

## 5. Radius

| Token | Value | Used for |
| --- | --- | --- |
| `radius.1` | 2 | Progress bars |
| `radius.2` | 4 | Checkboxes, dots, smallest chips |
| `radius.3` | 6 | Badges, colour bars, filter chips |
| `radius.4` | 8 | Buttons, inner cards, banners, text fields |
| `radius.5` | 10 | Course grid cells, weekday cells, period cells |
| `radius.6` | 12 | Controls, status-card header pills, alert cards |
| `radius.card` | 16 | Cards, status cards, large tiles |

**[M]** iOS currently has 14 distinct radius values, Android 15. Most collapse into the seven
above. `radius.5` (10) exists because the course timetable uses 10 consistently across all
three cell types on iOS — it is not a general-purpose value, so do not reach for it outside the
grid.

---

## 6. Elevation

**[T] There is no elevation. Cards are flat on both platforms.**

Verified: zero `.shadow` on any iOS card (`HamCardView` and `CommonStatusCard` both have none),
and Android's `HamCardView` is a plain `Box` with `.clip().background().clipToBounds()` — not a
Material3 `Card`/`Surface`, with no elevation parameter and no way for callers to opt in.
Across 1155 `.kt` files: `tonalElevation` 0 hits, `ElevatedCard`/`OutlinedCard` 0 hits, and the
four `Modifier.shadow` uses are all non-card (tab pill, floating back button, app icon, a
preview). `MaterialTheme` passes no `shapes` and no elevation scale.

Cards are separated purely by the `surface.secondary` vs `surface.primary` fill delta plus the
16 radius. **Adding elevation would be net-new work, not a parity fix** — this document does
not propose it.

Curiosity: Android defines `ham_card_shadow_color` (`#28000000` / `#19000000`) in
`colors.xml:88` and `values-night/colors.xml:55`, referenced only by a View-system window style
that Compose never reaches.

---

## 7. Iconography

| Token | Value | Used for |
| --- | --- | --- |
| `icon.xs` | 12 | Inline with caption text |
| `icon.sm` | 20 | List-row leading glyph |
| `icon.md` | 24 | Status-card header icon, chevrons, icon-button glyph |
| `icon.lg` | 32 | Small-card trailing decoration |
| `icon.xl` | 64 | Large tile, banner foreground |
| `icon.hero` | 72 | Banner foreground (tall variant) |
| `icon.watermark` | 128 | Card background watermark |

Two things to fix:

1. **[M] iOS leaves many icons unsized.** The status-card header icon is a bare
   `Image(systemName:)` with no `.font()` or `.frame()` (`CommonStatusCard.swift:52`), inheriting
   roughly 17pt, while Android pins 24dp. Size them explicitly.
2. **[C] Chevrons differ 3×.** iOS writes `.font(.system(size: 8))` in 22 places across 8 files;
   Android uses the Material default 24dp. Neither is chosen — 8 is small enough to look like an
   oversight, 24 is just whatever Material does. **Proposal: 12**, which is between them and
   already a spacing token. See [§10](#10-undecided).

### Watermark

**[C]** The decorative background icon is the least consistent element in the app. Nine distinct
sizes across thirteen live watermarks (iOS 128, 180; Android 60, 128×3, 144, 172, 200, 240,
250×2). Neither platform's declared default is the house style — Android's `size = 60.dp` is
used at 2 of 11 sites and iOS's `200` at 0 of 2.

Two rules:

1. **A size number does not mean the same thing on the two platforms.** iOS renders the
   watermark as `.font(.system(size:))` — an SF Symbol whose *ink* is much smaller than its box.
   Android uses `.requiredSize()` — the vector fills the whole box. A 128 iOS glyph and a 128dp
   Android icon are not optically equivalent. The spec must mandate a **fixed frame**
   (`.frame(width:height:)` + `.resizable()`) on iOS, not a font size.
2. Standardise on `icon.watermark` 128, alpha **0.12**, anchored **bottom-trailing** at offset
   **(16, 16)**, clipped to the card. Both platforms already agree on the anchor; the offsets
   and alphas are chaos.

Note that the most prominent iOS watermarks are hand-rolled outside the primitive entirely —
220pt crown (`ScoreMainViewMyScoreDataCard.swift:78`), 156pt person, 142pt doc, and four 128pt
glyphs. That is where the visual language actually lives.

---

## 8. Components

### 8.1 Card

**[T]** Both platforms already agree on the primitive. Nothing to change.

```
CARD                       radius 16 · padding 16 · bg surface.secondary · no shadow, no border
├── HEADER?                [plain]  transparent, inset 0 — inherits the card's 16
│                          [status] bg = brand @ 0.15, padding 12 all sides
│   ├── icon 24            leading
│   ├── title              17 / Bold / text.primary     (status: brand colour)
│   ├── subtitle           12 / Regular / text.secondary
│   └── chevron 24         trailing, optional
├── BODY                   padding = 16 (shared inset)
│   └── gap header→body    8 (plain card) · 0 (status card — the band's own padding separates)
├── DIVIDER?               1px, surface.tertiary, inset 0, vertical padding 4
└── FOOTER?                padding 16
```

| Property | Value | Mark |
| --- | --- | --- |
| radius | 16 | **[T]** `CardView.swift:23` = `Card.kt:53` |
| padding | 16, single inset shared by header and body | **[T]** `CardView.swift:22` = `Card.kt:48` |
| background | `surface.secondary` | **[T]** |
| header → body gap | 8 (plain) / 0 (status) | **[T]** the only spacing value both platforms independently agree on |
| title | 17 / Bold / `text.primary` | **[M]** iOS semibold → Bold per §3.2 |
| subtitle | 12 / Regular / `text.secondary` | **[T]** |
| divider | 1px `surface.tertiary`, inset 0, vertical padding 4 | **[C]** iOS `Divider()` has no colour token at all; Android uses `ham_lightGray` |

Both sides have **27 (iOS)** and **21 (Android)** hand-rolled cards that bypass the primitive.
They are listed in [Appendix A](#appendix-a--known-defects). Absorb them or document them, but
do not leave them as silent outliers.

### 8.2 Status card

| Property | iOS | Android now | Normative |
| --- | --- | --- | --- |
| radius | 16 (`:80`) | 12.dp (`:49`) | **16** |
| header padding | 12 all sides (`:71`) | v12 / h16 (`:67-68`) | **12 all sides** |
| content padding | 12 (`:22`) | 16.dp (`:53`) | **12** |
| header background | brand @ 0.15 (`:74`) | brand @ 0.15 (`:66`) | **brand @ 0.15** |
| header title | 17 Bold (`:54`) | 16 Bold (`:77`) | **17 Bold** |
| header icon | unsized (`:52`) | 24.dp (`:75`) | **24** |
| header → body gap | 0 (`:50`) | 0 | **0** |

**Android must also gain a `padding` parameter.** iOS's component takes one and
`StatusBusCard.swift:20` passes `0`; Android has no way to express that.

### 8.3 List row

| Property | iOS | Android now | Normative |
| --- | --- | --- | --- |
| height | implicit | implicit | **implicit** |
| icon plate | 40×40 circle, brand @ 0.1 | 40.dp circle, brand @ 0.1 | **40 @ 0.1** |
| icon glyph | 20 semibold | 25.dp (one row is 30) | **20** |
| icon → text gap | 8 | 8 | **8** |
| title | 17 semibold | 16 Bold | **17 Bold** |
| subtitle | 12 / `text.secondary` | 12 / `text.secondary` | **12 / secondary** |
| chevron | unsized ~17, `.gray` | 24.dp, `Color.Gray` | **12** per §10 |
| divider | system, no token | `ham_lightGray` | **surface.tertiary** |

Android's `MyViewLinkCard` sets the icon glyph to 25dp and, on the settings row, 30dp —
inconsistent with its own siblings. Fix to 20.

### 8.4 Buttons

Neither platform has a styled button component. Android's `HamButton` (`Button.kt:31`) supplies
only three things: press feedback (`alpha → 0.25f`), a `contentColor` that most call sites
override, and telemetry. It has **no padding, no height, no radius, no background, no border,
no ripple**. iOS has no `ButtonStyle` at all.

So the spec below is mostly **new work**, not a value change.

| Variant | Height | Padding | Radius | Background | Text | Mark |
| --- | --- | --- | --- | --- | --- | --- |
| **Filled primary** | 48 | h16 | 8 | `tint.active` (brand @ 1.0) | 17 / Bold / white | **[M]** |
| **Tinted** | 48 | h16 v12 | 12 | `tint.subtle` (brand @ 0.15 over base) | 17 / Bold / brand | **[T]** |
| **Borderless** | = text box | 0 | — | none | 17 / Regular / `text.link` | **[T]** |
| **Destructive (text)** | = text box | 0 | — | none | 17 / Regular / `text.danger` | **[T]** |
| **Destructive (outlined pill)** | 28 | h12 v6 | 8 | transparent, 1px `text.danger` @ 0.3 | 12 / Regular / `text.danger` | **[M]** iOS-only today |
| **Row-style tinted** | 52 | h16 | 12 | `tint.subtle` | 17 / Bold, 12 subtitle | **[M]** |
| **Large tile** | 150 | 16 | 16 | `tint.subtle` | 17 / Bold, 11 subtitle | **[M]** |
| **Icon button** | 44 target | 8 | circle | `surface.tertiary` | — | **[C]** |

Notes:

- **The SSO authorize button is the reference implementation** — v-pad 12, radius 12, alpha
  0.15, bold, brand-coloured text, and a grey @0.1 / grey disabled swap all agree across
  platforms (`SSOAuthorizationSheet.swift:437-447` vs `SSOAuthorizationSheet.kt:531-549`).
  Build the shared button to match it.
- **Android's `48.dp` explicit height appears 24 times** and is its only real standard. iOS has
  no equivalent. Take 48 for the standard button, 52 for the row-style variant (iOS's value).
- **Disabled state has no convention on either side.** iOS has one data point (`.opacity(0.5)`
  at `LibraryModifyBookingView.swift:105`); Android has one (`0.6f` at
  `SSOAuthorizationSheet.kt:470`). The only coherent disabled treatment on both is the SSO
  button's palette swap. **[C] Standardise on the palette swap** — brand @0.15 → grey @0.1,
  brand text → grey text — and reserve whole-view opacity 0.5 for genuinely unavailable
  controls.
- **Press feedback differs structurally.** Android dims the whole subtree to alpha 0.25 (which
  is very aggressive); iOS uses the native flash. **[C]** Android should adopt a ripple or a
  lighter dim; 0.25 is below most guidelines.
- **Minimum tap target: 44.** Neither platform enforces one. Android's smallest is 16×16dp
  (`LoginView.kt:191`, `CourseCommentItemView.kt:170`); iOS's is ~14×14pt. This is an
  accessibility defect independent of visual parity.

### 8.5 Chips and badges

**[T] The filter chip is the single most convergent control in the app** — radius 6, padding
h6 / v4, alpha 0.1, 12sp caption, brand-vs-grey for selected-vs-unselected. All eight metrics
match exactly (`CourseScoreResultViewSearchBar.swift:67-75` vs
`CourseScoreResultItemFilterFunctionView.kt:69-77`).

Use it as the model:

| Property | Value |
| --- | --- |
| radius | 6 |
| padding | h6 / v4 |
| selected | brand @ 0.10 background, brand text |
| unselected | `text.secondary` @ 0.10 background, `text.secondary` text |
| font | 12 / Bold |

Everything else in this family is chaos. iOS's 收藏座位 badge is an orange pill with no Android
counterpart; 上次预约 is green on iOS and `ham_blue` on Android; the bus line name is a pill on
iOS and plain bold text on Android. Badge radii in use across both platforms: 4, 5, 6, 8.
Neither platform has a reusable badge component — iOS repeats the recipe 7 times with 4 radii
and 3 sizes, Android has one extracted component.

Also: **neither platform has a chip with a close/remove (×) button.** Zero on both.

### 8.6 Controls

**[T] iOS delegates controls entirely to the system. Android uses Material3 with partial token
wiring.** This is largely a sanctioned divergence — you cannot make a SwiftUI `Toggle` look
like a Material `Switch` without a full custom implementation, and it is not worth it.

| Control | iOS | Android | Normative |
| --- | --- | --- | --- |
| **Switch** | native `Toggle`, no styling | `HamSwitch`: M3 `Switch`, track `ham_green` / `ham_lightGray` | Keep native. **Fix Android's checked track to `brand.sport` (#34C759)** — `ham_green` is currently the wrong green. |
| **Slider** | native `Slider`, no styling | M3 `Slider`, no custom colours | Keep native on both. |
| **Progress** | native `ProgressView` | hand-rolled bar: height 4, radius 2 | Keep native. Android's bar: **height 4, radius 2** |
| **Text field** | three ad-hoc helpers, no shared component | `HamTextField`: radius 8, bg `surface.secondary`, 1px `surface.tertiary` border, padding 8, 16sp body, cursor `text.link` | **Port `HamTextField`'s spec to iOS.** |
| **Segmented picker** | native `Picker(.segmented)` | `HamHorizontalPicker`: track radius 6 / `ham_gray` @0.40 / padding 2, thumb radius 5 / `ham_lightGray` | Keep native on iOS. **Fix Android's thumb radius to match the track (both 6)** — they differ today. |
| **Picker button** | — | `DatePickerButton` / `TimePickerButton` / `HamFixedTimePickerButton`: radius 8, padding v6 / h8, bg `Color.Gray` @0.15, 16sp body | **[C]** radius 8, padding v8 / h12, bg `surface.tertiary`, 17 / Regular |
| **Swipe to confirm** | height 50, radius 10, track grey @0.2, fill red | height 48, radius 12, track `ham_gray` @0.15, fill `ham_red` | **[M]** height 48, radius 12. Near-match already. |

### 8.7 Sheet

| Property | iOS | Android | Normative |
| --- | --- | --- | --- |
| radius | system (~12) | 24.dp (`Sheet.kt:65`) | keep per platform — see §10 |
| drag handle | system | **none** | **add: 36 × 5, pill, `text.secondary` @ 0.4, centred in the existing 32dp header** |
| background | system material | `surface.primary` | per platform |
| scrim | system | black @ 0.5 | per platform |

**Android's missing drag handle is the single most visible parity gap on the platform.** Users
have no affordance telling them the sheet is draggable.

### 8.8 Toast

| Property | Normative | Android now |
| --- | --- | --- |
| radius | 8 | 12 → **8** |
| padding | 16 all sides | same |
| icon | 36 | 32 → **36** |
| icon → text gap | 5 | 8 → **5** |
| title | 17 / Bold | 16 → **17** |
| subtitle | 12 / Regular | 16 → **12** |
| duration | 3300 ms | 2000 → **3300** |
| background | `feedback.*` | error uses #F44336 → **`text.danger`** |
| types | 5 (info, success, warning, error, neutral) | 3 → **5** |

**[C]** iOS's current toast radius is 0 (a plain rectangle). That is an omission, not a
decision. This is the one place the document knowingly departs from the measured iOS value.

### 8.9 Banner

| Property | Normative | Android now |
| --- | --- | --- |
| height | 200 | 180 → **200** |
| radius | 16 | same |
| title | 34 / Bold | 24 → **34** |
| foreground icon | 72 | same |
| background watermark | 36 @ 0.25, 6 × 9 grid | 72 @ 0.20, 5 × 10 → **36 / 0.25 / 6×9** |

### 8.10 Section header

**[M]** Exists only on iOS today. See [§3.7](#37-text-roles) row 2.

| Property | Value |
| --- | --- |
| font | 12 / Regular |
| colour | `text.secondary` |
| line limit | 1 |
| padding | 0 additional — spacing comes from the container |
| gap to the group below | 8 |

### 8.11 Empty state

**[C]** Neither platform has one. iOS renders empty text inline inside a plain card; Android
uses a bare full-screen `Column` with a 64dp red circle.

| Property | Value |
| --- | --- |
| font | 12 / Regular |
| colour | `text.placeholder` |
| container | inherits from the parent card |
| icon | `icon.xl` (64), `text.tertiary` |

---

## 9. Platform-sanctioned divergences

These **must** differ and are not defects:

- **Safe areas and status bars** — iOS safe-area insets; Android `statusBarsPadding()`.
- **Navigation affordances** — iOS edge-swipe back; Android system back.
- **System pickers** — date, time, album, and document pickers are platform-supplied.
- **Native controls** — switch, slider, and progress use each platform's native rendering
  ([§8.6](#86-controls)). Do not build custom cross-platform versions.
- **Navigation chrome** — iOS uses `UINavigationController`; Android renders an in-Compose
  header. Specify the resulting offset, not the implementation.
- **Widget configuration** — Android exposes an update-interval picker; WidgetKit owns refresh
  scheduling, so iOS correctly has no such screen.
- **Language** — Android has an in-app locale picker; iOS follows the system locale.
- **Type scaling** — Dynamic Type and sp both respond to OS font-size settings, with different
  curves. Values here are at the default scale.
- **Dark-mode mechanism** — asset catalogs vs `values-night`. Both must reach the same hexes.

---

## 10. Undecided

Needs a maintainer call before the relevant PR lands:

1. **Chevron size.** iOS 8pt (22 sites) vs Android's default 24dp. Proposal: **12**.
2. **Sheet radius.** iOS system ~12pt vs Android 24dp. Proposal: **keep Android at 24** — it
   suits large screens and iOS's value is system-controlled, so we cannot move it anyway.
   Document it rather than fight it.
3. **Stage the type change?** The `headline` 14 → 17sp bump is the largest reflow. Land the
   non-type fixes first, or one PR with full visual review?
4. **Toast radius.** Spec says 8; measured iOS value is 0. Confirm this deliberate departure.
5. **Body 17 vs 16.** iOS-as-baseline says 17, but Android's 16sp is tokenised and used 240
   times, and 10 of iOS's own raw `.system(size:)` sites are already at 16. Confirm 17, or
   accept 16 as a documented exception?
6. **Elevation.** This document specifies none, matching both platforms. Confirm we are not
   adding shadow as part of this work.

---

## 11. Review checklist

- [ ] Padding, radius, font, and colour all come from a token here.
- [ ] Every colour has a light and a dark value and is an adaptive resource.
- [ ] Text styles are explicit — no relying on an inherited default size.
- [ ] No new literal numbers in view code.
- [ ] `maxLines`/`lineLimit` set per [§3.4](#34-truncation), with `Ellipsis` on Android.
- [ ] Interactive elements meet the 44 minimum tap target.
- [ ] The same screen exists on the other platform with the same numbers, or the divergence is
      listed in [§9](#9-platform-sanctioned-divergences) or [§10](#10-undecided).
- [ ] Verified in light and dark mode.
- [ ] Verified with a populated account — empty states hide spacing and type differences.

---

## Appendix A — Known defects

Found while measuring. These are bugs, not style gaps, and most are one-line fixes.

### Colour wiring (Android)

| # | Defect | Location |
| --- | --- | --- |
| 1 | `ham_brand_sport` / `ham_brand_score` / `ham_green` / `ham_orange` / `ham_red` resolve to Material colours while the correct hexes sit unused in `colors.xml` | `Color.kt:53,60,63,79,82` vs `colors.xml:72,73,75` |
| 2 | `ham_text_secondary` is hardcoded Compose `Gray`, not a resource — cannot dark-mode | `Color.kt:29` |
| 3 | `values-night/colors.xml` keeps `gray` at #888888, so Android secondary text has **no dark variant at all** | `values-night/colors.xml:13` |
| 4 | 退出登录 renders in pure red #FF0000 in one place and #F44336 in another, for the identical string | `SyncLogoutView.kt:49` vs `UserCenterMainView.kt:183` |
| 5 | `<color name="link">#007AFF</color>` defined and referenced by zero Compose files | `colors.xml:14` |
| 6 | `dimens.xml` is dead (`R.dimen`: 0 hits) **and its values contradict the live font scale** — title 32 vs 24, title2 26 vs 20, title3 18 vs 16, headline 18 vs 14, caption2 10 vs 11 | `core/ui/.../values/dimens.xml` |

### Type (Android)

| # | Defect | Location |
| --- | --- | --- |
| 7 | `largeTitle` (28sp) has zero call sites | `Font.kt:16` |
| 8 | `title2` and `title3` are both 16sp — identical to `body`. Three names, one size. | `Font.kt:18,20` |
| 9 | `headline`, `caption`, `caption2` declare no `fontWeight` | `Font.kt:30,34,38` |
| 10 | `largeTitle`/`title`/`title2`/`title3`/`headline` declare no `color`, so hero numbers render uncoloured | `Font.kt:16-32` |
| 11 | `MaterialTheme.typography` (with the only three `lineHeight` declarations in the codebase) is referenced zero times | `HamTheme.kt:85-109` |
| 12 | 24 raw `fontSize = N.sp` sites bypass `HamFontStyle` entirely | various |
| 13 | 23 of 66 `maxLines` sites have no `TextOverflow.Ellipsis` — hard-clip with no ellipsis | various |
| 14 | The same seat number renders at three different sizes: 24sp, 32sp, 36sp | `SelectSeatCard.kt:287`, `LibraryMainViewQuickBookCard.kt:74`, `LastBookingCard.kt:38` |

### Type (iOS)

| # | Defect | Location |
| --- | --- | --- |
| 15 | No font, spacing, or shadow token file exists at all | `Ham/shared/` |
| 16 | 151 raw `.font(.system(size:))` sites — though 123 (81%) are SF Symbol icon sizing, not typography, and should become an icon scale instead | various |
| 17 | Three de-facto secondary colours: `ham_text_t2Color` (113 uses), raw `.gray` (38), raw `.secondary` (34 — a different colour entirely) | various |
| 18 | `Color.lightGray` #F0EFEF vs `Color.ham_lightGray` #EDEEEF — two greys under near-identical names | `Color+Ham.swift:45` vs `:31` |
| 19 | Course-period rail and sport clock times have no tabular figures, so digits jitter | `CourseViewBodyCourseNumView.swift:63`, `SportOrderViewAppointmentAreaCell.swift:146` |

### Component bugs

| # | Defect | Location |
| --- | --- | --- |
| 20 | Fill uses radius 10 while the clip uses radius 16 — visible corner artefact | `CourseViewDetailCourseInfoView.swift:27` vs `:31` |
| 21 | `HamHorizontalPicker` thumb radius 5 ≠ track radius 6 | `HorizontalPicker.kt:57,70` |
| 22 | `SportSelectItemView.kt:45` is a verbatim clone of `HamCardView` in a file that never imports it | `feature/sport/.../SportSelectItemView.kt` |
| 23 | Two competing Android shared cards: `HamCardView` (r16, `core/ui`) and `CommonStatusCard` (r12, `feature/status`, 8 call sites) | — |
| 24 | `PrintSheet.kt:13` is an empty stub; `PrintStatusCard.kt` is never called from any screen | `feature/print/` |
| 25 | `ScheduleCard.kt` (status) has zero call sites — iOS renders one | `feature/status/` |
| 26 | iOS `AboutPrivacyView.swift:12-18` renders a markdown link as unstyled, untappable text | `Ham/iOS/ui/about/` |
| 27 | Android duplicates the markdown-link parser verbatim in two files | `LoginView.kt`, `AboutView.kt` |
| 28 | `enabled = false` blocks the click but not `awaitFirstDown`, so the touch is swallowed silently | `Button.kt:65` |
| 29 | `CourseThemeSelectView.kt:124,170` pass `enabled = !selected` while an explicit ternary sets the colour — no visual effect | `feature/course/` |
| 30 | iOS print data layer ships compiled and unreferenced — no route, no strings, no view | `Ham/shared/business/print/` |
