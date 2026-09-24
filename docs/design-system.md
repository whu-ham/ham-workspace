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

Plus, optionally, the module's card watermark. That is the whole list — verified across both
platforms, where each brand token has between 2 and 35 use sites, nearly all in those places.

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
about **91** on Android; they are the single largest source of "these two apps feel different".

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
| Bare, unspecified animations | **352** `withAnimation {}` sites in 132 files | ~91 `AnimatedVisibility` / `AnimatedContent` / `animate*AsState` sites |
| Page push | **No duration written anywhere** — `Navigation.swift:91` wraps the push in a bare `withAnimation` | 400 ms in, 300 ms out — `NavHost.kt:45,55`, duplicated across two overloads |
| Sheet | `.sheet`, no detents, no drag indicator, no duration (0 uses of `presentationDetents`) | custom `HamSheet`: 85 % height, radius 24, scrim `0.5 × dragProgress`, quarter-height dismiss, duration unspecified |
| Toast dwell | **3.3 s** — `ToastView.swift:37` | **2.0 s** — `ToastManager.kt:150` |
| Toast exit | Slides *and* fades | **Fades only** — `ToastManager.kt:154` |
| Tab switch | `$tab.animation()`, unspecified, **plus a heavy haptic** | `fadeIn() togetherWith fadeOut()`, unspecified |
| Button press | none | **whole button dims to alpha 0.25** — `Button.kt:48` |
| Loading → content | hard swap at 51 `ProgressView` sites | crossfade via `AnimatedContent`, spec almost never given |
| List item | none | `.animateItem()` in one file only |
| Pull-to-refresh | **none** — 0 uses of `.refreshable` | one screen only — `StatusContainerView.kt:157` |
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
| Accessibility modifiers, **production-visible** | **4** — of 8 in the app; 5 of the 8 are `#if HAM_E2E` no-ops | 19 non-null of 290 `contentDescription` |
| Hints | **0** | n/a |
| Row / group merging | **0** `accessibilityElement(children:)` | **1** `mergeDescendants` — inside `HamButton` only, `Button.kt:66` |
| State semantics | 1 `accessibilityAddTraits`, 1 `accessibilityValue` | **0** `toggleable`, **0** `selectable` |
| Announcements | **0** `UIAccessibility.post` | **0** `liveRegion`, **0** `announceForAccessibility` |
| Test identifiers | **4** `accessibilityIdentifier` | **7** `testTag` |
| Touch-target enforcement | none | **0** `minimumInteractiveComponentSize` |
| Reduced motion | not checked | not checked |

iOS has **4** production-visible accessibility modifiers against **236** icons. `PrintPrepareView.swift:200,201,228`
is the only file in the app with more than one — it is the only existing pattern worth copying.

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
matters most. `HamButton` supplies **zero** padding (`Button.kt:101`), so a button wrapping a bare
icon inherits the icon's own size — which is why the back button is a 24 dp target.

#### Contrast

| Pair | Ratio | Verdict |
| --- | --- | --- |
| Android `text.secondary` on `surface.secondary` (`#F9F9F9`) | **3.37:1** | below AA 4.5:1 for body text |
| Android `text.secondary` on `surface.primary` (white) | **3.54:1** | below AA |

Two causes, and the second is the one that will not be fixed by editing a colour file:

1. `gray` is `#888888` in **both** `values/colors.xml:24` and `values-night/colors.xml:13`, so
   the dark variant is the same grey — the resource is adaptive in name only.
2. The Compose accessor **bypasses the resource entirely**. `Color.kt:28-29` is
   `get() = Gray` — a compile-time constant — whereas `ham_text_primary` directly above it uses
   `colorResource(...)`, which *is* adaptive. Every call site that writes
   `Color.ham_text_secondary` (`Card.kt:83`, `TextField.kt:78`) therefore cannot respond to dark
   mode even after the XML is corrected.

Fix both: make the accessor `colorResource`, and give the night value its own hex.

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

#### Current state

| | iOS | Android |
| --- | --- | --- |
| Shared loading component | **none** — stock `ProgressView()` repeated at ~40 sites | `HamLoadingProgressBar` — `core/ui/…/component/HamLoadingProgressBar.kt:20`, 61 sites, plus a blocking `LoadingModalView` |
| Skeleton | **one** — `.redacted` seat grid, `LibrarySelectSeatView.swift:368` | **none anywhere** |
| Shared empty component | **absent** | **absent** |
| Shared error component | `ErrorView` — `iOS/ui/common/view/ErrorView.swift:8`; default action **返回** `:22` | `ErrorView` — `core/ui/…/intro/ErrorView.kt:48`; default action **完成** `:60` |
| **Retry in the shared error component** | **no** | **no** |
| Connectivity monitoring | one `NetworkReachabilityManager`, refreshes remote config only — **never drives UI** | **none** — 0 hits for `ConnectivityManager` / `NetworkMonitor` / `isOnline` |
| Session-expired handling | `AuthPbInterceptor.swift:63` — `send` and `errorCaught` are pass-throughs, **no 401 handling** | no auth interceptor; `PbRequestHelper.kt:178,205` only add headers |
| Rate-limit state | none | none |

Three consequences worth stating plainly:

1. **The dominant error pattern on Android is a toast with no retry** — ~70 `ToastManager` call
   sites against 13 `ErrorView` sites. `ToastManager.kt:247` hardcodes 网络异常，请稍后重试 —
   text that promises a retry the UI does not offer.
