# Ham design system

How to build the Ham native apps. This is a specification, not a comparison — read it as
"build it this way". Values are measured from the existing code; where the two clients
disagreed, **iOS is the baseline**.

Three documents, used together:

| Document | Covers |
| --- | --- |
| **[`design-system.md`](design-system.md)** (this one) | Tokens and components — how things look |
| **[`screens.md`](screens.md)** | Every screen, both platforms — 129 screens with per-element values |
| **[`copy-and-strings.md`](copy-and-strings.md)** | Terminology, copy rules, string key naming — what things say |

A UI change usually needs all three.

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
  [shared](screens.md#11-shared-components-共享组件)
- [5. Shared flows](#5-shared-flows)
- [6. Platform rules](#6-platform-rules)
- [7. Undecided](#7-undecided)
- [8. Current divergences](#8-current-divergences)
- [9. Review checklist](#9-review-checklist)

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
│ header / nav bar          42 + status bar   │  ← Android renders in-Compose; iOS uses UINavigationController
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

#### Rules

1. **Every icon is explicitly sized.** Never inherit a default.
2. **An icon paired with text uses the pairing table**, not a hand-picked number.
3. **Chevrons are caption-paired** — 12 on iOS, 16 on Android. Not 8, not the Material default.
4. **Watermarks render at a fixed frame size**, not a font size. iOS renders an SF Symbol's ink
   smaller than its box while Android fills the box, so a font size and a frame size are not
   interchangeable. Watermarks sit at alpha **0.12**, anchored **bottom-trailing** at offset
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

## 3. Components

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
| watermark | `icon.watermark` 128 @ 0.12, bottom-trailing (16, 16), clipped |

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
| selected | brand @ 0.10 background, brand text |
| unselected | `tint.muted` — `text.secondary` @ 0.10 background, `text.secondary` text |
| font | 12 / Bold |

Status badge (the coloured bar or dot beside a value): 6 × 6, radius 3, in the entity's colour.

### 3.6 Controls

| Control | Specification |
| --- | --- |
| Switch | Native on each platform. Android: checked track = `brand.sport`, unchecked track = `surface.tertiary`. |
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

---

## 4. Screens

The per-screen specification lives in **[`screens.md`](screens.md)** — 129 screens across 12
modules, each with a layout diagram, ordered content blocks, a value table, its strings, and
its states.

This chapter covers only what applies to every screen.

### 4.1 What every screen has

| Element | Rule |
| --- | --- |
| Screen horizontal margin | **16** |
| Gap between cards | **8** |
| Screen background | `surface.primary` |
| Bottom spacing — tab-root screen | system navigation-bar inset **+ 80** |
| Bottom spacing — pushed/child screen | system navigation-bar inset **+ 24** |
| Top | content begins below a 42-tall header plus the status bar |

Bottom spacing is computed from the live system inset, never hardcoded.

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
| Print | Android | iOS ships an unreferenced data layer — see [§8](#8-current-divergences) |
| Sport status card | iOS | Android shows 5 cards where iOS shows 7 |
| Settings hub, language, widget settings | Android | iOS has no equivalent entry point |
| Share sheet, Siri shortcuts | iOS | Android has no equivalent |
| Bulletin list | iOS | Android's banner links straight to the detail |

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
platform bridge. Include a 刷新 action. **Screen spec:** [Webview screens](screens.md#webview-screens).

### 5.3 Sign-in

A shared login screen offers the available providers. Brand-coloured social buttons use their
own colours, not the app palette. **Screen spec:** see the login screen under
[§10 Auth and sign-in](screens.md#10-auth-and-sign-in-登录与授权).

### 5.4 Web content

Remote-config banners and announcements render three ways: an in-app webview push, an external
browser, or a full-text view rendered from a string. Pick by the config entry's action type.
**Screen spec:** [Webview screens](screens.md#webview-screens).

---

## 6. Platform rules

These **must** differ and are not defects:

| Area | iOS | Android |
| --- | --- | --- |
| Status bar / safe areas | safe-area insets | `statusBarsPadding()` |
| Back navigation | edge swipe | system back |
| Navigation chrome | `UINavigationController` | in-Compose header |
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
| Print is Android-only | iOS ships `Ham/shared/business/print/` — PrintApi, PrintCasClient, PrintRequestHelper, PrintService — compiled into the target but referenced by nothing. No route, no strings, no view. The data layer is complete; only UI is missing. |
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
| `title2` and `title3` are both 16sp, identical to `body` | `Font.kt:18,20` |
| `headline`, `caption`, `caption2` declare no `fontWeight` | `Font.kt:30,34,38` |
| Five tokens declare no `color`, so hero numbers render uncoloured | `Font.kt:16-32` |
| `MaterialTheme.typography` — the only three `lineHeight` declarations — is referenced zero times | `HamTheme.kt:85-109` |
| 24 raw `fontSize = N.sp` sites bypass `HamFontStyle` | various |
| 23 of 66 `maxLines` sites lack `TextOverflow.Ellipsis` | various |
| The same seat number renders at 24sp, 32sp, and 36sp | three files |

### 8.4 Type (iOS)

| Defect | Location |
| --- | --- |
| No font, spacing, or shadow token file exists | `Ham/shared/` |
| 151 raw `.font(.system(size:))` sites — 123 (81%) are SF Symbol icon sizing and belong in an icon scale, not a type scale | various |
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
- [ ] Interactive elements meet the 44 minimum tap target.
- [ ] `maxLines` is paired with `Ellipsis` on Android.
- [ ] The same screen exists on the other platform, or the divergence is listed in
      [§6](#6-platform-rules) or [§8](#8-current-divergences).
- [ ] Verified in light and dark mode.
- [ ] Verified with a populated account — empty states hide spacing and type differences.
