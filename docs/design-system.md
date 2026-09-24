# Ham design system

How to build the Ham native apps. This is a specification, not a comparison — read it as
"build it this way". Values are measured from the existing code; where the two clients
disagreed, **iOS is the baseline**.

Five documents, used together. The first three are the specification; the last two are the
work lists derived from it.

| Document | Covers |
| --- | --- |
| **[`design-system.md`](design-system.md)** (this one) | Tokens and components — how things look |
| **[`screens.md`](screens.md)** | Every screen, both platforms — 129 sections (**120 screens** + 9 analysis sections), per-element values |
| **[`copy-and-strings.md`](copy-and-strings.md)** | Terminology, copy rules, string key naming — what things say |
| **[`ui-parity.md`](ui-parity.md)** | Per-page UI work list — what each platform must change to match |
| **[`logic-parity.md`](logic-parity.md)** | Per-rule logic work list — business code, not appearance |

A UI change usually needs the first three. Before changing behaviour on one platform, check
`logic-parity.md` to see whether the other already does it your way.

Current divergences between the shipped clients are recorded in
[§8 Current divergences](#8-current-divergences) at the end, so this document stays readable
as a spec.

## Contents

- [1. Overview](#1-overview)
- [2. Foundations](#2-foundations)
- [3. Components](#3-components)
- [4. Screens](#4-screens) — the catalogue itself is in
  [`screens.md`](screens.md): [status](screens.md#1-status-dashboard-状态-status) ·
  [course](screens.md#2-course-timetable-课程表-course) ·
  [schedule](screens.md#3-schedule-日程-schedule) ·
  [library](screens.md#4-library-图书馆-library) · [sport](screens.md#5-sport-运动-sport) ·
  [score](screens.md#6-score-成绩-score) ·
  [course score](screens.md#7-coursescore-课程评分-coursescore) ·
  [my](screens.md#8-my-tab-我的-my) · [user center](screens.md#9-user-center-用户中心) ·
  [auth](screens.md#10-auth-and-sign-in-登录与授权) ·
  [shared](screens.md#11-shared-components-共享组件) · plus
  [navigation presentation](#44-navigation-presentation), [forms](#45-forms-and-text-entry),
  [confirmation](#46-confirmation-and-destructive-actions),
  [data loading](#47-data-loading-and-caching),
  [search / filter / sort](#48-search-filter-sort-pagination)
- [5. Shared flows](#5-shared-flows)
- [6. Platform rules](#6-platform-rules)
- [7. Undecided](#7-undecided)
- [8. Current divergences](#8-current-divergences)
- [9. Review checklist](#9-review-checklist)
- [10. Status and precedence](#10-status-and-precedence)

---

## 1. Overview

### 1.1 App structure

Three root tabs, **状态 as the default**:

```
┌─────────────────────────────────────────────┐
│                                             │
│              <screen content>               │
│                                             │
├─────────────────────────────────────────────┤
│   课程表        状态(默认)        我的       │  ← tab bar
└─────────────────────────────────────────────┘
```

| Tab | Chinese | Content |
| --- | --- | --- |
| Course | 课程表 | Week-by-week class grid |
| Status | 状态 | Dashboard of live module cards |
| My | 我的 | Account hub + function grid |

Everything else is reached from one of these three — mostly from the **function grid** on 我的,
which is the app's real navigation hub.

### 1.2 Navigation hub: the function grid

The 我的 tab carries a horizontally scrolling grid of module shortcuts. Each tile is driven by
a remote-config entry keyed by module name, and **each tile uses its module's brand colour**:

| Key | Label | Brand colour | Goes to |
| --- | --- | --- | --- |
| `library` | 图书馆 | `#007AFF` | Library home |
| `sport` | 运动 | `#34C759` | Sport home |
| `score` | 成绩 | `#FF9500` | Score home |
| `course_score` | 课程评分 | `#283593` | CourseScore search |
| `pay` | 付费 | `#BF360C` | Pay |
| `bus` | 校车 | `#A2845E` | Bus |
| `course` | 课程表 | `#1B5E20` | Course timetable |
| `schedule` | 日程 | `#01579B` | Schedule |

Tiles appear only when their config entry is present, so the grid can change without a release.
Two tiles are always available regardless: 设置 and 关于.

### 1.3 Page scaffold

Every scrollable screen has this shape:

```
┌─────────────────────────────────────────────┐
│ status bar                (system)          │
├─────────────────────────────────────────────┤
│ header / nav bar          42 + status bar   │  ← Android renders in-Compose; iOS uses NavigationStack
├─────────────────────────────────────────────┤
│                                             │
│  ┌───────────────────────────────────────┐  │  ← 16 horizontal margin
│  │ card                                  │  │
│  └───────────────────────────────────────┘  │
│                 8 gap                       │
│  ┌───────────────────────────────────────┐  │
│  │ card                                  │  │
│  └───────────────────────────────────────┘  │
│                                             │
│              (scrolls)                      │
│                                             │
│         bottom spacing (see below)          │
├─────────────────────────────────────────────┤
│ tab bar                (system inset)       │
└─────────────────────────────────────────────┘
```

| Property | Value |
| --- | --- |
| Screen horizontal margin | **16** |
| Gap between cards | **8** |
| Screen background | `surface.primary` |
| Bottom spacing — tab-root screen | system navigation-bar inset **+ 80** |
| Bottom spacing — pushed/child screen | system navigation-bar inset **+ 24** |

Bottom spacing is computed from the live system inset, never hardcoded.

---

## 2. Foundations

### 2.1 Units

**pt** (iOS) and **dp** (Android) are 1:1 — both density-independent. Type sizes are **pt** and
**sp**, also 1:1.

### 2.2 Colour

Every colour has a light and a dark value and is an adaptive resource. Never hardcode a hex in
view code.

#### Accent vs brand — read this first

There are two separate colour systems, and which one to use depends on **what the element
does**, not on which module it lives in.

| | `accent` | `brand.<module>` |
| --- | --- | --- |
| Value | `#007AFF` / `#0A84FF` | one per module |
| Means | **interactive** — this responds to a tap | **identity** — this belongs to a module |
| Used for | Links, buttons, selected state, today highlight, chevrons, switches, enabled state, date pickers, "前往本周" | Status-card header band, function-grid tile, module intro screen, module watermark |
| Varies by module? | **No** — the same blue everywhere | Yes |

**Interactive elements use `accent` on every module.** A button inside the library is blue, not
library-blue. A selected chip on the course screen is blue, not course-green. The module colour
identifies the module; it does not tint its controls.

**`brand.<module>` is used at exactly three places per module:**

1. the status-card header band on 状态,
2. the function-grid tile on 我的,
3. the module's intro / connect screen.

Plus, optionally, the module's card watermark.

#### Where brand is actually used — measured

The three-place rule is a target, and two modules do not meet it. Use-site counts, both
platforms:

| Token | iOS sites / files | Android sites / files | Meets the rule? |
| --- | --- | --- | --- |
| `brand.library` | 21 / — | 27 / 10 | indistinguishable — library's brand *is* `#007AFF`, the same hex as `accent` |
| `brand.sport` | **3** / 3 | **38** / 15 | **no** — see below |
| `brand.score` | 2 / 2 | 9 / 5 | marginal |
| `brand.course` | 7 / — | 16 / 5 | yes |
| `brand.schedule` | 2 / 2 | 6 / 3 | yes |
| `brand.coursescore` | 4 / — | 9 / 3 | yes |
| `brand.bus` | 3 / — | 9 / 4 | yes |
| `brand.pay` | 2 / — | 5 / 3 | yes |

**Android's sport module uses `brand.sport` far outside the three places** — as its working
colour across the whole booking flow: the primary CTA (`SportSelectFooterView.kt:82`
`.background(Color.ham_brand_sport)`), free-slot chips
(`SportSelectItemTimeView.kt:28`), selectable court cells
(`SportSelectItemCourtListDetailItemView.kt:47`), date chips (`SportSelectDateView.kt:43,60`),
the sport function card (`SportMainViewFunctionButtonCard.kt:61,71,79,85,91`) and the status-card
badge (`StatusSportCard.kt:122`).

**iOS does the same thing visually but without the token** — `Color.green` at 21 sites in
`iOS/ui/sport/`, `Color.orange` at **7** in the score screens, `Color.blue` at **75** across the app.
Those are raw system colours where a `ham_brand_*` token exists.

So the two clients agree on the *result* — both tint a module's own screens with its brand colour
— and disagree only on whether it is written as a token. That makes the strict reading of the
rule above wrong for 运动 and 成绩, and it is recorded as an open question in
[§7](#7-undecided) rather than silently changed here. Until it is settled: **use the token, not
the raw system colour**, and do not add new brand-tinted controls.

#### Dark mode — measured coverage

"Every colour has a light and a dark value" is a target. Measured:

| Platform | Adaptive | Fixed | Notes |
| --- | --- | --- | --- |
| iOS | **6 of 23** colours in `Color+Ham.swift` come from the asset catalog (`UIColor(named: "color/…")`) | **5 with no dark variant** — `ham_theme_bgColor` `#E2EDF2`, `ham_brand_pay` `#BF360C`, `ham_brand_coursescore` `#283593`, `ham_brand_course` `#1B5E20`, and `lightGray` `Color(red: 240/255, …)` | the other 12 are system colours (`Color.primary`, `.gray`, `.blue`, `.green`, `.orange`, `.brown`, `.red`, `.white`, `systemBackground`) and **do** adapt — they are fixed in name only |
| Android | **46 of 73** colour resources have a `values-night` entry | **27 with no `values-night`** — `core/ui` 24, `app` 3 | of those 27, 22 are deliberately mode-independent (`black`, `white`, the seven `*_alpha*` families, `light_gray_not_change`); **5 are not** |

The five that are not, on Android: `R.color.red` and `R.color.green` — **5 references from code**,
all of them rendering a light-mode value in dark mode:

| Reference | What it colours |
| --- | --- |
| `Color.kt:61` → `R.color.green` | `feedback.success` |
| `Color.kt:64` → `R.color.red` | `feedback.error` |
| `ToastManager.kt:123` → `context.getColor(R.color.red)` | the **error toast background** |
| `Color+Glance.kt:50` → `R.color.red` | widget |
| `Color+Glance.kt:66` → `R.color.green` | widget |

On iOS, `ham_theme_bgColor` is the visible one: #E2EDF2 is a pale blue-white that does not change
at night.

Rule: **a colour is either adaptive or named as fixed.** `_not_change`, `black`, `white` and the
alpha families already say so in their names; `red` and `green` do not, and the three hardcoded
iOS brand hexes need a dark counterpart in the asset catalog before the brand table above is true
at night.

#### Accent

| Token | Light | Dark |
| --- | --- | --- |
| `accent` | #007AFF | #0A84FF |
| `accent.subtle` | #007AFF @ 0.15 | #0A84FF @ 0.15 |
| `accent.muted` | #007AFF @ 0.10 | #0A84FF @ 0.10 |

On iOS this is the system blue — `AccentColor.colorset` is deliberately **empty**, so the app
inherits `UIColor.systemBlue` and adapts to dark mode for free. On Android it is
`R.color.ham_blue`.

#### Brand

Each module owns one brand colour for identity.

| Token | Light | Dark | Module |
| --- | --- | --- | --- |
| `brand.course` | #1B5E20 | #1B5E20 | 课程表 |
| `brand.schedule` | #01579B | #01579B | 日程 |
| `brand.library` | #007AFF | #0A84FF | 图书馆 |
| `brand.sport` | #34C759 | #30D158 | 运动 |
| `brand.score` | #FF9500 | #FF9F0A | 成绩 |
| `brand.coursescore` | #283593 | #283593 | 课程评分 |
| `brand.bus` | #A2845E | #AC8E68 | 校车 |
| `brand.pay` | #BF360C | #BF360C | 付费 |

**Collision worth knowing:** `brand.library` (#007AFF) is the **same value as `accent`**. On
library screens the module identity colour and the interactive colour are indistinguishable, so
you cannot tell from a screenshot whether a blue element there is identity or interaction. This
is the current state on both platforms and is not a bug — just be aware that the library module
has no distinct identity colour of its own.

#### Surface

| Token | Light | Dark | Used for |
| --- | --- | --- | --- |
| `surface.primary` | #F9F9F9 | #000000 | Screen background |
| `surface.secondary` | #FFFFFF | #0F0F0F | Cards |
| `surface.tertiary` | #EDEEEF | #0F0E0F | Chips, inactive cells, borders, dividers |
| `surface.tint` | #E6F1FF | #010D18 | Today / selected cells |

#### Text

| Token | Light | Dark | Used for |
| --- | --- | --- | --- |
| `text.primary` | #000000 | #FFFFFF | Titles, values, body |
| `text.secondary` | #8E8E93 | #98989D | Subtitles, captions, meta |
| `text.tertiary` | #8E8E93 @ 60% | #98989D @ 60% | Placeholder, disabled |
| `text.link` | = `accent` | = `accent` | Inline links — same value as `accent` |
| `text.danger` | #FF3B30 | #FF453A | Destructive text |

Define these as **semantic roles, not hex values**. iOS's `Color.primary` and `Color.gray` are
dynamic; writing #000000 into a spec would break iOS dark mode. The hexes above are the
*Android* implementation of each role.

`text.link` and `accent` are the same colour. The separate name exists so a reader can tell
"this text is a link" from "this control is interactive" at a glance — but they resolve to one
value and must stay in sync.

#### Feedback

| Token | Light | Dark |
| --- | --- | --- |
| `feedback.info` | #007AFF | #0A84FF |
| `feedback.success` | #34C759 | #30D158 |
| `feedback.warning` | #FFCC00 | #FFD60A |
| `feedback.error` | #FF3B30 | #FF453A |

#### Tint recipes

Tints are built from **`accent` by default**, and from `brand.<module>` only where the element is
deliberately carrying module identity.

| Recipe | Formula | Used for |
| --- | --- | --- |
| `tint.subtle` | `surface.secondary` + **accent** @ 0.15 | Buttons, large tiles, selected states |
| `tint.chip` | **accent** @ 0.10 background, **accent** text | Filter chips, small pills |
| `tint.active` | **accent** @ 1.0, white text | Filled primary buttons |
| `tint.muted` | `text.secondary` @ 0.10, `text.secondary` text | Unselected chips, inactive states |
| `tint.brand` | `surface.secondary` + **brand** @ 0.15 | Status-card header band, function-grid tile |

Three rules:

1. **Always put a tint over an opaque base.** A tint alone renders translucent against whatever
   is behind it.
2. **Tinted text is the same colour as its tint** — accent text on an accent tint, brand text on
   a brand tint. Not the label colour.
3. **Reach for `accent` unless you are colouring one of the three brand places.** When in doubt,
   it is accent.

### 2.3 Type

**iOS is the baseline for type.** The scale below is SwiftUI's, and Android moves to it —
including `body` at 17 rather than Material's 16. See
[§7](#7-undecided) for what that costs.

| Token | Size | Weight | Line height | Used for |
| --- | --- | --- | --- | --- |
| `largeTitle` | 34 | Bold | 40 | Large page title on 状态 |
| `title` | 28 | Bold | 34 | Hero values, banner titles |
| `title2` | 22 | Bold | 28 | Section values |
| `title3` | 20 | Bold | 28 | Card titles on secondary screens |
| `headline` | 17 | Semibold | 24 | Emphasised body |
| `body` | 17 | Regular | 24 | Body text, list titles, buttons |
| `bodyBold` | 17 | Bold | 24 | Card titles, list titles |
| `callout` | 16 | Regular | 22 | Secondary body |
| `subheadline` | 15 | Regular | 22 | Dense secondary text |
| `footnote` | 13 | Regular | 18 | Fine print |
| `caption` | 12 | Regular | 16 | Subtitles, meta, chips |
| `captionBold` | 12 | Bold | 16 | Chip labels, emphasised captions |
| `caption2` | 11 | Regular | 16 | Smallest labels |

**Android implementation status — read before using this table.** `HamFontSize`
(`core/ui/…/config/Font.kt:44-52`) has **8** members against the scale's 13, and every
`normative` value in [`screens.md`](screens.md) is written against the full 13:

| Token | Spec | Android today |
| --- | --- | --- |
| `largeTitle` | 34 | `28.sp` — and zero call sites |
| `title` | 28 | `24.sp` |
| `title2` | 22 | `20.sp` |
| `title3` | 20 | `16.sp` (same as `body`) |
| `body` / `bodyBold` | 17 | `16.sp` |
| `headline` | 17 | `14.sp` |
| `callout` | 16 | **absent** |
| `subheadline` | 15 | **absent** |
| `footnote` | 13 | **absent** |
| `caption` / `captionBold` | 12 | `12.sp` |
| `caption2` | 11 | `11.sp` |

So `callout`, `subheadline` and `footnote` are **not yet reachable on Android** — `grep -rn
"footnote\|callout\|subheadline" --include='*.kt'` returns zero. Adding them is Phase 2 of
`tasks/ios-android-ui-parity.md`; until then, do not hand-write the literal, add the token.

Rules:

- **Always name the token.** Never rely on an inherited default size — on iOS omitting
  `.font()` silently yields 17pt; on Android omitting `style` silently yields Material's 16sp.
- **Bold is the only emphasised weight.** Not semibold, not medium.
- **Numeric display uses rounded figures** at 28 and above, with **tabular figures** at 12.
  Tabular figures matter: the course period rail and the sport clock times otherwise jitter
  horizontally as digits change width.

### 2.4 Text roles

| Role | Size | Weight | Colour | Lines |
| --- | --- | --- | --- | --- |
| Screen large title | 34 | Bold | `text.primary` | 1 |
| Section header | 12 | Regular | `text.secondary` | 1 |
| Card title | 17 | Bold | `text.primary` | 2 |
| Card subtitle | 12 | Regular | `text.secondary` | 1 |
| Hero value | 28 | Bold, rounded | `text.primary` | 1 |
| List row title | 17 | Bold | `text.primary` | 1 |
| List row subtitle | 12 | Regular | `text.secondary` | 1 |
| Body | 17 | Regular | `text.primary` | unbounded |
| Caption / meta | 12 | Regular | `text.secondary` | 1 |
| Overline | 12 | Regular | `text.secondary` | 1 |
| Link | 12 | Regular, underlined | `text.link` | 1 |
| Destructive | 17 | Regular | `text.danger` | 1 |
| Placeholder | 12 | Regular | `text.tertiary` | unbounded |
| Chip label | 12 | Bold | on `tint.chip` | 1 |
| Numeric, small | 12 | Regular, tabular | `text.primary` | 1 |

On Android, `maxLines = n` must always be paired with `overflow = TextOverflow.Ellipsis`.

### 2.5 Spacing

| Token | Value | Used for |
| --- | --- | --- |
| `space.1` | 2 | Hairline gaps, grid gutters |
| `space.2` | 4 | Icon-to-label, tight stacks, divider padding |
| `space.3` | 8 | Default stack gap, card gap, internal padding |
| `space.4` | 12 | Status-card padding |
| `space.5` | 16 | Screen margin, card padding |
| `space.6` | 24 | Section separation |
| `space.7` | 32 | Hero offsets |

### 2.6 Radius

| Token | Value | Used for |
| --- | --- | --- |
| `radius.1` | 2 | Progress bars |
| `radius.2` | 4 | Checkboxes, dots |
| `radius.3` | 6 | Badges, colour bars, chips |
| `radius.4` | 8 | Buttons, inner cards, text fields, banners |
| `radius.5` | 10 | Course grid cells only |
| `radius.6` | 12 | Controls, alert cards |
| `radius.card` | 16 | Cards, status cards, large tiles |

### 2.7 Elevation

**None.** Cards are flat. Separation comes from the `surface.secondary` on `surface.primary`
fill delta plus the 16 radius — no shadow, no border, no elevation parameter.

### 2.8 Iconography

**The two platforms use different icon sets** — SF Symbols on iOS, Material Icons on Android.
They are different drawings with different optical metrics, so **the same nominal size does not
look the same size**. An absolute icon size is therefore *not* cross-platform normative.

What **is** normative is the **pairing**: an icon that sits with text takes the size bound to
that text role. Within one app, the same text role always gets the same icon size. That is what
makes the app look systematic.

#### Text-paired icons

| Text role | Text size | Icon — iOS | Icon — Android | Used for |
| --- | --- | --- | --- | --- |
| `caption2` / `caption` | 11–12 | **12** | **16** | Inline chevrons, chip glyphs, icons beside meta text |
| `body` / `bodyBold` / `headline` | 17 | **20** | **20** | List-row leading glyph, icons beside body copy |
| `title3` / `title2` | 20–22 | **24** | **24** | Section icons, icons beside section headings |
| `title` | 28 | **32** | **32** | Icons beside hero values |

The one place the platforms differ is the smallest step: an SF Symbol fills less of its box than
a Material icon at the same size, so iOS uses **12** where Android uses **16** for caption-paired
icons. Everything from `body` up is the same number on both.

#### Standalone and decorative icons

These are not paired with text, so the absolute value *is* normative on both platforms:

| Token | Value | Used for |
| --- | --- | --- |
| `icon.plate` | 40 | The circle behind a list-row leading glyph |
| `icon.badge` | 32 | Area badges, small-card trailing decoration |
| `icon.tile` | 64 | Large tile icon |
| `icon.empty` | 64 | Empty-state icon |
| `icon.banner` | 72 | Banner foreground icon |
| `icon.watermark` | 128 | Card background watermark |

#### Token names — the full vocabulary

§3 sizes icons with **short names**; this is what they mean. `icon.xs` … `icon.lg` are the
text-paired sizes above, and the rest are the standalone sizes. Both halves of the vocabulary
live here so §3 is readable on its own.

| Token | Value | Notes |
| --- | --- | --- |
| `icon.xs` | 12 iOS / 16 Android | Caption-paired. The only size that differs by platform. |
| `icon.sm` | 20 | Body-paired — list-row leading glyph |
| `icon.md` | 24 | Title3/title2-paired — section icons |
| `icon.lg` | 32 | `title`-paired |
| `icon.xl` | 64 | Large standalone icon — also the empty-state icon |
| `icon.plate` | 40 | The circle behind a list-row leading glyph — a container, not a glyph |
| `icon.badge` | 32 | Area badges, small-card trailing decoration |
| `icon.tile` | 64 | Large tile icon — same value as `icon.xl`, different role |
| `icon.empty` | 64 | Empty-state icon — use `icon.xl` |
| `icon.banner` | 72 | Banner foreground icon |
| `icon.watermark` | 128 | Card background watermark |

`icon.plate` is a container: it holds an `icon.sm` glyph at alpha 0.1 on a 40 circle. It is not
a size you draw an icon at.

#### Cross-platform glyph mapping

Sizes are paired above; **names are not**. SF Symbols and Material Icons are different drawings,
so the same concept is drawn differently unless someone chooses deliberately. Nobody has.

These are the surfaces where both platforms render **the same data** — the same remote-config
key, the same tab, the same card — and each picks its own glyph. They are the highest-confidence
comparisons available and the ones worth fixing first.

**Function grid — 8 keys, 5 agree** (`MyViewFunctionCard.swift:27-34` ↔ `MyViewFunctionComponentView.kt:85-187`)

| Key | iOS | Android | Agree |
| --- | --- | --- | --- |
| `library` | `books.vertical` | `Book` | ✅ |
| `score` | `doc.plaintext` | `Article` | ✅ |
| `pay` | `creditcard` | `CreditCard` | ✅ |
| `bus` | `bus.fill` | `DirectionsBus` | ✅ |
| `course` | `tablecells` | `TableChart` | ✅ |
| `schedule` | `calendar` | `CalendarMonth` | ✅ |
| `sport` | `sportscourt` | `SportsVolleyball` | ❌ |
| `course_score` | `chart.bar.xaxis` | `InsertChart` | ❌ |

**Bottom tab bar — 3 tabs, 1 agrees**

| Tab | iOS | Android | Agree |
| --- | --- | --- | --- |
| Course | `tablecells` — `MainTabView.swift:27` | `TableRows` — `MainBodyView.kt:85` | ❌ grid vs rows |
| Status | `cube` — `:28` | `Layers` — `:91` | ❌ cube vs stacked layers |
| My | `person` — `:29` | `Person` — `:97` | ✅ |

**Status-card headers — 6 cards, 4 agree**

| Card | iOS | Android | Agree |
| --- | --- | --- | --- |
| Course | `tablecells.fill` — `StatusCourseCard.swift:24` | `School` — `CourseCard.kt:85` | ❌ grid vs school |
| Sport | `sportscourt` — `StatusSportCardView.swift:16` | `SportsSoccer` — `StatusSportCard.kt:82` | ❌ court vs ball |
| Schedule / Bus / Library / Weather | `calendar` / `bus` / `books.vertical.fill` / `sun.max.fill` | `CalendarMonth` / `DirectionsBus` / `Book` / `WbSunny` | ✅ |

**Three systematic mismatches worth naming**

1. **Sport.** iOS draws a *court* (`sportscourt`); Android draws a *ball* (`SportsVolleyball`,
   `SportsSoccer`, `SportsTennis`). Every sport surface diverges for the same reason — fix the
   decision once, not per screen.
2. **Course.** iOS draws a *grid* (`tablecells`); Android draws a *school building* (`School`) on
   the status card but `TableChart` and `TableRows` elsewhere. Android is not even
   self-consistent here.
3. **Success and error.** iOS uses the filled-in-circle variants (`checkmark.circle.fill`,
   `xmark.circle.fill`); Android uses bare glyphs (`Done`, `Close`). Same states, different weight.

**Absent on both platforms:** notification bell, offline / wi-fi, visibility toggle. These are
roles both apps need and neither has an icon for — choose them deliberately here rather than
letting each screen invent one.

#### Rules

1. **Every icon is explicitly sized.** Never inherit a default.
2. **An icon paired with text uses the pairing table**, not a hand-picked number.
3. **Chevrons are caption-paired** — 12 on iOS, 16 on Android. Not 8, not the Material default.
4. **Watermarks render at a fixed frame size**, not a font size. iOS renders an SF Symbol's ink
   smaller than its box while Android fills the box, so a font size and a frame size are not
   interchangeable. Watermarks sit at alpha **0.15**, anchored **bottom-trailing** at offset
   **(16, 16)**, clipped to the card.

#### Current violations

| Platform | Violation | Should be |
| --- | --- | --- |
| iOS | Chevrons hardcoded at `.font(.system(size: 8))` in 22 places | 12 |
| iOS | Status-card header icon has no size at all | 24 |
| Android | List-row glyphs at 25 and 30 in different files | 20 |
| Android | Area badge 36 where iOS uses 32 | 32 |
| Both | Watermark sizes from 60 to 250 with no rule | 128 |

---

### 2.9 Motion

Motion is a foundation, like type or spacing: it is chosen once and referenced everywhere.
Today it is not. **Neither app has a motion scale** — durations are written inline, most sites
specify no duration at all, and **no duration, curve, or haptic event is shared between the two
clients**. The result is that two screens built to the same visual spec still feel like
different apps.

#### Duration

| Token | Value | Used for |
| --- | --- | --- |
| `motion.press` | **100 ms** | Press feedback |
| `motion.fast` | **150 ms** | Opacity toggles, chevron rotation, overscroll settle |
| `motion.medium` | **300 ms** | Fades, content swaps, toast, sheet, page push and pop |
| `motion.slow` | **500 ms** | Terminal-state reveal — the success card |

Four values. If a transition needs something else, it is using the wrong token.

#### Curves

| Token | Curve | Used for |
| --- | --- | --- |
| `motion.curve.standard` | ease-in-out | Default — anything not otherwise specified |
| `motion.curve.enter` | ease-out (decelerate) | Content arriving |
| `motion.curve.exit` | ease-in (accelerate) | Content leaving |

**Every animation names a duration and a curve.** A bare `withAnimation { … }` on iOS or a bare
`animate*AsState` on Android is a defect: it inherits a platform default, and the two platform
defaults are different curves with different durations. There are **352** such sites on iOS and
**129** measured sites on Android (`AnimatedVisibility(` / `AnimatedContent(` /
`animate*AsState` in `src/main`, across 72 files); they are the single largest source of "these two apps feel different".

#### What animates

| Interaction | Specification |
| --- | --- |
| **Page push and pop** | Slide from the trailing edge. **300 ms, symmetric** — `curve.enter` in, `curve.exit` out. Not 400 in / 300 out. |
| **Sheet** | Slides from the bottom edge, **300 ms**. Height 85 % of the screen on Android; iOS declares its detents explicitly rather than inheriting them. Top corners **radius 24** on Android. |
| **Sheet scrim** | Black, max alpha **0.5**. The scrim animates on its own curve — it is **not** coupled to drag progress. Dismiss threshold is **one quarter** of the sheet height. |
| **Toast** | Fade *and* slide from the top, **300 ms** in and out (a toast must animate out, not vanish). Auto-dismiss **3.0 s**. Radius **12**. |
| **Tab switch** | **Instant.** Do not animate tab content — a crossfade on every tab tap reads as slowness, not polish. |
| **Button press** | The platform's native press treatment — SwiftUI's default on iOS, the Material state layer on Android. **Not** a custom alpha fade. |
| **Loading → content** | Crossfade, **300 ms**. Never a hard swap. |
| **List item insert / remove** | **150 ms**, on both platforms. |
| **Terminal-state card** | Rise 100 with a fade, **500 ms**, revealed after **800 ms**. |

#### Haptics

Haptics fire for **commitments only** — something the user just committed to, or just destroyed.
Not for navigation.

| Event | Specification |
| --- | --- |
| Booking, order, or payment succeeds | Heavy impact — iOS `.heavy`, Android `EFFECT_HEAVY_CLICK` @ 50 ms |
| Destructive action confirmed | Heavy impact |
| Tab change | **None** |
| Pull-to-refresh completion | **None** |
| Picker tick | Platform default — this is a system control, not our feedback |

#### Reduced motion

**Both apps must honour the system reduced-motion setting.** Neither does today — there is no
`UIAccessibility.isReduceMotionEnabled` check on iOS and no animation-scale query on Android, so
turning the setting on changes nothing. When it is on, every duration collapses to **0** and
positional motion becomes an instant opacity change.

Doing this is only cheap if durations live in tokens. Centralise them first.

#### Current state

| | iOS | Android |
| --- | --- | --- |
| Distinct stated durations | 4 — 0.3 s, 0.5 s, 1 s, 3 s (the last two are a LiDAR sweep loop) | 3 — 150 ms, 300 ms, 400 ms |
| Bare, unspecified animations | **352** `withAnimation {}` sites in 132 files | **129** `AnimatedVisibility` / `AnimatedContent` / `animate*AsState` sites in 72 files |
| Page push | **No duration written anywhere** — `Navigation.swift:122,128` wraps the push in a bare `withAnimation` | 400 ms in, 300 ms out — `NavHost.kt:45,55`, duplicated across two overloads |
| Sheet | `.sheet`, no detents, no drag indicator, no duration (0 uses of `presentationDetents`) | custom `HamSheet`: 85 % height, radius 24, scrim `0.5 × dragProgress`, quarter-height dismiss, duration unspecified |
| Toast dwell | **3.3 s** — `ToastView.swift:37` | **2.0 s** — `ToastManager.kt:150` |
| Toast exit | Slides *and* fades | **Fades only** — `ToastManager.kt:154` |
| Tab switch | `$tab.animation()`, unspecified, **plus a heavy haptic** | `fadeIn() togetherWith fadeOut()`, unspecified |
| Button press | none | **whole button dims to alpha 0.25** — `Button.kt:48` |
| Loading → content | hard swap at 51 `ProgressView` sites | crossfade via `AnimatedContent`, spec almost never given |
| List item | none | `.animateItem()` in one file only |
| Pull-to-refresh | hand-rolled, one screen: `scrollY < -128` slides a pill down, release fires `onRefresh()` — `StatusUpdateView.swift:15,57-63`, `StatusView.swift:72-74`; plus **1** native `.refreshable` on the course centre (`CourseCenterView.swift:84`). The status-dashboard motion is ours and unsponsored | native, one screen — `rememberPullToRefreshState`, 300 px — `StatusContainerView.kt:157`; **plus the shared scroll container `BounceScrollView`, instantiated at 8 sites** — the two shells (`NavigationView.kt:95`, `HomeContainer.kt:67`) and 6 screens — **which bounce and do not refresh** |
| Reduced motion | not checked | not checked |

**Highest-severity motion defect.** The most-used iOS transition is not ours and is not
SwiftUI's. `.fade` resolves to `AnyTransition.fade` vendored by the **SDWebImageSwiftUI pod**
(`Pods/SDWebImageSwiftUI/…/Transition.swift:24`), which is defined as
`asymmetric(insertion: .opacity, removal: .identity)`. Every one of the **13** `.fade` sites
therefore **fades in and cuts out** — the disappearance is unanimated. Nine of them pass a
duration; three do not import the pod in-file and were not verified without a build. A design
system must not source its primary transition from an image-loading dependency: define a
symmetric `AnyTransition.fade` in app code and delete the dependency.

### 2.10 Accessibility

Both apps are currently **unusable with a screen reader**, and neither honours the system text
size or reduced-motion setting. This is not a divergence between the platforms — they fail in
the same direction — which is why it belongs in the specification rather than in the parity
work list.

#### Rules

1. **Every icon that carries meaning has a label.** A decorative icon sitting beside a text
   label is not labelled individually — the row is merged into one accessibility element that
   reads its text. Silence is only correct when something else speaks for the element.
2. **Minimum touch target: 44 pt on iOS, 48 dp on Android.** These differ and that is fine —
   it is a platform rule, like the icon set. What is normative is the *result*: no interactive
   element is smaller than its platform's minimum. Grow the hit area, not the drawing.
3. **Text survives the largest system text size without clipping.** Text containers get a
   minimum height, not a fixed one. This is why the type scale is in `sp` on Android
   (`Font.kt:45-52`) and why SwiftUI's automatic scaling must not be fought with fixed frames.
4. **Transient feedback is announced.** A toast that only appears is invisible to a screen
   reader user; it must post an announcement. So must a loading state that lasts.
5. **Every interactive element has a test identifier.** This doubles as the labelling
   discipline's audit trail: an element with no identifier has usually had no thought given to
   how it is reached.

#### Current state

| | iOS | Android |
| --- | --- | --- |
| Icon call sites in app code | **236** `Image(systemName:)` in `iOS/` (287 counting tests, widgets, watchOS) | **218** `Icon(` |
| Accessibility modifiers, **production-visible** | **4** — of 8 in the app; 4 of the 8 are `#if HAM_E2E` no-ops | **7** real strings of 290 `contentDescription` — 271 `null`, 8 `""`, 4 more in test comments |
| Hints | **0** | n/a |
| Row / group merging | **0** `accessibilityElement(children:)` | **1** `mergeDescendants` — inside `HamButton` only, `Button.kt:66` |
| State semantics | 1 `accessibilityAddTraits`, 1 `accessibilityValue` | **0** `toggleable`, **0** `selectable` |
| Announcements | **0** `UIAccessibility.post` | **0** `liveRegion`, **0** `announceForAccessibility` |
| Test identifiers | **3** `accessibilityIdentifier` — **all inside `#if HAM_E2E`**, so a release build has **none** | **7** `testTag` |
| Touch-target enforcement | none | **0** `minimumInteractiveComponentSize` |
| Reduced motion | not checked | not checked |

iOS has **4** production-visible accessibility modifiers against **236** icons. Three of the four
are in `shared/ui/print/PrintOptionsForm.swift:124,125,152` — the print options form, extracted out
of the print screen on 2026-09-24, is the only
place in either app where accessibility was designed rather than omitted, and it is the pattern to
copy. All three `accessibilityIdentifier` sites (`Route.swift:320,350,367`) sit inside
`#if HAM_E2E`, so Rule 5 is unmet in a release build on iOS: **no element in the shipping app has
a test identifier.**

**The nulls are absence, not intent.** A `contentDescription = null` is correct only inside a
container that merges descendants *and* contains a text sibling. The app has **one**
`mergeDescendants` — inside `HamButton` (`Button.kt:66`) — and **zero**
`accessibilityElement(children:)` on iOS. The shared card, the status card and the My links card
apply no merging to their rows, so their null-description icons have no parent to inherit a label
from.

`HamButton` makes this worse rather than better for icon-only buttons: it merges descendants and
sets `role = Role.Button`, but when its content is `Image(contentDescription = null)` the merged
node has **no text at all**, so TalkBack announces an unnamed "button". Every icon-only button
built on it is in this state — the back button (`NavigationView.kt:143`), every status-card
chevron (`CommonStatusCard.kt:82`), the schedule popover close (`SchedulePopoverCard.kt:125`).

#### Touch targets — measured

| Control | iOS | Android | Minimum | Result |
| --- | --- | --- | --- | --- |
| Course timetable cell, 7-day | **45.9 × 31.5 pt** (iPhone SE) — `CourseViewBodyCourseItemView.swift:59` | **41.7 × 40.3 dp** — `CourseMainViewBodyCell.kt:179` | 44 / 48 | **both fail, both dimensions on Android** |
| Course timetable cell, 5-day | 61.8 × 31.5 pt (SE), 64.8 × 43.0 pt (14) | 60.0 × 40.3 dp | 44 / 48 | fail on height |
| Period rail | **42 pt** | **32 dp** | 44 / 48 | **both fail** |
| Back button | — | **24 × 24 dp** — `NavigationView.kt:143` | 48 | fail |
| Status-card chevron | — | **24 × 24 dp** — `CommonStatusCard.kt:82` | 48 | fail |
| Course-detail icon buttons | **≈33 × 33 pt** — `CourseViewDetailFunctionView.swift:39` | — | 44 | fail |
| Schedule drawer toggle | **80 × 33.7 pt** — `ScheduleView.swift:223` | — | 44 | fail |
| Bottom tab bar | 49 pt (native `TabView`) | 56 dp item (M3 default, no override) | 44 / 48 | **pass** |
| Function-grid tile | 52 pt — `MyViewFunctionCard.swift:130` | not measured | 44 | pass |

The timetable is the worst case on both platforms and is also the screen where the tap target
matters most. `HamButton` supplies **zero** padding — the `Surface(` at `Button.kt:60-102` takes no
`contentPadding` argument at all — so a button wrapping a bare
icon inherits the icon's own size — which is why the back button is a 24 dp target.

#### Contrast

**Requirement.** Body text meets **4.5:1** against its background; large text (≥18pt, or ≥14pt
bold) and non-text UI meet **3:1**. Both in light and dark mode. Nothing in either app currently
enforces this, and the word "contrast" appeared once in the whole document set before this table.

**Measured, light mode** (WCAG relative luminance, from the hexes in [§2.2](#22-colour)):

| Pair | Ratio | Verdict |
| --- | --- | --- |
| `text.primary` on `surface.primary` / `secondary` / `tertiary` | 19.95 / 21.00 / 18.08 | pass |
| `text.secondary` #888888 on `surface.primary` #F9F9F9 | **3.37** | **fails 4.5** |
| `text.secondary` #888888 on `surface.secondary` #FFFFFF | **3.54** | **fails 4.5** |
| `text.secondary` #888888 on `surface.tertiary` #EDEEEF | **3.05** | **fails 4.5**, only just clears 3 |
| Spec's own `text.secondary` #8E8E93 on #FFFFFF | **3.26** | **fails 4.5** — the spec value is no better than the shipped one |
| Spec's own `text.secondary` #8E8E93 on #EDEEEF | **2.81** | **fails even 3:1** |
| `text.tertiary` (#8E8E93 @ 60% → #BBBBBE) on #FFFFFF | **1.92** | **worst in the set** — invisible, not merely low |
| `caption2` 11sp `text.secondary` on `surface.primary` | **3.37** | fails 4.5 at the smallest size — worst combination |
| #888888 on dark `surface.primary` #000000 | 5.92 | pass — the hardcoded grey is only a light-mode problem |

So **`text.secondary` and `text.tertiary` need new values, not new wiring.** #8E8E93 and
#888888 both fail, and 60% opacity on top of that cannot be rescued. Raise the base grey until
it clears 4.5:1 on all three surfaces, and define `text.tertiary` as a **solid** colour, not an
alpha.

**Text on a brand fill** — the status-card header band and the function-grid tile both put the
brand hex at full strength on the same brand at 0.15 alpha over `surface.secondary`
(`CommonStatusCard.kt:66,78` / `CommonStatusCard.swift:74`, `MyViewFunctionCard.swift:131-133`):

| Brand | Android | iOS | Verdict |
| --- | --- | --- | --- |
| `coursescore` #283593 | 7.99 | 7.99 | pass |
| `course` #1B5E20 | 6.21 | 6.21 | pass |
| `schedule` #01579B | 5.82 | not verified (asset catalog) | pass |
| `pay` #BF360C | 4.45 | 4.45 | passes 4.5 marginally |
| `library` #007AFF | 3.30 | 3.30 | title passes 3:1, 12sp subtitle fails |
| `bus` #A2845E | 3.00 | 3.00 | title passes 3:1, 12sp subtitle fails |
| `sport` | 2.42 (Material #4CAF50) | **1.97** (#34C759) | **fails even 3:1** |
| `score` | **1.92** (#FF9800) | **1.95** (#FF9500) | **fails even 3:1** |

Green and orange are the failures: a mid-tone brand colour on a pale tint of itself cannot work,
at any size. **Do not use the brand hex as the text colour on its own tint.** Use a darker
`brand.<module>.text` — one token per module, measured to 4.5:1 — or `text.primary`.

**Toasts** (`ToastType+UI.swift:8-32`) put white text on a system fill: `.info` #007AFF **4.02**,
`.success` #34C759 **2.22**, `.warning` #FFCC00 **1.51**, `.error` #FF3B30 **3.55**. Three of
four fail. Toast fills need their own darker shades.

Two causes of the `text.secondary` failure, and the second will not be fixed by editing a colour
file:

1. `gray` is `#888888` in **both** `values/colors.xml:24` and `values-night/colors.xml:13`, so
   the dark variant is the same grey — the resource is adaptive in name only.
2. The Compose accessor **bypasses the resource entirely**. `Color.kt:28-29` is
   `get() = Gray` — a compile-time constant — whereas `ham_text_primary` directly above it uses
   `colorResource(...)`, which *is* adaptive. Every call site that writes
   `Color.ham_text_secondary` (`Card.kt:83`, `TextField.kt:78`) therefore cannot respond to dark
   mode even after the XML is corrected.

Not computable statically: course text over a **user-chosen timetable photo**
(`CourseMainViewBodyCell.kt:169-178`, `CourseViewBodyCourseItemView.swift:113-124`). Both
platforms pick black or white from the background luminance, which is sound over a flat colour
and unbounded over a photo. Require a scrim under the text.

The spec defines `text.tertiary` ([§2.2](#22-colour)); **Android has no such token** — only
primary and secondary exist — so placeholders and disabled labels silently fall back to
secondary.

#### What is *not* broken

Android declares every text token in `sp` (`Font.kt:45-52`) and SwiftUI text scales by default,
so **text does scale on both platforms** — nothing caps it. The failure mode is clipping and
overlap, not text that refuses to grow: iOS has 195 `.frame(width:` and 158 `.frame(height:`,
Android has 257 `.height(` and 198 `.size(`. Say this precisely — "text does not scale" would be
false.

Related: Android has **71** `maxLines` sites against **46** `TextOverflow.Ellipsis`, so ~25 places
truncate without telling the reader the text was cut.

---

## 3. Components

### 3.1 Card

The workhorse. Radius 16, padding 16, `surface.secondary`, flat.

```
┌──────────────────────────────────────────────────────┐
│ CARD                              radius 16, pad 16  │
│                                                      │
│  Title                          17 / Bold / primary  │  ← HEADER (optional)
│  Subtitle                       12 / Regular / sec.  │
│         ───────────── 8 ──────────────               │
│  <body content>                 padding 16           │
│                                                      │
│  ──────────────────────────────────────────────────  │  ← DIVIDER (optional)
│  <footer content>               padding 16           │
└──────────────────────────────────────────────────────┘
```

| Property | Value |
| --- | --- |
| radius | 16 |
| padding | 16 — a single inset shared by header, body, and footer |
| background | `surface.secondary` |
| header → body gap | 8 (emitted only when a title exists) |
| divider | 1px `surface.tertiary`, inset 0, vertical padding 4 |
| watermark | `icon.watermark` 128 @ **0.15**, bottom-trailing (16, 16), clipped |

### 3.2 Status card

A card with a coloured header band. Used only on the 状态 dashboard.

```
┌──────────────────────────────────────────────────────┐
│▓▓ [icon] 图书馆                              ▸ ▓▓▓▓▓▓│  ← band: brand @ 0.15, padding 12
├──────────────────────────────────────────────────────┤
│                                                      │
│  <body content>                  padding 12          │
│                                                      │
└──────────────────────────────────────────────────────┘
     radius 16 · background surface.secondary
```

| Property | Value |
| --- | --- |
| radius | 16 |
| band background | brand @ 0.15 |
| band padding | 12 all sides |
| band icon | `icon.md` (24), brand colour |
| band title | 17 / Bold, brand colour |
| band → body gap | **0** — the band's own padding separates them |
| body padding | **12**, overridable (the bus card passes 0) |
| trailing chevron | `icon.xs` (12), brand colour, present only when the card navigates |

### 3.3 List row

A tappable row inside a card, or standalone.

```
┌──────────────────────────────────────────────────────┐
│  ╭────╮                                              │
│  │ ic │  Title                              ▸        │
│  ╰────╯  Subtitle                                    │
└──────────────────────────────────────────────────────┘
   40×40 circle       8 gap        17/Bold    12 chevron
   brand @ 0.1                     12/Regular subtitle
```

| Property | Value |
| --- | --- |
| icon plate | 40 × 40 circle, brand @ 0.1 |
| icon glyph | `icon.sm` (20) |
| icon → text gap | 8 |
| title | 17 / Bold, `text.primary` |
| subtitle | 12 / Regular, `text.secondary` |
| chevron | `icon.xs` (12), `text.secondary` |
| divider between rows | 1px `surface.tertiary`, inset 0, vertical padding 4 |

### 3.4 Buttons

| Variant | Height | Padding | Radius | Background | Text |
| --- | --- | --- | --- | --- | --- |
| Filled primary | 48 | h16 | 8 | `tint.active` | 17 / Bold / white |
| Tinted | 48 | h16 v12 | 12 | `tint.subtle` | 17 / Bold / `accent` |
| Borderless | = text | 0 | — | none | 17 / Regular / `text.link` |
| Destructive text | = text | 0 | — | none | 17 / Regular / `text.danger` |
| Destructive pill | 28 | h12 v6 | 8 | transparent, 1px `text.danger` @ 0.3 | 12 / Regular / `text.danger` |
| Row-style | 52 | h16 | 12 | `tint.subtle` | 17 / Bold |
| Large tile | 150 | 16 | 16 | `tint.subtle` | 17 / Bold + 11 subtitle |
| Icon button | 44 target | 8 | circle | `surface.tertiary` | — |

Rules:

- **Disabled** = palette swap, not opacity: `tint.subtle` → `tint.muted`, brand text →
  `text.secondary`. Reserve whole-view opacity 0.5 for genuinely unavailable regions.
- **Minimum tap target 44**, including icon buttons and inline links. This is a hard floor.
- Every button has a pressed state.

### 3.5 Chips and badges

| Property | Value |
| --- | --- |
| radius | 6 |
| padding | h6 / v4 |
| selected | `tint.chip` — **accent** @ 0.10 background, **accent** text |
| unselected | `tint.muted` — `text.secondary` @ 0.10 background, `text.secondary` text |
| font | 12 / Bold |

Status badge (the coloured bar or dot beside a value): 6 × 6, radius 3, in the entity's colour.

### 3.6 Controls

| Control | Specification |
| --- | --- |
| Switch | Native on each platform, **`accent` when checked**. Android currently ships `brand.sport` — change it: a switch is interactive, so it takes `accent` ([§2.2](#22-colour)). |
| Slider | Native on each platform, no custom colours. |
| Progress bar | Height 4, radius 2, track `surface.tertiary`, fill in context colour. |
| Text field | Radius 8, background `surface.secondary`, 1px `surface.tertiary` border, padding 8, body text, placeholder `text.tertiary`, caret `accent`. |
| Segmented control | Track radius 6, track `text.secondary` @ 0.40, track padding 2, thumb radius 6, thumb `surface.tertiary`. Thumb radius must equal track radius (they differ today). |
| Picker button | Radius 8, padding v8 / h12, background `surface.tertiary`, 17 / Regular. |
| Swipe to confirm | Height 48, radius 12, track `surface.tertiary`, fill `text.danger`, 3px handle. |
| Checkbox | `icon.md` (24), radius 4, `accent` when checked. |

### 3.7 Sheet

| Property | Value |
| --- | --- |
| radius | platform default (iOS system ≈ 12, Android 24) |
| drag handle | 36 × 5 pill, `text.secondary` @ 0.4, centred in a 32-tall header |
| background | `surface.primary` |
| scrim | black @ 0.5, tap to dismiss |
| detent | 85% of screen height, overridable |

The drag handle is mandatory — without it users have no affordance telling them the sheet is
draggable.

### 3.8 Toast

| Property | Value |
| --- | --- |
| radius | 8 |
| padding | 16 all sides |
| icon | 36 |
| icon → text gap | 5 |
| title | 17 / Bold |
| subtitle | 12 / Regular |
| duration | 3300 ms |
| position | top, below the status bar |
| types | info, success, warning, error, neutral — background `feedback.*` |

### 3.9 Empty state

| Property | Value |
| --- | --- |
| icon | `icon.xl` (64), `text.tertiary` |
| text | 12 / Regular, `text.tertiary` |
| container | inherits from the parent card |

### 3.10 Non-content states

Every screen has five states, not one: **loading, empty, error, offline, and loaded**. Only the
last is documented per screen in [`screens.md`](screens.md); the other four are specified once,
here, and every screen inherits them.

**The rule that matters most: an error state must offer a way out.** Every one of the four
non-loaded states either resolves itself or gives the user an action. A state with no action and
no exit is a dead end.

| State | Specification |
| --- | --- |
| **Loading** | A spinner occupying the space the content will occupy — never a blocking modal, except while a committed action is in flight (payment, booking). Use the **shared** loading component; do not repeat `ProgressView` at the call site. Where the layout's shape is known in advance, a skeleton is better than a spinner. |
| **Empty** | [§3.9](#39-empty-state). One shared component per platform. Never a bare string centred in a blank card. |
| **Error** | Icon 64, title, the reason, and a **retry** action. The shared error component's default action is **retry**, not dismiss. |
| **Offline** | A persistent banner, not a toast — connectivity loss is a state, not an event. Content already loaded stays readable. |
| **Session expired** | Its own state, with its own copy: the session ended, sign in again. Not a generic network error. |
| **Rate-limited / blocked** | Show the server's message verbatim. Do not rewrite it into a generic failure. |
| **Permission denied** | A blocking state whose action is **打开设置**, not 重试. The app cannot grant a permission it does not hold, so a retry button is a lie — the only way out is the system settings screen. Every screen that needs camera, location, notifications or exact alarms must render this state. |

#### Current state

| | iOS | Android |
| --- | --- | --- |
| Shared loading component | `HamAsyncContentLoadingView` — `iOS/ui/common/container/HamAsyncContentView.swift:142`, added by `fe97e1b2` (#132); **adopted by 4 screens** (three course-center pages + `AuthorizedAppsView`). `ProgressView` is still hand-rolled at **52** sites in 44 files | `HamLoadingProgressBar` — `core/ui/…/component/HamLoadingProgressBar.kt:20`, 61 sites, plus a blocking `LoadingModalView` |
| Skeleton | **one** — `.redacted` seat grid, `LibrarySelectSeatView.swift:368` | **none anywhere** |
| Shared empty component | **absent in substance** — `HamAsyncContentPlaceholder` (`HamAsyncContentView.swift:156-160`) is now the shared default, but it renders a zero-height `Color.clear`, so the user still sees nothing | **absent** |
| Shared error component | **two** — `ErrorView` — `iOS/ui/common/view/ErrorView.swift:8`, default action **返回** `:22`; and `HamAsyncContentFailedView` — `HamAsyncContentView.swift:178-207` | `ErrorView` — `core/ui/…/intro/ErrorView.kt:48`; default action **完成** `:60` |
| **Retry in the shared error component** | **split** — `ErrorView`: **no**, still defaults to 返回. `HamAsyncContentFailedView`: **yes** — a `common.retry` button at `:189-200`, and all four adopted screens pass `retry: { vm.fetchNextPage() }` | **no** |
| Connectivity monitoring | one `NetworkReachabilityManager`, refreshes remote config only — **never drives UI** | **none** — 0 hits for `ConnectivityManager` / `NetworkMonitor` / `isOnline` |
| Session-expired handling | `AuthPbInterceptor.swift:63` — `send` and `errorCaught` are pass-throughs, **no 401 handling**. *One exception:* the library module recovers — `error.tokenExpired` renders a tappable red `LibraryRetryLoginView` (`iOS/ui/library/common/retry-login/`), re-signing in without leaving the screen | no auth interceptor; `PbRequestHelper.kt:178,205` only add headers. *Same exception:* `feature/library/…/ui/common/LibraryRetryLoginView.kt` |
| Rate-limit state | none | none |
| **Retry actions, app code** | **4** — `CourseScoreCourseDetailErrorView.swift:33`, `StatusBusCard.swift:31`, `StatusWeatherCard.swift:58`, `PrintPrepareView.swift:45` | **5** wired to a control, of 7 retry strings |
| **Route to system Settings** | **none** — 0 hits for `openSettingsURLString` or `UIApplication.open` in all of `iOS/` | **1** — `Settings.ACTION_REQUEST_SCHEDULE_EXACT_ALARM`, `AlarmPermissionSheet.kt:143`; the only hit in the app, and it is an Android-only feature |
| Pull-to-refresh | hand-rolled overscroll: `scrollY < -scrollThreshold` fires `onRefresh()`, `StatusUpdateView.swift:57-63`; **1** use of `.refreshable`, on the course centre (`CourseCenterView.swift:84`) — so the hand-rolled gesture and the system gesture now coexist | one screen — `StatusContainerView.kt:157` |

Five consequences worth stating plainly:

1. **The dominant error pattern on Android is a toast with no retry** — ~70 `ToastManager` call
   sites against 13 `ErrorView` sites. `ToastManager.kt:247` hardcodes 网络异常，请稍后重试 —
   text that promises a retry the UI does not offer.
2. **Neither app knows it is offline.** The user sees a generic failure, not "you are offline".
3. **Session expiry is indistinguishable from a network error** — *except in the library
   module*, which is the only place on either platform that recovers. It proves the pattern is
   implementable; it has simply not been generalised.
4. **Two call sites label a retry as a dismissal.** iOS course center calls
   `ErrorView(title: 请求失败) { vm.doRequest() }` and inherits the default **返回** label;
   Android print does the same and inherits **完成**. Both do retry — they just tell the user
   they are going back. Each is a one-argument fix: pass the label.
5. **Error and empty render the same pixels** — *except on the four screens that adopted
   `HamAsyncContentView`.* Android's `when (loadState) { … else -> {} }` and iOS's fall-through
   to an empty list mean a failed load and a genuinely empty result are indistinguishable — and
   neither offers a retry. `HamAsyncContentState` fixes exactly this: `.empty` and `.failed` are
   separate cases (`HamAsyncContentView.swift:71-72`) and the failed view always draws
   (`:181-206`) with a retry. The ~50 un-adopted iOS screens still have the defect.

**iOS has the component now; it is not yet the rule.** `HamAsyncContentView`
(`iOS/ui/common/container/HamAsyncContentView.swift:244`) and its `HamAsyncContentState`
(`:67-108`) are the shared loading/error/empty/content container this section asks for, and its
state derivation is deliberately *data wins* — while paging, a loading status with a non-empty
value becomes `.loadingMore(value)` (`:87-88`) so old content stays on screen instead of flashing
a spinner. Only **4 of ~54** screens use it; 58 hand-rolled `loadState == .` ladders remain.
`CourseScoreCourseDetailView` is a deliberate holdout — it wants the opposite rule (an error
covers existing content), noted in the code at `:81-83`.

#### Where a retry exists — measured

Across ~11 modules only **4 iOS screens and 6 Android screens** offer a retry, and of 14
comparable cases only **4 match**. The sharpest divergence:

| Case | iOS | Android |
| --- | --- | --- |
| Booking screen, seat list fails | **backs out only** — `ErrorView(title: 预约失败, backAction:)`, `LibraryBookErrorView.swift:22-25` | **retries** — `buttonText = stringResource(R.string.library_retry)` → `vm.fetchSeatList()`, `LibraryBookView.kt:134-141` |

Also worth stating: Android's **scan screen has no camera-denial handling at all**. Scan Kit
"reports to nobody" for a denied camera permission — the in-code comment at
`QrCodeScanView.kt:205-208` records that the callback yields only `CAMERA_INIT_ERROR` and the
gallery permission code, and that `scankit/b.java` returns without opening the camera. A denied
permission therefore produces a black screen indefinitely. iOS does better — it toasts
相机权限未开启 / 请前往"设置"开启 — and then cannot take the user there, because the app has
no Settings route (see the table above). Both halves of the fix are missing on both platforms:
iOS has the words but no destination, Android has neither.

---

## 4. Screens

The per-screen specification lives in **[`screens.md`](screens.md)** — 129 sections across 12
modules (**120 screens** plus 9 analysis sections), each with a layout diagram, ordered content
blocks, a value table, its strings, and its states.

This chapter covers what applies to every screen: the page scaffold, the module index, and the
cross-screen **patterns** — how a screen is presented, how a form behaves, how loading and
confirmation work.

### 4.1 What every screen has

| Element | Rule |
| --- | --- |
| Screen horizontal margin | **16** |
| Gap between cards | **8** |
| Screen background | `surface.primary` |
| Bottom spacing — tab-root screen | system navigation-bar inset **+ 80** |
| Bottom spacing — pushed/child screen | system navigation-bar inset **+ 24** |
| Top | content begins below a **42**-tall header plus the status bar |

Bottom spacing is computed from the live system inset, never hardcoded.

**Header height is 42 everywhere.** Android ships two values and only one is right:

| Where | Value | Source |
| --- | --- | --- |
| Android tab roots | 42 — correct | `TOOLBAR_HEIGHT` — `ScreenExtension.kt:56`, used by `HomeContainer.kt:70,87,101` |
| Android pushed screens on `HamNavigationView` | 42 — correct | `NavigationView.kt:97,105,112` |
| Android pushed screens on `HamNavigationView2` | **36 — wrong** | `HamNavigationViewUIConfig.headerHeight = 36.dp` — `NavigationView.kt:197`, read at `:287,:326` |

`HamNavigationView2` is used by 6 pushed screens (`CourseCenterView`, `CourseCenterRankView`,
`CourseCenterCommentHistoryView`, `CourseDetailCommentView`, `SportOrderCenterView`,
`SSOAuthorizationSheet`). No document previously said which value a new screen takes, so the
split has been re-litigated per screen. The answer is 42; pass `headerHeight = 42.dp` until the
default is corrected.

### 4.2 Module index

| Module | Screens | Notes |
| --- | --- | --- |
| 状态 Status | 10 | The dashboard; cards ranked by relevance |
| 课程表 Course | 7 | Fixed grid, not scrolling cards |
| 日程 Schedule | 5 | Hero card + pinned group tabs + list |
| 图书馆 Library | 12 | Seat and room booking |
| 运动 Sport | 13 | Venue booking; the order screen is the most complex in the app |
| 成绩 Score | 12 | Grades, GPA, F2 calculation |
| 课程评分 CourseScore | 8 | Course reviews and grade distributions |
| 我的 My | 10 | Account hub and the function grid |
| 用户中心 User center | 21 | Account, devices, passkeys, authorized apps |
| 登录与授权 Auth | 8 | Login, CAS, SSO consent, QR |
| 共享组件 Shared | 15 | Error, empty, loading, toast, sheet, webview, RN container |
| 独立页面 Standalone | 8 | Bus, pay, privacy, changelog, debug, automation, share |

### 4.3 Screens that exist on one platform only

Recorded in `screens.md` per screen. The ones that matter most:

| Screen | Platform | Consequence |
| --- | --- | --- |
| Sport status card | iOS | Android shows 5 cards where iOS shows 7 |
| Settings hub, language, widget settings | Android | iOS has no equivalent entry point |
| Share sheet, Siri shortcuts | iOS | Android has no equivalent |
| Bulletin list | iOS | Android's banner links straight to the detail |

### 4.4 Navigation presentation

Four ways to get to a screen. Nothing previously said which to use, so the choice has been made
per screen and the two clients have drifted. **Push is the default**; the other three need a
reason.

| Presentation | Use it when | Measured today |
| --- | --- | --- |
| **Push** — iOS `NavigationLink` / `router.push`, Android `navController.navigate` | The user tapped **content**: a row, a card, a list item, a function-grid tile. The new screen is a *place*. | iOS **67** `NavigationLink(` + 9 `router.push(`; Android 131 `navController.navigate(` |
| **Sheet** — iOS `.sheet`, Android `HamSheet` | The task is **about the current screen** and returns to it. Five cases: (1) a module intro / CAS gate ([§5.1](#51-cas-verification-gate)); (2) a login-provider webview; (3) an app-level flow launched from the root — pay, SSO authorization, changelog; (4) a transient picker — colour, alarm permission; (5) a terminal success or result state. | iOS 21 `.sheet(`; Android 13 `HamSheet(` call sites. The two lists match case for case. |
| **Full-screen takeover** — iOS `.fullScreenCover`, Android a separate `Activity` | The task **brings its own chrome** and must not be dismissed by dragging down: image crop, a document webview (privacy policy), receiving a shared file (print), social-account linking. | iOS 4 `.fullScreenCover(` (`CourseSettingViewUISection.swift:157`, `LoginView.swift:158`, `SyncSocialAccountView.swift:69,84`); Android 4 activities (`ImageCropActivity`, `PrivacyActivity`, `PrintShareFileActivity`, `MainActivity`) |
| **Overlay** — `.overlay` / `Box` | **Decoration and in-place chrome only** — watermarks, badges, counts, a scrim. Never a whole screen. | iOS **42** `.overlay(` |

Mechanics of a sheet are in [§3.7](#37-sheet): 85 % height, radius 24, scrim to 0.5,
quarter-height drag to dismiss.

Three consequences of the rule:

- **Do not use a sheet for a place.** A screen the user will navigate *from* — course detail,
  a booking, a settings page — pushes.
- **iOS has no `.popover(` anywhere** (0 sites), so the 300-wide schedule detail popover in
  `screens.md` §3 is a sheet on iOS, and Android's is a half-height sheet while iOS's is a
  full-screen morphing overlay. That is a recorded divergence, not a rule — pick one, and the
  rule above says **sheet**.
- **Android has 34 `Dialog(` sites; iOS has 1 `.alert(` and 1 `.confirmationDialog(`.** The
  confirmation pattern is therefore built almost entirely on Android. See [§4.6](#46-confirmation-and-destructive-actions).

### 4.5 Forms and text entry

Both clients build forms the same way, and both build them **per field** — there is no shared
form pattern, only a shared habit. Measured on the schedule editor and the user-center editors:

| | iOS | Android | Agree? |
| --- | --- | --- | --- |
| Text-field label | **Placeholder-only** + leading icon — `TextField("输入日程名称", …)` after `Image(systemName: "flag.fill")` — `ScheduleInsertView.swift:37-42` | **Hint-only**; the schedule editor hand-builds a `BasicTextField` rather than using the shared component — `InsertEditHomeView.kt:146-165` | **yes** |
| Non-text field | leading label, trailing value, chevron — `ScheduleInsertView.swift:51-70,92-117` | same — `InsertEditHomeView.kt:176-189,200-214` | **yes** |
| Required marking | **never** — no `必填`, no asterisk, no colour; "optional" is written into the placeholder (`输入地点(可选)`) | **never** — `grep -rn "必填" --include='*.kt'` → 0 | **yes** |
| Validation timing | **on submit only** — `validate()` from `save()` — `:282-311`, called `:314` | **on submit only** — first statement of `commit()` — `InsertEditViewModel.kt:179-186` | **yes** |
| Error placement | **toast** — `ToastUtils.showError`; zero inline field errors | **toast** — `ToastManager.showError`; `HamTextField` has **no error slot at all** | **yes** |
| Error copy | localized string for client rules; server message passed through for gRPC (`ToastUtils.showGRPCError`) | string resource for client rules; `ToastManager.showGrpcError(e)` for gRPC | **yes** |
| Keyboard declared | **3** `.keyboardType` in the whole UI tree — 1 print, 2 debug | **4** `KeyboardOptions` — 3 in one print screen, 1 `imeAction` on the comment editor | **yes** (both: almost never) |
| Submit placement | **two ship** — bottom full-width filled (schedule `:258-270`) *and* nav-bar trailing 保存 (profile `:75-81`) | **the same two** — bottom 48 dp (`InsertEditHomeView.kt:305-364`) *and* nav-trailing (`UserCenterInfoView.kt:65-75`) | **yes** |
| Shared field component | **none in use** — `TextEdit.swift:10` has zero call sites; 13 hand-built `TextField(` outside tests and the component itself | `HamTextField` exists (`TextField.kt:37`) but is used at only 10 call sites; the flagship editor bypasses it | **no** |

Rules:

1. **Create/edit a record → the primary action is a full-width, 48-tall primary button at the
   bottom of the form.** Editing one value in place (nickname, group name) → nav-bar trailing
   text button. Do not use both in one app for the same form type; that is what ships today and
   it is not a decision.
2. **A field error belongs below the field, not in a toast.** A toast names the problem and not
   the field, then disappears. Add an error slot and a label slot to `HamTextField` and adopt it
   on iOS, which currently has no field component at all.
3. **Validation fires on submit.** Keep it. Per-keystroke validation on a 24-hour-time field is
   worse than a clear error on tap.
4. **Declare the keyboard.** A numeric field gets a number pad; today only the print screen does
   (`shared/ui/print/PrintOptionsForm.swift:147`, `PrintShareFilePrepareView.kt:253,274,326`).
5. **Required fields are marked**, or none are. Today "optional" is written into the placeholder
   and required is unmarked, so the user learns by failing.

**Accessibility consequence of hint-only:** an iOS `TextField` prompt survives as the
accessibility label after the user types; an Android hint does not — once the field has text,
`HamTextField` reports no label at all. See [§2.10](#210-accessibility).

### 4.6 Confirmation and destructive actions

Measured across 13 destructive actions and 5 logout entry points:

| Action | iOS | Android |
| --- | --- | --- |
| Log out — user center | **no confirm** — `UserCenterView.swift:66-75` | **no confirm** — `UserCenterMainView.kt:181-183` |
| Log out — CAS settings | **no confirm** — `CasSettingView.swift:85` | **no confirm** — `CasSettingMainView.kt:78-80` |
| Log out — dedicated screen | — | **no confirm** — `SyncLogoutView.kt:44-51` |
| Delete schedule | **no** (behind 更多操作) — `ScheduleInsertView.swift:241-251`; single delete is a two-tap inline collapse — `ScheduleItemDetailView.swift:117-146` | **no** — `InsertEditViewModel.kt:166-177` |
| Delete schedule group | **no** — `ScheduleGroupEditView.swift:73-85` | **no** — `GroupEditView.kt:141` |
| Deactivate account (注销) | inline disclosure, **a lone 确定 with no cancel** — `SyncLogoutView.swift:38-60` | inline disclosure, same copy — `SyncLogoutView.kt:88-102` |
| Revoke authorized app | **YES** — `.alert`, cancel + destructive — `AuthorizedAppsView.swift:35-46` | **YES** — `AlertDialog`, dismiss + confirm in `ham_red` — `AuthorizedAppsView.kt:117-147` |
| Remove login device | **no** — `SyncLoginDeviceView.swift:118-132` | **no** — `UserCenterDeviceView.kt:72` |
| Delete passkey | **no** — `UserCenterPasskeyConfigItemView.swift:26-31` | **no** — `UserCenterPasskeyConfigView.kt:201-203` |
| Cancel library booking | slide-to-confirm (`Unlocker`, 95 %) — `LibraryModifyBookingView.swift:133-158` | slide-to-confirm (`HamLocker`) — `LibraryModifyBookingView.kt:126-138` |
| Delete starred seat | — | **no** — `StarredSeatSettingView.kt:177-193` |
| Delete course / class | — | **no**, fires from a dropdown — `CourseMainViewDropDownMenuCell.kt:186-228` |
| Remove from 想上 | **no**, `allowsFullSwipe` deletes with no confirm — `CourseCenterSelfWantPageView.swift:62-68` | — |

**11 of 13 destructive actions have no confirmation, and no logout anywhere confirms.** Each
platform has exactly **one** correct confirmation — the same one, revoking an authorized app —
and it is the only place either app uses a native alert for a decision: iOS has **1** `.alert(`
in the entire UI tree, Android has **1** `AlertDialog` used as a confirmation (2 imports
them pickers and a privacy notice). There is no shared confirm component on either platform.

Rules:

1. **Confirm before anything that ends a session, ends an account, or destroys data** — logout,
   注销, deleting a schedule, group, course, device, passkey, starred seat or theme, revoking
   access. Today none of these confirm except one.
2. **Do not confirm a reversible action.** Cancelling a booking already uses slide-to-confirm on
   both platforms — keep it; it is a commitment gesture, not a dialog.
3. **Use the platform's native alert**: iOS `.alert` with `role: .destructive` on the action and
   `role: .cancel` on cancel; Android `AlertDialog` with dismiss left, confirm right, confirm in
   `feedback.error`. `AuthorizedAppsView` on either side is the reference implementation.
4. **Copy shape: title = the action noun, message = a question, buttons = cancel + the action.**
   `确定要取消对该应用的授权吗？` is the pattern.
5. **Never a lone 确定.** The deactivate-account disclosure on both platforms offers no cancel —
   the only way out is collapsing the disclosure. That is not a confirmation.

### 4.7 Data loading and caching

| Screen | iOS | Android | Agree? |
| --- | --- | --- | --- |
| Status dashboard | **cache-first** — persisted snapshot in `init()`, then a 15 s poll — `StatusCourseCardViewModel.swift:41-48,60-71` | starts empty, polls a **local Room DAO** every 5 s — `CourseCardViewModel.kt:76-84` | **no** — iOS shows stale data immediately, Android shows nothing until the first fetch lands |
| Course timetable | **cache-first**, but the cache is the in-memory `CourseContext`, not a Realm read — `CourseService.swift:116-138` (`invalidate()` `:150`) | **cache-first**, Room — `doInit()` reads `queryCourseGrid` — `CourseMainViewModel.kt:129-132` | yes |
| Score | **cache-first**, Realm — `ScoreService.swift:13-14` | **cache-first**, Room — `doInit()` reads `queryScore()` — `ScoreMainViewModel.kt:214-217` | yes |
| Schedule | **local only** — `@ObservedResults`, no network in the read path — `ScheduleView.swift:14` | **local only** — live Realm result — `ScheduleHomeViewModel.kt:24-25` | yes |
| Library home | **network-first** — `init()` fetches, no disk cache — `LibraryMainViewModel.swift:30-37` | **network-first** — the empty list is `LibraryContext.kt:60`, never seeded from disk; `init {}` fetches at `LibraryMainViewModel.kt:64-73` | yes |
| Sport home | **network-first** — `SportMainViewModel.swift:25-60` | **network-first** — `SportMainViewCurrentOrderCardVM.kt:25-36` | yes |

Persistence: iOS uses `UserDefaults` (App Group), `@AppStorage`, Realm and
`NSUbiquitousKeyValueStore`; Android uses Room (`ham.db`), MMKV, Realm and CCKV. **Both use
Realm for the schedule.**

**No stale-while-revalidate policy exists on either platform.** `grep -rn
"staleWhileRevalidate\|cacheThenNetwork\|CachePolicy"` returns **4** hits and none of them is an SWR
policy: iOS sets a URL-cache policy (`AFRequestBuilder.swift:148`
`.reloadIgnoringLocalAndRemoteCacheData`) and Android sets Coil image-cache policies
(`ImageLoaderHelper.kt:40-41`). The status dashboard is the closest thing to SWR, and only on iOS.

Pull-to-refresh: iOS has **1** `.refreshable` (`CourseCenterView.swift:84`) and a **hand-rolled**
indicator on the status dashboard — drag past **128 pt** and a pill slides down reading
`已请求刷新`; releasing fires `vm.contentVM.updateAllCard()` and the pill fades after 2 s —
`StatusUpdateView.swift:15,31-32,55-61`, wired at `StatusView.swift:72-74`. Android has **1**
real `rememberPullToRefreshState` (`StatusContainerView.kt:157`, **300 px** threshold) plus the
shared scroll container `BounceScrollView`, instantiated at **8** sites
(`NavigationView.kt:95`, `HomeContainer.kt:67`, `ScheduleMainViewGroupTabView.kt:56`,
`ScheduleMainViewGroupExpandView.kt:88`, `MyMainViewContainer.kt:90`,
`SelectSeatCard.kt:65`, `LibraryBookView.kt:210`, `SportSelectHeaderView.kt:129`) — two of which
are the shells that host **54** screen call sites (`HamNavigationView(` 50, `HomeContainer(` 4).
So two screens on iOS and one on Android refresh, at thresholds that differ by more than 2×, and
the container every other Android screen scrolls in promises a refresh it does not perform. (The iOS pill is `Color.blue`, a raw system
colour — see [§2.2](#22-colour).)

Rules:

1. **Local-first for anything the user has already seen** — course, score, schedule. Render from
   the local store on appearance; refresh in the background.
2. **Network-first for live state** — bookings, current orders. A spinner from empty is correct
   when the data is about *now*.
3. **Stale-while-revalidate is the rule**: render the persisted value, revalidate in the
   background, replace on success, **keep the stale value on failure**. Write it once as a shared
   policy; today it exists on one screen on one platform.
4. **Persist the last good value for the dashboard.** iOS does; Android does not, so its cards
   are blank on every cold start until the first fetch.
5. **Pull-to-refresh on every list that can go stale**, not just the status dashboard. Android's
   `BounceScrollView` — every screen that scrolls inside a shell — must either refresh or stop
   bouncing; gesture feedback that does nothing is worse than no gesture. iOS's hand-rolled pill
   should become `.refreshable`, as the course centre already is.

### 4.8 Search, filter, sort, pagination

The main search surface is course-score search → result list. Long lists elsewhere: course
comments, 想上 history, comment history, score rank, authorized apps, library history, library
seats.

| | iOS | Android | Agree? |
| --- | --- | --- | --- |
| Where the query lives | **one** `@Published var keyword` on `CourseScoreMainViewModel`, injected as `@EnvironmentObject` and shared by home, search bar and result — `CourseScoreMainViewModel.swift:32` | `@AssistedInject` `var keyword by mutableStateOf("")` on `CourseScoreSearchViewModel`, one per search screen, partly carried in the route — `CourseScoreSearchViewModel.kt:87` | **no** |
| Debounce | **none** — `.onChange(of: em.keyword)` fires a gRPC `queryCourse` on **every keystroke**, guarded only by a blank check and a stale-response check — `CourseScoreSearchViewSearchBar.swift:35-37`, `CourseScoreMainViewModel.swift:39-51` | **none needed** — `searchCourse` is an explicit call on submit or history tap, not a keystroke hook — `CourseScoreSearchViewModel.kt:154-168` | **no** — iOS requests per character |
| Filter control | 3 inline chips in an `HStack` under the search bar — `CourseScoreResultViewSearchBar.swift:50-56` | 3 chips in a `LazyRow` — `CourseScoreResultItemFilterFunctionView.kt:31-66` | **yes**, near-identically styled |
| Filter chip styling | selected = blue text on 10 % blue; unselected = grey on 10 % grey — `:64-81` | selected = `ham_blue` @10 %; unselected = `ham_gray` @10 %; `RoundedCornerShape(6.dp)`, `HamFontStyle.caption` — `:70-86` | **yes** |
| Filter gated? | **no** | **yes — A/B flag** `enableCourseDetailConfig.enableCourseScoreResultFilter` — `CourseScoreSearchViewModel.kt:100-102` | **no** |
| Filter persisted? | **no**, defaults to `.totalDesc` every time — `CourseScoreResultViewModel.swift:23` | **no**, `mutableStateOf(TOTAL_DESC)` — `CourseScoreSearchViewModel.kt:130` | yes |
| Search *history* persisted | **yes**, Realm, capped to 30 — `CourseScoreResultViewModel.swift:97-103`, `CourseScoreHomeViewHistoryCard.swift:14,24` | **yes**, MMKV-backed `LSKV` — `CourseScoreContext.kt:26-36` | yes |
| Sort control | **none separate** — the 3 chips *are* the sort; all three hard-coded descending — `CourseScoreResultViewSearchBar.swift:51-53` | same — `CourseScoreResultItemFilterFunctionView.kt:31-66` | yes |
| Sort elsewhere | **none** — `grep -rn "sorted\|sort(" --include='*.swift' iOS/ui/coursescore` → 0 | **none user-facing** — only status-card and semester ordering | yes |
| Pagination mechanism | cursor, `resp.nextRequestCursor`, finish when `item.isEmpty` — `CourseScoreResultViewModel.swift:50-79` | same cursor design — `CourseScoreSearchViewModel.kt:194-224` | yes |
| Page size | server-decided for score and comments; **10** for authorized apps — `AuthorizedAppsViewModel.swift:14` | server-decided; **20** for comments — `CourseCommentViewModel.kt:139-166`; **10** for authorized apps — `AuthorizedAppsViewModel.kt:34` | **no** for comments (server vs 20) |
| Load-more trigger | second-to-last item appears — `CourseScoreResultView.swift:26-30` | second-to-last item appears — `CourseScoreResultView.kt:221-227` | **yes** |
| Footer loading indicator | **absent** on the score result — `searchResultLoadState` is published (`CourseScoreResultViewModel.swift:16,85`) but never rendered (`CourseScoreResultViewBody.swift:20-37`) | **absent** on the score result; only the initial full-box spinner | **no indicator at all** |
| Footer indicator elsewhere | **yes** — `ProgressView()` as a trailing row for want/comment history — `CourseCenterSelfWantPageView.swift:82-87` | trailing row while paging, `HamLoadingProgressBar()` on first load — `CourseCenterWantView.kt:60,160` | yes |
| Result count | **never** on the score result. Nav titles are static (`想上历史`, no count) — `CourseCenterSelfWantPageView.swift:97` | never on the score result; **count in the nav title** for want/rank (`想上历史(%1$d)`) and a filtered seat count in library | **no** |

Rules:

1. **Debounce search input — 300 ms, or fetch on explicit submit.** A gRPC request per keystroke
   is what iOS does today; it is not acceptable. `grep -rn "debounce"` finds debouncing in exactly
   two places — the status card-order manager's `SmartDebouncer`
   (`StatusContentViewModel.swift:65`) and the course-centre view model
   (`CourseCenterViewModel.swift:124`, 400 ms) — and **not** in search. So the idiom exists in the
   codebase; search is the one place that needs it and does not have it.
2. **The query belongs to the screen, not to a shared singleton.** iOS's
   `@EnvironmentObject CourseScoreMainViewModel` means a keyword typed on the search screen is
   still live on the home screen; Android's per-screen `@AssistedInject` VM is the better shape.
3. **A filter ships to everyone or to no one.** Android's three chips sit behind
   `enableCourseScoreResultFilter`; that is an experiment, not a design, and it makes the two
   platforms differ for reasons no designer chose.
4. **Separate *what is included* from *what order*.** Today sort and filter are the same chip
   row, all descending, with no ascending toggle. Either give the row one name and add
   ascending/descending, or split it into a filter row and a sort control.
5. **Persist the filter** the way you already persist history. Today the order resets to
   `TOTAL_DESC` on every visit on both platforms while the keywords survive.
6. **Render a trailing loading row whenever a page is in flight, and an end-of-list marker when
   it is not.** `searchResultLoadState` is already computed on both platforms and thrown away on
   the one screen that pages. iOS does render one for want/comment history — that is the pattern.
7. **Show the result count on any filtered or searched list.** Android does for want, rank and
   library seats; iOS does not; neither does for the score result, which is the list that most
   needs it.

### 4.9 Lists, grids, and card columns

Every module's main screen, by container:

| Screen | iOS | Android | Agree? |
| --- | --- | --- | --- |
| Status dashboard | `ScrollView { VStack(spacing: 0) }` — hand-stacked card column, **not lazy** — `StatusView.swift:22-23` | `Column` in a bouncy scroll host — `StatusContainerView.kt:90,103,130` | yes |
| Course timetable | **hand-built absolute grid** — `GeometryReader` + `ZStack` + `.offset(x:y:)`, 5-or-7 × 13 — `CourseViewBodyCourseBodyView.swift:18-48` | **hand-built absolute grid** — `Box` + `.offset`, size from `onSizeChanged` — `CourseMainView.kt:159-182` | yes |
| Library home | `ScrollView { VStack }` — `LibraryMainView.swift:15-16` | `Column` in a bounce host — `LibraryMainView.kt:40,80` | yes |
| Sport home | `ScrollView { VStack }` — `SportMainView.swift:15-16` | `Column` in a bounce host — `SportMainView.kt:30` | yes |
| Score | `ScrollView { VStack }` — `ScoreMainView.swift:18-20` | **`LazyColumn`** — `ScoreMainView.kt:105` | **no** — the only lazy main list on Android |
| Schedule list | **`LazyVStack`** in a `ScrollView` — `ScheduleView.swift:351,385` | **`LazyColumn`** — `ScheduleMainViewItemListView.kt:86` | yes |
| "My" function grid | **horizontal `ScrollView` of hand-laid `HStack` rows**, driven by cloud config `row`/`col` — `MyViewFunctionCard.swift:49-67` | **`LazyHorizontalStaggeredGrid(rows = StaggeredGridCells.Fixed(row))`**, same config — `MyViewFunctionComponentView.kt:201-205` | **no** |
| CourseScore home | `VStack` of cards + a **custom** wrapping-flow chip layout — `CourseScoreHomeViewHistoryCard.swift:22-23` | `VStack` + `LazyHorizontalStaggeredGrid`, `row = 8` — `CoursScoreMainView.kt:134-135` | **no** |
| CourseScore result | `ScrollView { LazyVStack(spacing: 8) }` — `CourseScoreResultViewBody.swift:21` | `LazyColumn(spacedBy(8.dp), contentPadding = 16.dp)` — `CourseScoreResultView.kt:216-219` | yes |
| Library floor/book grid | *(no equivalent found)* | `LazyVerticalGrid(columns = GridCells.Fixed(4))` — `LibraryBookView.kt:387-389` | unmeasured |

Counts: Android has **17** `LazyColumn(` call sites and **3** files using any `Grid`/`StaggeredGrid`;
iOS has **10** `LazyVStack` and **0** `LazyHStack`, **6** `LazyVGrid` files (all small pickers), and ≈**2**
real SwiftUI `List`s — both chosen because `swipeActions` only exists on `List` rows
(`CourseCenterSelfWantPageView.swift:35-36`).

Dividers:

| List | iOS | Android | Agree? |
| --- | --- | --- | --- |
| User-center settings rows | `Divider()` after each of 5 rows — `UserCenterView.swift:45,47,49,51,53` | `HamDivider()` in the same 5 positions — `UserCenterMainView.kt:138,145,152,159,166` | yes |
| Schedule list | **no dividers** — `Spacer().frame(height: 12)`, one lone `Divider()` between  upcoming and past — `ScheduleView.swift:386-396` | **no dividers** — `Spacer(Modifier.height(16.dp))` — `ScheduleMainViewItemListView.kt:164` | yes in kind, **12 vs 16** in value |
| CourseScore result / comments | no dividers, `spacing: 8` | no dividers, `spacedBy(8.dp)` | yes |
| Course-center want / comment / rank | `Divider()` between rows, `listRowSeparator(.hidden)` — `CourseCenterSelfWantPageView.swift:59,72` | `HamDivider()` — `CourseCenterWantCardView.kt:53,59` | yes |
| Form / editor option rows | `Divider()` — `ScheduleInsertGroupView.swift:43,65`, `ScheduleInsertAlarmView.swift:48` | `HamDivider()` — `InsertEditSelectGroupView.kt:92,127`, `InsertEditSelectAlarmView.kt:71` | yes |
| Divider component | **none** — **76 raw `Divider()`** call sites, no shared thickness or colour | **`HamDivider`** — `Divider.kt:17-24`, **66** call sites, plus `HamDividerVertical` `:26-33` | **no** |

Rules:

1. **A grid is for a bounded, fixed set of tiles. A list is for unbounded data.** That is the
   rule the code already follows: grids appear only for the cloud-configured function launcher,
   the library 4-column floor grid, search-history chips and small colour/time pickers.
2. **Anything that can exceed ~30 items is lazy.** iOS's score screen uses
   `ScrollView { VStack }` where Android uses `LazyColumn`; iOS's status, library and sport
   homes are non-lazy card columns too. Make them lazy.
3. **The timetable is the documented exception.** Neither platform uses a lazy grid for the
   5-or-7 × 13 course grid; both compute cell size and place items with `offset`. Keep it, and
   keep it in one file per platform.
4. **Use the platform's real grid for the function launcher.** iOS hand-lays `HStack` rows
   inside a horizontal `ScrollView` from the same cloud config that Android feeds to
   `LazyHorizontalStaggeredGrid`. One of them is a reimplementation of the other.
5. **Cards are separated by spacing, rows by a divider.** 8 between cards in a column (both
   agree today); a hairline divider between the rows of a settings or option list (both agree
   today). Do not put a divider between cards, and do not separate settings rows with padding.
6. **Schedule rows are 12 apart on both.** iOS ships 12; Android ships 16. The row already has
   its own padding; 12 is the value.
7. **Ship a divider component on iOS.** 76 raw `Divider()` sites with no shared thickness or
   colour is 76 chances to drift; Android's `HamDivider` is the model.

---

## 5. Shared flows

Four sequences recur across modules. They describe **order and state transitions**; the visual
spec for each screen involved lives in [`screens.md`](screens.md).

### 5.1 CAS verification gate

Several modules sit behind the university's 信息门户 (CAS) single sign-on. The pattern is
identical everywhere:

1. User enters a module without a valid session.
2. After a short delay (0.3s), an intro sheet slides up: module icon, title, a one-line
   explanation, and a 登录 button.
3. Tapping opens the CAS web login. On iOS the module often presents its own sheet; Android
   navigates an internal graph.
4. Terminal states: a spinner, then 验证成功 (auto-dismiss) or 验证失败 with 重新登录.
5. Dismissing without logging in pops back to where the user came from.

**Colours:** the intro uses the module's brand colour. Failure states use `feedback.error`.
**Screen spec:** [Intro / connect screen](screens.md#intro--connect-screen-连接页).

### 5.2 Captcha

The education and sport flows route a CAPTCHA through a bundled local HTML page
(`education-captcha-page.html`, `sport-captcha-page.html`) and receive the token back through a
platform bridge. Include a 刷新 action. **Screen spec:** [Webview screens](screens.md#webview-screens-网页视图--platforms-both).

### 5.3 Sign-in

A shared login screen offers the available providers. Brand-coloured social buttons use their
own colours, not the app palette. **Screen spec:** see the login screen under
[§10 Auth and sign-in](screens.md#10-auth-and-sign-in-登录与授权).

### 5.4 Web content

Remote-config banners and announcements render three ways: an in-app webview push, an external
browser, or a full-text view rendered from a string. Pick by the config entry's action type.
**Screen spec:** [Webview screens](screens.md#webview-screens-网页视图--platforms-both).

---

## 6. Platform rules

These **must** differ and are not defects:

| Area | iOS | Android |
| --- | --- | --- |
| Status bar / safe areas | safe-area insets | `statusBarsPadding()` |
| Back navigation | edge swipe | system back |
| Navigation chrome | SwiftUI `NavigationStack` — `Navigation.swift:22` | in-Compose header |
| Native controls | SwiftUI `Toggle` / `Slider` / `ProgressView` | Material3 |
| Icon set | SF Symbols | Material Icons |
| Icon size | **Follow the pairing in [§2.8](#28-iconography).** Sizes may differ between platforms because the drawings differ; within one app the same text role always gets the same icon size. | |
| System pickers | platform-supplied | platform-supplied |
| Widget configuration | none — WidgetKit owns refresh | in-app update-interval picker |
| Language | follows system locale | in-app locale picker |
| Type scaling | Dynamic Type | sp |
| Dark mode | asset catalogs | `values-night` |

Specify the **result** (content begins below a 42-tall header plus the status bar), not the
implementation.

---

## 7. Undecided

### Resolved

1. ~~Body 17 vs 16.~~ **Resolved: 17.** Text follows iOS, so the iOS type scale in
   [§2.3](#23-type) is normative, including `body` at 17.
2. ~~Chevron size.~~ **Resolved: caption-paired — 12 on iOS, 16 on Android.** Not a single
   cross-platform number: SF Symbols and Material Icons are different drawings, so an absolute
   icon size is not cross-platform normative. The *pairing* is — see
   [§2.8](#28-iconography).

### Still open

3. **Stage the type change?** Raising Android `headline` 14 → 17 is the largest reflow in the
   migration. Land non-type fixes first, or one pass with full visual review?
4. **Toast radius.** Spec says 8; the shipped iOS value is 0 (a plain rectangle, presumably an
   oversight).
5. **Elevation.** Spec says none, matching both platforms. Confirm we are not adding shadow as
   part of this work.
6. **Is a module's brand colour that module's accent?** [§2.2](#22-colour) says interactive
   elements take `accent` everywhere and lists brand at three places. Both clients actually
   tint a module's own screens with its brand colour: Android `brand.sport` at 38 sites
   including the booking CTA (`SportSelectFooterView.kt:82`) and the slot chips
   (`SportSelectItemTimeView.kt:28`); iOS the same result via raw `Color.green` at 21 sites.
   Two coherent answers — (a) **strict**: `accent` everywhere interactive, and the sport and
   score flows change colour; (b) **module accent**: inside its own module a brand colour may
   act as the accent, on every module, and the rule becomes "accent on shared surfaces — 状态,
   我的, standalone — brand inside the module". (b) matches what ships and preserves module
   identity; (a) is simpler and matches the contrast findings, since green-on-green-tint fails
   at 1.97–2.42:1. **Decide before applying [§2.2](#22-colour) to the sport and score screens.**
7. **Contrast floor.** [§2.10](#210-accessibility) now requires 4.5:1 body / 3:1 large. Confirm
   we are adopting WCAG AA as the bar, since meeting it changes `text.secondary`, retires
   `text.tertiary`'s 60% alpha, and needs a `brand.<module>.text` per module.
8. **Search: as-you-type, or on submit?** [§4.8](#48-search-filter-sort-pagination) forbids a
   request per keystroke, but permits either a 300 ms debounce or an explicit submit. iOS
   currently fetches on every character and Android only on submit, so the two clients are the
   two options. Pick one experience.
9. **Is the score-result filter shipping?** [§4.8](#48-search-filter-sort-pagination) rule 3 says
   a filter ships to everyone or to no one. Android gates it behind
   `enableCourseScoreResultFilter`; iOS does not gate it. Say which is true.
10. **Confirm the submit split in [§4.5](#45-forms-and-text-entry) rule 1** — bottom-of-form for
    create/edit, nav-bar trailing for editing one value in place. Both platforms currently ship
    both placements; this rule assigns them by form type rather than by screen.

---

## 8. Current divergences

What the two shipped clients do differently today. Each entry is a task, not a spec.

**This section lists app-wide defects.** Every screen in [§4](#4-screens) also carries its own
`Divergences` list, covering per-screen differences — together they are the full work queue.

### 8.1 Feature gaps

| Gap | Detail |
| --- | --- |
| ~~Print is Android-only~~ — **retracted** | Print ships on **both** clients and is near-identical. An earlier revision of this table said iOS had only an unreferenced data layer; that was wrong. iOS has `Route.libraryPrint` / `Route.printPrepare` (`Route.swift:33-34`), `LibraryPrintView.swift`, `PrintPrepareView.swift` and an E2E suite. The real divergences are: the share hint is two different strings, iOS picks the file behind an action sheet while Android opens the document picker directly, iOS replaces the printer list with `没有找到打印机` where Android appends it, and Android re-fetches the printer list on every recomposition. — [§12 Print](screens.md#12-print-打印--platforms-both) |
| ~~Sport status card is iOS-only~~ — **retracted** | Retracted 2026-09-24. `StatusViewCardType` **does** list `Sport` (`StatusViewCardScoreManager.kt:40`), `StatusView.kt:250-252` composes it, and `StatusSportCard.kt` is 212 lines of real UI with an order card, area number and pay row. An earlier survey said otherwise; it was wrong. |
| ~~Android schedule status card is an empty stub~~ — **retracted** | Retracted 2026-09-24. `ScheduleCard.kt` is called at `StatusView.kt:247` and is fed by `ScheduleCardViewModel` (Realm). The file's own header comment still says "until now this was an empty container" — the container has since been filled, and the stale comment is what the earlier survey read. |
| **Android renders all 7 status cards** | Both platforms render the same seven: library, weather, course, bus, schedule, sport, plus the CAS alert card (`CasErrorCardView()` composed above the card-order loop, `StatusView.kt:227`). The earlier "5 of 7, no sport, no schedule" was wrong on both counts. What actually differs is the card *container* — radius and padding — which every card inherits, not which cards exist. |
| Android weather card shows no data-source attribution | iOS links Apple's required WeatherKit attribution. Android fetches CMA data with a spoofed browser UA and displays no credit. |
| RN bundles are 6 commits apart | Android at `4f3d241`, iOS at `0939555`; one of the six is a bug fix. |
| Language and widget settings are Android-only | May be correct as-is — see platform rules. |

### 8.2 Colour wiring (Android)

| Defect | Location |
| --- | --- |
| `ham_brand_sport` / `ham_brand_score` resolve to Material colours (#4CAF50 / #FF9800) while the correct hexes sit unused in `colors.xml` | `Color.kt:79,82` vs `colors.xml:73,75` |
| `ham_text_secondary` is hardcoded Compose `Gray`, not a resource — no dark variant | `Color.kt:29` |
| **27 colour resources have no `values-night` entry** — 24 in `core/ui`, 3 in `app`. 22 are deliberately fixed; `red` and `green` are not, and 5 code references render a light-mode value at night, including the error toast background | `Color.kt:61,64`, `ToastManager.kt:123`, `Color+Glance.kt:50,66` — see [§2.2](#22-colour) |
| 退出登录 renders #FF0000 in one place and #F44336 in another for the identical string | `SyncLogoutView.kt:49` vs `UserCenterMainView.kt:193` |
| `<color name="link">` defined and referenced by zero Compose files | `colors.xml:14` |
| `dimens.xml` is dead and its values contradict the live font scale | `core/ui/.../values/dimens.xml` |

### 8.3 Type (Android)

| Defect | Location |
| --- | --- |
| `largeTitle` has zero call sites | `Font.kt:16` |
| `title3` is 16sp, identical to `body`, so `body`/`bodyBold`/`title3` are three names for one size | `Font.kt:22` (`title3` = `16.sp` at `:48`, `body` = `16.sp` at `:49`) |
| `callout` (16), `subheadline` (15) and `footnote` (13) are specified in [§2.3](#23-type) but **do not exist on Android** — `grep -rn "footnote\|callout\|subheadline" --include='*.kt'` returns zero. `HamFontSize` has 8 members; the scale has 13 | `Font.kt:44-52` |
| Six tokens declare no `color`, so text using them renders uncoloured rather than in `text.primary` | `Font.kt:16,18,20,22,30,32` |
| `headline`, `caption`, `caption2` declare no `fontWeight` | `Font.kt:30,34,38` |
| `MaterialTheme.typography` — the only three `lineHeight` declarations — is referenced zero times | `HamTheme.kt:85-109` |
| 24 raw `fontSize = N.sp` sites bypass `HamFontStyle` | various |
| 71 `maxLines` sites vs 46 `TextOverflow.Ellipsis`, so ~25 truncate without an ellipsis — the cut is silent | various |
| The same seat number renders at 24sp, 32sp, and 36sp | three files |

### 8.4 Type (iOS)

| Defect | Location |
| --- | --- |
| No font, spacing, or shadow token file exists | `Ham/shared/` |
| **131** raw `.font(.system(size:))` sites — of the 140 counted before the `main` update, 119 (85%) size an SF Symbol and belong in an icon scale, not a type scale; 19 size text, 2 were ambiguous | `grep -rn "\.font(\.system(size:" --include='*.swift' Ham` |
| Three de-facto secondary colours: `ham_text_t2Color` (113), raw `.gray` (38), raw `.secondary` (34, a different colour) | various |
| `Color.lightGray` #F0EFEF vs `Color.ham_lightGray` #EDEEEF — two greys under near-identical names | `Color+Ham.swift:45` vs `:31` |
| No tabular figures on the course period rail or sport clock times | two files |

### 8.5 Component and behaviour bugs

| Defect | Location |
| --- | --- |
| Fill uses radius 10 while the clip uses radius 16 — visible corner artefact | `CourseViewDetailCourseInfoView.swift:27` vs `:31` |
| Segmented thumb radius 5 ≠ track radius 6 | `HorizontalPicker.kt:57,70` |
| `SportSelectItemView.kt:45` is a verbatim clone of `HamCardView` | feature/sport |
| Two competing Android shared cards: `HamCardView` (r16) and `CommonStatusCard` (r12) | core/ui vs feature/status |
| `PrintSheet.kt` is an empty stub; `PrintStatusCard.kt` is never called | feature/print |
| Android's 在地图打开 is a no-op; iOS implements it | `ReservedCard.kt:91-98` |
| Android omits 添加到系统日历 where iOS has it commented out — inverted | library |
| iOS `AboutPrivacyView` renders a markdown link as unstyled, untappable text | `Ham/iOS/ui/about/` |
| Android duplicates the markdown-link parser verbatim in two files | `LoginView.kt`, `AboutView.kt` |
| `enabled = false` blocks the click but not `awaitFirstDown`, swallowing the touch | `Button.kt:79` (down) vs `:85` (the `enabled` gate) |
| `CourseThemeSelectView.kt:125` and `:171` pass `enabled = !selected` while a ternary sets the colour — no visual effect | feature/course |
| Android silently removes the sport current-order block on error — no inline error or retry | `SportMainViewCurrentOrderCard.kt:60` |
| Neither platform has a loading state on the sport main screen, though iOS's view model tracks one | `SportMainViewModel.swift:19` |
| 21 Android and 27 iOS hand-rolled cards bypass the shared primitive | various |
| Android has no shared section-header component; iOS has it in one screen only | — |
| No minimum tap target enforced: Android's smallest is 16×16dp, iOS's ~14×14pt | various |
| iOS fires a gRPC `queryCourse` on **every keystroke** — no debounce | `CourseScoreSearchViewSearchBar.swift:35-37` |
| Android gates the score-result filter behind an A/B flag; iOS does not | `CourseScoreSearchViewModel.kt:100-102` |
| `searchResultLoadState` is computed on both platforms and never rendered as a footer | `CourseScoreResultViewModel.swift:16,85`, `CourseScoreResultViewBody.swift:20-37`, `CourseScoreSearchViewModel.kt:119,195-219`, `CourseScoreResultView.kt:216-227` |
| iOS's shared `TextEdit` has **zero** call sites; 14 hand-built `TextField`s | `iOS/ui/common/compoment/TextEdit.swift:10` |
| Android's `HamTextField` has no error slot, and the flagship schedule editor bypasses it | `TextField.kt:37`, `InsertEditHomeView.kt:146-165` |
| Android's shared scroll container `BounceScrollView` (8 instantiation sites, 2 of them the shells that host 54 screens) overscroll-bounces without refreshing | `NavigationView.kt:95`, `HomeContainer.kt:67` |
| Pull-to-refresh thresholds differ >2× — iOS 128 pt hand-rolled, Android 300 px native | `StatusUpdateView.swift:15`, `StatusContainerView.kt:157` |
| Android's status dashboard starts empty and never shows a persisted snapshot; iOS does | `CourseCardViewModel.kt:76-84` |
| iOS writes **76** raw `Divider()`; Android has a shared `HamDivider` at 66 sites | iOS various / `Divider.kt:17-24` |
| Schedule row spacing is 12 on iOS and 16 on Android | `ScheduleView.swift:388,395` / `ScheduleMainViewItemListView.kt:164` |
| iOS's score screen is `ScrollView { VStack }` where Android's is `LazyColumn` | `ScoreMainView.swift:18-20` / `ScoreMainView.kt:105` |
| iOS hand-lays the function-launcher grid; Android uses `LazyHorizontalStaggeredGrid` from the same cloud config | `MyViewFunctionCard.swift:49-67` / `MyViewFunctionComponentView.kt:201-205` |
| The deactivate-account disclosure offers 确定 with **no cancel**, on both platforms | `SyncLogoutView.swift:38-60`, `SyncLogoutView.kt:88-102` |
| **No logout anywhere confirms** — 5 entry points, 0 confirmations | [§4.6](#46-confirmation-and-destructive-actions) |
| Print's options form uses literal `Color.blue`, not the library brand token, because the share extension cannot link `Color+Ham.swift` | `shared/ui/print/PrintOptionsForm.swift:15-16` / `LibraryPrintView.swift:72` |

---

## 9. Review checklist

- [ ] Padding, radius, font, and colour all come from a token here.
- [ ] Interactive elements (links, buttons, selected state, toggles) use `accent`, not the
      module brand colour — unless the element is one of the three brand places in
      [§2.2](#22-colour).
- [ ] User-visible text follows [`copy-and-strings.md`](copy-and-strings.md) — approved
      terminology, no hardcoded strings, keys in the shared namespace.
- [ ] Every colour has a light and a dark value and is an adaptive resource.
- [ ] Text styles are explicit — no relying on an inherited default size.
- [ ] The screen is **presented** correctly — push by default, sheet only for one of the five
      cases in [§4.4](#44-navigation-presentation).
- [ ] The header is **42** — not 36 ([§4.1](#41-what-every-screen-has)).
- [ ] Every icon has a label, or is explicitly decorative — [§2.10](#210-accessibility).
- [ ] Duration and easing come from [§2.9](#29-motion); reduced motion is honoured.
- [ ] Loading, empty, error and offline are all handled — [§3.10](#310-non-content-states).
- [ ] A field error appears **on the field**, not only in a toast —
      [§4.5](#45-forms-and-text-entry).
- [ ] Destructive actions confirm, or are one of the reversible exemptions in
      [§4.6](#46-confirmation-and-destructive-actions).
- [ ] Lists render from local state first, revalidate in the background, and **keep the stale
      value on failure** — [§4.7](#47-data-loading-and-caching).
- [ ] Search input is debounced or submits explicitly — no request per keystroke.
- [ ] Paging renders a trailing loading row, and a marker when the list ends —
      [§4.8](#48-search-filter-sort-pagination).
- [ ] A list that can exceed ~30 items is lazy, and a grid is only for a bounded tile set —
      [§4.9](#49-lists-grids-and-card-columns).
- [ ] Dividers come from a component with a token thickness and colour, not a raw `Divider()`.
- [ ] Interactive elements meet the 44 minimum tap target.
- [ ] `maxLines` is paired with `Ellipsis` on Android.
- [ ] The same screen exists on the other platform, or the divergence is listed in
      [§6](#6-platform-rules) or [§8](#8-current-divergences).
- [ ] Verified in light and dark mode.
- [ ] Verified with a populated account — empty states hide spacing and type differences.

---

## 10. Status and precedence

### 10.1 Precedence

Two sentences in this set looked like opposite rules. They are not, and here is the resolution:

- `design-system.md` opens with "Values are **measured** from the existing code." That describes
  *where a number came from*.
- `screens.md` opens with "Status: **proposed** — the values here are the target, not
  necessarily what ships today." That describes *what the number means*.

**When the code and this specification disagree, this specification wins.** The shipped value is
evidence; the `normative` column is the requirement. A disagreement is not a reason to change the
spec — it is a work item, and it belongs in [`ui-parity.md`](ui-parity.md) or
[`logic-parity.md`](logic-parity.md) until the code is fixed. Where a screen's `iOS` / `Android`
columns differ from its `normative` column, the platform columns are the evidence and `normative`
is what to build.

Corollary: do **not** "correct" a token to match the code. If §2.3 says `body` is 17 and Android
ships 16, Android has a bug.

### 10.2 Status

All five documents are **proposed**. They carry no version number, so **cite by section, not by
version** — "§2.3 Type" — until a versioning scheme is agreed. `proposed` means:

- nothing here is enforced by a linter, a test, or a code owner;
- a value is *settled* when both clients ship it, not when it is written down;
- [`ui-parity.md`](ui-parity.md) and [`logic-parity.md`](logic-parity.md) are the open work
  against this spec, and shrink as items land.

### 10.3 Changing a value

A token change touches four places. Change all four in the same commit, or the documents drift
from each other the way the two clients did:

1. the token or component section here (§2 or §3);
2. every `screens.md` row that cites that token — use the token, never a literal;
3. the corresponding row in `ui-parity.md` — and **delete** it once both clients match;
4. `logic-parity.md` if the change is behavioural rather than visual.

### 10.4 Still not decided

Recorded here rather than invented, because they need an owner:

| Question | Why it is open |
| --- | --- |
| Who owns this specification? | No named owner or reviewer. `logic-parity.md` §12 has ~30 rows of "needs a product decision" with no decider. |
| How is it versioned? | No scheme. Needed before it can be cited in code review. |
| What happens to a retired token or screen? | No deprecation rule. §8 lists dead code (`PrintStatusCard.kt`, `dimens.xml`) with no record of removal once done. Two entries here were **retracted** after a re-check proved the code was live — see §8.1 — so the list itself needs re-verification before anyone acts on it. |
| How is a new screen added? | `screens.md` documents the eight fields of a section but not how to propose one or who measures it. |
| What settles a §7 open question? | No escalation path. |