2. **Neither app knows it is offline.** The user sees a generic failure, not "you are offline".
3. **Session expiry is indistinguishable from a network error** on both platforms, so the user
   is told to retry something that cannot succeed.

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
| **Push** — iOS `NavigationLink` / `router.push`, Android `navController.navigate` | The user tapped **content**: a row, a card, a list item, a function-grid tile. The new screen is a *place*. | iOS 68 `NavigationLink(` + 9 `router.push(`; Android 131 `navController.navigate(` |
| **Sheet** — iOS `.sheet`, Android `HamSheet` | The task is **about the current screen** and returns to it. Five cases: (1) a module intro / CAS gate ([§5.1](#51-cas-verification-gate)); (2) a login-provider webview; (3) an app-level flow launched from the root — pay, SSO authorization, changelog; (4) a transient picker — colour, alarm permission; (5) a terminal success or result state. | iOS 21 `.sheet(`; Android 13 `HamSheet(` call sites. The two lists match case for case. |
| **Full-screen takeover** — iOS `.fullScreenCover`, Android a separate `Activity` | The task **brings its own chrome** and must not be dismissed by dragging down: image crop, a document webview (privacy policy), receiving a shared file (print), social-account linking. | iOS 4 `.fullScreenCover(` (`CourseSettingViewUISection.swift:157`, `LoginView.swift:158`, `SyncSocialAccountView.swift:69,84`); Android 4 activities (`ImageCropActivity`, `PrivacyActivity`, `PrintShareFileActivity`, `MainActivity`) |
| **Overlay** — `.overlay` / `Box` | **Decoration and in-place chrome only** — watermarks, badges, counts, a scrim. Never a whole screen. | iOS 52 `.overlay(` |

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

*Pending measurement — see `/tmp/audit2/patterns.md`.*

### 4.6 Confirmation and destructive actions

*Pending measurement.*

### 4.7 Data loading and caching

*Pending measurement.*

### 4.8 Search, filter, sort, pagination

*Pending measurement.*

### 4.9 Lists, grids, and card columns

*Pending measurement.*

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

---

## 8. Current divergences

What the two shipped clients do differently today. Each entry is a task, not a spec.

**This section lists app-wide defects.** Every screen in [§4](#4-screens) also carries its own
`Divergences` list, covering per-screen differences — together they are the full work queue.

### 8.1 Feature gaps

| Gap | Detail |
| --- | --- |
| ~~Print is Android-only~~ — **retracted** | Print ships on **both** clients and is near-identical. An earlier revision of this table said iOS had only an unreferenced data layer; that was wrong. iOS has `Route.libraryPrint` / `Route.printPrepare` (`Route.swift:33-34`), `LibraryPrintView.swift`, `PrintPrepareView.swift` and an E2E suite. The real divergences are: the share hint is two different strings, iOS picks the file behind an action sheet while Android opens the document picker directly, iOS replaces the printer list with `没有找到打印机` where Android appends it, and Android re-fetches the printer list on every recomposition. — [§12 Print](screens.md#12-print-打印--platforms-both) |
| Sport status card is iOS-only | Android's `StatusViewCardType` lists no `Sport`. |
| Android schedule status card is an empty stub | `ScheduleCard.kt` has zero call sites; iOS renders one. |
| Android weather card shows no data-source attribution | iOS links Apple's required WeatherKit attribution. Android fetches CMA data with a spoofed browser UA and displays no credit. |
| RN bundles are 6 commits apart | Android at `4f3d241`, iOS at `0939555`; one of the six is a bug fix. |
| Language and widget settings are Android-only | May be correct as-is — see platform rules. |

### 8.2 Colour wiring (Android)

| Defect | Location |
| --- | --- |
| `ham_brand_sport` / `ham_brand_score` resolve to Material colours (#4CAF50 / #FF9800) while the correct hexes sit unused in `colors.xml` | `Color.kt:79,82` vs `colors.xml:73,75` |
| `ham_text_secondary` is hardcoded Compose `Gray`, not a resource — no dark variant | `Color.kt:29` |
| 退出登录 renders #FF0000 in one place and #F44336 in another for the identical string | `SyncLogoutView.kt:49` vs `UserCenterMainView.kt:183` |
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
| 140 raw `.font(.system(size:))` sites — 119 (85%) size an SF Symbol and belong in an icon scale, not a type scale; 19 size text, 2 are ambiguous | `grep -rn "\.font(\.system(size:" --include='*.swift' Ham` |
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
| `enabled = false` blocks the click but not `awaitFirstDown`, swallowing the touch | `Button.kt:65` |
| `CourseThemeSelectView.kt:124` passes `enabled = !selected` while a ternary sets the colour — no visual effect | feature/course |
| Android silently removes the sport current-order block on error — no inline error or retry | `SportMainViewCurrentOrderCard.kt:60` |
| Neither platform has a loading state on the sport main screen, though iOS's view model tracks one | `SportMainViewModel.swift:19` |
| 21 Android and 27 iOS hand-rolled cards bypass the shared primitive | various |
| Android has no shared section-header component; iOS has it in one screen only | — |
| No minimum tap target enforced: Android's smallest is 16×16dp, iOS's ~14×14pt | various |

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
| What happens to a retired token or screen? | No deprecation rule. §8 lists dead code (`PrintStatusCard.kt`, `ScheduleCard.kt`, `dimens.xml`) with no record of removal once done. |
| How is a new screen added? | `screens.md` documents the eight fields of a section but not how to propose one or who measures it. |
| What settles a §7 open question? | No escalation path. |
