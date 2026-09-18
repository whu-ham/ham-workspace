# Ham design system

How to build the Ham native apps. This is a specification, not a comparison — read it as
"build it this way". Values are measured from the existing code; where the two clients
disagreed, **iOS is the baseline**.

This document covers how things look. **[`copy-and-strings.md`](copy-and-strings.md)** covers
what they say — terminology, copy rules, and string key naming. They are companion documents;
a UI change usually needs both.

Current divergences between the shipped clients are recorded in
[§8 Current divergences](#8-current-divergences) at the end, so this document stays readable
as a spec.

## Contents

- [1. Overview](#1-overview)
- [2. Foundations](#2-foundations)
- [3. Components](#3-components)
- [4. Screens](#4-screens) — [status](#status-dashboard) · [course](#course-timetable) ·
  [schedule](#schedule) · [library](#library) · [sport](#sport) · [score](#score) ·
  [course score](#course-score) · [my &amp; user center](#my-and-user-center) ·
  [shared](#shared-screens)
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

| Token | Value | Used for |
| --- | --- | --- |
| `icon.xs` | 12 | Inline with caption text, chevrons |
| `icon.sm` | 20 | List-row leading glyph |
| `icon.md` | 24 | Status-card header, icon buttons |
| `icon.lg` | 32 | Small-card trailing decoration, area badges |
| `icon.xl` | 64 | Large tiles, empty states |
| `icon.hero` | 72 | Banner foreground |
| `icon.watermark` | 128 | Card background watermark |

Every icon must be explicitly sized. Watermarks render at a **fixed frame size** (not a font
size) at alpha **0.12**, anchored **bottom-trailing** at offset **(16, 16)**, clipped to the
card — iOS renders an SF Symbol's ink smaller than its box, Android fills the box, so a font
size and a frame size are not interchangeable.

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

Every screen in the app, in the order a user meets them. Each entry is prescriptive: build it
this way. Values are the normative ones from [§2](#2-foundations); where a platform currently
differs, the entry says so.

Screens are grouped by module. Modules are reached from the three tabs, mostly through the
function grid on 我的 ([§1.2](#12-navigation-hub-the-function-grid)).

---

### Status dashboard


**Layout:**
```
┌───────────────────────────────────────────┐
│ ↓ 64 (A)                                  │
│ 状态                                      │ A large title, 32 bold
│                                           │
│▓▓▓▓▓▓ daily photo, 300 tall, parallax ▓▓▓▓│ B
│                                           │
│ ┌───────────────────────────────────────┐ │
│ │ ⚠ 信息门户登录失败         [重新登录] │ │ C CAS — always first
│ └───────────────────────────────────────┘ │
│ ┌───────────────────────────────────────┐ │
│ │ ☀ 天气                            (D) │ │ D header band, brand @15%
│ ├───────────────────────────────────────┤ │
│ │ 28.5°  晴                          (E) │ │ E content, padding 16
│ │ 洪山区 · 14:32                         │ │
│ │ ┌───────────────────────────────────┐ │ │ F forecast, 5 rows
│ │ │ 今日 晴  ☀31.0°  ☾22.0°           │ │ │
│ └───────────────────────────────────────┘ │
│ ┌───────────────────────────────────────┐ │
│ │ ⬒ 图书馆                            › │ │
│ …课程 / 日程 / 校巴 / 运动, 8 apart      │ │
│                                           │
│░░░░░ bottom fade 85, bg_b1 @ 0.75 ░░░░░░░│ F
└───────────────────────────────────────────┘
```

**Page shell:**

| Element | Value |
| --- | --- |
| Large title | `状态`, **32 bold**, `ham_text_primary`, leading-aligned, `maxWidth: .infinity` |
| Title top inset | **status-bar height + 28** |
| Title → cards gap | **32** |
| Page background | `ham_bg_b1` — set explicitly on the root |
| Horizontal padding | **16**, applied once on the scroll column; cards never add their own |
| Daily photo (B) | **300** tall, `fillMaxWidth`, top-aligned behind the scroll view, `scaledToFill`/`Crop`, cross-fade 0.5 s, **not tappable** |
| Photo stretch / parallax | stretch `300 + max(0, -scrollY)` on overscroll; parallax `offset(y = min(0, -scrollY))` |
| Photo selection | one random pick per day, cached; keep the cached URL if it is still in the server list; wait **10 s** before swapping on first load of a session |
| Title colour | flip to white/black by photo luminance, animated |
| Collapse | continuous alpha ramp: `alpha = clamp01(1 + (scrollY - titleBottomY) / titleBottomY)`; collapsed bar = `ham_bg_b1 @ 0.7`, status-bar top padding, title at **title2 = 20 bold** |
| Pull-to-refresh | platform-native indicator; fires at **128** overscroll units; on fire call `update()` on every card VM; iOS additionally shows its pill (h 36, radius 18, bg `ham_bg_b2`, shadow black@0.1 r5 y5, `arrow.clockwise` w36, label `已请求刷新`, light haptic, auto-dismiss 2 s + 0.3 s fade) |
| Periodic refresh | every **2 min** while the page is visible, plus on foreground/appear |
| Bottom fade (F) | `ham_bg_b1 @ 0.75`, height **85**, pinned to the bottom |
| Bottom spacing | trailing spacer **64** + system navigation-bar inset |

⚠ iOS: title 34, gap 30, banner 200, refresh threshold 128 pt with the pill only, 96 pt spacer, fade is iOS<26-only, no periodic timer. Android: title 32, gap 42 (10+32), banner 300, threshold 300 px Material indicator, no bottom fade, no luminance adaptation, no foreground trigger, no page background set.

**Card container (all cards except CAS):**

| Element | Value |
| --- | --- |
| Corner radius | **16**, clipped |
| Background | `ham_bg_b2` |
| Header band fill | brand colour **@ 0.15 alpha**; foreground (icon, title, chevron) = brand colour |
| Header padding | **16 horizontal / 12 vertical** |
| Header icon | **24**, tinted brand |
| Icon → title gap | **4** |
| Header title | **16 bold** (`bodyBold`) |
| Chevron | `chevron.right` 24, tinted brand, min **44** hit target; rendered **only** when the card has a navigation destination |
| Content padding | **16** |
| Inter-card gap | **8** |
| Size animation | `animateContentSize()` on the card; cross-fade the whole list when order changes |

⚠ iOS: radius 16, header pad 12 uniform, content pad 12, icon unsized, title 17, list `VStack` with **no** spacing arg, no `animateContentSize`. Android: radius 12, content pad 16, header 16/12, icon 24, title 16, `spacedBy(8.dp)`.

**Brand colours:** library `#007AFF` · course `#1B5E20` · schedule `#01579B` · bus `#A2845E` · sport `#34C759` · weather `#FF9800` · CAS/error `#F44336`.
⚠ iOS uses stock `Color.blue/.brown/.green/.orange` instead of tokens for library, bus, sport, weather.

**Card ordering:**

1. Each card reports an integer score. **Score `< 0` hides the card.** Nothing else hides it.
2. Order = score **descending**. **Ties break by the card's default score** (table below), descending — deterministic, never dictionary order.
3. The CAS alert is **not scored**: it renders above the list whenever CAS is invalid.
4. Persist the order and restore it on next launch; recompute on every change.
5. Recompute through a debouncer: apply **immediately** if this is the first submission or >**10 s** since the last one, otherwise after **5 s** of quiet. Animate the reorder.

| Card | Default | Runtime scores |
| --- | --- | --- |
| casAlert | 999 (unscored, always first) | — |
| weather | 5 | **120** success during 06:00–09:00 / 13:00–14:00 / 16:30–18:30 · **100** no location permission and no cache · **60** fetch error and no cache · **10** success otherwise, or location error |
| library | 4 | **60** in-progress booking · **30** request error · **20** ended-today only · **−1** nothing to show |
| course | 3 | **50** week has courses · **−1** week empty |
| schedule | 0 | **60** has an upcoming item · **5** none |
| bus | 2 | **40** a bus is approaching the nearest stop · **−1** not logged in to CAS, or >20 km away |
| sport | 1 | **60** unpaid order · **30** paid/active order · **−1** no orders |

⚠ iOS: visibility is a separate `Set`, so a −1 score does **not** hide the card (library bug); ties are unordered; order is persisted. Android: −1 sentinel is honoured, ties fall back to enum ordinal, order is **not** persisted and `reset()` zeroes every score on each refresh; course and bus are never scored, weather is never scored on failure; no `sport`/`casAlert` enum cases.

---

#### CAS alert (信息门户登录失败) — `#F44336`

**Shows when:** CAS session is invalid. Renders above the scored list; never participates in ordering. Not a `CommonStatusCard`.

**Contents:**
1. Red container with a leading `HStack(spacing: 4)`: filled warning triangle + message text.
2. `Spacer`.
3. Trailing `重新登录` chip → CAS settings screen.

**Values:**

| Element | Value |
| --- | --- |
| Background | `#F44336` |
| Corner radius | 16 |
| Padding | 16 all sides |
| Icon | filled warning triangle, 24, `#FFFFFF`, gap 4 to the text |
| Message | `信息门户登录失败`, 16 bold, `#FFFFFF` |
| Action chip | `重新登录`, 16, padding h8 v4, fill `#FFFFFF @ 0.7`, radius 8, text `ham_text_primary` (**not** white) |
| Tap target | the chip only |
| Animation | none (it is outside the ordered list) |
| a11y | icon `contentDescription = null`; the adjacent text carries the label |

**Strings:** `信息门户登录失败` · `重新登录`

**States:** binary — absent when CAS is valid, present when invalid. No loading / error / empty.

⚠ iOS: radius 12, padding 8/8, chip text inherits the outer `.white` and renders white-on-white (defect); the pill is the only tap target. Android: radius 16, padding 16, one combined string `请重新登录信息门户`, 12 sp caption, whole card tappable.

---

#### Weather (天气) — `#FF9800`

**Shows when:** always (score ≥ 0 in every state). No chevron — the header band is not tappable.

**Contents:**
1. Header band: `sun.max.fill` + `天气`.
2. Red error banner (conditional): message, 1 line, + `重试` chip → re-fetch. Rendered **above** any stale content, both may show at once.
3. Permission state (conditional): `未获取地理位置权限` + `去授权` link, replacing the body.
4. Current conditions: temperature, condition text, `city · HH:mm` sub-line; inline spinner beside the temperature while refreshing.
5. Forecast container (conditional, non-empty list): **5 rows** of weekday / condition / ☀max / ☾min.
6. Attribution link, right-aligned.

**Values:**

| Element | Value |
| --- | --- |
| Temp | `28.5°` — one decimal, **36 rounded**, `ham_text_primary` |
| Condition | 16, `ham_text_primary` |
| Sub-line | `洪山区 · 14:32` — 12, `ham_text_secondary` |
| Brief-block spacing | 2; block start padding 2, bottom padding 12 |
| Error banner | `#F44336`, radius 8, padding h12 v8, bottom 12 |
| Error message | 16 bold, `#FFFFFF`, 1 line |
| Retry chip | `重试`, padding 6, `#FFFFFF @ 0.7`, radius 8 |
| Permission text | 12, `ham_text_secondary`; `去授权` 16 in `#007AFF`, padding top 4 |
| Forecast container | `ham_gray @ 0.1`, radius 8, padding h16 v12 |
| Forecast row gap | 12 |
| Forecast weekday / condition | 16; weekday padding end 8, condition padding start 8 |
| Forecast day / night icon | sun / moon, **12**, tinted `ham_text_primary`, gap 4 |
| Forecast temps | 16 bold; day↔night gap 8 |
| Attribution | ` Weather`, 11, `ham_text_secondary` → WeatherKit legal page |
| Spinner | 20, stroke 2, `ham_gray` |
| Refresh | every **5 min**; on resume if >1 h stale; on pull-to-refresh; cache the last response |

**Strings:** `天气` · `重试` · `未获取地理位置权限` · `去授权` · `获取地理位置遇到了错误` · `获取天气数据时遇到了异常` · `今日` · `昨日` · `明日` · `周%1$s` · `未知`

**States:** loading → spinner (full-body when no cache, inline beside the temperature when cached) · error → red banner + `重试`, stale data retained · permission missing → dedicated prompt · loaded → brief + forecast · empty → brief block omitted, attribution only. No skeleton/shimmer.

⚠ iOS: system `.orange` (not `#FF9800`), forecast padding v12, 5 min self-poll + cache, WeatherKit, attribution present, no `去授权` affordance (reuses the error banner), sub-line uses raw `subLocality` (defect renders `" · 14:32"` when nil). Android: `#FF9800`, forecast padding v8, no cache, weather.com.cn via AMap, **no attribution**, `-1` never reported on failure, `refreshWeatherInfo()`'s 30-min guard is dead code.

---

#### Library (图书馆) — `#007AFF`

**Shows when:** score ≥ 0, i.e. the user has a library token **and** at least one booking in `RESERVE`, `CHECK_IN`, `AWAY`, or one ended today. **One full card per booking**, 8 apart. Chevron → library home.

**Contents:**
1. Header band: `books.vertical.fill` + `图书馆` + chevron.
2. `%@离开` away line (only when `AWAY`).
3. Status block — two mutually exclusive branches:
   - `RESERVE`: begin time (brand blue, bold) + `开始` + countdown-to-start.
   - `CHECK_IN` / `AWAY`: `已学习%@` + progress bar + `begin` … `end`.
   - Ended today: `已结束` + `累计学习%lld分钟`, and nothing else.
4. Divider.
5. Seat block: seat number, location, `date begin-end`.
6. Right-aligned `变更预约` chip → modify-booking screen.

**Values:**

| Element | Value |
| --- | --- |
| Content spacing | 10 |
| Divider | 1 dp `#EDEEEF`, vertical padding 4 |
| Away line | `12:30离开` — 16, `ham_text_secondary` |
| Reserve begin time | **20 bold**, `#007AFF` |
| Reserve `开始` label | 20, `ham_text_primary` |
| Countdown | 12; **`#F44336` when ≤ 30 min to start**, else `ham_text_primary` |
| Countdown cadence | **1 s** |
| Countdown buckets | `< −60 s` → `超过%lld分钟` · `[−60, 0]` → `超过不到1分钟` · `(0, 60]` → `还有不到1分钟` · `(60, 3600)` → `剩余%lld分钟` (**round up**) · `≥ 3600` → `剩余%lld小时` |
| Studied line | `已学习%@`, 16 bold |
| Studied buckets | `< 60 s` → `不到1分钟` · else `%lld分钟` (**round up**) |
| Progress bar | height **4**, radius 2, fill **green when `CHECK_IN`, orange when `AWAY`**, value = elapsed ÷ (end − begin) clamped to [0, 1] |
| Progress labels | `begin` … `end`, 12 |
| Seat number | **24 bold**, `ham_text_primary` |
| Location / date | 12, `ham_text_primary` |
| `变更预约` chip | 12, `ham_text_secondary`, padding h4 v2, bg `ham_text_secondary @ 0.2`, radius **6** |
| Error title / body | `加载时遇到了错误` 16 bold + message 12 |
| Spinner | 20, stroke 2, `ham_gray` |
| Refresh | on resume and on pull-to-refresh; render the cached booking list before the first network call |

**Strings:** `图书馆` · `%@离开` · `开始` · `不久后` · `变更预约` · `加载时遇到了错误` · `超过%lld分钟` · `超过不到1分钟` · `还有不到1分钟` · `剩余%lld分钟` · `剩余%lld小时` · `已学习%@` · `%lld分钟` · `已结束` · `累计学习%lld分钟`

**States:** loading → spinner inside the card · error → error title + message (one silent retry on token expiry) · empty / no token → score −1, card removed · ended today → the ended variant · reserve / check-in / away as above. No skeleton/shimmer.

⚠ iOS: seat number 28, reserve time 22, has the ended state and the green/orange tint, `变更预约` is a grey chip, caches bookings, 1 s countdown. Android: seat number 24, reserve time 20, **no ended state** (finished bookings empty the list), `ham_blue` progress for both sub-states, `变更预约` is bare blue 16 sp text with no chip, no cache (blank first frame), countdown at **10 s** with floored minutes and a 2 h hour threshold, `加载时遇到错误` (missing 了).

---

#### Course (课程) — `#1B5E20`

**Shows when:** the current week has at least one course (score −1 otherwise). **No chevron** — the header band is not tappable.

**Contents:**
1. Header band: `tablecells.fill` + `课程`.
2. Header tip block — one of:
   - Upcoming today: `%@后上%@` + the focused course row.
   - In progress: `正在上%@` + course caption + in-class progress bar + `from` … `to`.
   - Tomorrow, first period: `明日早八` + row. Tomorrow otherwise: `明日第一节课将在%@后开始` + row.
   - Week empty: `下周无课程` (Saturday after 21:00) or `本周无课程`.
   - Week view, all done: `本周课程已上完`. Day view, nothing left: `今日无课，好好休息` / `明日无课，好好休息`.
3. Divider.
4. Week-grid progress strip (**week view only**).
5. Course rows for every course except the focused one.
6. `没有其它课程` when only the focused course remains — rendered **after** the rows.
7. Right-aligned view toggle (hidden when the week is empty).

**Values:**

| Element | Value |
| --- | --- |
| Body spacing | 10; header column 8; row list **5** |
| Divider | 1 dp `#EDEEEF`, vertical padding **8** |
| Tip primary text | 16 bold, `ham_text_primary` |
| Tip secondary / times | 12, `ham_text_primary` |
| In-class progress | height **4**, radius 2, fill = the course colour, value = elapsed ÷ (end − begin) **clamped to 1** |
| Time buckets | `> 120 min` → `%lld小时` · `≥ 1 min` → `%lld分钟` · else `不到1分钟` |
| Week strip | height **4**, radius 2, track `ham_gray @ 0.15`, bottom padding 8 |
| Strip cells | **13 periods × (5 or 7) weekdays** = 65 or 91; cell = the course colour, free slot = clear |
| Strip marker | `arrowtriangle.up.fill`, 12, `ham_gray @ 0.8`, x-offset −5, at `width × weekProgress` |
| Course row | rail **6** wide, radius 3, full height, fill = course colour (`ham_lightGray` when ended) · gap **4** · name 16 bold + desc 12 (`ham_text_secondary` when ended) |
| Desc format | `[周X ]N-M节 地点` — weekday prefix in week view only, location when non-empty |
| Toggle | `切换到日视图` / `切换到周视图`, 12, `#007AFF`, right-aligned; tapping refetches and re-ranks |
| "Today" cut-off | **21:00**; `showNextWeek` = Saturday after 21:00 |
| Refresh | every **15 s**, plus on the course-updated notification, on resume, and on toggle |

**Strings:** `课程` · `没有其它课程` · `切换到周视图` · `切换到日视图` · `%@后上%@` · `正在上%@` · `明日早八` · `明日第一节课将在%@后开始` · `下周无课程` · `本周无课程` · `本周课程已上完` · `今日无课，好好休息` · `明日无课，好好休息` · `%lld小时` · `%lld分钟` · `不到1分钟` · `%@节` · `周%1$s`

**States:** loading → none (render from cache; a cold first launch shows `本周无课程`) · error → none (degrade to the empty-week tip) · empty week → score −1, card removed · empty day → the "no class" tip, toggle still shown · no remaining rows → `没有其它课程`. No skeleton/shimmer.

⚠ iOS: row gap 6, body spacing 10, cut-off 21:00, refresh 15 s, has the week strip, hides the card on an empty week, toggle labels read the **current** view (`日视图` / `周视图`) and `没有其它课程` sits **above** the rows, `本周课程已上完` resolves to `本周的课程已经全部结束啦～辛苦啦！💪`. Android: row gap 4, list spacing 5, cut-off **21:30**, refresh 5 s, **no week strip** (`weekProgress` computed but never drawn), the card is never hidden or scored (always sorts last), toggle labels are action-oriented, `今日课程已上完` is day-scoped with no week-scoped equivalent.

---

#### Schedule (日程) — `#01579B`

**Shows when:** always (score 5 when there is nothing upcoming, 60 when there is). Chevron, hero row, each listed row and `其它日程` all → the schedule screen.

**Contents:**
1. Header band: `calendar` + `日程` + chevron.
2. Summary line — one of `本周待完成%lld项日程` (week list non-empty) / `待完成%lld项日程` (week empty, list non-empty) / `未添加日程`.
3. Hero block (only when there is a first item): name, begin time, location (when non-empty), big countdown number + unit, content preview (when non-empty), related-course rail + name (when non-empty).
4. Divider.
5. `无日程` when there is nothing further to list.
6. Up to **4** more rows (index 1…4; index 0 is the hero), each showing name + begin time + countdown.
7. Right-aligned `其它日程` link when there are ≥ 5 items.

**Values:**

| Element | Value |
| --- | --- |
| Content spacing | **12** |
| Divider | system hairline |
| Summary line | 16 bold |
| Hero name | 16 bold, `ham_text_primary` |
| Hero begin time / location | 12, `ham_text_secondary`; location 1 line |
| Hero countdown number | **48 bold**; unit 12 with `%@剩余` / `%@前`, y-offset 8 |
| Hero content preview | 12, 3 lines, padding top 8 |
| Related-course rail | 6 wide, radius 3, fill `getRandomColor(courseId)`; name 12 bold; gap 4; padding top 8 |
| Hero row padding | top 8 |
| Row countdown number | **28 bold**; unit 12, y-offset 4; `HStack(alignment: .top, spacing: 4)`; row padding vertical 4 |
| Countdown colour | **orange when expired**, else `#01579B` |
| `无日程` | 12, `ham_text_secondary` |
| `其它日程` | 12, right-aligned |
| Buckets | `≤ 1 min` → 1 分钟 · `< 120 min` → N 分钟 · `< 48 h` → N 小时 · `< 10000 d` → N 天 · else 10000 天 |
| Data | local store, items not yet finished, ascending by `begin`; week window = Monday + 7 days |
| Refresh | every **10 s** and on pull-to-refresh |

**Strings:** `日程` · `本周待完成%lld项日程` · `待完成%lld项日程` · `未添加日程` · `无日程` · `其它日程` · `%@剩余` · `%@前` · `分钟` · `小时` · `天`

**States:** loading → none (the local query is synchronous; render nothing until `inited`) · error → none · empty → `未添加日程` in the summary plus `无日程` in the body, no hero. No skeleton/shimmer.

⚠ Android ships only an unrendered `ScheduleCard` stub: a header band with an empty content lambda, zero call sites, no `Schedule` branch in the render `when`, no view-model, no chevron (it omits `clickAction`). iOS defects: `待完成%lld项日程` is formatted with `weekScheduleList.count` inside the branch that only runs when that list is empty, so it always reads `待完成0项日程`; `无日程` is gated on counts > 1, so it can appear alongside a populated hero.

---

#### Bus (校巴) — `#A2845E`

**Shows when:** logged in to CAS **and** the nearest stop is within **20 km**; score −1 otherwise. Hidden entirely when there is no stop to show. Chevron → the campus-bus screen.

**Contents:**
1. Header band: `bus` + `校巴` + chevron.
2. Red error banner (conditional): message + `重试`.
3. Spinner (conditional): full-body when there is no stop yet, inline beside the stop name while refreshing.
4. Stop block: nearest stop name + `距你%1$dm`.
5. Divider + line list — one row per line serving that stop, collapsed by default.
6. Collapsed row: line-name chip, `first → last` route, one bus glyph per approaching bus, arrival headline, status sub-line, expand chevron.
7. Expanded row: horizontal timeline — `当前` marker, then one column per bus.

**Values:**

| Element | Value |
| --- | --- |
| Content padding | **16** (the bus card is not exempt) |
| Stop name | 16 bold, `ham_text_primary`; column spacing 2 |
| Distance | `距你1234m`, 12, `ham_text_secondary` |
| Divider | 1 dp `#EDEEEF`, vertical padding 8 |
| Line list | spacing **8**, bottom padding 12 |
| Line-name chip | 12 bold, `#007AFF`, bg `#007AFF @ 0.2`, radius **4**, padding h8 v4 |
| Route text | `first → last`, 11, `ham_text_secondary`, gap 3 |
| Bus glyphs | one per approaching bus, **8**, `ham_gray`, gap 0 |
| Arrival headline | number at **28 rounded bold** + `站` 12 (y −3); or `已到站` / `即将到达` in the same style; or `未发车` at 16 rounded |
| Status sub-line | `当前到达%@站` / `当前正前往%@站`, 11, `ham_text_secondary` |
| Expanded columns | `当前` **56** wide, each bus **64** wide, gap 0, horizontal scroll with no indicators, padding h12 top 12 |
| Expanded marker | `person.fill` 16 in `#616161`, frame height 18, 12 spacer below |
| Expanded bus label | black pill, radius 4, padding 2, text 11 white; content `%lld站` / `已到站` / `即将到达` / `""` when the bus has passed |
| Expanded stop name | 11, `ham_gray`, max width 48 |
| Expanded track | height **4**, `#616161 @ 0.2`, radius 2, y-offset 22 |
| Expanded fade | opacity `(1 / (i + 1)) * 0.5 + 0.5` |
| Expand / collapse | `chevron.up` / `chevron.down`; animate the height; persist the expanded state across data refreshes |
| Error banner | `#F44336`, radius 8, padding h12 v8; message 16 white 1 line; `重试` chip padding 6, white @ 0.7, radius 8 |
| Spinner | 20, stroke 2, `ham_gray` |
| Refresh | every **60 s**, on CAS login change, on pull-to-refresh |
| Stop selection | nearest stop across all lines; dedupe lines by name, then by line id; loop lines match on the last stop |

**Strings:** `校巴` · `重试` · `距你%1$dm` · `加载时遇到了错误` · `获取地理位置异常` · `更新校巴信息异常` · `站` · `%lld站` · `已到站` · `即将到达` · `未发车` · `到达` · `前往` · `当前%@` · `当前` · `把%@添加到实况活动` · `清除所有实况活动`

**States:** logged out or > 20 km → score −1, nothing rendered · first load → full-body spinner · refreshing with data → inline spinner · error → red banner + `重试`, stop block and line list suppressed · no buses on a line → `未发车` · bus has passed → blank label · partial line failure → the whole card goes to error. No skeleton/shimmer.

⚠ Android: content padding 16 but no error banner and **no retry**, no line-name chip (plain `#01579B` text), no route summary, no bus-count glyphs, no `当前` prefix, no Live-Activity menu, `> 100 km` hide threshold (5× looser), 5 s polling, and a **vertical** dot rail instead of the horizontal timeline (28 dp rows, 2 dp × 10 dp connectors, 8 dp dots with a 4 dp `ham_bg_b1` hole in `#01579B`; the first connector has no background and is invisible). iOS: content padding **0**, 20 km threshold, its 60 s refresh loop is declared but never started (refreshes on login change and pull-to-refresh only), and `expand` survives data refresh.

---

#### Sport (运动) — `#34C759`

**Shows when:** there is at least one recent order (score 60 unpaid / 30 active / −1 when none). **One full card per order**, 8 apart. Chevron → the sport screen; `去支付` → the sport pay screen. Nothing else is tappable.

**Contents:**
1. Header band: `sportscourt` + `运动` + chevron.
2. Status line (order status name).
3. Court-number badge.
4. Venue line `sportType | stadium`.
5. Time range line.
6. Payment block (unpaid orders only): divider, then either the deadline + `去支付`, or the not-yet-payable pair.

**Values:**

| Element | Value |
| --- | --- |
| Content padding | 16; block spacing 10 |
| Status line | 16 bold, `ham_text_primary` |
| Court badge | **32 × 32**, radius **6**, fill **`#34C759`** (must be explicit), text 22 rounded bold, `#FFFFFF` |
| Venue line | `羽毛球 | 桂园体育馆`, 12 |
| Time line | `2026-09-19 10:00-11:00`, 12 |
| Divider | system hairline, inside the payment block only |
| Deadline (payable) | `请于%@前完成支付`, 12, **`#F44336`** |
| Deadline (not yet) | `当前未处于可支付时间` 12 bold + `请于%@-%@完成支付` 12 |
| `去支付` button | 12, `#FFFFFF` on `#007AFF`, padding h8 v8, radius 6, right-aligned |
| Analytics | `sport_pay_btn` with `location = "current_order_card"` |
| Refresh | on init and on pull-to-refresh; no timer |

**Strings:** `运动` · `去支付` · `请于%@前完成支付` · `请于%@-%@完成支付` · `当前未处于可支付时间` · `待付款` · `待使用` · `使用中` · `已使用` · `已取消` · `已退款` · `未知`

**States:** loading → spinner inside the card (do **not** unmount it) · error → the standard red banner with `重试` · empty → score −1, card removed · unpaid and payable → deadline + `去支付` · unpaid before the window → the two-line notice, no button · paid/using/ended/cancelled/refunded → no payment block. No skeleton/shimmer.

⚠ **iOS only** — Android has no sport card at all (no composable, no `Sport` enum case, no `status_sport*` strings), though `ham_brand_sport`, the sport feature module and the `sport_pay_*` strings all exist. iOS defects: the whole card sits inside `ForEach(orderList)`, so loading and error render **nothing** (`loadState` is written but never read); the court badge has **no explicit fill**, so its white text is white-on-white in dark mode; `updateScore`/`setVisibility` are never called, so the card is frozen at score 1; the seven order-status labels are hardcoded Chinese literals used as localisation lookup keys.

---

#### Divergences

- **Cards implemented:** iOS 7/7; Android 4/7. **Sport is iOS-only** (Android has the domain, colour, routes and strings but no card). **Android's schedule card is an uncomposed stub** — `ScheduleCard.kt` is a header band with an empty content lambda, zero call sites, no `Schedule` branch in the render `when`, no view-model; `StatusViewCardType.Schedule` is scored at 0 but dropped by `else -> {}`.
- **Scoring:** iOS keeps 7 scored types with a separate `invisibleCardType` set (so −1 does **not** hide a card); Android keeps 5 types, encodes hiding in the score, hides nothing by default, never persists order, and zeroes every score on refresh. Android never scores course or bus; iOS never scores bus or sport.
- **Geometry:** card radius 16 (iOS) vs 12 (Android); content padding 12 vs 16; header padding 12 uniform vs 16/12; header icon unsized vs 24; inter-card gap unspecified (SwiftUI `VStack` with no `spacing`) vs 8.
- **Page shell:** large title 34 vs 32; title→cards gap 30 vs 42; banner 200 vs 300; refresh threshold 128 pt pill vs 300 px Material indicator; iOS has a bottom fade (85, iOS<26 only) and a foreground trigger, Android has a 2-minute poll and no fade; Android sets no page background and falls through to the Material theme colour.
- **Type:** iOS uses SwiftUI text styles (17 pt body, 28 pt title, 34 pt largeTitle); Android uses the `HamFontStyle` scale (16/24/32 sp). Neither uses `HamFontStyle.largeTitle` (28 sp) for the status title.
- **Brand colours:** only course `#1B5E20` and schedule `#01579B` match exactly. Library, bus, sport and weather are stock UIKit colours on iOS; weather and CAS have no brand token on either platform.
- **Per-card:** library — Android lacks the ended-today state, the green/orange progress tint and the `变更预约` chip, and counts down at 10 s with floored minutes and a 2 h hour threshold. Course — Android lacks the week-grid strip, never hides or scores the card, uses a 21:30 cut-off and inverted toggle semantics relative to iOS's Chinese strings. Bus — Android replaces the horizontal timeline with a vertical rail, drops the retry affordance, the line chip, the route summary and the `当前` prefix, and hides 5× further out. Weather — Android has no attribution (legal gap), no cache, and never scores a failure.
- **Known defects:** iOS CAS chip renders white-on-white; iOS weather sub-line loses the city name (`??` binds looser than `+`); iOS schedule always reads `待完成0项日程`; iOS sport badge has no fill; iOS bus refresh loop is dead code; Android's first bus rail connector is invisible; Android's dead second weather card (`WeatherCard.kt`, `StatusViewWeatherCard`) must not be documented or revived.
- **Localisation:** iOS ships `zh-Hans`/`en`/`ja` `.lproj` catalogs; Android ships `values` (Chinese default, **no `values-zh`**), `values-en`, `values-ja`. The only untranslated user-visible string in the module is ` Weather`.


---

### Course timetable

#### Course timetable (课程表)
**Purpose:** One week of the term as a 13-period × 5-or-7-day grid, with week navigation, a course detail overlay, and long-press add/paste. **Entry:** iOS tab 1; Android `CourseRoutePath.HOME`. **Platforms:** both.

**Layout:**
```
┌────────────────────────────────────────┐
│ STATUS BAR                    h = sbH  │  iOS ignoresSafeArea
├────────────────────────────────────────┤
│ HEADER  [‹]  [本周 / 第N周]  [⚙] [›]   │  h 50 (iOS) / sbH+56 (Android)
├────────────────────────────────────────┤  ↓ 8
│ WEEKDAY │ 42 │ 一  二  三  四  五  …   │  h 36
├────────────────────────────────────────┤  ↓ 2
│ BODY    │ 1  │ ▢ ▢ ▢ ▢ ▢               │  13 equal rows
│         │ .. │       row h = avail/13  │  H pad 2
│         │ 13 │ ▢ ▢ ▢ ▢ ▢               │
├────────────────────────────────────────┤
│ tab-bar clearance             h 90     │
└────────────────────────────────────────┘
```
Cell `w = (boxW − pad·cols − railW − 2·hPad)/cols`, `h = (boxH − bodyTop − pad·13)/13`, `bodyTop = weekdayRowH + headerH + sbH`. iOS 26 header: `‹`/`›` glass buttons at the edges, week label in a glass capsule `minWidth 120`, height 50, H pad 10.

**Content blocks, in order:**
1. **Header / week bar** — prev chevron, week label, settings gear, next chevron; Android renders it only when `semesterBeginDate != null`. Tap: chevrons change week; label opens the week-jump menu; gear → settings.
2. **Weekday row** — rail spacer + 5 cells (Mon–Fri) or 7 (Sun–Sat) per `showWeekend`; each shows the weekday name over the date `M-d`. Tap: nothing.
3. **Period rail** (13 rows) — period number, or start/end times; tap toggles between the two.
4. **Course cells** — one block per course, height `rowH × spans + padding`. Tap: detail. Long-press: menu.
5. **Empty cells** — placeholder chips; long-press → add/paste menu, only when `displayWeek > 0`.
6. **Detail overlay** — see next screen.

**Values:**
| Element | Value | |
| --- | --- | --- |
| Header height / H pad | 50 (iOS, below status bar) / `sbH + 56` (Android); H pad 10 (iOS) / 12 (Android) | |
| Header material | `.systemThinMaterial` blur, `.glassEffect()` on iOS 26; Android `ham_bg_b2`, or `black/white @ 0.75f` over a wallpaper | |
| Chevron + gear | chevron 25×25 (iOS legacy) / 24×32 (iOS 26) / 24 Material `ChevronLeft\|Right` tint `ham_blue`; gear 25×25 `gearshape` → settings, on every platform | (diverges: iOS 26 has no gear; Android has none at all) |
| Week label | primary `本周` / `第N周` / `放假中`, `bodyBold` semibold; secondary `第N周` / `还有N周开学` `caption` 12; jump links `ham_blue` | |
| Week-jump menu | weeks `1…20`, check mark on the current week | (diverges: iOS 1–19) |
| Weekday row | height 36, rail width 42, body H pad 2 | (diverges: Android 48 / 32) |
| Weekday cell | radius 10; bg `ham_lightGray`, `ham_lightBlue` when today; name `caption` 12; date `caption2` 11; today → `ham_blue` else primary | (diverges: Android radius 12, `@0.15f` fills, date `caption` 12) |
| Period rail | number `caption` 12; times `caption` 12; highlighted → `ham_blue` else primary | (diverges: Android times `caption2` 11) |
| Course chip | name bold + instructor + location, all `caption2` 11, H pad 2, top pad 2; bg `displayColor` + `black@0.25` in dark, over a wallpaper thin material + colour @ 0.5 at `courseOpacity`; text colour contrast-derived | (diverges: Android name `captionBold` 12, others `caption` 12, pad 4) |
| Empty chip / row gap | radius 10 `ham_lightGray`, opacity 0.001 with a wallpaper; row gap and item V pad 2 | (diverges: Android gap 4) |
| Week swipe | horizontal drag, threshold 60 (iOS) / 20 (Android) → prev/next | |
| Long-press | haptic; menu 编辑 / 复制 / 剪切 / 删除这节课 / 删除该门同时间段的课 / 删除这门课; empty cell → 添加 / 粘贴\<课程名\>; pressed cell scales 1.1 | (diverges: iOS native `contextMenu`, no scale, no haptic) |
| Scrim / clearance | `Black @ 0.7f` behind the dropdown and the detail sheet; bottom 90 (iOS) / `bottomWithTabBarHeight() + 8` (Android) | |

**Strings:** `本周` · `第N周` · `放假中` · `还有N周开学` · `前往本周` · `前往第一周` · `编辑` · `复制` · `剪切` · `删除这节课` · `删除该门同时间段的课` · `删除这门课` · `添加` · `粘贴<课程名>` · tab `CURRICULUM`. Weekday names: long 星期一…星期日, short 周一…周日 — use short when `showWeekend` is on or the language is not zh. Android keys `course_week_format` `第%d周`, `course_week_current`, `course_week_go_current`, `course_week_go_first`, `course_week_until_start`, `course_menu_*`, `course_weekdays_full` / `_short`.

**States:** **Loading** — none; the grid renders and fills in. **Empty (no term start date)** — centred column, title `课程表未设置` `title2` 20, subtitle `请设置开学日期后再使用课程表` `body` 16, padding 24, spacing 8; the week bar still renders (diverges: iOS has no such state). **Empty (term set, no courses)** — grey chips, long-press → 添加. **Before term (`displayWeek <= 0`)** — label `放假中`, secondary `还有N周开学`, action `前往第一周`; navigation skips week 0; add/paste disabled.

#### Course detail (课程详情)
**Purpose:** A course's info, grade distribution, and linked schedules. **Entry:** tap a course cell. **Platforms:** both.

**Layout:**
```
┌────────────────────────────────────────┐
│ sbH spacer + 72                        │
├────────────────────────────────────────┤
│ Scroll, spacing 16, H pad 16           │
│  ┌ INFO ─────────────────────────────┐ │
│  │ ● 高等数学              dot 8      │ │
│  │ 必修 · 3.0 学分    caption 12      │ │
│  │ 张三 / 教三 301                    │ │
│  ├──────────────────────────────────┤ │
│  │ SCORE CARD        (conditional)  │ │
│  ├──────────────────────────────────┤ │
│  │ SCHEDULE CARD     (conditional)  │ │
│  └──────────────────────────────────┘ │
│ [⚙] [+日程] [✕]  top-trailing, pad 16  │
└────────────────────────────────────────┘
```
Presentation stays platform-native: iOS a full-screen `.overlay` morphing out of the grid cell (`matchedGeometryEffect`, spring `response 0.6 / dampingFraction 0.8`); Android a half-height sheet (`slideInVertically{it/2} + fadeIn`). Both: H pad 16, card radius 16, spacing 16 between cards.

**Content blocks, in order:**
1. **Colour dot + name** — dot 8 `CircleShape` in the course colour, 8 gap, name `bodyBold` 16 (diverges: iOS 10).
2. **Type · credit** — `caption` 12, `"<type> · <credit> 学分"`; render only when `courseType` is non-empty, append the credit part only when `credit > 0` (diverges: iOS `%.1f`, Android `%.2f`).
3. **Instructor** / **location** — `body` 16, 4 gap, each conditional on non-empty.
4. **Score card** — conditional; see States.
5. **Linked-schedule card** — conditional on a non-empty list; title `关联了N项日程`; row = name `bodyBold` + primary, time `caption` + secondary, V pad 4, rows spaced 8; tap opens the schedule. iOS adds a countdown (28 bold + `caption` unit at offset y 8, `ham_darkBlue`, `.orange` when expired) and a `Divider` between future and past (diverges: Android has no countdown and prints the timestamp twice per row).
6. **Action bar** — add-schedule + close (iOS also gear → edit); height 32, radius 8, `ham_gray @ 0.75f`, icon pad 4 tint white, label `body` 16 white, container `spacedBy(8)` H pad 16 (diverges: iOS 17pt circular ultra-thin-material buttons at offset (−16, 65)).

**Values:**
| Element | Value | |
| --- | --- | --- |
| Container H pad / card spacing | 16 / 16 | (diverges: iOS spacing 0 + per-card bottom pad 8) |
| Info card / dot / name | `HamCardView` padding 16 radius 16 / dot 8 / name `bodyBold` 16 | (diverges: iOS dot 10, name 17 bold) |
| Credit format | `"<type> · %.1f 学分"` | (diverges: Android `%.2f`) |
| Schedule title | `关联了N项日程`, counting **all pending** items | (diverges: iOS counts future-only) |
| Countdown | 28 bold + `caption` unit; `ham_darkBlue`, `.orange` when expired | (diverges: Android none) |
| Scrim / icons | `Black @ 0.7f` (Android), iOS `displayColor@0.3` + `.systemMaterial`; `gearshape.fill`, `calendar.badge.plus`, `xmark` | |

**Strings:** `学分` · `关联了N项日程` · `添加日程` · `%@剩余` · `%@前` · `暂无这节课的成绩统计信息` · `%d位同学的成绩` · `均分` · `暂无该课程的给分数据`. Android keys `course_credit_format` `· %s学分`, `course_linked_schedules`, `course_add_schedule`, `course_score_total_students`, `course_score_average`, `course_score_empty`.

**States:** **Loading (score)** — progress indicator inside a `HamCardView`. **Loaded, with data** — name `bodyBold` maxLines 2, instructor `caption`, 8 gap, total-students `caption`, one row per range (label width 48, bar height 4 radius 2, count `caption` start pad 4); right column: divider height 120 at end 81, `均分` `caption` + `"%.1f"` at `title` 24, comment button (icon `Forum` 20 tint `ham_gray`, bg `ham_gray@0.2f`, pad 4). **Loaded, no data** — `暂无这节课的成绩统计信息` `body` 16, box padding 12. **Error** — render the message at `body` 16 (diverges: iOS silently omits the card). **Card hidden** — when the education fetch never ran or the course id has the custom prefix (diverges: `customize` vs `cust` → unify on `cust`). **Dismiss** — close button, tap-outside, or the timetable-changed notification. **Schedule popover (Android only)** — full-screen `Black @ 0.75f`, `HamCardView` padding 32, name `title2` 20, timestamp `body`, three icon buttons (delete orange / edit darkBlue / close); countdown refreshes every 10 s.

#### Course settings (课程表设置)
**Purpose:** Term start date, background + opacities, weekend toggle, reset, theme entry, help. **Entry:** gear in the timetable header or the My-tab function card. **Platforms:** both.

**Layout:**
```
NavBar  title = 课程表设置
├─ Scroll, H pad 16, block spacing 16 ────
┌ 课程表设置 ────────────────────────────┐
│ 开学日期              [2025-09-01]     │
│ 2025-2026年度 第一学期    caption 12    │
│ 显示周末                      [switch] │
│ 获取课程表                      (blue) │
├────────────────────────────────────────┤
│ 个性化                                 │
│ [preview 200 × 200·ratio, r10]         │
│ 背景透明度 ──slider──                  │
│ 课程透明度 ──slider──                  │
│ 移除背景图片                     (red) │
├────────────────────────────────────────┤
│ 配色设置   前往配色设置         (blue) │
├────────────────────────────────────────┤
│ 其它   重置课表 → 确认重置课表   (red) │
├────────────────────────────────────────┤
│ 帮助  [? watermark 128, offsetY 4]     │
│ 如何添加、编辑课程      headlineBold   │
│ 在课表空白处长按可添加课程…   caption  │
└────────────────────────────────────────┘
```

**Content blocks, in order:**
1. **Term start date** — label `开学日期` `body` 16 + date chip (`Gray @ 0.15f`, radius 8, pad (6, 8), text `body` 16 primary); tap opens the native picker. Below it the computed semester string `caption` 12 secondary, `<y1>-<y2>年度 第<一|二|三>学期` (diverges: iOS uses a `labelsHidden` `DatePicker` with no label).
2. **Weekend toggle** — `显示周末` `body` 16 + switch; track on `#34C759`, off `ham_lightGray`.
3. **Fetch timetable** — `获取课程表` `body` 16 `ham_blue` → fetch sheet (diverges: iOS a full row with a 40×40 circular blue icon (`arrow.up.right` 20 semibold on `blue@0.1`), bold title `按照开学日期更新课程表`, `caption` subtitle `从教务系统获取课程表`, trailing `chevron.right`).
4. **Background / personalisation** — with an image: preview + two sliders + remove; without: a single `设置背景图片` button; animate between the states.
5. **Colour settings** — card title `配色设置` + `前往配色设置` `body` 16 `ham_blue`.
6. **Reset** — `重置课表` `body` 16 `ham_red`; two-tap confirm, second tap reads `确认重置课表`.
7. **Help** — watermark `?` 128 offsetY 4, card top pad 48; header `如何添加、编辑课程` `headlineBold` 14; body `caption` 12.

**Values:**
| Element | Value | |
| --- | --- | --- |
| Nav title | `课程表设置` | (diverges: iOS `课程设置`) |
| Page bg / H pad / block spacing | `ham_bg_b1` / 16 / 16 | (diverges: iOS spacing 24, Android 8) |
| Section headers | none — cards carry their own titles | (diverges: iOS `caption` headers 基础信息 / 自定义配置 / 帮助) |
| Background preview | width 200, height `200 × ratio`, `ratio = max(screenH,W)/min(screenH,W)`, clip radius 10 | (diverges: iOS `ratio = screenH/screenW`) |
| Preview mock chips | 3 chips at 1/5 preview width, heights ×4 / ×3 / ×5, radius 8, at `courseGridAlpha`; plus a top bar chip 180×40 radius 5, shadow radius 5 offset (0,2) `black@0.1` | |
| Sliders | `背景透明度` `0…0.85`, `课程透明度` `0.15…1`; thumb + active track `ham_blue`; row spacing 4; label `body` 16 | (diverges: Android stores `1 - alpha` clamped `<= 0.95`, effective `0.05…1` for both) |
| Remove / choose | `移除背景图片` `ham_red` · `设置背景图片` `ham_blue` | (diverges: iOS `移除图片` / `选择图片`) |
| Reset | two-tap confirm, re-arming; success toast `已重置课程表`; error toast `重置课表时遇到了错误` | (diverges: iOS one tap then permanently disabled; Android toasts unconditionally with no VM result) |
| Help watermark | 128, offsetY 4, card top pad 48 | (diverges: iOS 180 at offset (0, 30)) |

**Strings:** `课程表设置` · `开学日期` · `决定获取哪一学期的课表` · `<y1>-<y2>年度 第<N>学期` · `按照开学日期更新课程表` · `从教务系统获取课程表` · `获取课程表` · `背景设置` / `个性化` · `设置课程表的背景` · `设置背景图片` · `背景透明度` · `课程透明度` · `移除背景图片` · `高级设置` · `显示周末等` · `显示周末` · `配色设置` · `前往配色设置` · `其它` · `重置课表` · `确认重置课表` · `已重置课程表` · `帮助` · `如何添加、编辑课程` · `在课表空白处长按可添加课程，在课程出长按可编辑该课程。` · `如何快速查看某一周的课程表` · `长按顶部的“第几周”，可快速选择周数。` · `重置课表时遇到了错误` · date picker `未设置` / `确定` / `取消`.

**States:** **No background image** — single `设置背景图片` button. **Background set** — preview + sliders + remove. **Image picking** — system picker → cropper locked to the screen aspect ratio. **Reset** — two-tap as above. **Loading / error** — none (toasts only).

#### Theme selection (配色)
**Purpose:** Choose the built-in or custom palette; import/export by QR. **Entry:** `前往配色设置`. **Platforms:** both.

**Layout:**
```
NavBar  title = 配色
├─ Scroll, H pad 16, card spacing 16 ─────
┌ 默认配色 ─────────────────── [选取] ──┐
│ ▪▪▪▪▪▪▪▪  24, r4, gap 4                │
├────────────────────────────────────────┤
│ 自定义配色 ───────────────── [选取]    │
│ ▪▪▪▪▪▪▪▪  or  无自定义配色             │
│ 编辑颜色                       (blue)  │
├────────────────────────────────────────┤
│ 扫码导入自定义配色             (blue)  │  whole card = button
├────────────────────────────────────────┤
│ 分享我的自定义配色             (blue)  │  tap → QR
│ [QR 200]                               │
└────────────────────────────────────────┘
```

**Content blocks, in order:**
1. **Default palette card** — title `默认配色` `bodyBold` `weight(1f)`, read-only swatch grid, `选取` button.
2. **Custom palette card** — title `自定义配色`, swatch grid or `无自定义配色`, `编辑颜色` `body` 16 `ham_blue` → custom-theme screen, `选取` button.
3. **Import row** — whole card is the button, `扫码导入自定义配色` `body` 16 `ham_blue` → scanner, auto pop-back.
4. **Share card** — `分享我的自定义配色` `body` 16 `ham_blue`; tap toggles the QR with animation.

**Values:**
| Element | Value | |
| --- | --- | --- |
| Title / bg / padding / card spacing | `配色` / `ham_bg_b1` / 16 / 16 | |
| Swatch | 24 × 24, radius 4, grid gap 4 | (diverges: Android 20) |
| `选取` | `body` 16; `ham_blue` when selectable, `ham_gray` + disabled when already active | |
| Empty custom label | `无自定义配色` | (diverges: Android renders an empty grid only) |
| QR / payload | 200 tall, `.interpolation(.none)`, scaled to fit, centred, `CIFilter.qrCodeGenerator`; payload = JSON array of `#RRGGBB` hex strings — identical on both platforms | |
| Import validation | ≤ 30 entries, each a valid 6-digit hex with optional `#`; on success toast `成功从二维码导入配色` then pop | |
| Share card visibility | only when the custom list is non-empty | (diverges: Android always shows it and always shows the custom `选取`) |

**Strings:** `配色` · `默认配色` · `自定义配色` · `选取` · `无自定义配色` · `编辑颜色` · `扫描配色分享二维码` · `扫码导入自定义配色` · `成功从二维码导入配色` · `分享我的自定义配色`.

**States:** **No custom palette** — `无自定义配色` label, custom `选取` hidden, share card hidden. **Selected theme** — its `选取` disabled and greyed. **QR expanded** — local state, animated. **Loading / error** — none.

#### Custom theme (自定义配色)
**Purpose:** Add, list, and delete the colours in the custom palette. **Entry:** `编辑颜色`. **Platforms:** both.

**Layout:**
```
NavBar 自定义配色        [清空所有配色]   trailing, red; disabled+grey when empty
├─ Scroll, H pad 16, item spacing 8 ─────
┌────────────────────────────────────────┐
│ ▪ #RRGGBB                  删除 (red)  │  swatch 24, r6
│ ────────────────────────────────────── │  Divider between rows
│ ▪ #RRGGBB                  删除        │
├────────────────────────────────────────┤
│ [colour well]              添加颜色    │  preview bar 32×10 r4
└────────────────────────────────────────┘
```
Android picker: bottom-sheet ring picker, `showAlphaBar = false`.

**Content blocks, in order:**
1. **Colour list** — one card, one row per colour separated by a `Divider`; row `HStack(spacing: 4)` (diverges: Android renders one `HamCardView` per colour, 8 apart).
2. **Row** — swatch 24 × 24 radius 6, hex text `body` 16 primary `weight(1f)`, `删除` `body` 16 `ham_red`.
3. **Add row** — colour well (24 box radius 6 `ham_gray@0.3f` + `ColorLens` icon 20 tint `ham_gray` + preview bar 32×10 radius 4) + `添加颜色` `body` 16 `ham_blue`.
4. **Clear-all** — trailing toolbar, `清空所有配色`, `ham_red`, disabled + grey while the list is empty.

**Values:**
| Element | Value | |
| --- | --- | --- |
| Title / bg / padding | `自定义配色` / `ham_bg_b1` / 16 | |
| Swatch | 24 × 24, radius 6 | (diverges: Android 20, radius 4) |
| Spacing | 4 within a row; 8 between cards | |
| Picker | opacity disabled; default `Color.gray`, reset to gray after each insert | |
| Max colours | 30 — reject with a toast | (diverges: Android has no cap) |
| Duplicate colour | reject with an error toast; compare the hex string, not the platform colour object | (diverges: Android compares raw `Color` ARGB floats) |
| Clear-all | red, disabled when empty; success toast after clearing | (diverges: Android always enabled, no toast) |
| Insert | animate the list insertion | (diverges: Android none) |

**Strings:** `自定义配色` · `删除` · `添加颜色` · `清空所有配色` · `最多选择30种颜色` · `存在相同的颜色` · `已清空所有配色`.

**States:** **Empty** — list card hidden, clear-all disabled. **Max / duplicate** — toast, add rejected. **After clear-all** — success toast. **Loading / error** — toasts only.

#### Course edit / add (编辑课程 / 添加课程)
**Purpose:** Create or modify a course: colour, name / instructor / location, week / period / weekday slots. **Entry:** long-press menu (添加 / 编辑) or the gear on the detail sheet. **Platforms:** both.

**Layout:**
```
NavBar  添加课程  /  编辑课程
├─ Scroll, H pad 16, block spacing 8 ─────
┌ 背景色 ────────────────────────────────┐
│ ▪▪▪▪▪▪▪▪  24 r4 gap 8   │  [well]     │
├────────────────────────────────────────┤
│ 基本信息          label column width 64│
│ 课程名  [输入课程名]                    │
│ 讲师    [输入讲师]                      │
│ 地址    [输入课程地址]                  │
├────────────────────────────────────────┤
│ 周数  1…25      36 r8, gap 8           │
│ ────────────────────────────────────── │
│ 节数  1…13                             │
│ ────────────────────────────────────── │
│ 星期  周日…周六                         │
├────────────────────────────────────────┤
│ [ 保存这门课 ]   h48 r12 blue@0.15     │
└────────────────────────────────────────┘
```

**Content blocks, in order:**
1. **Colour** — swatch grid from the active theme + a system colour well (opacity disabled); applies immediately.
2. **Basic info** — fields 课程名 / 讲师 / 地址 with hints 输入课程名 / 输入讲师 / 输入课程地址; label column width 64; field radius 8, `ham_bg_b2`, border `1 ham_lightGray`, padding 8, text `body` 16, hint `body` secondary (diverges: Android uses `Bookmark` / `Person` / `Map` icons and no text labels).
3. **Time setting** — sections 周数 / 节数 / 星期 separated by dividers, labels `bodyBold` 16. Slot buttons 36 × 36 radius 8, gap 8; selected `ham_blue @ 0.2`, unselected `ham_gray @ 0.1`; end cells take the outer radii. Weeks and periods are multi-select (tapping outside a range of ≥2 resets to that single value, otherwise extend/shrink at the edges); weekday is single-select.
4. **Commit** — one full-width button, height 48, radius 12, `ham_blue @ 0.15f`, `headlineBold` 14 `ham_blue`.

**Values:**
| Element | Value | |
| --- | --- | --- |
| Title | `添加课程` when creating, `编辑课程` when editing | (diverges: Android always `编辑课程`) |
| Background / H pad / block spacing | `ham_bg_b1` / 16 / 8 | |
| Swatch | 24 × 24, radius 4, gap 8; selected ring `stroke(gray@0.3, 2)` | (diverges: Android 20, gap 4, `1 ham_gray` border) |
| Slot button | 36 × 36, radius 8, gap 8; selected `ham_blue@0.2`, unselected `ham_gray@0.1` | (diverges: Android 32 with a `1 ham_lightGray` border and the course colour as fill) |
| Week / period count | `weekMax` from shared config, default **25**; periods 13 | (diverges: iOS `timetableWeekTotal` defaults to 17) |
| Commit button | height 48, radius 12, `ham_blue@0.15f`, `headlineBold` 14 `ham_blue` | (diverges: iOS 17pt, radius 8, `blue@0.1`, plus per-section floating `保存` / `重置时间` at offset (−16, 16)) |
| Custom course id | prefix `cust` + uuid | (diverges: iOS `customize `) |

**Strings:** `添加课程` · `编辑课程` · `保存这门课` · `课程背景色` / `背景色` · `基础信息` / `基本信息` · `课程名` · `输入课程名` · `讲师` / `授课人` · `输入讲师` · `地址` / `地点` · `输入课程地址` · `时间设置` · `周数` · `节数` · `星期` · weekday short names. Commit variants: `新增课程` / `新增<name>` / `重置这门课` / `修改这门课的基本信息`. Toasts: `课程名不能为空` · `信息不完整` · `周次不能为空` · `节数不能为空` · `未选择星期` · `存在冲突的课程` · `<name> - 第N周 第X[-Y]节` · `保存失败`.

**States:** **Initial (add)** — week / period / weekday prefilled from the tapped cell; colour = a random colour from the active theme; id `cust <uuid>`. **Initial (edit)** — fields populate asynchronously; show an empty form until resolved. **Empty name** — error toast `课程名不能为空`, abort. **Empty weeks / periods / weekday** — error toast with the matching subtitle (周次不能为空 / 节数不能为空 / 未选择星期) (diverges: iOS silently aborts). **Conflict** — error toast `存在冲突的课程` + `<name> - 第N周 第X[-Y]节`. **Save failure** — error toast `保存失败`, do not pop. **Success** — pop back and post the timetable-changed notification (diverges: iOS stays open with a success toast). **Loading / error** — toasts only.

#### Fetch courses (获取课程)
**Purpose:** Authenticate against the university portal (CAS) and pull the term's courses. **Entry:** the `获取课程表` row in settings. **Platforms:** both.

**Layout:**
```
┌────────────────────────────────────────┐
│ [关闭]                          TextBtn │  14, ham_blue
│   [logo 48 r8] [link] [tablecells 48]  │  spacing 12; module glyph @0.25
│   获取课程              title 24 Bold  │  top 32, bottom 4
│   将按照设定的开学日期获取课程  body 16 │  bottom 16
│  ┌──────────────────────────────────┐  │  r12, ham_lightGray, pad 16, gap 8
│  │ (public) 通过信息门户             │  │  icon 24 ham_blue
│  │         从信息门户登录教务系统…    │  │  caption 12
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

**Content blocks, in order:**
1. **Dismiss** — `关闭`, `headline` 14 `ham_blue` (diverges: iOS `取消` in the navigation bar).
2. **Header cluster** — logo 48 × 48 radius 8, link glyph, module glyph 48 at `ham_brand_course @ 0.25`; spacing 12.
3. **Title + subtitle** — `获取课程` `largeTitle` 28 bold (top pad 32, bottom 4); subtitle `将按照设定的开学日期获取课程` `body` 16 (bottom 16).
4. **Method list** — one row: icon `Public` 24 tint `ham_blue`, title `通过信息门户` `bodyBold` 16, subtitle `从信息门户登录教务系统获取课程` `caption` 12, trailing chevron. Tap starts the CAS flow.
5. **Captcha / login web view** — bundled captcha page in a web view; sub-route title `验证码验证`.
6. **Terminal states** — loading, success, error.

**Values:**
| Element | Value | |
| --- | --- | --- |
| Presentation / logo | modal sheet (iOS wraps a `NavigationView`, Android a `HamSheet`); logo 48 × 48 radius 8 | (diverges: iOS 56, radius 12) |
| Cluster spacing | 12 | (diverges: iOS 16) |
| List container | radius 12, `ham_lightGray`, padding 16, row spacing 8 | (diverges: iOS radius 8, `gray@0.15`, row spacing 24) |
| Module colour | `ham_brand_course` `#1B5E20` | |
| Native vs RN | gate on the shared CCKV flag `rnComponentConfig["enable"]` containing `RNFetchCourseView` | |
| Loading | centred progress indicator | |
| Success | `SuccessView`: full-screen Lottie `lottie_congrats.json`, detail after 500 ms with `slideInVertically{it/2} + fadeIn`; check icon 64 on a `ham_blue` circle pad 8; title 24; message `body` 16; 16 gap; button height 48, `ham_blue@0.15f`, radius 12, `headlineBold` 14, default text `完成`; vibrate on appear | |
| Error | `ErrorView`: same layout, 200 ms delay, close icon 64 on a `ham_red` circle | |
| Back handling | clear the back-stack on success; back from success/error returns to the intro | |

**Strings:** `获取课程` · `将按照设定的开学日期获取课程` · `通过信息门户` · `从信息门户登录教务系统获取课程` · `从信息门户验证` · `将进入武汉大学信息门户网页验证你的身份` · `验证码验证` · `正在更新` · `获取成功` / `更新成功` · `愉快使用吧` · `获取失败` / `更新失败` · `关闭` / `取消` · `完成` · `研究生教务系统` (route defined, entry disabled).

**States:** **Portal not linked** — the row routes to CAS login. **Portal linked** — straight to the captcha/RN fetch, appearing with an animation on login success. **Loading** — centred indicator. **Success** — `SuccessView`, clear the back-stack. **Error** — `ErrorView` with the message; retry resets to the initial state and returns to the intro. **Empty** — n/a.

##### Divergences

- **Timetable header:** iOS 26 has no settings gear, Android has none at all → settings only from the My tab; the spec requires the gear everywhere.
- **Week jump:** iOS 1–19 (`1..<max(currentWeek, 20)`), Android 1–20 → unify on 1–20.
- **Before term:** iOS labels `放假中`, Android `还有N周开学` with no holiday string → ship both.
- **No-term-set empty state:** Android-only centred `课程表未设置`; iOS has none.
- **Grid metrics:** iOS radius 10 / rail 42 / weekday row 36 / row gap 2 vs Android 12 / 32 / 48 / 4.
- **Chip fills:** iOS solid `ham_lightGray` & `ham_lightBlue`; Android `ham_gray@0.15f` & `ham_blue@0.15f`.
- **Type inversion:** iOS weekday-date 11 / period-times 12; Android 12 / 11. Course chip text: iOS all `caption2` 11; Android name `captionBold` 12 + others `caption` 12.
- **Long-press:** iOS native `contextMenu`; Android `DropdownMenu` with a 1.1× scaled clone, haptics, and a View-system drag (`CourseGridItemView`) vs iOS `onDrag`/`onDrop`. Paste label `粘贴<课程名>` vs bare `粘贴`.
- **Detail:** dot 10 vs 8; name 17 vs 16; credit `%.1f` vs `%.2f`; iOS counts *future* schedules, Android *pending*; iOS countdown 28pt, Android prints the timestamp twice; iOS omits the score card on error, Android renders it.
- **Custom-course prefix:** `customize ` vs `cust` → unify on `cust` (the score-card gate depends on it).
- **Settings:** title `课程设置` vs `课程表设置`; iOS has three `caption` section headers and no `其它` card, and groups weekend + reset + colour into the background section.
- **Opacity sliders:** iOS `0…0.85` / `0.15…1`; Android inverts to an effective `0.05…1` for both.
- **Reset:** iOS one tap then permanently disabled with an error toast; Android two-tap re-arming confirm, an unconditional success toast, no error path.
- **Theme:** swatch 24 vs 20; iOS hides the `无自定义配色` label, the custom `选取`, and the share card when empty; Android always shows them.
- **Custom theme:** swatch 24/r6 vs 20/r4; iOS one card with dividers, Android one card per colour; clear-all disabled+grey when empty vs always red+enabled; iOS caps at 30 with a toast, Android has no cap; duplicate check compares hex vs raw ARGB floats; iOS toasts after clearing, Android is silent.
- **Edit/add:** iOS separate routes/titles and per-section floating saves; Android one route titled `编辑课程` with a single commit button. Save button 17pt/r8/`blue@0.1` vs 14sp/h48/r12/`ham_blue@0.15f`.
- **Slot buttons:** iOS 36 with blue/gray fills; Android 32 with a border and theme-colour fill. Field labels 64-wide text vs icon-only.
- **Week count:** iOS `timetableWeekTotal` 17 vs Android `weekMax` 25 → unify on 25.
- **Validation:** iOS silently aborts on empty weeks/periods/weekday, Android toasts; iOS stays open after a successful edit, Android pops.
- **iOS localisation gap:** nine `String(localized:)` literals in the add/edit view models have no `Localizable.strings` entry and render as raw keys — `存在冲突的课程`, `保存失败`, `保存颜色失败`, `已保存背景颜色`, `保存课程基础信息失败`, `已保存基础信息`, `已重置课程时间`. Android has full `values` / `values-en` / `values-ja` coverage.
- **Intro:** logo 56/r12 vs 48/r8; cluster spacing 16 vs 12; list container radius 8 / `gray@0.15` / gap 24 vs radius 12 / `ham_lightGray` / gap 8; dismiss `取消` (nav bar) vs `关闭` (body button); terminal strings `正在更新`/`更新成功`/`更新失败` vs `获取成功` + `愉快使用吧`/`获取失败` with Lottie, haptics and back-stack clearing; row copy full sentences vs short forms.
- **Dead code:** `CourseEditViewSeekCell.kt` unreferenced; the post-graduate route and `strings.xml:37–38` unreachable; `CourseViewDetailViewModel.swift:62–73` scroll-to-dismiss commented out; `CourseMainViewDetailView.kt:304–350` commented out; `CourseViewBodyCourseItemView.swift:163–165` empty-cell tap handler empty.


---

### Schedule

#### Schedule home (日程)
**Purpose:** Show a hero card counting down to the nearest upcoming item, a horizontally scrolling group filter, and a list of schedule items split into future and past; tapping an item opens a detail popover. **Entry:** home/tab route `schedule`. **Platforms:** both. **Primary schedule screen.**
*(The audit also covers insert/edit, group edit and group position — out of scope here. Its home coverage is complete: hero, tabs, panel, list and popover all have values.)*

**Layout:**
```
┌────────────────────────────────────────────┐
│ nav 日程 (inline)                    [ + ] │ + → new schedule
├────────────────────────────────────────────┤
│ ┌ hero card, pad 16, r16 ────────────────┐ │ hidden when scrolled
│ │ 期末考试          │  12               │ │ past 50 and list > 5
│ │ 2026-01-08 09:00  │  天               │ │ number 52 Bold white,
│ │ 📍 教三楼 301                          │ │ plate ham_darkBlue or
│ │ ▌ 高等数学                             │ │ ham_orange when past
│ └────────────────────────────────────────┘ │
│ ┌ group tab bar, h 60, bg ham_bg_b2 ────┐ │ pad lead 16, gap 24
│ │ 📁 全部 [3] 📁 学习 [2] 📁 生活 [1] ▿ │ │ + 80 trailing fade
│ └────────────────────────────────────────┘ │ + chevron button
│ ┌ group expand panel, h 350 ────────────┐ │ header h 46, rows gap 24
│ │ [+]        编辑位置                   │ │
│ │ 📁 学习 [2]      期末考试           › │ │
│ └────────────────────────────────────────┘ │
│ list, pad 16, rows 12 apart                │
│ ┌ row, r12, bg ham_bg_b2, pad 16 ────────┐ │
│ │ 期末考试                 12            │ │ title 24 Bold
│ │ 2026-01-08 09:00         天            │ │ caption, y +4
│ │ ▌ 高等数学                             │ │
│ └────────────────────────────────────────┘ │
│ ── divider, pad top 12 ──  (future | past) │
│ trailing spacer = nav-bar height + 24      │
└────────────────────────────────────────────┘
 overlay: detail popover (w 300, r 12, scrim black @ 0.75)
```

**Content blocks, in order:**
1. Hero card — nearest item: name, begin timestamp, optional location row, optional related-course chip, right countdown plate. Conditional: shown only when a nearest item exists **and** (scroll offset < **50** or the filtered list has ≤ **5** items). Tap → detail popover for that item.
2. Group tab bar — `全部 [n]` then one chip per group, horizontally scrolling, trailing fade + chevron. Tap a chip → filter and scroll the list to the top; tap the chevron → open/close the group panel (chevron rotates 90°).
3. Group expand panel — header with an add-group button and `编辑位置`, then one row per group (icon, name, count badge, next future item name, chevron). Conditional: `无群组` when there are no groups. Tap a row → group edit.
4. List — future items ascending by `begin`, then past items descending, separated by a divider. Row: name, begin timestamp, optional related-course chip, countdown number + unit. Tap → detail popover.
5. Detail popover — header countdown block, name, date range, optional location, optional note (scrollable, max 200), optional related-course chip, then the action row. Close via scrim tap or the close button; actions are a two-stage `删除日程` and an edit button.

**Values:**

| Element | Value |
| --- | --- |
| Screen / hero card | bg `ham_bg_b1`, nav title `日程` inline, trailing `+` → new schedule; hero padding **16**, radius **16**, bg `ham_bg_b2`, top spacer **0** ⚠ iOS reserves 95 pt |
| Hero text | name `title2` 20 Bold, 2 lines; date `caption` 12 `ham_text_secondary`; location row `caption` only when non-empty; left padding **16** all round ⚠ Android start 12, vertical 12 |
| Hero countdown | number **52** Bold white, unit `body` 16 white at `offset(y: 16)`, plate padding h **16**, bg `ham_darkBlue` or `ham_orange` once past/in progress ⚠ iOS hero is always `ham_darkBlue`; iOS number 50, unit offset 12 |
| Hero scroll gate | hide when offset.y ≥ **50** **and** the filtered list has > **5** items ⚠ Android has no gate — the hero is permanently pinned |
| Related-course chip | 6-wide radius-3 bar (`fillMaxHeight`) + name `captionBold`, course colour with `ham_blue` fallback, top pad **8** |
| Tab bar | height **60**, bg `ham_bg_b2`, leading pad **16**, chip gap **24**, trailing spacer 64 |
| Tab chip | icon 20 (`group.icon`, fallback `Folder`) + gap **4** + label (`bodyBold` selected, else `body`), label max width **80**; tint selected `ham_text_primary` / unselected `ham_text_secondary` ⚠ Android hardcodes `Folder` and never reads `group.icon` |
| Count badge | `captionBold` `ham_text_secondary`, pad v **4** / h **8**, `ham_lightGray` radius **12**, min width **25** ⚠ Android has no badges anywhere |
| Selected indicator | tab width minus 20 horizontal padding, height **6**, radius **3**, `ham_blue`, bottom-anchored, animated ⚠ iOS uses a fixed 50-wide bar with `matchedGeometryEffect` |
| Trailing fade | **80** wide, 3-stop gradient (clear → `ham_bg_b2` → `ham_bg_b2`) ⚠ Android 45 dp, hand-drawn 4-stop |
| Expand chevron | `chevron.right`, padding 8, translucent circle plate, `offset(x: -8, y: 12)`, rotates 90° when open ⚠ Android circle is `ham_lightGray`, `padding(end = 4.dp)` |
| Group panel | height **350**, bg `ham_bg_b1`; header height **46**, padding h 16; row gap **24** ⚠ Android 300 dp, gap 8. Header: add button `plus` in a radius-8 `ham_lightGray` plate, padding 8; `编辑位置` right-aligned, `body`, `ham_blue`. Row: icon 20 `ham_text_secondary` from `group.icon`, name `bodyBold`, count badge (radius **10**), next **future** item name `caption`, `chevron.right` ⚠ Android shows the first item regardless of time and has no badge |
| Panel scrim | `black @ 0.5` over the hero, `@ 0.75` over the list; tap dismisses ⚠ Android uses 0.75 in both |
| List padding | leading **16**, horizontal **16**, row gap **12**, trailing spacer = nav-bar height + **24** ⚠ iOS 16/12/100+1, Android 24/16/116; future/past separated by a `Divider()` with top pad **12** when both buckets are non-empty ⚠ Android has no separator |
| Row container | radius **12**, bg `ham_bg_b2`, inner padding **16** ⚠ iOS radius 10, Android 16; name `bodyBold` 16 `ham_text_primary`; date `caption` 12 `ham_text_secondary` |
| Row countdown | number `title` 24 Bold, unit `caption` at `offset(y: 4)`, gap **2**; `ham_darkBlue` for not-started and in-progress, `ham_orange` when expired |
| Countdown suffix | `%@剩余` in progress, `%@前` once expired, bare unit when not started ⚠ Android shows only the bare unit |
| Countdown buckets | ≤1 min → `1 分钟`; <2 h → 分钟; <2 d → 小时; <10000 d → 天; else `10000 天` ⚠ Android's thresholds differ and it adds a `0 分钟` bucket |
| Time model | future/past and expiry derive from **`end`** when present, else `begin` — an in-progress item is neither future nor past ⚠ Android uses `begin` only, so in-progress items count as past |
| Popover | width **300**, radius **12**, bg `ham_bg_b2`, centred; scrim `black @ 0.75`; entrance slide-up + spring ⚠ iOS radius 10 with an `ultraThinMaterial` backdrop |
| Popover header | height **125**, full width, `ham_darkBlue` (or `ham_orange` when expired); number **72** Bold white; unit `body` white at `offset(x: 8, y: 20)` ⚠ iOS header is content-driven, number 80 pt |
| Popover body | padding **12**; name `title2` 20; date range `begin – end` (⚠ iOS prints `begin` twice — fix); location row when non-empty; note in a scroll view capped at **200** |
| Popover actions | `删除日程` two-stage (first tap arms, second deletes, label animates in), radius 6, `ham_red` 1-pt stroke, padding 4; edit `gearshape` button, radius 6, `ham_blue` 1-pt stroke; action-row top pad **24** ⚠ Android: single-tap 28 dp icon delete and a `ham_blue @ 0.15` settings plate, top pad 8 |
| Close | `xmark` top-trailing, padding 6, translucent circle, `offset(x: -8, y: 8)` ⚠ Android: 32 dp `ham_gray @ 0.75` circle, 8 dp margins |
| Refresh | recompute the clock at least once a minute so countdowns stay live ⚠ Android sets it once and once more after 2000 ms, then freezes |

**Strings:** 日程 · 全部 · 无日程 · 无群组 · 编辑位置 · 删除日程 · %@剩余 · %@前 · 分钟 · 小时 · 天 · 新建日程 / 编辑日程 (entry).
⚠ Every iOS schedule-home string is a hardcoded literal; only the countdown units and 剩余/前 are localised. Android resources everything except the unit labels.

**States:** empty (no items) → `无日程`, `caption`, `ham_text_secondary`, centred, 20 top padding ⚠ Android renders a blank list. Empty (no groups) → `无群组`, same treatment, inside the panel ⚠ Android renders a header-only panel. Loading → none; the data source is live, so the first frame is simply empty ⚠ Android's Flow starts as an empty list. Error → none. Panel open → panel at full height, scrims over hero and list, chevron rotated. Popover open → scrim + centred card, dismissible by scrim tap or close. Check-off → **none**; no completion field or affordance exists on either platform (iOS has a dead `isChecked` state) — the only way to finish an item is to delete it or let it expire.

##### Divergences
- iOS-only: chip/panel count badges, hero scroll-hide, hero location row, hero related-course chip, empty states.
- `end`-aware three-state countdown and the 剩余/前 suffixes: iOS only; Android is two-state on `begin`.
- Group icons: iOS renders `group.icon`; Android hardcodes `Folder` and never writes the field.
- Popover: iOS shows end time, location and note; Android shows none of the three.
- Delete: two-stage on iOS, immediate single-tap on Android.
- Popover scrim: iOS dismisses on tap; Android's scrim is not clickable.
- Clock: iOS recomputes every render; Android freezes after 2 s.
- Countdown unit buckets and the hero countdown colour differ.


---

### Library

#### 1. Library home (图书馆)
**Purpose:** Library hub — announcements, statistics opt-in, in-flight bookings, quick-book launcher, function grid. **Entry:** app library tab. **Platforms:** both.
**Layout:**
```
┌──────────────────────────────────────────┐
│ NavBar "图书馆" (iOS large / Android 32sp bold under 42dp toolbar) │
│ Scroll, bg bg_b1, pad-h 16, gap 16       │
│ A Banner carousel                    180 │
│ B Retry-login banner    (conditional)    │
│ C Analytics card        (conditional)    │
│ D Current-booking card(s)      (0..n)    │
│ ── divider + gap 16 ──  (if D present)   │
│ E Quick-book card       (conditional)    │
│ F Function grid                      142 │
│   ┌──────────┬─────────────────────────┐ │
│   │ 查看房间 │ 历史预约                │ │
│   │ (full h) ├─────────────────────────┤ │
│   │          │ 设置                    │ │
│   └──────────┴─────────────────────────┘ │
│ G Print card (gap 8) │ spacer 128        │
└──────────────────────────────────────────┘
```
**Content blocks, in order:**
1. **Banner carousel** — page 0 = announcements (book icon + `图书馆公告` + red `boardTip` when present); pages 1..n = remote banners (`bannerType: image|text`, `clickAction: innerWebview|outerWebview|fullText`). Conditional: always. Tap: announcements → Board (§11); remote → its action.
2. **Retry-login banner** — red card, title + `请点击重新登录`. Conditional: token invalid. Tap: re-run fast login.
3. **Analytics card** — consent prompt, or per-group sparkline + success-rate %. Conditional: remote flag on AND consent not declined. Tap: agree / disagree.
4. **Current-booking cards** — one per reserve / check-in / away booking. Conditional: list non-empty. Tap: expand toggle.
5. **Divider** — Conditional: block 4 present.
6. **Quick-book card** — seat row, divider, time row, reserve button. Conditional: a starred or last-booking seat exists. Tap: seat → Select seat; time → Select time; reserve → Quick book.
7. **Function grid** (3 cells) + **Print card**. Conditional: always. Tap: 查看房间 → Book; 历史预约 → History; 设置 → Settings; 打印 → Print.
8. **Bottom spacer.**
**Values:**
| Element | Value | Note |
| --- | --- | --- |
| Screen bg / content pad | `bg_b1` / `16` | iOS leaves block spacing implicit; Android `16` |
| Banner height / radius / Banner icon / title | `180` / `16` / `72` / title 24sp bold, accent | iOS 200 / iOS 28pt |
| Banner watermark | book glyph `72` @0.2, 5×10 | iOS 36pt @0.25, 6×9 |
| Carousel auto-advance / Board tip | `5 s`; dots hidden when only one page / red, `lineLimit(3)`, caption | Android: single static page, no timer, no tip / Android: absent |
| Booking card radius / Status strip | `16` / bold white on status colour, leading, pad h `16` / v `8` | Android v `12` |
| Seat number / location+time | title 24sp bold / caption `12` | iOS 28pt |
| Expand actions / Expand toggle | 刷新, 在地图打开, 添加到系统日历, 变更或取消预约 — icon `24` + text / caption `12`, `text_t2`, trailing pad `16`, bottom pad `12` | iOS: icon frame `20`, no calendar action / iOS also rotates the chevron −90° |
| Grid height / gap / radius | `142` / `8` / `16` | iOS 150 |
| Room cell | centred: chair icon `64` + bodyBold title + chevron `20` accent + caption subtitle; fill accent @0.15 | iOS fill @0.13 |
| History / Settings cells | leading: bodyBold title + chevron, caption subtitle, trailing icon `32` accent @0.75; fill `lightGray @0.1`; pad `12` | iOS leading pad `16` + overlay; Android mixes 8 (history) / 12 (settings) |
| Print cell | fill accent @0.15, radius `16`, pad `12`, icon `48` @0.75 | iOS: absent |
| Quick-book card / Seat badges | `HamCardView` 16/16; seat number 28sp bold / 收藏座位 orange / 上次预约 green — caption, white, pad `4`, radius `5` | Android 32sp / Android: absent |
| Quick-book divider | `1dp` divider, v-gap `10` | |
| Time row | `今天\|明天 HH:mm-HH:mm`, bodyBold accent | iOS: plain body |
| Reserve button | `48` tall, radius `12`, accent @0.25, centred bodyBold accent | iOS: text pill, radius `10`, accent @0.1, `.padding()` |
| Analytics bars / success-rate text | `2`×`30`, gap `1`, radius `1`; `<0` gray @0.4, `<0.5` orange, else green; row width `216` / body `17` above caption `请求成功率` | Android: bars `20`, `weight(1f)` / Android title2 20sp |
| Retry banner / Bottom spacer | red, radius `16`, person watermark `128` @0.25 / `128` | Android: `60` @0.15 (card default) / Android `navBar + 24` |
**Strings:** 图书馆 · 图书馆公告 · 查看房间 / 查看空余的座位预约 · 历史预约 / 你的历史预约记录 · 设置 / 图书馆预约选项 · 打印 / 在图书馆公共打印机打印 · 刷新 · 在地图打开 · 添加到系统日历 · 变更或取消预约 · 展开 / 收起 · 收藏座位 · 上次预约 · 预约 · 今天 · 明天 · 数据统计 · 请求成功率 · 昨天%@ · 登录失败 - %1$s · 请点击重新登录. (iOS hardcodes nearly all of these; Android uses `strings.xml`.)
**States:** **Loading** — none for bookings; the retry banner shows its own spinner while re-logging in. **Empty** — omit blocks 4+5, omit 6 if no starred/last seat, no empty copy. **Error** — toast only, screen unchanged. **Token expired** — block 2. **Consent pending** — card 3 prompt; declining hides it permanently. **Not logged in** — present the gate (§2); dismissing without login pops the tab.

#### 2. Library gate / connect intro (连接图书馆)
**Purpose:** Obtain a CAS session, exchange it for a library session, report success or failure. **Entry:** library tab with no session. **Platforms:** both.
**Layout:**
```
┌──────────────────────────────────────────┐
│ Sheet, bg bg_b1                          │
│ [取消]  (iOS inline toolbar / Android TextButton 14sp accent)           │
│        [logo 48 r8] ⛓ [book 48 @0.25]    │  row gap 12
│  连接图书馆            title 24sp bold    │  pad top 32 / bottom 4
│  ┌ Column(pad h16 → pad16, r12, bg lightGray, spacedBy 8) ────────────┐ │
│  │ 🌐 从信息门户验证               ▸      │ icon 24 accent; subtitle caption │
│  │   将进入武汉大学信息门户网页验证…       │ │
│  └──────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```
Success / error replace the whole body: Lottie congrats background → centred column with check (or cross) icon `64` on a coloured circle pad `8`, title 24sp, message body 16sp, spacer `16`, button `48` / radius `12` / accent @0.15 / headlineBold accent. (iOS: text pill radius `8`, accent @0.1, `maxWidth 350`.)
**Content blocks, in order:**
1. **Close** — iOS `取消` toolbar item, Android `关闭` text button. Tap: dismiss.
2. **Identity row** — app logo, link glyph, module book icon.
3. **Title** `连接图书馆` (+ optional subtitle, omitted when empty).
4. **Choice card** — `从信息门户验证` / `将进入武汉大学信息门户网页验证你的身份`. Tap: CAS web view (`https://cas.whu.edu.cn/authserver/login?service=…`, cache and cookies cleared first).
5. **Exchange** — full-screen spinner while the CAS ticket is exchanged. Conditional: after a successful CAS login.
6. **Result** — success or error page, one button back to the app.
**Values:**
| Element | Value | Note |
| --- | --- | --- |
| Presentation | modal sheet, bg `bg_b1` | |
| Logo / link / module icon | `48` r`8` / link glyph default / book `48` @0.25 | iOS logo `56` r`12`, row gap `16` |
| Icon row gap / title | `12` / title 24sp Bold, pad top `32`, bottom `4` | iOS 28pt, no explicit padding |
| Choice container | radius `12`, `lightGray` `#EDEEEF`, pad h `16` then `16`, rows spaced `8` | iOS radius `8`, `gray @0.15`, rows spaced `24` |
| Choice row / Result icon / Result stack | icon `24` accent + bodyBold title + caption subtitle + trailing chevron / check `64` on accent circle pad `8` white; error cross `64` on red circle / centred column, pad `16`, gap `8`; slide-up half-height + fade, `500 ms` + haptic | iOS: green check; no error screen at all / iOS gap `4`, pad `32`, spring, `0.8 s` |
| Button | `48` / radius `12` / accent @0.15 / headlineBold accent, h-pad `4` | iOS text pill r`8`, `maxWidth 350`, label `返回` |
| Loading | full-screen centred spinner between CAS success and result | iOS: none — the intro page just waits |
**Strings:** 连接图书馆 · 取消 / 关闭 · 从信息门户验证 · 将进入武汉大学信息门户网页验证你的身份 · 验证成功 · 你可以开始使用图书馆了 · 验证失败 · 完成. (Android has five unused dead strings for web / education / captcha paths — do not build them.)
**States:** **Loading** — block 5. **Empty** — n/a. **Error** — error page; button returns to block 4. **Already CAS-authenticated** — skip the web view, go to block 5. **Dismissed without login** — pop the tab (iOS `0.3 s`; Android shows the sheet after `500 ms`).

#### 3. Book a room (查看房间)
**Purpose:** Choose building → room → time range → seat, then submit. **Entry:** home grid `查看房间`. **Platforms:** both.
**Layout:**
```
┌──────────────────────────────────────────┐
│ Header column, bg gray @0.1, pad-h 12, spacedBy 8,               │
│   pad-top statusBar+42, pad-bottom 12                            │
│  ▸ 请选择图书馆 / <building>   bodyBold  │  chevron rotates 270° │
│  [building chips] h-scroll r8 gray@0.15 │  pad v 6, gap 8, ends 12│
│  [room cards w75] h-scroll r8 gray@0.15 │  pad v 12, gap 8, ends 4│
│  🕐 [08:00] → [22:30]      [今天|明天]   │  HamTab 32            │
├──────────────────────────────────────────┤
│ Body Box(fillMaxSize)                    │
│  LazyVerticalGrid(Fixed 4), gaps 8/8,    │
│   contentPad top 56, bottom 96+nav, h16  │
│  [⚡][☀] filter at offset (16,16)        │
│  Confirm banner (bottom, slides up)      │
└──────────────────────────────────────────┘
```
**Content blocks, in order:**
1. **Building selector** — title button toggling the chip row. Conditional: chip row visible while no building is selected or the user expanded it. Tap: toggle.
2. **Building chips** — icon + name each. Conditional: block 1 expanded. Tap: select.
3. **Room cards** — floor, `free/total`, name. Conditional: a building is selected. Tap: select, auto-scroll into view.
4. **Time row** — begin chip, arrow, end chip, day segmented control. Tap: chips open a 30-minute time dialog.
5. **Seat grid** — 4 columns. Conditional: room selected and list loaded. Tap: select seat.
6. **Filter buttons** — power ⚡, window ☀. Conditional: room selected. Tap: toggle (accent on, gray off).
7. **Confirm banner** — `已选择` / `seat | location` / `day HH:mm-HH:mm` + reserve button. Conditional: seat selected. Tap: submit.
8. **Result screen** — success card (check icon, title, info card with seat + location + `yyyy-MM-dd HH:mm-HH:mm`, done button) or error screen.
**Values:**
| Element | Value | Note |
| --- | --- | --- |
| Header bg / padding / Title / chevron | `gray @0.1`; top `statusBar+42`, h `12`, bottom `12`, inner `8` / bodyBold / chevron `8` accent, rotate `0`→`270` | iOS `bg_b1`, `.padding([.h,.t])` 16, no inner spacing |
| Chip row / Room row | bg `gray @0.15`, radius `8`, pad v `6`, gap `8`, ends `12`, icon `16`, caption / bg `gray @0.15`, radius `8`, pad v `12`, gap `8`, ends `4`, card width `75` | iOS: bg @0.2, pad v `8`, ends `16`, per-building SF icons / iOS card width `80` |
| Room card / free-total colour | icon + `NF` captionBold + `\|` + `free/total` caption + name caption centred; `<=5` red, `<=10` orange, else green | iOS: binary `free==0 ? red : green` |
| Time row | begin/end chips (radius `8`, pad v `6`/h `8`) + `HamTab` height `32` | iOS: inline `UIDatePicker`, chip width `85`, row height `50` |
| Time bounds / Seat grid | begin `08:00`–`23:00`; end `08:30`–`23:30`; default end `22:30`; today clamps to the next half hour; `minEnd = begin + 30 min` / 4 fixed columns, gaps `8`/`8`, pad top `56`, bottom `96 + navBar`, h `16` | iOS `LazyVGrid` adaptive `minimum 70`, spacing `10`, insets `48`/`128` |
| Seat cell / Power / window bars | radius `8`, bg `accent @0.15` selected else `gray @0.15`, icon `20` + number title3, pad v `4`/h `6` / `4`-tall full-width strips, green / orange | iOS: bg @0.2, body bold 17, default-size icon |
| Filter buttons | icon `32` on `colour @0.25`, pad `6`, radius `8`, container `bg_b1 @0.95`, offset `(16,16)` | iOS `36×32` + blur material |
| Confirm bar / CTA | `bg_b1 @0.95` + `gray @0.2`, radius `8`, pad `8`, h-pad `8`, v-pad `16` / `预约`, bodyBold white, pad v `4`/h `16`, radius `8`, accent, fills height | iOS: blur material, radius `8`, bottom safe area / iOS: pad v `12`/h `16` |
| Placeholder icon / Skeleton / retry | `64`, `gray` / centred spinner; retry button `128` wide, radius `8`, `gray @0.2`, pad v `16` | iOS: `96` at `gray @0.45` / iOS: 48 redacted cells, no retry button |
**Strings:** 请选择图书馆 · 请选择房间 · 今天 / 明天 · 已选择 · `%1$s %2$s-%3$s` · 预约 · 没有多余的座位了 · 预约成功 · 预约失败 · 完成 · 预约信息 · `%1$s%2$d楼%3$s` · 重试 · 遇到了错误 · 未设置. (iOS says `没有多余的座位`. Building names come from the server; Android maps them to `library_info_building`, `library_medical_building`, `library_main_building`, `library_engineering_building`.)
**States:** **Loading** (seat list) — centred spinner. **Empty** (no room) — icon + `请选择房间`. **Empty** (no seats) — icon + `没有多余的座位了`. **Empty** (filters match nothing) — same view. **Error** — icon + message + `重试`. **Booking** — CTA swaps to an inline spinner; header and grid stay. **Success** — result screen replaces everything. **Booking error** — toast, stay. **Token expired** — toast only.

#### 4. Select seat (选择座位)
**Purpose:** Pick a seat and return it to the caller. **Entry:** quick-book seat row, starred-seat screens. **Platforms:** both.
**Layout:**
```
┌──────────────────────────────────────────┐
│ Scroll, pad 16, gap 16                   │
│ ─ Last-booking shortcut card (optional) ─│
│ ─ Starred-seat shortcut card (optional) ─│
│ ─ HamCardView 16/16 ──────────────────── │
│   房间                        bodyBold   │
│   [building chips] h-scroll, gap 8       │
│   [room cards w60] r8 gray@0.15          │
│   ── divider ──                          │
│   座位                        bodyBold   │
│   [⚡ 40×32 r8][☀ 40×32 r8]    …   [↻]   │
│   FlowRow(maxH 400, gaps 12/12, centred) │
│   选择该座位                  body accent│
└──────────────────────────────────────────┘
```
**Content blocks, in order:**
1. **Last-booking shortcut** — seat number `36sp` bold + `上次预约` badge + location + time + select link. Conditional: a last booking exists and isn't the already-selected seat. Tap: return that seat.
2. **Starred-seat shortcut** — per entry: seat number, location, weekday chip, select link. Conditional: starred seats configured. Tap: return that seat.
3. **Room section** — building chips + room cards. Tap: select building / room.
4. **Seat section** — filter buttons (power, window) + refresh + seat grid. Tap: toggle filters; refresh re-fetches; seat selects.
5. **Confirm** — text button. Conditional: a seat is selected. Tap: return the seat and pop.
**Values:**
| Element | Value | Note |
| --- | --- | --- |
| Container | scroll view, pad `16`, card gap `16` | iOS: raw `VStack` + bottom overlay, no card, no nav title |
| Card | `HamCardView` `16`/`16` | iOS has no card around the picker |
| Building chip | icon `16` + caption; accent selected, else `text_t2` | iOS: per-building SF icons |
| Room card | width `60`, radius-`8` `gray @0.15` row, pad v `12`, ends `4`; icon + `NF` captionBold + `free/total` + name caption centred | iOS width `80`, ends `8`, `\|` prefix |
| Room `free/total` colour | `<=5` red, `<=10` orange, else green | iOS binary red / green |
| Filter buttons | `40×32`, radius `8`, `colour @0.25`, pad h `2`/v `4`; power green, window orange | iOS `36×32` + blur material, no refresh |
| Refresh / Seat grid | `40×32`, `gray @0.15`, tint `gray` / `FlowRow`, gaps `12`/`12`, `maxHeight 400`, centred, container `gray @0.15` radius `8` pad v `8` | iOS: absent / iOS `LazyVGrid` adaptive `70`, spacing `10` |
| Seat number / Attribute badges / weekday chip | `24sp` bold, accent selected else `text_t2` / 14 dp icons on a `6`-radius pill, `(accent\|gray) @0.25`, pad v `4`/h `4`; `ham_yellow`, bg `@0.2`, radius `6`, pad `4`, body | iOS body bold 17 / iOS: 4-tall colour strips; iOS: absent |
| Confirm | inline text button, body accent | iOS: bottom blur bar, `已选择` + `选择` |
| Placeholders | distinct strings for no-room / empty-room / filtered-empty | iOS renders one identical view for all three |
| Loading | centred spinner | iOS: 48 redacted cells |
**Strings:** 选择座位 · 房间 · 座位 · 选择该座位 · 选择房间继续 · 该房间暂无座位 · `筛选条件下无座位，共剩余%1$d个座位` · 首选座位 · 上次预约 · `%1$s预约` · 每天 · 周日…周六 · `%1$dF` · 遇到了错误.
**States:** **Loading** — centred spinner. **Empty** (no room) — `选择房间继续`. **Empty** (room has no seats) — `该房间暂无座位`. **Empty** (filters match nothing) — `筛选条件下无座位，共剩余%1$d个座位`. **Error** — `遇到了错误` + toast, refresh available. **No time range** — this screen lists the room's full layout; do not send or apply availability filtering. **Confirmed** — write the seat to the caller's return channel and pop.

#### 5. Select time (选择预约时间)
**Purpose:** Edit begin time, end time and the tomorrow flag. **Entry:** quick-book time row, starred-seat duration. **Platforms:** both.
**Layout:**
```
┌──────────────────────────────────────────┐
│ Nav title "选择预约时间" (inline / 36dp)  │
│ Column(pad 16, spacedBy 16)              │
│  Row(gap 8): [begin chip] - [end chip]   │
│              ␣  预约明天        [Switch] │
│  ┌────────────────────────────────────┐  │
│  │ 确定  white, 48, r12, accent@0.85  │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```
**Content blocks, in order:**
1. **Begin chip** — `HH:mm` or `未设置`. Tap: time dialog, 30-minute interval.
2. **Separator** `-`.
3. **End chip** — same, bounded by `begin + 30 min`. Tap: time dialog.
4. **Tomorrow toggle** — `预约明天` + switch (track green on / `lightGray` off). Tap: toggle; switching to today raises begin to the next half hour.
5. **Confirm** — writes `{begin, end, tomorrow}` to the caller's return channel and pops.
**Values:**
| Element | Value | Note |
| --- | --- | --- |
| Container | `Column`, pad `16`, gap `16` | iOS: `ScrollView` + leading `VStack`, `.padding()` 16, gap `0` |
| Section labels / Dividers | none beyond the `-` separator / none | iOS has `开始时间` / `结束时间` labels / iOS: two `Divider().padding(.vertical)` |
| Time control | chip (radius `8`, pad v `6`/h `8`) opening a 30-minute dialog | iOS: `UIDatePicker` `.wheels`, locale `en_GB` |
| Layout / Tomorrow control | both pickers on one row separated by `-` / switch on the same row as the pickers | iOS stacks and centres each picker / iOS: `Toggle` below a divider |
| Spacer before CTA / CTA / time bounds | `16` / `确定`, headlineBold `14sp` white, `48`, radius `12`, accent @0.85, h-pad `12` / begin `08:00`–`23:00`; end `08:30`–`23:30`; default end `22:30` | iOS `30` / `确认`, radius `10`, `.padding()` |
**Strings:** 选择预约时间 · 开始时间 · 结束时间 · 预约明天 · 确定 · `-` · 未设置. (iOS labels the button `确认` — standardise on `确定`.)
**States:** **Loading / error** — none; purely local state. **Begin moved past end** — push end to `begin + 30 min`. **End moved before `begin + 30 min`** — revert end. **Tomorrow toggled** — begin min becomes `08:00` (tomorrow) or next half hour (today); raise begin if below. **Dismissed** — pop without writing; caller keeps its previous values.

#### 6. Quick book (快速预约)
**Purpose:** Fire the smart-book call for the pre-configured seat and time, then report the outcome. **Entry:** home quick-book reserve button. **Platforms:** both.
**Layout:**
```
┌──────────────────────────────────────────┐
│ bg bg_b1, fill                           │
│ ─ loading ─  animation 200 tall          │
│   正在快速预约            title 24sp      │
│   请稍等                  body 16sp      │
│ ─ success ─  booking success card (§3/8) │
│ ─ error ─   cross 64 on red circle       │
│   快速预约失败            title          │
│   <errorMessage>          body           │
│   [ 返回  48 r12 accent@0.85 ]           │
└──────────────────────────────────────────┘
```
**Content blocks, in order:**
1. **Loading** — animation `200` tall full width, title, subtitle, spacer `16`. Conditional: call in flight.
2. **Success** — shared booking success card (check icon `64` on accent circle pad `8`; title; info card with seat + location + `yyyy-MM-dd HH:mm-HH:mm`; done button), plus a `座位发生了变更` caption when the server assigned a different seat. Conditional: success. Tap: done pops.
3. **Error** — cross icon, title, message, back button. Conditional: failure. Tap: button pops.
**Values:**
| Element | Value | Note |
| --- | --- | --- |
| Background / transition | `bg_b1` / crossfade | iOS: no transition |
| Loading indicator | Lottie `lottie_flying.json`, `200` tall, full width | iOS: linear `ProgressView` with synthetic fill |
| Loading text | title `24sp` + body `16sp`, column pad `16`, gap `8` | iOS: progress-bar label only, hardcoded `正在预约` / `很快就好` |
| Nav title (loading) | `快速预约` | iOS: none |
| Success icon / Seat-changed hint | check `64` on accent circle, pad `8`, white / caption under the title | iOS: white-on-green / Android: absent |
| Error icon / title / detail / Error CTA | cross `64` on red circle pad `8` / title `24sp` / body `16sp` / `返回`, headlineBold white, `48`, radius `12`, accent @0.85, h-pad `12` | iOS: `xmark.circle.fill` 64; title + caption / iOS: bold text pill, radius `12`, accent @0.15, `maxWidth 350` |
| Re-entry guard | start the call only when state is unload, once | iOS also never resets its progress value |
**Strings:** 快速预约 · 正在快速预约 · 请稍等 · 快速预约失败 · 预约成功 · 座位发生了变更 · 完成 · 返回.
**States:** **Loading** — block 1. **Success (same seat)** — block 2 without the hint. **Success (seat reassigned)** — block 2 with `座位发生了变更`. **Error** — block 3. **Token expired** — toast only. **Empty** — n/a; the caller always supplies a target seat.

#### 7. Modify booking (变更预约)
**Purpose:** Change an existing booking's time window, or cancel / end it. **Entry:** home booking card `变更或取消预约`. **Platforms:** both.
**Layout:**
```
┌──────────────────────────────────────────┐
│ Nav title "变更预约" (inline / 36dp)      │
│ Scroll, pad 16, gap 16                   │
│ ┌ HamCardView(title = 更改预约时间) ────┐ │
│ │ 开始时间                    [chip]    │ │  row gap 8
│ │ 结束时间                    [chip]    │ │
│ │ [ 更改时间  48 r12 accent@0.85 ]      │ │
│ └────────────────────────────────────────┘ │
│ ┌ HamCardView(title = 取消预约) ─────────┐ │
│ │ ┌────────────────────────────────────┐ │ │  48 tall, r12, gray@0.15
│ │ │ 滑动取消预约 ▸  ← drag → red fill  │ │ │
│ │ └────────────────────────────────────┘ │ │
│ └────────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```
**Content blocks, in order:**
1. **Time card** — labelled `开始时间` / `结束时间` rows with 30-minute chips, spacer `8`, then the change button. Tap: chips open the time dialog; button submits.
2. **Cancel card** — swipe-to-confirm slider. Tap: drag right past the threshold to cancel (reserve) or end (check-in / away).
**Values:**
| Element | Value | Note |
| --- | --- | --- |
| Container / card | scroll, pad `16`, gap `16`, bg `bg_b1` / `HamCardView` `16`/`16`, title bodyBold | |
| Time rows / Pickers | label body + chip, rows spaced `8` / 30-minute chips; bounds seeded from the booking (`minBegin = begin`, `maxEnd = end`, `maxBegin = end − 30 min`, `minEnd = nextHalfHour(begin)`) | iOS: bare pickers, no labels / iOS: inline `UIDatePicker`, width `80`, row `minHeight 50`, leading pad `-8` |
| Change CTA / Disabled CTA | `更改时间`, headlineBold white, `48`, radius `12`, accent @0.85, h-pad `12` / `enabled = false`, fill `gray` | iOS: `更改`, radius `10`, accent + blue shadow r`3` y`2` / iOS also `.opacity(0.5)` |
| Slider | `48` tall, radius `12`, bg `gray @0.15`, red fill, fling threshold `0.85` of distance | iOS: `50`, radius `10`, bg `gray @0.2`, threshold `95`, dynamic shadow and `1 + %*0.05` scale |
| Slider label | `滑动取消预约` bodyBold red + red chevron, start pad `32`, gap `8` | iOS: `滑动以取消预约`, `.padding(.trailing)` only |
| Loading | centred spinner in place of the button / slider | iOS keeps the button visible with an inline spinner |
| Feedback | toast `操作成功` + `已更改预约时间` / `已取消预约`, then refresh home data and pop | |
| Interlock | block the other action while one is in flight | iOS: no interlock |
**Strings:** 变更预约 · 更改预约时间 · 开始时间 · 结束时间 · 更改时间 · 取消预约 · 滑动取消预约 · 已更改预约时间 · 已取消预约 · 操作成功. (iOS reaches `更改` / `变更预约` through the `CONFIRM` / `MODIFY_RESERVATION` keys — use the Chinese values.)
**States:** **Pristine** — change button enabled from the start (no dirty check). **Change in flight** — spinner replaces the button. **Change success** — toast, refresh, pop. **Change error** — toast, button returns. **Cancel in flight** — spinner replaces the slider. **Cancel success** — toast, remove any calendar event, refresh history, pop. **Cancel error** — toast, slider returns. **Non-actionable booking** — no-op, no feedback.

#### 8. History (历史预约)
**Purpose:** Show past bookings grouped by day with a per-day study total. **Entry:** home grid `历史预约`. **Platforms:** both.
**Layout:**
```
┌──────────────────────────────────────────┐
│ Nav title "历史预约" (inline / 36dp)      │
│ Retry-login banner (conditional, pad 16) │
│ Scroll, bg bg_b1, pad-h 16               │
│  per day cluster:                        │
│   今天 \| yyyy-MM-dd  -  共计120分钟      │  caption
│   ┌────────────────────────────────────┐ │  bg bg_b2, r16
│   │▌│ 012      │              08:00    │ │  12 status rail
│   │ │ 履约中    │              10:00    │ │  seat col 48, caption
│   │ │ 信息馆3楼…│                       │ │  trailing pad 16
│   └────────────────────────────────────┘ │
│   gap 16 between clusters                │
└──────────────────────────────────────────┘
```
**Content blocks, in order:**
1. **Retry-login banner** — red card. Conditional: token invalid. Tap: re-login then refetch.
2. **Day clusters** — date header (or `今天`) plus `共计%lld分钟` when greater than zero, then one card of rows. Conditional: data present.
3. **Rows** — status rail, seat number + status word, location, begin over end.
**Values:**
| Element | Value | Note |
| --- | --- | --- |
| Background / padding | `bg_b1` / pad-h `16` | |
| Cluster card | bg `bg_b2`, radius `16`; clusters separated by `16` | Android: one `HamCardView` for the whole list, rows spaced `8`, dividers between rows |
| Date header / Status rail | caption `12` / `12` wide, status colour, `@0.15` when unknown | Android: no grouping at all / Android: no rail — status is a coloured word |
| Seat column | width `48`, number body bold, status caption in the status colour | Android: `fillToConstraints`, status body 16sp |
| Location / times / Row padding / total | caption multiline leading / begin stacked over end, caption, trailing pad `16` / v `4`; total counts check-in and away (begin→now) and stop (end−begin) | Android: single `date begin-end` line / Android: no row padding |
| Loading | centred spinner filling the screen | Android: no loading state — empty looks identical to loading |
| Error banner | red retry card + `16` padding | iOS shows it for every error class, not just token expiry |
**Strings:** 历史预约 · 没有历史记录 · 今天 · `共计%lld分钟` · 未知. (iOS hardcodes the first two; Android's title reads `历史记录` and its empty copy `暂无历史记录` — standardise on the values above. Android's `library_total_study_minutes` / `library_total_study_note` are unused.)
**States:** **Loading** — centred spinner. **Empty** — `没有历史记录`, `text_t2`, full width. **Error** (generic) — toast only. **Token expired** — block 1. **Total unavailable** — omit the `- 共计N分钟` suffix.

#### 9. Settings (设置)
**Purpose:** Account, captcha notice, starred seat, local cache data, statistics, diagnostics. **Entry:** home grid `设置`. **Platforms:** both.
**Layout:**
```
┌──────────────────────────────────────────┐
│ Nav title "设置" (inline / 36dp)          │
│ Scroll, bg bg_b1, pad 16, gap 8          │
│ 1 账号信息 / 用以预约座位、获取预约信息   │  watermark person 156 @0.15
│   学号  <userId> (label w65)             │
│   [ 登录  48 r12 accent@0.85 ]           │
│ 2 验证码识别设置 / 查看你的验证码账号…    │  watermark Settings 200
│   已默认开启验证码识别                   │
│ 3 收藏座位 / 使用快速预约或自动预约时…    │  watermark Star 250 offsetX 48
│   [每天] 012  信息馆3楼…  (or 未设置)    │
│   去设置                                 │
│ 4 本地数据 / 本地存储的图书馆数据可以…    │  watermark cloud 128 offsetY 12
│   数据日期: 暂无             caption     │
│   🏛 从图书馆更新数据  📄 复制当前数据    │
│ 5 统计服务 / 查看图书馆可用程度          │  conditional on remote flag
│   开启统计服务                  [Switch] │
│ 6 其他 / 更改基础数据、诊断账号状态      │  watermark device 250 offsetX 48
│   登录序列号  ******…        caption     │
│   重置账号状态 → 确认重置账号状态   red  │
└──────────────────────────────────────────┘
```
**Content blocks, in order:**
1. **Account** — subtitle, `学号` + value, login button. Tap: re-login.
2. **Captcha** — subtitle, note. Tap: none.
3. **Starred seat** — subtitle, weekday chip + seat + location per entry or `未设置`, link to the starred-seat screen. Tap: link.
4. **Local data** — subtitle, last-update date, update and copy buttons. Tap: refresh the cached library data / copy it to the clipboard.
5. **Statistics** — subtitle, enable switch. Conditional: remote flag on. Tap: toggle.
6. **Other** — subtitle, masked login serial, two-step reset. Tap: first tap arms, second resets the account and pops.
**Values:**
| Element | Value | Note |
| --- | --- | --- |
| Container / card | lazy scroll, pad `16`, card gap `8` / radius `16`, pad `16`, bg `bg_b2`, title bodyBold, subtitle caption `text_t2` | iOS: `ScrollView` + `VStack`, pad-h 16, cards adjacent |
| Watermarks | account person `156`; captcha Settings `200`; starred Star `250` offsetX `48`; local-data cloud `128` offsetY `12`; other device `250` offsetX `48`; all `lightGray @0.15`, bottom-leading | iOS offsets captcha / starred / local-data by `y +16 / +16 / +12`; no `其他` card |
| Field label width | `65` (学号, 登录序列号) | iOS: none |
| Primary button / Accent note | `48`, radius `12`, accent @0.85, headlineBold white, h-pad `12` / body `text_t1`, no accent colour | iOS: plain text button `重新登录` / iOS: tinted accent |
| Weekday chip | `每天` white on accent, radius `6`, pad v `2`/h `6` | |
| Link / switch / Action rows | body accent text button / track green on, `lightGray` off / icon `16` + body accent text, gap `8` | iOS: `NavigationLink` / system `Toggle` / iOS: local-data icon resizable `16×16` |
| Token masking | `token.replaceRange(0, 6, "******")`, caption | iOS: absent |
| Loading (local data) | spinner + `正在更新基本信息`, replaces both buttons | iOS: spinner + `正在更新`, replaces the date line and button |
| Data date | `数据日期: %@` caption, `暂无` when unset | iOS always shows `暂无` (never assigned); Android omits the line |
| Destructive / copy | two-step confirm, body red / copy button present | iOS: both absent |
| Statistics gate | card shown only when the remote flag is on | iOS: card always present |
**Strings:** 设置 · 账号信息 / 用以预约座位、获取预约信息 · 学号 · 登录 · 登录成功 · 验证码识别设置 / 查看你的验证码账号、是否启用验证码识别 · 已默认开启验证码识别 · 收藏座位 / 使用快速预约或自动预约时将自动选用收藏座位 · 未设置 · 去设置 · 统计服务 / 查看图书馆可用程度 · 开启统计服务 · 本地数据 / 本地存储的图书馆数据可以让你更快查看图书馆房间 · 正在更新基本信息 · 数据日期: %@ · 暂无 · 从图书馆更新数据 · 复制当前数据 · 更新成功 · 其他 / 更改基础数据、诊断账号状态 · 登录序列号 · 重置账号状态 · 确认重置账号状态 · 遇到了错误 · 操作成功. (iOS copy differs: `你的图书馆系统认证信息`, `设置验证码账号信息、是否使用验证码`, `该版本已默认开启验证码识别功能`, `重新登录` — use the values above.)
**States:** **Login in flight** — spinner + label, button disabled. **Login success / failure** — toast. **Starred seat set / unset** — entry rows vs `未设置`. **Local data refreshing** — spinner replaces the buttons. **Local data date** — real last-update date. **Statistics off** — card hidden. **Reset** — two-step, then reset and pop. **Empty / error** — toasts only.

#### 10. Starred seat settings (首选座位设置)
**Purpose:** Configure the seats, time window and attribute filters used by quick and automatic booking. **Entry:** Settings `收藏座位`. **Platforms:** both.
**Layout:**
```
┌──────────────────────────────────────────┐
│ Nav title "首选座位设置" (inline / 36dp)  │
│ 1 Explain: 首选座位是什么 + description  │  watermark bulb 128
│ 2 HamCardView                            │
│   座位列表                    [✎] [＋]   │  bodyBold
│   012                bodyBold            │
│   信息馆3楼…          caption (1 line)   │
│   08:00-22:30         caption            │
│                     [每天]  [删除]       │
│   未添加座位           caption t2        │  when empty
│ 3 偏好 / 当首选座位被占用时…              │
│   ⚡ 电源                      [Switch]  │
│   ☀ 靠窗                      [Switch]   │
│ 4 [ 确定  48 r12 ]        (insert only)  │
└──────────────────────────────────────────┘
```
The insert sub-screen adds three cards above the submit button: **seat** (number title 24sp + location body + change/choose link), **duration** (`开始时间` / `结束时间` chips), **weekday** (seven `32` squares plus `每天`).
**Content blocks, in order:**
1. **Explainer card** — what a starred seat is. Conditional: always.
2. **Seat list** — header with edit toggle and add button; one row per entry (seat, location, time, weekday chip, delete when editing). Conditional: rows only when configured. Tap: add → insert screen; pencil → edit mode; delete → remove.
3. **Preferences** — power and window switches. Tap: toggle.
4. **Insert form** (sub-screen) — seat, duration, weekday, submit. Tap: seat → Select seat; chips → time dialog; weekday chips → select; submit → validate and commit.
**Values:**
| Element | Value | Note |
| --- | --- | --- |
| Container / card | lazy scroll, pad `16`, gap `16` / `16`/`16`, title semibold, subtitle caption `text_t2` | iOS: `ScrollView` + `VStack` `.padding()` 16 |
| Seat row | number bodyBold + location caption 1-line ellipsis + time caption | iOS: number body bold 17, location body multiline, pickers inline on the parent screen |
| Empty CTA | `未添加座位` (list) / `去选择`, `更改` (insert) | iOS: `设置座位` placeholder inside the seat card |
| Time pickers | chips + 30-minute dialog, bounds `08:00`–`23:00` both ends; labels `开始时间` / `结束时间` body | iOS: inline `UIDatePicker`, frame `80×50`, leading pad `-8`, no labels |
| Attribute toggles | switch + `电源` / `靠窗` body + 24 icon (green / orange) | iOS: `Toggle` + `仅` + `title3` icon |
| Weekday chip | `ham_yellow`, bg `@0.2`, radius `6`, pad v `4`/h `6`, body | iOS: single every-day entry, no chip UI |
| Weekday selector / Edit affordances | `32` squares, radius `8`, bg `colour @0.2`, headlineBold, `FlowRow` gaps `8`/`4` / pencil `24`, add `28`, `取消编辑` text, `删除` body red | iOS: absent / iOS: absent |
| Explainer card / ordering | present, bulb watermark `128` / `weekday * 1000 + (beginTime − 08:00)/60000` | iOS: absent / n/a |
| Save affordance | explicit `确定`, `48`, radius `12`, accent @0.85, disabled `gray @0.5` | iOS: no button — every change writes through immediately, `已自动保存` caption centred, `16` top padding |
| Conflict handling | reject overlapping entries (same weekday, or an every-day entry, with overlapping times) with a toast | iOS: n/a |
**Strings:** 首选座位设置 · 添加首选座位 · 首选座位是什么 · 座位 · 座位列表 · 未添加座位 · 持续时间 · 开始时间 · 结束时间 · 星期 · 每天 · 日…六 · 偏好 · 电源 · 靠窗 · 取消编辑 · 删除 · 更改 · 去选择 · 确定 · 存在时间冲突的座位 · 首选座位.
**States:** **Loading / error** — none. **Empty** — `未添加座位`; the add button stays visible. **Seat configured** — entry row with edit affordances. **Seat picked** — returned through the caller's channel and written immediately. **Time invalid** (end ≤ begin) — submit disabled, fill `gray @0.5`. **Overlapping entry** — toast, insert rejected. **Editing while empty** — pencil hidden, only `＋`

#### 11. Board / announcements (图书馆公告)
**Purpose:** Render the library notice-board HTML. **Entry:** home banner announcements cell. **Platforms:** both.
**Layout:**
```
┌──────────────────────────────────────────┐
│ Nav title "图书馆公告" (36dp, accent     │
│   back chevron, 16dp start margin)       │
│ Scroll, pad-h 16, top-leading            │
│  <server HTML rendered as rich text>     │
└──────────────────────────────────────────┘
```
**Content blocks, in order:**
1. **Notices** — the server HTML rendered as rich text. Conditional: always. Tap: links follow the platform default.
**Values:**
| Element | Value | Note |
| --- | --- | --- |
| Container / Renderer | scroll, pad-h `16`, top-start aligned / platform rich-text HTML renderer; typography and colour come from the payload | iOS: `NSAttributedString` HTML → `AttributedString`; Android: `Html.fromHtml(…, FROM_HTML_OPTION_USE_CSS_COLORS)` in a `TextView` |
| Nav title / Loading | `图书馆公告` / centred spinner | iOS: none / both platforms render blank until the HTML arrives |
**Strings:** 图书馆公告 · 遇到了错误.
**States:** **Loading** — centred spinner. **Empty** — no-notices placeholder. **Error** — toast; do not fail silently.

#### 12. Print (打印)
**Purpose:** Send a file to a library printer and list the available print stations. **Entry:** home print card. **Platforms:** Android only — **iOS must add this screen and its home entry point**.
**Layout:**
```
┌──────────────────────────────────────────┐
│ Nav title "打印" (36dp)                   │
│ Column(pad-h 16), spacer 16              │
│ ┌──────────────────────────────────────┐ │  168 tall, r16, accent@0.15
│ │       ＋ 72 accent                   │ │  centred
│ │    添加文件      body accent         │ │
│ └──────────────────────────────────────┘ │
│ 或者，通过其他APP（例如QQ/微信）分享…    │  caption t2, pad v 8
│ spacer 36 │ 打印机位置 bodyBold │ sp 8   │
│ ── AnimatedContent(state) ──             │
│  Loading: [spinner] 加载中               │
│  Error:   加载异常                       │
│  Success: LazyColumn(gap 8, pad-b 16)    │
│   ┌────────────────────────────────────┐ │  r16, gray@0.15, pad 16
│   │ 🖨 48 │ <name>  bodyBold, 2 lines   │ │
│   │       │ <stat>  caption, 3 lines   │ │
│   └────────────────────────────────────┘ │
│   (empty) 没有找到打印机                 │
└──────────────────────────────────────────┘
```
**Content blocks, in order:**
1. **Add file** — large drop zone. Conditional: always. Tap: system document picker (`*/*`), then hand the URI to the print flow.
2. **Share hint** — caption. Conditional: always.
3. **Section header** `打印机位置`. Conditional: always.
4. **Printer list** — one row per station (icon, name, status). Conditional: loaded.
5. **Empty row** `没有找到打印机`. Conditional: loaded but zero stations.
**Values:**
| Element | Value |
| --- | --- |
| Content padding / top spacer / Drop zone | `16` / `16` / height `168`, radius `16`, fill accent @0.15, centred column |
| Add icon / label / Share hint | `72` accent / body accent / caption `12`, `text_t2`, pad v `8` |
| Spacers around header | `36` before, `8` after |
| Header / Loading row | bodyBold, `text_t1` / spinner + `加载中` body, gap `2` |
| Printer row | radius `16`, fill `gray @0.15`, pad `16`, gap `8` |
| Printer icon / Printer name / status | `48`, `gray` / bodyBold 2 lines / caption 3 lines, both ellipsis |
| List spacing | `8`, bottom content pad `16` |
| Empty row / Transitions | body `16` / `AnimatedContent` crossfade |
| Picker | `ActivityResultContracts.OpenDocument()`, MIME `*/*` |
**Strings:** 打印 · 在图书馆公共打印机打印 · 添加文件 · `或者，通过其他APP（例如QQ/微信）分享文件到Ham中打印` · 打印机位置 · 加载中 · 加载异常 · 没有找到打印机.
**States:** **Loading** — spinner + `加载中`. **Loaded** — printer rows. **Loaded, empty** — `没有找到打印机` below the header. **Error** — `加载异常` plus a toast carrying the real message. **File picked** — hand the URI to the print flow. **Picker cancelled** — stay. **Recomposition** — refresh the printer list on entry only, not on every recomposition.

##### Divergences
- **Home banner:** iOS 200 pt pager, N remote banners, red `boardTip`, 5 s auto-advance; Android 180 dp single static page, no tip, no remote banners.
- **Home grid:** iOS 150 pt / three entries vs Android 142 dp / four (adds Print); cell fill accent @0.13 vs @0.15.
- **Home quick book:** visibility differs (iOS: starred-or-last seat exists; Android: a seat is currently selected); seat 28pt vs 32sp; iOS has coloured badges, Android none; CTA text pill r10 @0.1 vs 48 dp bar r12 @0.25.
- **Home analytics:** iOS bars 30 pt, row width hard-coded 216, working consent card; Android bars 20 dp, `weight(1f)`, consent card unreachable (`if (canUse == null)` on a non-null Boolean).
- **Home booking actions:** Android's `在地图打开` is a dead button; iOS has no `添加到系统日历` (commented out); status strip v-pad 8 vs 12.
- **Retry-login copy:** iOS hardcodes `登录信息过期` for every failure class; Android reports the real error and has an in-banner loading state.
- **Status colours:** iOS maps `STOP` green and `MISS`/`LEAVE_EARLY` red; Android omits `STOP` (falls through to `text_t2`) on home and history.
- **History:** iOS groups by date with per-day totals and a 12 pt status rail; Android is a flat id-descending list with coloured status words and no totals. Titles differ (`历史预约` vs `历史记录`); empty copy differs (`没有历史记录` vs `暂无历史记录`).
- **Book:** iOS skeleton (48 cells) vs Android plain spinner; iOS has no retry button; placeholder icons 96 @0.45 vs 64 gray; chip pad v 8 / ends 16 vs 6 / 12.
- **Select seat:** iOS `LazyVGrid` adaptive 70 + bottom blur footer, no card, no nav title, no refresh; Android `FlowRow` 48 dp cells + inline text button + starred/last shortcut cards + refresh; room card 80 vs 60.
- **Select time:** iOS stacks wheel pickers with labels and `确认`; Android puts both chips on one row with `确定`.
- **Modify:** iOS dirty-checks the change button, has no action interlock, uses a 95 %-threshold scaling slider; Android always enables, interlocks, uses an 0.85 fling threshold; labels `更改` vs `更改时间`.
- **Settings:** iOS has no `其他` card, no copy-data button, no token masking, no reset; Android has no data-date line; three card strings differ (see §9).
- **Starred seat:** iOS stores one every-day entry with inline pickers and auto-save; Android stores a per-weekday list with an insert screen, weekday grid and conflict check.
- **Board:** iOS has no title and fails silently (`catch { return }`); Android has a title and toasts.
- **Print:** Android only — iOS has no screen, no home entry, no strings.
- **Strings:** iOS hardcodes ~30 library labels that already exist in `Localizable.strings`; Android routes everything through `strings.xml` (exceptions: `"-"`, `"未设置"`, unused `"向右滑动解锁"`).
- **Dead code:** iOS `LibraryPCLoginWebView`, `LibraryEducationQuickLoginView`, `LibraryBannerImageCover`, `LibrarySettingViewMoreCard`, `LibraryPreferredSeatSettingView2`, write-only `currentBookingLoadState`; Android `library/login/page` route (never registered), `LibraryMainViewPrintStatusCard`, and 11 unused strings (`library_verify_with_web`, `library_verify_from_education`, `library_verify_from_education_hint`, `library_education_auth_not_found`, `library_input_captcha_verify`, `library_reservation_failed`, `library_print_tasks`, `library_file_placeholder`, `library_total_study_minutes`, `library_total_study_note`, `library_agree`/`library_disagree`).


---

### Sport

#### Home (运动)
**Purpose:** Landing tab — bulletin banner, live orders, one-tap quick order, three entry tiles. **Entry:** Sport tab. **Platforms:** both.

**Layout:**
```
┌─────────────────────────────────────────────┐ nav 运动 (.inline)
│ ScrollView · bg b1 · pad 16 · block gap 8   │
│ ┌ 1 Banner ──────── h200 r16 bg b2 ───────┐ │ pager · 5 s auto-advance
│ ├ 2 CurrentOrderCard ─ 0..n ──────────────┤ │ status strip 32 + body
│ ├ 3 QuickOrderCard ─── if starred ────────┤ │ court · 今天|明天 · CTA
│ ├ 4a 查看场馆 (h160) ┬ 4b1 订单中心  ~76 ─┤ │ gap 8
│ │                    └ 4b2 设置      ~76 ─┤ │
│ └─────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

**Content blocks, in order:**
1. **Banner** — pager: fixed decorative page 0 + up to 5 bulletins. Page 0 = centred `sportscourt.fill`/`Stadium` 72 in brand + `场馆预约公告` over a 6 × 9 decorative icon grid; dots only when the list is non-empty; auto-advance every 5 s. Tap: page 0 → bulletin list; page n → detail.
2. **Current orders** — one card per live order: status strip, court badge, venue line, time line, pay block when unpaid. Conditional: section omitted when empty. Tap: → order/pay.
3. **Quick order** — remembered court: badge, 收藏 chip, sport type, court/time, 今天/明天 picker, CTA. Conditional: omitted with no starred config. Tap: CTA → quick-order flow.
4. **Function card** — 查看场馆 → select-area & order · 订单中心 → order centre · 设置 → settings.

**Values:**

| Element | Value | Note |
|---|---|---|
| Screen padding | 16 h/v, block gap 8 | i: no v-pad, no gap token |
| Banner h/r/bg · grid | 200 / 16 / b2 · 6 × 9 @ 36, gap 8, `gray@0.2` | a: 180 · 5 × 10 @ 56, no gap |
| Banner title | `title` bold, brand | i: `.title.bold()` = 17pt |
| Bulletin title/date/body | `bodyBold` 1 line / `caption` t2 `yyyymmddhhmm` / `body` 5 lines | a: date not t2, body unlimited |
| Bulletin pad · gap · 查看详情 | 16 all · 4 date→body · `caption` + `arrow.up.right` blue, bottom-trailing | a: no bottom pad, no 查看详情 |
| Page dots | native, when list non-empty | a: hand-rolled 8dp circles |
| Order card · status strip | r16, bg b2, body pad 12 · `bodyBold` white on blue, pad v8 h12 | — |
| Court badge (standalone) | 32 × 32 r6, bg t1, `title2` rounded bold, fg b1 | a: 36 × 36 |
| Venue line · time line | `{sportType} \| {stadium.title}` `caption` · `yyyymmddhhmm-HHmm` | a: `address` · `yyyyMMdd HHmm-HHmm` |
| Court watermark | sport SF Symbol 128, `gray@0.15`, trailing | a: absent |
| Pay row / out of window | red `caption` `请于{x}前完成支付` + `去支付` white on blue, pad h8 v8, r8, `headline` · else `captionBold` `当前未处于可支付时间` + `caption` range, no button | i: 去支付 r6 |
| 收藏 chip · 预约 CTA | `caption` orange on `orange@0.15`, r4, pad h4 v2, star 16 · CTA full width, pad v16, r8, bg `brand@0.25`, fg brand, `bodyBold` | a: chip `@0.25`, star 18 · CTA v12 `@0.20` |
| Function tile | h160 left / ~76 right, gap 8, r16 | i: 150 / ~71 |
| Left tile | icon 64, bg `brand@0.15` r16, title `bodyBold` brand + caret 8, subtitle `caption` centred, pad 16 | a: icon 72; i: subtitle `caption2` |
| Right tile | bg `gray@0.10` r16, pad 16, title `bodyBold` t1 + caret, subtitle `caption`, trailing icon 32 `brand@0.70` | a: icon 56 `@0.65` |

**Strings:** `运动` · `场馆预约公告` · `查看详情` · `查看场馆` / `查看空余的运动场馆预约` · `订单中心` / `你的历史预约记录` · `设置` / `场馆预约选项` · `收藏` · `今天` · `明天` · `预约` · `去支付` · `请于%@前完成支付` · `当前未处于可支付时间` · `请于%@-%@完成支付`. Status names are data-driven.
**States:** loading — none; page 0 only until bulletins arrive. Empty bulletins — page 0, dots hidden. Bulletin error — silent (log only). Empty orders — section omitted.

#### Select area & order (选择场地并预约)
**Purpose:** Book a court — sport type → date → venue → court → contiguous slots → submit from the pinned footer. **Entry:** home 查看场馆. **Platforms:** both. Body is shared with the picker screen.

**Layout:**
```
┌───────────────────────────────────────────────────────┐
│ root VStack(spacing 0) · bg b1 · ignore bottom safe   │
├───────────────────────────────────────────────────────┤
│ ▓▓ PINNED HEADER (outside scroll) · bg gray@0.10      │
│   pad h16, top = safe + toolbar, bottom 8 · gap 8     │
│  ┌ A ──────────────────────────────────────────────┐  │
│  │ 请选择运动类别 / <type>  bodyBold t1 + ▾ caret   │  │ tap toggles B
│  ├ B CHIP STRIP (if expanded ‖ no type) ────────────┤  │ H-scroll
│  │ bg gray@0.15 · r8 · pad v6 · chip gap 20 · h16   │  │
│  │  ┌───┐ ┌───┐ ┌───┐  icon 32 + caption 12         │  │ sel brand / unsel gray
│  │  └───┘ └───┘ └───┘                               │  │
│  ├ C DATE ROW ──────────────────────────────────────┤  │
│  │  ◀ 32×32 │  2026年9月19日  │  ▶ 32×32            │  │ ±1 day
│  └──────────────────────────────────────────────────┘  │
├───────────────────────────────────────────────────────┤
│ ░░ SCROLLING BODY · vertical scroll + bounce           │
│   Column pad v16 · card gap 8 · card pad h16           │
│   ┌─ venue card · bg b2 · r16 · pad 16 ─────────────┐  │ tap = expand
│   │ ┌────────┐ title    bodyBold                    │  │
│   │ │ img    │ address  caption                     │  │
│   │ │104 r12 │ 已闭馆   caption red (status == 0)   │  │
│   │ └────────┘ ¥{low}起 bodyBold             ⌄ ⟨180°│  │
│   │ ── COLLAPSED ───────────────────────────────────│  │
│   │  预约时间段  caption                             │  │
│   │  [08-09] [09-10] [10-11]  chips r8 pad6 @0.25    │  │
│   │ ── EXPANDED ────────────────────────────────────│  │
│   │  court rows · gap 8 · divider between            │  │
│   │  ┌ badge 24×24 r6 ┬ chips + 详情 ▸ (collapsed)   │  │
│   │  │                │ SLOT GRID     (expanded)     │  │
│   │  │ ┌─────┐ ┌─────┐ ┌─────┐ adaptive 110–150     │  │
│   │  │ │08:00│ │09:00│ │10:00│ gap 8 · h56 · r8     │  │
│   │  │ │09:00│ │10:00│ │11:00│                      │  │
│   │  │ │学专 ¥30 余2│ │ …    │ │ …    │             │  │
│   │  │ │  选择 ▸     │ │      │ │      │ bar h15     │  │
│   │  │ └─────┘ └─────┘ └─────┘                      │  │
│   │  └────────────────┴──────────────────────────────│  │
│   └──────────────────────────────────────────────────┘  │
├───────────────────────────────────────────────────────┤
│ ▓▓ PINNED FOOTER — iff ≥1 slot selected                │
│   system thin-material blur · r8 · pad 8 · +bottom safe│
│   已选择           caption                             │
│   {venue}-{n}号场  caption 1 line                      │
│   {date} {start}-{end} caption      ‖ [ 预约 ]         │
└───────────────────────────────────────────────────────┘
```

**Content blocks, in order:**
1. **Sport-type button** — `请选择运动类别` or the selected type, `bodyBold` t1, trailing caret rotated -90° (collapsed) / 0° (expanded). Tap: toggle the chip strip.
2. **Sport-type chip strip** — conditional: visible when expanded **or** nothing selected. Chip = icon 32 + `caption` label; selected brand, unselected gray. Tap: set the type, collapse, clear the slot selection, refetch venues. Resolve icons from the **localised** title, never a zh-Hans literal.
3. **Date row** — ◀ / `yyyy年M月d日` / ▶, step ±1 day; tap the date for the native picker. Changing the date clears the selection and refetches. Default: today, rolled to tomorrow after 18:00.
4. **Venue card (repeating)** — image, title, address, `已闭馆` (status == 0), `¥{low}起`, expand caret. Tap the whole card to toggle expand; expanding fetches that venue's courts for the date.
5. **Court rows (expanded only)** — leading court-number badge; collapsed shows brief chips + `详情 ▸`, expanded shows the slot grid. Conditional: empty → `无可用时间段`; loading → spinner; error → `加载时遇到了错误`. Rows separated by a divider, gap 8.
6. **Slot tiles** — start/end stacked, `学专` badge when student-exclusive, `¥{lightingPrice + price}`, `余{remaining}`, CTA bar. Three visuals: **bookable** brand tint + brand bar + `选择 ▸`; **full** gray tint + gray bar; **selected** solid brand fill, white times + divider + `已选择`, no bar. Tap: no-op when full; else insert/remove, resetting when the court changes; enforce contiguity and a hard cap of `maxAppointTimerPeriod`. When `maxAppointTimerPeriod == 1`, one tap inserts **and submits immediately**, skipping the footer.
7. **Pinned footer** — `已选择` / `{venue}-{n}号场` / `{date} {first.start}-{last.end}` + 预约 CTA. Conditional: hidden with no selection, and hidden entirely while submitting.

**Values:**

| Element | Value | Note |
|---|---|---|
| Header bg / pad | `gray@0.10` / h16, bottom 8, gap 8 | i: transparent, no v-gap |
| Type title · caret | `bodyBold` t1 · caret 8 brand, rotate -90/0 | a: `ArrowRight` rotate 270/0 |
| Chip strip | bg `gray@0.15`, r8, pad v6, chip gap 20, h-pad 16 | i: pad 16 all, gap 8 |
| Chip icon / label · colours | 32 / `caption` 12, gap 8 · selected brand, unselected gray | i: icon 24 |
| Date arrow · format | 32 × 32, r8, bg `brand@0.15`, icon t1 · `yyyy年M月d日` | a: 36; i: system `DatePicker` |
| Venue card · image · 已闭馆 · price | r16, pad 16, bg b2 · 104 × 104, r12, fill-bounds, crossfade · `caption` red, animated in when `status == 0` · `¥{lowPrice}起` format string, `bodyBold` | i: 85 × 85 r8, concatenates hardcoded `起` |
| Expand caret · title→summary · time chip | `chevron.down.circle` 24 t1, rotate 180 expanded · 8 · chip `caption`, pad 6, r8, `brand@0.25` / `gray@0.25` | i: no caret size, gap 16; a: chip h40 `@0.2` |
| Court row gap · badge · 详情 | 8 · 24 × 24, r6, bg t1, `body` rounded bold, fg b1 · `caption` + caret 16 brand, gap 2 | i: gap 16, caret 8; a: badge 28 `title2` fg b2, caret 20 offset -4 |
| Slot grid · tile h / r / bg | adaptive `min 110 / max 150`, gap 8 · 56 / 8 / `brand@0.15` (or `gray@0.15`) unselected, brand @1 selected | a: `FlowRow` fixed 120, tile h50; i: bg 0.25 |
| Slot tile text · bottom bar | `caption` 12 rounded, pad top 4 / h8 · bar h15, fill = tile colour, `caption` white, `选择 ▸` | a: `caption2` 11, no pad, implicit bar |
| 学专 badge · 余 · price | `caption2` bold, fg b2 on t1, r4, pad h4 v2 · `余%d` `caption` · `¥{lightingPrice + price}` | a: `captionBold` white/black no pad · `caption2` · `price` only |
| Selected row | times + divider + `已选择`, all `caption` white | a: white 12dp divider |
| Footer | system thin-material blur, r8, pad 8 + bottom safe | a: b1@0.95 + gray@0.2 |
| Footer text | `已选择` / `{venue}-{n}号场` / `{date} {start}-{end}`, `caption` | a: venue \| address, no court no. |
| CTA · transition | `预约`, white on solid brand, pad v12 h16, r8, fills footer height · slide in/out from bottom + fade | — |

**Strings:** `请选择运动类别` · `预约时间段` · `已闭馆` · `¥%1$s起` · `加载时遇到了错误` · `无可用时间段` · `详情` · `学专` · `余%1$d` · `选择` / `预约` (one-tap variant) · `已选择` · `%@-%lld号场` · `预约`.
**States:** idle — header + body, footer when a selection exists. Submitting — full-screen spinner, header and footer removed. Success — success screen. Error — error screen with retry. Court-list error — inline `加载时遇到了错误`. Slot-detail error — inline `加载时遇到了错误` (a: today; i: toast + collapse). Empty slots — `无可用时间段`.

#### Select / picker (选择场地)
**Purpose:** Same browser; the footer returns a selection to the caller instead of ordering. **Entry:** starred-order setting → 去设置. **Platforms:** both. **Layout:** identical to select-area & order (pinned header / scrolling body / pinned footer), CTA `确定`.
**Content blocks:** 1–6 as above, with these overrides: slot tiles show **times + 学专 only** (no price, no `余`, no dividers), width 100 (order 120); tile colours use real availability (both hardcode `isFree = true` today); tap never submits and has **no** availability guard. **Footer** — `确定`; tap writes `SportSelectAreaViewResult` (date, time list, court detail, venue, sport type) to the caller's saved state and pops; if any field is null, do nothing.
**Values:** all as select-area & order except: tile width 100; bottom bar always brand; tile bg `brand@0.15`; h56 / r8; content `caption` 12; CTA `确定`, pad v12 h16, r8, white on brand. **Strings:** `确定` · `已选择` · `%@-%lld号场` · `选择` + all shared select strings. **States:** loading — centred spinner. Empty venue list — render an empty state (i: renders nothing). Error — none today; reuse `加载时遇到了错误`.

#### Captcha (输入验证码)
**Purpose:** Solve the bundled local HTML captcha and hand a token back to the caller. **Entry:** pushed by the order or quick-order flow when the API demands a challenge. **Platforms:** both.

**Layout:**
```
┌───────────────────────────────────────────────┐
│ ▓▓ nav 输入验证码 (.inline)          [刷新]   │
├───────────────────────────────────────────────┤
│ ░░ WebView — bundled sport-captcha-page.html  │ fills the rest
└───────────────────────────────────────────────┘
```

**Content blocks, in order:** 1. **Title bar** — `输入验证码`; trailing `刷新` → reload. 2. **Captcha web view** — bundled asset; the page posts `{"token": …}` to the native bridge; on receipt write the token to the caller's saved state (or post the notification keyed by request id) and pop. An `autoDismiss = false` variant must exist so the composable can be embedded inline.
**Values:** title system nav · refresh toolbar trailing `body` blue · asset `sport-captcha-page.html` (a: `file:///android_asset/web/…`) · bridge `captchaValidateToken` (a: `android.postCaptchaValidateToken`).
**Strings:** `输入验证码` (add to the iOS catalog — bare literal today) · `刷新`. **States:** loading — owned by the page. Error — none; a failed challenge never fires the bridge. Success — handoff + pop.

#### Pay (支付)
**Purpose:** How to settle a placed order — WeChat mini-program, or a WHU unified-payment URL. **Entry:** 去支付 from the home card, success screen, or order centre. **Platforms:** both.

**Layout:**
```
┌───────────────────────────────────────────────┐ nav 支付
│ ScrollView · bg b1 · pad 16 · leading         │
│ ┌ 1 METHOD 1 (always) ──────────────────────┐ │
│ │ [方法1] 前往微信小程序“场馆预约”继续支付   │ │
│ ├ 2 METHOD 2 (if loaded, else spinner) ──────┤ │ divider pad v8
│ │ [方法2] 从武汉大学统一支付网页支付         │ │
│ │ 您在武汉大学统一支付的交易与Ham无关。      │ │
│ │ ─ collapsed (default) ─ gap 16 ──         │ │
│ │ 核对支付信息                               │ │
│ │ ┌ pretty JSON · h256 · r12 · gray@0.2 ──┐ │ │
│ │ └───────────────────────────────────────┘ │ │
│ │ [      确认      ] blue r12 v16 · gap 32  │ │
│ │ ─ expanded ──                             │ │
│ │ 长按下方文本复制到浏览器打开               │ │
│ │ ┌ URL · caption · selectable · r12 · p8 ┐ │ │
│ └───────────────────────────────────────────┘ │
└───────────────────────────────────────────────┘
```

**Content blocks, in order:**
1. **Method 1** — always visible, instructional, no tap.
2. **Method 2** — conditional on the payment-info fetch (spinner while fetching, hidden on failure) + disclaimer.
3. **Payment-info panel (default)** — `核对支付信息` + pretty JSON in a fixed-height scrollable panel, then `确认` → expand.
4. **URL panel (expanded)** — `长按下方文本复制到浏览器打开` + the selectable URL `https://gym.whu.edu.cn/hsdsqhafive/transit/formSubmit.html?url={url}&json={json}&signature={sig}`, built with RFC-3986 query escaping.

**Values:**

| Element | Value | Note |
|---|---|---|
| Title · method chip | `支付` · `body` rounded bold, white on blue, r8, pad v4 h8 | i: no title, no fill; a: no chip |
| Method description / disclaimer | `bodyBold` / `body` t2 | i: description bold, disclaimer unstyled |
| Divider · pre-panel gap · pre-button gap | pad v8 · 16 · 32 | a: pre-button gap 4 |
| JSON panel · label | h256 scrollable, r12, bg `gray@0.2`, pad h16 v4, leading · `核对支付信息` `bodyBold` | a: unbounded — clips; i: label unstyled |
| 确认 | white on blue, r12, pad v16, full width | a: v12 |
| URL hint · URL text | `长按下方文本复制到浏览器打开` `body` · `caption`, selectable, r12, pad 8, bg `gray@0.2` | i: unstyled; a: bare read-only field |
| Initial state | **JSON + 确认** first | a: starts on the URL — inverted |

**Strings:** `支付` · `方法1` / `前往微信小程序“场馆预约”继续支付` · `方法2` / `从武汉大学统一支付网页支付` · `您在武汉大学统一支付的交易与Ham无关。` · `核对支付信息` · `确认` · `长按下方文本复制到浏览器打开` · `获取付费信息失败` (a: declared, unused — render on fetch failure).
**States:** loading — spinner in place of method 2. Error — `获取付费信息失败` (neither renders it today). Success — the JSON/URL panel is the success state.

#### Quick order (正在预约)
**Purpose:** Place the remembered court in one tap. **Entry:** home quick-order CTA. **Platforms:** both.

**Layout:**
```
┌───────────────────────────────────────────────┐
│ full screen · bg b1 · centred · pad 16        │
│   ◔ determinate ring                          │
│   正在预约  body   (→ 很快就好 at > 0.95)     │
└───────────────────────────────────────────────┘
  then, in place: success screen │ error screen
  captcha: pushed full screen over the top
```

**Content blocks, in order:** 1. **Progress** — determinate ring + label; advance on a 10 ms timer, clamp at 0.95 so motion is always visible; label `正在预约` → `很快就好` past 0.95. 2. **Captcha** — pushed full screen when the API demands a challenge; resume the request on token receipt. 3. **Result** — swap the whole screen for the success screen (tap → dismiss) or the error screen (tap → dismiss).
**Values:** bg b1, content centred, ring pad 16, label `body` t1. **Strings:** `正在预约` · `很快就好`. **States:** loading — ring. Error — error screen with the API message. Success — success screen.

#### Starred-order setting (收藏预约设置)
**Purpose:** Pick the court/time remembered for one-tap ordering, and the day offset. **Entry:** settings → 去设置. **Platforms:** both.

**Layout:**
```
┌───────────────────────────────────────────────┐ nav 收藏预约设置
│ ScrollView · bg b1 · pad 16 · card gap 8      │
│ ┌ 1 · bg b2 · r16 · pad 16 ────────────────┐ │
│ │ badge 32×32 r6                            │ │
│ │ sport type          caption               │ │
│ │ {title}（{address}） caption              │ │
│ │ HHmm-HHmm           caption               │ │
│ │ ── or 未设置 ──  gap 12 ──                │ │
│ │ [       去设置       ] r8 pad v16         │ │ → picker
│ └───────────────────────────────────────────┘ │
│ ┌ 2 · only when configured ─────────────────┐ │
│ │ 预约明天                          [ ◐ ]   │ │ switch
│ └───────────────────────────────────────────┘ │
└───────────────────────────────────────────────┘
```

**Content blocks, in order:**
1. **Configured court card** — badge, sport type, `{title}（{address}）`, `HHmm-HHmm`, or `未设置` when unconfigured; then the full-width `去设置` CTA → picker. On return, write the picker result (venue, sport type, court no., start, end, `weekday = -1`, tomorrow flag) into the starred config.
2. **Day-offset row** — `预约明天` + switch bound to the item's `tomorrow`. Conditional: rendered only when a court is configured. Persist on every change.

**Values:**

| Element | Value | Note |
|---|---|---|
| Title · card | `收藏预约设置` (i: none) · bg b2, r16, pad 16, gap 8 | — |
| Badge · venue · time · badge gap | 32 × 32 r6, bg t1, `title2` rounded bold, fg b1 · `{title}（{address}）` `caption` · `HHmm-HHmm` `caption` · 4 | a: `StadiumAreaIcon` 36 `title` 24; i: title only, no gap |
| 去设置 CTA | full width, r8, pad v16, bg `brand@0.15`, fg brand, `bodyBold` | i: `@0.25` fg t1; a: r12 |
| Day row · selection filter | `预约明天` `body` + trailing switch · read the `weekday == -1` entry | a: `firstOrNull()` |

**Strings:** `收藏预约设置` · `预约明天` · `未设置` · `去设置`. **States:** empty — `未设置` replaces the court lines, day row hidden. Success — returning from the picker replaces the config in place. No loading or error states.

#### Order centre (订单中心)
**Purpose:** Order history, by hosting the university's own list page. **Entry:** home 订单中心. **Platforms:** both.

**Layout:**
```
┌───────────────────────────────────────────────┐
│ ░░ status-bar spacer                          │
│ ◀     订单中心                       [刷新]   │ header h36, bg b1
├───────────────────────────────────────────────┤
│ ░░ WebView · https://gym.whu.edu.cn/          │
│      hsdsqhafive/pages/order/orderList        │
└───────────────────────────────────────────────┘
```

**Content blocks, in order:**
1. **Title bar** — fixed `订单中心` (never swap to the page title or `加载中`), back button, trailing `刷新` → `reload()`.
2. **Web view** — fills the content area. Before each page load inject the session token `window.localStorage.setItem('wxtoken', '<auth without "Bearer ">')`, guarded to `gym.whu.edu.cn`; if the loaded URL contains `cas`, redirect back to the order-list URL.

**Values:** header h36, bg b1 · title `bodyBold` t1, 1 line, max width 300 · refresh `body` blue, 8 from the trailing edge · back chevron blue, 16 from the leading edge. The content region is out of scope for the design system (remote HTML).
**Strings:** `订单中心` · `刷新`. **States:** loading — none native. Empty/error — owned by the remote page. Session expiry — the `cas` redirect guard. Missing token — validate before injecting; never build JS by interpolating an unescaped token.

#### Bulletin list (公告)
**Purpose:** All venue announcements. **Entry:** home banner page 0. **Platforms:** both (iOS only today — add the Android route and make banner page 0 tappable).

**Layout:**
```
┌───────────────────────────────────────────────┐ nav 公告
│ ScrollView · bg b1 · pad 16 · card gap 8      │
│ ┌ card · bg b2 · r16 · pad 16 ──────────────┐ │
│ │ title    bodyBold                         │ │
│ │ time     caption t2 · yyyymmddhhmm        │ │
│ │ ─ gap 8 ─                                 │ │
│ │ content  body, unlimited                  │ │
│ └───────────────────────────────────────────┘ │ … 1 per bulletin, newest first
└───────────────────────────────────────────────┘
```

**Content blocks, in order:** one repeating card — title, creation time, 8 gap, full body. Tap: → bulletin detail.
**Values:** pad 16 · card bg b2, r16, pad 16, gap 8 · title `bodyBold` t1 · time `caption` t2 · content `body` t1. Titles/bodies are translatable (`TSText` → `translationTask(zh → preferred)`, iOS 26+).
**Strings:** `公告` only; content is data-driven. **States:** loading — spinner or skeleton (today: blank). Empty — empty-state placeholder (today: blank). Error — inline message + retry (today: toast only). Fetch on appear.

#### Bulletin detail (公告)
**Purpose:** Read one announcement. **Entry:** home banner or bulletin list. **Platforms:** both. **Layout:** as the bulletin-list card, full width, no list. **Content blocks:** title → time → body; no actions, no share/copy. **Values:** identical to the list card (pad 16, r16, bg b2, gap 8, `bodyBold` / `caption` t2 / `body`); the card container is required (a: bare column today). **Strings:** `公告` only. **States:** no loading (arrives in the route argument). Empty — placeholder for empty fields. Error — catch deserialisation failure and show a message instead of crashing.

#### Settings (运动设置)
**Purpose:** Manage the sport session/token and the starred court. **Entry:** home 设置. **Platforms:** both.

**Layout:**
```
┌───────────────────────────────────────────────┐ nav 运动设置
│ ScrollView · bg b1 · pad 16 · card gap 8      │
│ ┌ 1 · bg b2 · r16 · pad 16 ────────────────┐ │
│ │ 账号信息               bodyBold           │ │
│ │ 你的体育场所系统认证信息  caption t2       │ │
│ │ ─ gap 16 ─                                │ │
│ │ 登录时间：{t} / 过期时间：{t}  caption    │ │ or 未登录
│ │ ─ gap 8 ─   [登录]                        │ │
│ ├───────────────────────────────────────────┤ │
│ │ 2 收藏预约       bodyBold                 │ │
│ │ {subtitle}       caption t2               │ │
│ │ ─ gap 16 ─                                │ │
│ │ badge 32 │ type / area / 明天 HHmm-HHmm    │ │ or 未设置
│ │ ─ gap 8 ─   [去设置]                      │ │
│ ├───────────────────────────────────────────┤ │
│ │ 3 其它设置       bodyBold                 │ │
│ │ 停止使用、诊断信息等  caption t2          │ │
│ │ ─ gap 8 ─   [重置]  body red              │ │
│ └───────────────────────────────────────────┘ │
└───────────────────────────────────────────────┘
```

**Content blocks, in order:**
1. **Account card** — title + subtitle; `登录时间：{t}` and `过期时间：{t}` when the token decodes, else `未登录`. Conditional: append ` 已过期` in red when past due; spinner replaces the button while logging in. Tap `登录` → authenticate; on failure fall back to the CAS web-login sheet.
2. **Starred-order card** — title + subtitle; the configured court (badge, sport type, area, `明天`/`今天` + `HHmm-HHmm`) or `未设置`. Tap `去设置` → starred-order setting.
3. **Other card** — `其它设置` + subtitle + destructive `重置`. Tap: confirm, reset the sport context, pop.

**Values:**

| Element | Value | Note |
|---|---|---|
| Title · card · card title / subtitle | `运动设置` (i: none) · bg b2, r16, pad 16, gap 8 (i: cards flush) · `bodyBold` t1 / `caption` t2 | i: subtitle left primary |
| Title→body · body→action gap | 16 · 8 | a: 8 · 4 (auth card) |
| Body text · action label | `caption` t1, expiry red when expired · `body` blue | i: default button styling |
| Badge · badge→text gap | 32 × 32, r6, bg t1, `title2` rounded bold, fg b1 · 4 | a: 36; i: no gap |
| 未设置 · 重置 · time format | `caption` t1 · `body` red · `登录时间：%@` / `过期时间：%@` (full-width colon, no space) | i: 未设置 inherits 17pt, 重置 commented out; a: half-width colon + space |

**Strings:** `运动设置` · `账号信息` / `你的体育场所系统认证信息` · `登录时间：%@` · `过期时间：%@` · `过期时间：%@ 已过期` · `未登录` · `登录` · `登录成功` (toast) · `取消` (web sheet) · `收藏预约` / `在设置中设置收藏的运动场地和时间，然后在首页快速预约` · `明天` · `今天` · `未设置` · `去设置` · `其它设置` / `停止使用、诊断信息等` · `重置`.
**States:** logged in — both time lines, expiry red + `已过期` when past due. Not logged in — `未登录`. Undecodable token — treat as not logged in. Login loading — spinner replaces the button. Login success — toast `登录成功`. Login failure — toast, then the CAS web sheet. Reset — confirm, reset, pop.

#### Intro / connect (连接体育场所预约)
**Purpose:** First-run gate — explain verification, offer CAS, run the sport login, celebrate success. **Entry:** sport tab when unauthenticated. **Platforms:** both. Presented as a sheet.

**Layout:**
```
┌───────────────────────────────────────────────┐
│ [取消]                                        │
│        ┌────┐                                 │
│        │logo│56×56 r12 ⟶ link ⟶ icon 48 @25%  │ gap 16
│        └────┘                                 │
│   连接体育场所预约       title 28 bold        │ gap 32 above
│   使用前，Ham需要验证你的在校信息  body t2    │ gap 8
│ ┌ panel · #EDEEEF · r12 · pad 16 ───────────┐ │ gap 16 above
│ │ ⬤ 从信息门户验证                      ›   │ │ row: icon 24 blue, gap 8
│ │   将进入武汉大学信息门户网页验证你的身份   │ │ items gap 8
│ └───────────────────────────────────────────┘ │
└───────────────────────────────────────────────┘
  sub-routes: CAS web login → loading → success │ error
```

**Content blocks, in order:**
1. **Dismiss** — `取消`; also wired to the system back gesture.
2. **Header** — logo 56 × 56 r12, `link` glyph, module icon 48 at 25% brand.
3. **Title** — `连接体育场所预约`, `title` bold t1. **4. Subtitle** — `使用前，Ham需要验证你的在校信息`, `body` t2.
5. **Choice list** — one CAS row: icon 24 blue, 8 gap, bold title, `caption` subtitle, gray chevron. Tap: skip straight to login if already CAS-authenticated, else open the CAS web view.
6. **Post-login** — loading step (centred full-screen spinner) → success step or error step.

**Values:**

| Element | Value | Note |
|---|---|---|
| Dismiss · logo / module icon | `取消` · 56 × 56 r12 / 48 @ brand 25%, gap 16 | a: `关闭` · 48 r8, gap 12 |
| Header→title · title→subtitle · subtitle→panel | 32 · 8 · 16 | a: 4 between title/subtitle |
| Title · subtitle | `title` 28 bold t1 · `body` t2 | a: 24 · subtitle omitted entirely |
| Panel | r12, `#EDEEEF`, pad 16, item gap 8 | i: r8 `gray@0.15`, item gap 24 |
| Row icon / title / subtitle / chevron | 24 blue, gap 8 · `bodyBold` t1 · `caption` t2 · gray | a: chevron t1 |
| Success check · title / message | 64 green fill, white glyph · `title` t1 `验证成功` / `body` t1 `你可以开始使用体育场所预约了` | a: 64 on a blue circle pad 8; i: `登录成功`, no message |
| Gap before button · success button | 16 · full width h48, r12, bg `blue@0.15`, `headlineBold` blue, `完成` | i: 32 · `去使用`, r8 outlined, max 350 |
| Entry animation · Lottie · haptics | slide up from 50% + fade after 0.5 s · `lottie_congrats` play once speed 1 · vibrate | i: spring, offset 100, delay 0.8 s, no vibrate |
| Error screen | icon 64 red, `title` title, `body` message, gap 16, `重新登录` | i: absent (toast only) |

**Strings:** `连接体育场所预约` · `使用前，Ham需要验证你的在校信息` · `从信息门户验证` / `将进入武汉大学信息门户网页验证你的身份` · `取消` · `验证成功` / `你可以开始使用体育场所预约了` · `完成` · `验证失败` / `登录场馆预约失败` · `重新登录`.
**States:** idle — intro. CAS-authenticated — skip CAS, go to login. CAS web login — web view with a back action. Loading — centred full-screen spinner. Success — success screen with Lottie, vibrate, `完成`. Error — error screen with `重新登录` (resets state, returns to the intro). Dismiss — sheet close.

#### Order success (预约成功)
**Purpose:** Confirm the booking and offer payment. **Entry:** returned by the order or quick-order flow. **Platforms:** both.

**Layout:**
```
┌───────────────────────────────────────────────┐ full screen · bg b1
│ Lottie congrats (background, play once)       │
│     ✓ 64 green                                │ centred · gap 4
│     预约成功     title 28                     │
│     {hint}      caption (if any)              │
│     ─ gap 16 ─                                │
│ ┌ order card · bg b2 · r8 · 16px blue accent ┐│
│ │▌badge 32×32 r6                             ││ pad lead 28, trail 8, v8
│ │▌venue title       caption                  ││
│ │▌{start}-{end}     caption                  ││
│ │▌────────────────────────────────────────── ││ divider
│ │▌请于{x}前完成支付 caption red      [去支付] ││ button blue r8 pad 8
│ └────────────────────────────────────────────┘│
│     ─ gap 32 ─                                │
│     [      完成      ] max 350, r8            │
└───────────────────────────────────────────────┘
```

**Content blocks, in order:**
1. **Celebration** — full-screen Lottie `lottie_congrats`, play once, plus a vibration.
2. **Check + title** — 64 green check, `预约成功` `title`, optional `caption` hint.
3. **Order card** — 16-wide brand-blue leading accent bar; badge, venue title, `{start}-{end}`, divider, pay row. Conditional: in the pay window show the red deadline + `去支付`; otherwise bold `当前未处于可支付时间` + the range line and no button. Tap `去支付` → pay.
4. **完成** — dismisses.

**Values:**

| Element | Value | Note |
|---|---|---|
| Entry animation | slide up 50% + fade after 0.5 s | i: spring, offset 100, delay 0.8 s |
| Check icon · title | 64 green fill, white glyph · `预约成功` `title` t1 | a: 64 on blue circle pad 8; i: `预定成功` |
| Hint | `caption` t1, only when non-empty | — |
| Card | bg b2, r8, 16-wide blue leading accent, pad lead 28 / trail 8 / v8 | a: `HamCardView` titled `预约信息`, no accent |
| Badge · card text | 32 × 32, r6, bg t1, `title2` rounded bold, fg b1 · `caption` t1 | a: `StadiumAreaIcon` 36 |
| Pay row · out of window | red `caption` deadline · `去支付` white on blue, pad h8 v8, r8, `headline` · `captionBold` `当前未处于可支付时间` + `caption` range | i: r6; a: reuses the deadline format for the range |
| Time format · 完成 | `{start}yyyymmddhhmm-{end}HHmm` · h48, r12, bg `blue@0.85`, `headlineBold` white, full width | i: `返回`, r8 outlined, max 350 |

**Strings:** `预约成功` · `预约信息` · `请于%@前完成支付` · `当前未处于可支付时间` · `请于%@-%@完成支付` · `去支付` · `完成`. **States:** success only; render empty strings when the API omits venue or times.

#### Order error (预约失败)
**Purpose:** Report a failed booking and let the user back out. **Entry:** returned by the order or quick-order flow. **Platforms:** both.

**Layout:**
```
┌───────────────────────────────────────────────┐
│ full screen · bg b1 · centred · gap 4         │
│        ✕ 64 red                               │
│        预约失败    title 28                   │
│        {message}  body t2                     │
│        ─ gap 32 ─                             │
│        [   返回   ] w128 · r8 · pad v16       │
└───────────────────────────────────────────────┘
```

**Content blocks, in order:** 1. **Icon** — 64 red `xmark.circle.fill` / `ErrorOutline`. 2. **Title** — `预约失败`, `title` t1. 3. **Message** — the API error, or `遇到了错误` when empty. 4. **返回** — dismisses.
**Values:** bg b1, centred, gap 4 · icon 64 red · title `title` t1 · message `body` t2 · gap 32 · button width 128, r8, pad v16, bg `gray@0.20`, `body` gray label. **Strings:** `预约失败` · `返回` · `遇到了错误`. **States:** error only.

##### Divergences (current, to be closed)

- **Vocabulary:** 8 zh-Hans strings use 预定 against the app standard 预约 — CTA `预定`, `场馆预定公告`, `你的历史预定记录`, `场馆预定选项`, `收藏预定`, `连接体育场所预定`, `预定成功`, `预定失败`. iOS ships both `你的历史预定记录` and `你的历史预约记录` in `Localizable.strings`; ja/en already collapse to one term.
- **Home:** i: Banner → CurrentOrder → QuickOrder → Function; a: swaps the middle two.
- **Bulletin list:** iOS-only screen; a: no route, banner page 0 untappable. **Banner:** i: 5 s auto-advance, 5-bulletin cap, `查看详情`; a: none. **Bulletin detail:** i: card container; a: bare column; time→body gap 8 vs 4.
- **Settings:** i: no nav title, "其它设置/重置" card commented out, never renders `未登录`; a: ships both but never renders `已过期`. Favourite label i: `隔天`/`当天`; a: `明天`/`今天`. Filter i: `weekday == -1`; a: `firstOrNull()`. Login failure i: CAS web sheet; a: toast only.
- **Intro:** a: omits the subtitle, adds loading and full-screen error routes with `重新登录`, skips CAS when already authenticated, dismiss labelled `关闭`; i: none. Success copy differs (`登录成功`/none/`去使用` vs `验证成功`/message/`完成`); check icon plain green vs blue circle; a vibrates.
- **Order centre:** a: has `刷新` and a `cas` redirect guard; i: neither.
- **Select screen:** i: chip strip auto-opens when nothing selected; a: does not. i: sport icons keyed on hardcoded zh-Hans; a: localised titles. Chip icon 24/gap 8 vs 32/gap 20. Date arrow 32 vs 36. Venue image 85 r8 vs 104 r12. Court badge 24 vs 28. Tile h56 r6 `@0.25` vs h50 r8 `@0.15`. Bottom-bar label i: `预定`/`选择` by `canQuickOrder`; a: always `选择`. Slot price i: `lightingPrice + price`; a: `price`. Contiguity enforced in the view model (i) vs the tile (a). Footer hidden during submit (i) vs left up (a). Footer line i: `{venue}-{n}号场`; a: `{title} | {address}`, never the court number.
- **Picker:** CTA `确定` (i) vs `预定` (a); both hardcode `isFree = true`; i: dismisses unconditionally after `confirm()`; a: no-ops on a partial selection.
- **Captcha:** i: title + `刷新`; a: neither.
- **Pay:** i: no nav title, `方法N` as chips (no explicit fill), starts on the JSON panel, JSON panel h256, escapes the query with `.urlHostAllowed`; a: title, plain text labels, starts on the URL panel, unbounded JSON height, `URLEncoder`. `.urlHostAllowed` is wrong for a query value — use RFC-3986.
- **Quick order:** i: determinate ring with `正在预约`/`很快就好`; a: indeterminate spinner, captcha inline instead of pushed.
- **Starred setting:** i: no nav title, venue title only, CTA `设置` at `@0.25`; a: titled screen, `{title}（{address}）`, CTA `去设置`.
- **iOS hygiene:** the module bypasses `ham_*` tokens almost entirely and mixes three localisation mechanisms (`String(localized:)`, bare `Text("…")`, `LocalizedStringKey(…)`), leaving several literals as unverifiable catalog keys. Android uses tokens and `strings.xml` consistently.


---

### Score

**Shared tokens.** Screen bg `ham_bg_b1`. Card: padding 16, radius 16, bg `ham_bg_b2`, clipped; card title `bodyBold`,
title→content 8; card gap in a scroll column 8; scroll horizontal padding 16. Type scale: largeTitle 28 · title 24 ·
title2 20 · title3 16 · body 16 · headline 14 · caption 12 · caption2 11 (`Bold` = same size at W700). Text
`ham_text_primary` / `ham_text_secondary` / `ham_gray` (disabled). Accent: `ham_blue` actions, `ham_red` destructive,
`ham_orange` expired, `ham_brand_score` score accent. Buttons: height **48**, radius **12**, `headlineBold`, fill
`ham_blue @ 0.85` white (primary) or `ham_blue @ 0.15` `ham_blue` (secondary). Row: label `body` + subtitle `caption`,
switch→label gap 4. Divider: full card width, vertical pad 4. ⚠ iOS has no fixed scale (SwiftUI semantics: `.title`≈28,
`.title2`≈22, `.body`≈17, `.caption`≈12); card metrics match.

---

#### Score root / gate (成绩)
**Purpose:** Decide whether the score feature is usable — scores fetched at least once **and** biometric lock passed — and re-lock when the app backgrounds. **Entry:** home/tab route `score`. **Platforms:** both.

**Layout:**
```
┌──────────────────────────────────────┐
│ nav 成绩 (inline)                    │
├──────────────────────────────────────┤
│ one of, cross-faded:                 │
│  A fetched && auth ok → Score main   │ fills remaining height
│  B auth failed        → Auth error   │ centred (see Face ID section)
│  C never fetched      → blank +      │ connect sheet auto-presented
└──────────────────────────────────────┘
```

**Content blocks, in order:**
1. Auth gate — render **A**, **B** or **C**. Conditional: **C** auto-presents the connect sheet **500 ms** after appear.
2. Connect sheet — first-run fetch flow titled `连接成绩`; dismissing it while still unfetched pops the screen.
3. Re-lock — on app background reset to unauthenticated; on foreground re-run biometric auth.

**Values:** nav title `成绩` inline · root bg `ham_bg_b1` · gate transition cross-fade (`fadeIn togetherWith fadeOut`) · auto-present delay **500 ms** ⚠ iOS 300 ms · re-lock on app background / lifecycle stop.

**Strings:** 成绩 · 连接成绩 · 保护你的成绩数据 (biometric prompt reason) · 请重新验证 · 你已开启成绩保护，请开启生物认证权限 · 验证失败.
⚠ iOS hardcodes the four auth strings; Android resources them.

**States:** empty (never fetched) → block 2. Loading → render nothing; auth resolves within a frame, no spinner. Error → block B. Select-mode → n/a.

##### Divergences
- Auto-present 300 ms iOS / 500 ms Android; iOS re-locks via `scenePhase`, Android via `ON_STOP`.
- iOS hardcodes the auth/biometric messages while localising the rest.

---

#### Score main (成绩)
**Purpose:** Show aggregate GPA/F2 statistics, three entry actions and the per-semester score list, with a multi-select mode that recomputes statistics from the selected subset. **Entry:** from the gate once scores exist and auth passes. **Platforms:** both. **Primary score screen.**

**Layout:**
```
┌────────────────────────────────────────────┐
│ nav 成绩 (inline)                          │
├────────────────────────────────────────────┤
│░░░ select header ░░░░░░░░░░░░░░░░░ h ≈ 92 ░│ pinned; select-mode only
├────────────────────────────────────────────┤
│ scroll, padding h 16                       │
│  [ spacer 92 ]                 select only │
│  ┌ 成绩概览, watermark School 240 @.15 ──┐ │
│  │ GPA 3.72            title 24 Normal   │ │
│  │ • 平均成绩: 90.12        caption      │ │
│  │ ── divider, maxW 200, pad v 12 ──    │ │ only if years exist
│  │ 年度成绩             bodyBold         │ │ rows gap 4, newest first
│  │ 大四 • 综测成绩: … 平均成绩: … (GPA…)│ │
│  └────────────────────────────────────────┘ │
│  gap 8                                      │
│  ┌ function card h 56, r16, ham_lightGray ┐ │ pad 8, col gap 8
│  │ [获取成绩] [选择] [设置]               │ │ 3 equal columns
│  └────────────────────────────────────────┘ │
│  gap 8                                      │
│  ┌ semester card (repeat, newest first) ─┐ │
│  │ 2024-2025 第二学期     ▸ / 全选       │ │ header gap 4
│  │ • 平均成绩: 88.40 (GPA3.60)           │ │
│  │  ── gap 10 ──                         │ │ rows gap 12
│  │ ▌ 高等数学                 96         │ │ bar 6×32 r6; title2 20
│  │   王老师 · 专业课 · 5.0 学分          │ │ caption 12
│  └────────────────────────────────────────┘ │
│  trailing spacer = nav-bar height + 24      │
└────────────────────────────────────────────┘
 overlay: full-screen privacy blur when backgrounded
```

**Content blocks, in order:**
1. Pinned select header — three dot+text statistic rows, then right-aligned `退出`. Conditional: select mode only; it overlays the list, cleared by the **92** spacer; tapping `退出` exits select mode **and clears the selection**.
2. Stat card `成绩概览` — GPA value, one `平均成绩` line, then (conditional: any year data) divider + `年度成绩` + one row per year, newest first. Hidden in select mode. Watermark `School` **240**, `ham_brand_score @ 0.15`, offsetY **64**, bottom-trailing, non-interactive.
3. Function card — three equal-width buttons `获取成绩`/`更新成绩信息`, `选择`/`自选成绩然后统计`, `设置`/`编辑成绩选项`. Hidden in select mode. Tap → fetch sheet / toggle select mode / push Settings.
4. Semester card — header (title, dot + `平均成绩: %.2f (GPA%.2f)`), right side `chevron` collapse toggle in normal mode or `全选`/`全不选` in select mode. Conditional: the select-all control only when the semester has ≥1 selectable row. Default **expanded**, state persisted per card.
5. Score row — colour bar, name + `instructor · courseType · credit 学分`, score, optional comment button, optional checkbox. Comment button: normal mode **and** comment enabled **and** upload permission; tap → course detail if logged in, else login prompt. Checkbox: select mode only, disabled when the course has no score or is disabled, hidden when unselectable. Struck through when enabled but excluded from F2 — never in select mode.
6. Privacy overlay — full-screen blur over the whole screen while backgrounded.

**Values:**

| Element | Value |
| --- | --- |
| Scroll padding | horizontal **16**; top 16 + header + status bar; bottom **16**; card gap **8** |
| Watermark | `School` 240, `ham_brand_score @ 0.15`, offsetY 64 ⚠ iOS `crown.fill` 220, orange `@ 0.1`, offset (60, 20) |
| `成绩概览` / GPA | `bodyBold` / `title` 24 **Normal** ⚠ iOS `.title` ≈28 pt |
| `年度成绩` | `bodyBold`; divider above with vertical pad **12**, `maxWidth 200` ⚠ Android pad 4 |
| Year rows | gap **4**; dot 6×6 r6 coloured `getRandomColor("<year>678")`; text `caption`; labels `大一`…`大五` from a 6-entry array with a blank leading slot ⚠ Android stops at 大四 — a 5th year renders blank |
| Precision | GPA `%.2f`, 平均成绩 `%.2f`, 综测成绩 `%.6f`, credits `%.1f` ⚠ Android prints 平均成绩 and the header GPA `%.6f` |
| Function card | height **56**, radius **16**, bg `ham_lightGray`, inner pad **8**, column gap **8** ⚠ Android radius 12, iOS bg `gray @ 0.1` |
| Function button | title `bodyBold` + `arrowtriangle.right.fill` **8** in `ham_brand_score`, subtitle `caption2` 11 ⚠ Android subtitle `caption` 12, arrow is an unscaled `ArrowRight` |
| Semester title / dot | `bodyBold`; dot 6×6 r6 from `"<courseId>66"`, fallback `ham_gray` ⚠ Android fallback `ham_blue @ 0.15`; header gap **4**, header→rows **10**; `全选`/`全不选` `body` `ham_blue` plain text button |
| Row gap / colour bar | gap **12** ⚠ iOS 15 pt, Android 8 dp; bar **6 × 32**, radius **6** ⚠ iOS 6×35 r6, Android 6×28 r3 + 3 dp top pad |
| Row text | name `body` 16 (disabled `ham_gray`); meta `caption` 12 `ham_text_secondary`, max 2 lines; score `title2` 20, `"--"` when absent, `ham_gray` when disabled ⚠ iOS `.title2` ≈22, Android `title` 24/Normal |
| Row controls | comment: glyph **14** on a 32×32 radius-8 plate `ham_gray @ 0.2`, tint `ham_gray` ⚠ Android `Forum` at 20 dp. Checkbox: filled/outline pair at `title2`, tint `ham_blue` checked / `ham_text_primary` unchecked / `ham_gray` disabled, leading pad **3**, hidden (not dimmed) when unselectable |
| Select header | pinned top, translucent `systemThinMaterial`, padding h 16 / v 8, rows gap 4, clearance **92** ⚠ Android opaque `ham_bg_b1`, clearance 92 dp. Rows: `• 综测成绩: %.6f` + `平均成绩: %.2f (GPA%.2f)` · `总学分: %.1f 必修学分: %.1f 选修学分: %.1f` · `专业必修: %.1f 专业选修: %.1f 跨专业课程: %.1f`; `退出` right-aligned, `body`, `ham_blue`, end pad 8 |
| Bucketing | cross-college-major → `跨专业课程`; else 专业必修 = 专业+必修, 专业选修 = 专业+选修. Recompute on every selection change |
| Bottom / privacy | trailing spacer = nav-bar height + **24**; backgrounded → full-screen `systemThinMaterial` blur, ignores safe area ⚠ Android has no blur — the gate blanks content instead |

**Strings:** 成绩 · 成绩概览 · GPA %.2f · 平均成绩 · 年度成绩 · 综测成绩 · 大一 大二 大三 大四 大五 · 获取成绩 · 更新成绩信息 · 选择 · 自选成绩然后统计 · 设置 · 编辑成绩选项 · 第一学期 / 第二学期 / 第三学期 · 平均成绩: %.2f (GPA%.2f) · 全选 · 全不选 · 学分 · 跨学院专业课 · 总学分 · 必修学分 · 选修学分 · 专业必修 · 专业选修 · 跨专业课程 · 退出.
⚠ iOS hardcodes `跨学院专业课` and mixes ASCII `:` with full-width `：` (`必修学分：12.0`). Use ASCII `:` everywhere.

**States:** empty (no semesters) → GPA + `平均成绩` only, yearly block hidden, function card present; no empty copy. Loading → none; data loads async, screen empty until it lands, no spinner. Error → not representable. Select mode on → stat card and function card hidden, 92 spacer, pinned header, per-semester `全选`, row checkboxes, statistics recomputed from the selection. Backgrounded → privacy blur.

##### Divergences
- Select mode: iOS hides the stat card and clears the selection on exit; Android keeps the card and the ticks.
- iOS watermark is a crown (orange, 220); Android a school glyph (brand, 240).
- Row gap 15 pt vs 8 dp; colour bar 35/r6 vs 28/r3.
- iOS suppresses strikethrough in select mode; Android keeps it.
- iOS gates the comment button on `enableCourseComment` only; Android also requires `permittedUploadScore`.
- iOS persists per-card expand state; Android's `remember` resets on scroll-off.
- Android lacks the 大五 label and prints 平均成绩/GPA to 6 decimals.

---

#### Update / fetch score (获取成绩)
**Purpose:** Authenticate against the education portal (CAS) and pull the score list. **Entry:** `获取成绩` on the main screen, or auto-presented from the root the first time (then titled `连接成绩`). **Platforms:** both.

**Layout:**
```
┌──────────────────────────────────────┐
│ 取消                                 │
│   ┌────┐ 🔗 ┌────────┐               │ logo 56 r12 · link · icon 48
│   │logo│    │  icon  │               │   ham_brand_score @ 0.25
│   获取成绩             title 24 Bold │ row gap 16
│ ┌──────────────────────────────────┐ │ r12 ham_lightGray, pad 16
│ │ ▸ 🌐 通过信息门户              › │ │ title bodyBold
│ │   从信息门户登录教务系统获取成绩 │ │ subtitle caption
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
 then: CAS login → captcha (if needed) → fetching → success
       → first run only: Face ID enable → dismiss
       on failure: error state
```

**Content blocks, in order:**
1. Intro shell — `取消`, centred logo + link + feature icon row, title, choice container.
2. Choice row `通过信息门户` / `从信息门户登录教务系统获取成绩` — `Public` icon 24 in `ham_blue`, gap 8, title `bodyBold`, subtitle `caption`, chevron. Tap → CAS login. Conditional: when CAS is disabled the row becomes `从信息门户验证` / `将进入武汉大学信息门户网页验证你的身份`.
3. CAS login — web login; on success go to captcha, or straight to fetching when the RN fetch module is enabled.
4. Captcha — bundled `education-captcha-page.html` in a web view, JS bridge named `captchaValidateToken`.
5. Fetching — centred progress indicator + `正在获取成绩`.
6. Success — `获取成功` + `愉快使用吧` + `返回`. Tap → Face ID step on first run, else dismiss.
7. Failure — `获取失败` + `message，hint` + a retry/dismiss button.

**Values:**

| Element | Value |
| --- | --- |
| Logo / link / icon | **56×56 r12** / default size / **48** in `ham_brand_score @ 0.25`; row gap **16** ⚠ Android logo 48 r8, gap 12 dp |
| Title / container | `title` 24 Bold, **16** below the icon row and **16** above the choice container ⚠ iOS 28 pt; container radius **12**, `ham_lightGray`, outer pad h 16 + inner pad 16, rows gap 8 ⚠ iOS radius 8, `gray @ 0.15`, gap 24 |
| Loading | centred spinner + `正在获取成绩` ⚠ iOS shows `正在更新`, Android shows no label |
| Success | `checkmark.circle.fill` **64** white on green; title `title` 24; message `body` 16; button full width h 48 r 12 `ham_blue @ 0.15` `headlineBold`, label `返回`; entrance slide+fade ~300 ms after 200 ms + success haptic ⚠ iOS opacity/offset-y-100 spring after 800 ms + heavy haptic; Android `slideInVertically` after 500 ms + vibrate |
| Failure | `xmark.circle.fill` **64** white on `ham_red`; title `title` 24; message `body` 16; button h 48 r 12 `ham_blue @ 0.15`; entrance slide+fade after 200 ms ⚠ iOS error state is static |
| RN kill-switch | read the **score** fetch flag; when on delegate the fetch UI to RN module `RNFetchScoreView` |
| Cleanup | on success pop the CAS and captcha steps off the stack |

**Strings:** 获取成绩 · 连接成绩 (first run) · 取消 · 通过信息门户 · 从信息门户登录教务系统获取成绩 · 从信息门户验证 · 将进入武汉大学信息门户网页验证你的身份 · 验证码验证 · 正在获取成绩 · 获取成功 · 愉快使用吧 · 获取失败 · 返回.
⚠ iOS hardcodes 更新成功 / 更新失败 / 正在更新 / 返回; Android resources 获取成功 / 获取失败. Use the 获取 wording.

**States:** empty → n/a (fixed single row). Loading → block 5. Success → block 6. Error → block 7. Select-mode → n/a.

##### Divergences
- Terminology: iOS 更新成功/失败, Android 获取成功/失败.
- iOS reads RN flag `RNFetchCourseView`, Android `RNFetchScoreView` — different flags.
- iOS's captcha view is full-screen; Android hosts it in a nav container titled `验证码验证`.
- iOS builds the choice list conditionally; Android renders one row and branches on tap.

---

#### Score settings (成绩设置)
**Purpose:** Configure F2 calculation method, biometric protection, zero-score filtering, per-course enablement and reset. **Entry:** `设置` on the main screen. **Platforms:** both.

**Layout:**
```
┌──────────────────────────────────────┐
│ nav 成绩设置 (inline)                │
│ scroll, pad 16, cards 8 apart        │
│ ┌ 保护成绩数据 ────────────────────┐ │
│ │ 使用生物识别保护成绩        [ ○] │ │ red caption when unavailable
│ └───────────────────────────────────┘ │
│ ┌ F2计算方式 ──────────────────────┐ │
│ │ ○ 使用计算机学院F2计算方式       │ │ B1+B2×0.002（B2最多选8门）
│ │ ─────────────────────────────── │ │
│ │ ○ 使用其它F2计算方式             │ │ B1×0.98+B2×0.02
│ │ ─────────────────────────────── │ │
│ │ ○ 自定义计算方式                 │ │ 使用自定义的JavaScript计算F2
│ │   编辑代码 ›                     │ │ disabled while code empty
│ │ ─────────────────────────────── │ │
│ │ ○ 使用其它计算方式(基于JavaScript)│ │ 选择计算方式 ›
│ └───────────────────────────────────┘ │
│ ┌ 偏好 ────────────────────────────┐ │
│ │ 忽略0分成绩                 [ ○] │ │
│ └───────────────────────────────────┘ │
│ ┌ 成绩启用 ─────┐┌ 重置成绩 ───────┐ │
│ │ 编辑成绩启用状态›││ 重置所有成绩数据│ │ ham_blue / ham_red
│ └───────────────┘└─────────────────┘ │
└──────────────────────────────────────┘
```

**Content blocks, in order:**
1. Protection card — switch row `使用生物识别保护成绩`. Conditional: when biometrics are unavailable/forbidden the switch is disabled and the red `caption` hint `无法访问你的生物识别模块` appears. Tap to turn **off** → require biometric re-auth; if auth fails, revert the switch.
2. F2 method card — four mutually exclusive rows, each a bold label + `caption` description. The custom-JS row carries an `编辑代码` link (disabled while the code is empty); the marketplace row carries `选择计算方式`. Tapping the marketplace row with nothing selected yet opens the picker instead of selecting.
3. Preference card — `忽略0分成绩` switch, bound to the zero-score filter applied when loading scores.
4. Enable card — navigation row `编辑成绩启用状态` in `ham_blue` → enable screen.
5. Reset card — destructive row `重置所有成绩数据` in `ham_red`; tap → reset, then return to the score home.

**Values:** screen padding **16**, card gap **8**, card 16/16/`ham_bg_b2` · card title `bodyBold`, title→content **8** · row label `body` 16, subtitle `caption` 12, switch→label gap **4** · subtitle `ham_text_secondary`, tips `ham_red` · divider full card width, top pad **4** · navigation link `caption` in `ham_blue` · destructive `ham_red`. ⚠ iOS's dead `重置` / `重置成绩列表` card must be deleted.

**Strings:** 成绩设置 · 保护成绩数据 · 使用生物识别保护成绩 · 无法访问你的生物识别模块 · F2计算方式 · 使用计算机学院F2计算方式 · B1+B2×0.002（B2最多选8门） · 使用其它F2计算方式 · B1×0.98+B2×0.02 · 自定义计算方式 · 使用自定义的JavaScript计算F2 · 编辑代码 · 使用其它计算方式(基于JavaScript) · 选择计算方式 · 偏好 · 忽略0分成绩 · 成绩启用 · 编辑成绩启用状态 · 重置成绩 · 重置所有成绩数据.
⚠ iOS hardcodes `B1×0.98+B2×0.02` and spells the marketplace label `JavaScript`; Android resources both as `Javascript`. Use `JavaScript`.

**States:** empty → no rows in the enable screen; show the standard empty placeholder ⚠ neither platform does. Loading → none. Error → none. Select-mode → n/a.

##### Divergences
- Android has the 偏好 / 忽略0分成绩 card; iOS has no zero-score filter.
- iOS has the 自定义计算方式 row and the JS editor; Android has neither.
- iOS renders the enable list inline here; Android links to a separate route.
- Card order: the five above (Android's); iOS omits Preference.
- iOS reverts the protection switch when re-auth fails; Android leaves it on.
- Android pops to the score home after reset; iOS does not, and does not confirm.

---

#### Score enable (成绩启用)
**Purpose:** Toggle `isEnable` per course so disabled courses are excluded from GPA/F2. **Entry:** `编辑成绩启用状态` in Settings. **Platforms:** both.

**Layout:**
```
┌──────────────────────────────────────┐
│ nav 成绩启用 (inline)                │
│ scroll, pad 16, rows 8 apart         │
│  高等数学                      [ ○]  │ name body, 1 line
│  王老师                              │ instructor caption
│  ──────────────────────────────────  │ between rows only
│  线性代数                      [ ○]  │
└──────────────────────────────────────┘
```

**Content blocks, in order:**
1. Row — course name (`body`, 1 line) + instructor (`caption`) + trailing switch, checked from the set of enabled ids. Tap → write through and recompute statistics.
2. Divider between rows only, never after the last.

**Values:** full-screen scrolling list, padding **16**, row gap **8** ⚠ iOS: the same rows inline in a Settings card, no dividers, disabled-first sort · name `body` 16 `lineLimit 1`, instructor `caption` 12 `lineLimit 1` ⚠ Android has no line limits · switch checked → `ham_blue`.

**Strings:** 成绩启用 · 编辑成绩启用状态.
**States:** empty → standard empty placeholder ⚠ neither platform renders one. Loading → none. Error → none. Select-mode → n/a.

##### Divergences
- iOS: inline card, no dividers, disabled courses sorted first. Android: dedicated screen with dividers.

---

#### Score intro / connect (连接成绩)
**Purpose:** First-run flow shown when the user has never fetched scores. **Entry:** auto-presented from the root. **Platforms:** both.

**Layout:** the intro shell of the fetch screen retitled `连接成绩` (logo 56 r12, link, feature icon **48** in
`ham_brand_score @ 0.25`, title `title` 24 Bold, choice container radius 12 `ham_lightGray` pad 16) holding one CAS
choice row `从信息门户验证` / `将进入武汉大学信息门户网页验证你的身份`.

**Content blocks, in order:**
1. Intro shell titled `连接成绩` with one CAS choice row.
2. CAS login → captcha → fetching → success, identical to the fetch sheet.
3. Face ID enable step. Conditional: only when the device offers biometrics.
4. Completion — dismiss; the root gate then renders the main screen.

**Strings:** 连接成绩 · 取消 · 从信息门户验证 · 将进入武汉大学信息门户网页验证你的身份.
**States:** empty → n/a. Loading → fetching block. Error → failure block. Select-mode → n/a.

##### Divergences
- iOS: three sub-screens swapped by notification flags, with a possible blank frame before init resolves. Android: one sheet with a route per step, no blank frame.
- Intro icon is `.orange` on iOS, `ham_brand_score` on Android.

---

#### Face ID enable + Face ID / auth error (开启生物识别保护 · 验证失败)
**Purpose:** Offer the biometric lock after the first successful fetch, and block with a retry when unlock fails. **Entry:** last step of the connect flow (enable); rendered by the root gate when auth fails (error). **Platforms:** both.

**Layout (enable / error):**
```
┌──────────────────────────────────────┐
│ Column, pad 16, centred, gap 8       │  enable
│  🔒  (72, ham_gray, top pad 16)      │
│ ┌──────────────────────────────────┐ │
│ │ 使用生物识别保护成绩        [ ○] │ │
│ │ 无法访问你的生物识别模块         │ │ red caption, conditional
│ └──────────────────────────────────┘ │
│  gap 16                              │
│ [          确定          ] h48 r12   │
└──────────────────────────────────────┘
┌──────────────────────────────────────┐
│ centred column, gap 8, bg ham_bg_b1  │  error
│   🔒   (72, tap = retry)             │
│   验证失败           title 24 Bold   │
│   <message>          body 16         │
└──────────────────────────────────────┘
```

**Content blocks, in order (enable):** 1. Lock icon **72** `ham_gray`, centred, top pad 16 ⚠ iOS 48 pt. 2. Switch row `使用生物识别保护成绩` (`body`) + switch; when biometrics are unavailable the switch is disabled and the red `caption` hint appears. 3. Confirm button — full width, h 48, r 12, `ham_blue @ 0.15`, `headlineBold` `ham_blue`, label `确定`; tap → persist and dismiss.
**Content blocks (error):** 1. Lock icon **72** `ham_gray`, itself the retry button → re-run biometric auth. 2. Title `验证失败` `title` 24 Bold. 3. Message `body` 16 — `请重新验证` when the user failed or cancelled, `你已开启成绩保护，请开启生物认证权限` when the device has no enrolment.

**Values:** centred column, padding 16, gap **8**, bg `ham_bg_b1` · enable entrance fade-in after **200 ms** · min hit target 44 · commit `enableFaceId` **only** when biometrics are permitted ⚠ Android writes unconditionally.

**Strings:** 使用生物识别保护成绩 · 无法访问你的生物识别模块 · 确定 · 验证失败 · 请重新验证 · 你已开启成绩保护，请开启生物认证权限 · 保护你的成绩数据 (prompt reason).
⚠ iOS's enable label is the device-specific `开启Face ID保护你的成绩数据`; use the platform-neutral wording. iOS hardcodes the error title and both messages.

**States:** empty → n/a. Loading → n/a. Error → the unavailable branch (enable) / this screen is the error state. Select-mode → n/a.

##### Divergences
- Lock icon 48 pt iOS / 72 dp Android.
- iOS bails out of the enable screen immediately when biometrics are not permitted; Android shows the disabled state inline.
- iOS resets `faceIdUnlockState` then re-authenticates; Android sets `Unload` then `beginAuth()`.

---

#### JS calc picker + detail (选择计算方式)
**Purpose:** Browse the RN marketplace of community F2 calculators and inspect one. **Entry:** `选择计算方式` in Settings, then item tap. **Platforms:** both.

**Layout (picker):** nav title `选择计算方式` (inline) over an RN container hosting module `RNScoreCalcView` that fills the
body; safe areas respected. The module emits an open-detail event carrying the item and the host pushes the detail screen
(ignore the event if the item cannot be resolved). No layout values — the body is 100 % RN.

**Layout (detail):**
```
┌──────────────────────────────────────┐
│ scroll, padding 16                   │
│ ┌────────┐ <title>    title2 20 Bold │
│ │ icon   │ <brief>      caption, 2ln │
│ │ 64 r12 │ [ 选择 / ✓ 已选择 ]       │
│ └────────┘                           │
│ 👤 <author>  📅 <date> 更新          │ captionBold, pad v 8
│ ── divider, pad v 8 ──               │
│ 【更新日志】              captionBold│
│ <updateBrief>            caption, ≤8 │
│ <desc or 暂无简介>          caption  │ gap 8
│ ── divider, pad v 8 ──               │
│ 代码详情                          ›  │ body 16, ham_blue
└──────────────────────────────────────┘
```

**Content blocks, in order (detail):** 1. Header row — icon + title + brief + select button; tapping selects the calculator, after which it shows a checkmark + `已选择`. 2. Author/date row — `person` and `calendar` glyphs **16** `ham_gray`, gap 8 (inner 2), vertical pad 8; date only when non-empty; author capped at 200 wide, 1 line. 3. Update log — heading `【更新日志】` `captionBold`, body `caption` capped at **8** lines. 4. Description `caption`, falling back to `暂无简介` when empty. 5. `代码详情` row — `body` 16 `ham_blue` + `chevron.right` tinted `ham_blue`; shown only when the item has a url and is not an APP-type item.

**Values (detail):**

| Element | Value |
| --- | --- |
| Outer padding / icon | **16** all round; icon **64×64**, radius **12**, `ham_gray @ 0.2` ⚠ Android 72 dp; icon→text gap **8** ⚠ Android 16 dp |
| Title / brief | `title2` 20 Bold, 1 line / `caption` 12, 2 lines, `ham_text_secondary` |
| Select button | pad h **8** / v **8**, `ham_gray @ 0.2`, radius **8**; label `body` 16 `ham_blue`; selected = checkmark **24** + `已选择`, gap 2 ⚠ Android v-pad 4; iOS label inherits `Color.primary` |
| Author / date | `captionBold` 12, 1 line, max width 200 ⚠ iOS `caption`, author 2 lines, max 150; divider vertical pad **8** ⚠ Android 16 dp |
| Update log / description | heading `【更新日志】` `captionBold`; body `caption`, max 8 lines ⚠ iOS unlimited; description `caption`, empty → `暂无简介` ⚠ iOS renders blank |

**Strings:** 选择计算方式 · 选择 · 已选择 · %1$s 更新 · 【更新日志】 · 暂无简介 · 代码详情.
**States:** empty → `暂无简介` fallback. Loading/error → owned by the RN module (picker); none on the detail. Select-mode → n/a.

##### Divergences
- iOS ignores the top and bottom safe areas in the picker; Android keeps them.
- iOS matches the selected item by title **and** url; Android resolves from a static url→item map and bails when absent.
- iOS merges `【更新日志】` and the log body into one `\n` string; Android uses two.
- Icon 64 vs 72; icon→text gap 8 vs 16; divider padding 8 vs 16.

---

#### JS F2 editor (JS代码编辑)
**Purpose:** Author custom JavaScript that computes the F2 score. **Entry:** `编辑代码` in Settings. **Platforms:** both (⚠ iOS only today; Android must add the route).

**Layout:**
```
┌──────────────────────────────────────┐
│ nav JS代码编辑 (inline) [重置] [保存] │
├──────────────────────────────────────┤
│ pad 16, cards 8 apart                │
│ ┌──────────────────────────────────┐ │
│ │ inputJsonStr的示例数据为         │ │
│ │  read-only JSON sample, h 128,   │ │
│ │  r 16 ─────────────────────────  │ │
│ │ 请完成calc方法，返回相应的数据   │ │
│ └──────────────────────────────────┘ │
│ ┌──────────────────────────────────┐ │
│ │ JS editor, radius 16, theme ocean│ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

**Content blocks, in order:**
1. Example card — label, read-only JSON sample editor (h **128**, r **16**), instruction line.
2. Editable JS editor — radius **16**, theme `ocean`, bound to the code; seeds with the default template when nothing is stored.
3. Toolbar `重置` — restore the seed template.
4. Toolbar `保存` — validate then persist: evaluate the script against a one-item fixture (高等数学 / 王老师 / 5.0 学分 / 96 分), call `calc`, require a `[Double, [String]]` result. Success → persist, toast `保存成功`, notify that scores changed. Failure → toast `存在语法错误` / `请修复语法错误，然后重试`.

**Values:** outer padding **16**, card gap **8** · card 16/16/`ham_bg_b2`, screen bg `ham_bg_b1` · sample editor height **128** · editor radius **16** · nav title `JS代码编辑` inline with two trailing toolbar buttons.

**Strings:** JS代码编辑 · inputJsonStr的示例数据为 · 请完成calc方法，返回相应的数据 · 重置 · 保存 · 保存成功 · 存在语法错误 · 请修复语法错误，然后重试.
⚠ The seed JSDoc comment block is hardcoded on iOS; resource it.

**States:** empty → never (the editor seeds the template). Loading → none. Error → toast only, not an inline state. Select-mode → n/a.

##### Divergences
- Android has no route and no 自定义计算方式 row in Settings.

---


---

### Course score

#### Permission gate (权限请求)

**Purpose:** Block the module until the user consents to anonymised score upload and has fetched scores once. **Entry:** sheet over Search home, after a 500 ms delay, once per session. **Platforms:** both.

**Layout:**

```
┌────────────────────────────────────┐
│ [取消]                             │  44
│     🔒 56 brand · 权限请求 28 bold  │
│   "给分"需要使用你的成绩数据  16     │
│ ┌────────────────────────────────┐ │
│ │ 使用"给分"前Ham会自动…          │ │  h 350, scrollable, r10
│ │ 你的以下信息将会被上传到服务器    │ │  gray .1, h-pad 16
│ │  - **学号不可逆特征值** 用于…     │ │  body 16, spacing 16
│ │  - **脱敏成绩信息** 用以提供数据  │ │  4 spacer each side
│ │  - **设备信息** 辨别你是否…       │ │  of the bullet list
│ │ 数据库里的成绩信息不能逆向定位…   │ │
│ └────────────────────────────────┘ │  ⊘ warning / [授权] 48
└────────────────────────────────────┘  mutually exclusive
```

**Content blocks, in order:** 1. `取消` — dismisses; without consent this pops the whole module. 2. Lock glyph `lock.fill` 56, brand blue. 3. Title `权限请求`. 4. Subtitle. 5. Scrollable consent panel, fixed height, bold on the three **terms** only. 6. Footer: `授权` button **if** scores fetched, else the red warning column + the CCKV `courseScoreEnableDescription` line.

**Values:**

| Element | Value | Note |
| --- | --- | --- |
| Consent panel | h 350, r10, gray 0.1, h-pad 16, body 16, spacing 16 | Android h320, r12, gray 0.2 |
| Bullets | `"- "` prefix, term bold, rest regular | iOS embeds `**bold**` in one localized string; Android splits each bullet into 2 resources |
| Warning | `xmark.circle.fill` 16 red; copy red, hint `.caption` | Android uses `Close` on a filled circle |
| `授权` button | full width, h48, r12, brand @ 0.15 fill, brand text, v-pad 16 | Android h48; iOS is a plain padded fill |
| Padding / gate | 16 / `permitUploadScore && hasFetchScore` | iOS omits `permitUploadScore` |

**Strings:** `取消` · `权限请求` · `"给分"需要使用你的成绩数据` · `使用"给分"前Ham会自动将你的成绩数据匿名发送到Ham的服务器上，作为该功能的数据来源。` · `你的以下信息将会被上传到服务器` · `- **学号不可逆特征值** 用于下次更新数据` · `- **脱敏成绩信息** 用以提供数据` · `- **设备信息** 辨别你是否正常使用Ham` · `数据库里的成绩信息不能逆向定位到任何一个人，Ham也不会将这些成绩作为非法用途` · `同意后，每次使用"给分"前，Ham会自动上传你的成绩信息。` · `你尚未获取成绩信息，因此无法使用"给分"` · `请前往"成绩"页面获取成绩后重试` · `授权`. The CCKV description line is server text.

**States:** *has scores* → `授权`. *no scores* → red warning, no button. *dismissed without consent* → pop the module after 0.6 s. *loading* → none. Consent write-back also uploads scores.

---

#### Search home (课程评分 / 给分)

**Purpose:** Module landing — brand mark, fake search bar, past-keyword cloud. **Entry:** tab / root route. **Platforms:** both.

**Layout:**

```
┌────────────────────────────────────┐
│ 给分                      👤 我的数据│  44
│            ▤ 48 brand              │  32 v-pad
│ ┌────────────────────────────────┐ │
│ │ 搜索  24 bold, t2              │ │  r16, gray .15
│ └────────────────────────────────┘ │  h-pad 16, bottom 4
│ [📖 高等数学] [👤 张三] … ≤30       │  4 rows, gap 4, h-pad 16
│ 其他服务 16bold · [(icon) Title  > ]│  card pad 16, r16
│                Subtitle            │
└────────────────────────────────────┘  bg ham_bg_b1
```

**Content blocks, in order:** 1. Toolbar trailing `我的数据` — gated on `EnableCourseCenter`; logged in → Course center, anonymous → login dialog; person glyph + label, gap 4, brand, bold. 2. Brand mark `chart.bar.xaxis` 48 brand, centred, 32 v-pad, not tappable. 3. Search bar — full-width round-rect; tap → Search with a matched-geometry morph over 0.3 s. 4. Keyword cloud — horizontal scroll, newest first, max 30; tap commits the keyword and opens the result list; 1-line names. 5. `其他服务` — header 16 bold, then one card per remote-config item (`title-content[locale]`, `subtitle-content[locale]`, `url`, `iconURL`), icon clipped to a circle; tap opens the URL and logs `promotion_btn`.

**Values:**

| Element | Value | Note |
| --- | --- | --- |
| Search bar | r16, gray 0.15, inner pad 16 h / 10 v, outer h-pad 16, bottom 4; label 24 bold t2 | Android inner 24 h, gray 0.2, label 24 |
| Brand icon | 48, brand, 32 v-pad | Android 64 |
| History rows / cap | 4 rows, `prefix(30)`, gap 4, h-pad 16, newest first | Android 8 rows, uncapped |
| History chip | r8, gray 0.15, pad 8 h / 6 v, text t1 @ 0.65, 1 line; book glyph (course) / person glyph (instructor), icon 18, gap 4 | Android r10, pad 4 h, t2, unlimited lines, icon gap 2 |
| `其他服务` header / card | 16 bold, spacing 8 / card pad 16 r16 | **absent on Android** |
| `我的数据` | icon + label, gap 4, brand, bold | |

**Strings:** `给分` · `我的数据` · `搜索` · `其他服务`. External-service text is server data.

**States:** *empty history* → collapse to zero height, no placeholder. *empty `其他服务`* → hide header and card. *not logged in* → button still shown, tap opens login. *loading / error* → none (local storage). *permission not granted* → replaced by the gate.

---

#### Keyword suggestions (搜索建议)

**Purpose:** Live autocomplete rows as the user types. **Entry:** tapping the home search bar or a history chip (pre-filled). **Platforms:** both.

**Layout:**

```
┌────────────────────────────────────┐
│ [<] [输入课程名或授课人        ] [⊗]│  56, v-pad 8
├────────────────────────────────────┤  divider, 16 spacer above
│  🔍 高<b>等数</b>学             ↗   │  h-pad 16, top 8, gap 8
│  🔍 高等代数 (张三)             ↗   │
└────────────────────────────────────┘  bg ham_bg_b1
```

**Content blocks, in order:** 1. Back chevron — pops to home; system back button hidden. 2. Text field — 1 line, autofocus after 0.3 s, re-query on every keystroke (no debounce), blank input not sent. 3. Clear — visible only when non-blank; clears the field **and** the result list and resets load state. 4. Divider. 5. Suggestion rows — leading search glyph, hit text with the `<strong>` run emphasised, trailing `arrow.up.right`; tap commits the suggestion, **does not** copy it into the field, opens the result list.

**Values:**

| Element | Value | Note |
| --- | --- | --- |
| Field | 20, 1 line | |
| Clear icon | `multiply.circle.fill`, trailing pad 8 | Android always visible, clears text only |
| Row pad / gap | h-pad 16, top 8, gap 8; 16 spacer above the divider | Android h-pad 12, no spacer |
| Leading icon | search glyph 24, `.foregroundStyle(gray, gray 0.25)`, 8 gap to text | Android 16 |
| Hit text | `<strong>` → t1, plain → t2, body 16 | Android sets a 14sp span size then overrides with a 16sp style |
| Highlight parsing | scan `<strong>`/`</strong>`; on malformed markup fall back to plain text — **never crash, never drop the tail** | iOS force-unwraps (crash); Android swallows the remainder |

**Strings:** `输入课程名或授课人`. Row text is the server `hit` field.

**States:** *no input* → no rows. *loading* → no visual. *error* → none. *dismiss* → chevron or edge-swipe (`start.x < 50`, `translation.width > 80`).

---

#### Course result list (课程搜索结果)

**Purpose:** Paginated course cards for a committed keyword. **Entry:** committing a suggestion or history keyword. **Platforms:** both.

**Layout:**

```
┌────────────────────────────────────┐
│ ┌────────────────────────────────┐ │
│ │ 🔍 高等数学                     │ │  r16, gray .1 over b1, pad 16
│ └────────────────────────────────┘ │  shadow gray .5 r16
│ [人数倒序] [均分倒序] [中位数倒序]  │  gap 8, h-pad 16, v-pad 8
│ ┌────────────────────────────────┐ │
│ │ 高等数学 (2 lines)     ┃  均分  │ │  card pad 16, r16
│ │ 张三                   ┃  84.5  │ │  divider 120, col 64
│ │ 128位同学的成绩 (8)     ┃ 中位:90 │ │  bar max 180 h4 r2
│ │ 0-59 ▓▓▓▓▓▓▓ 12 (4)    ┃   [↗]  │ │
│ └──────────────────────────[💬]───┘ │
│  gap 8 · bottom spacer navBar + 24  │
└────────────────────────────────────┘
```

**Content blocks, in order:** 1. Keyword echo header — search glyph 20 + keyword, 1 line; tap → back to the suggestions surface. 2. Sort chips — always rendered, order `人数倒序` / `均分倒序` / `中位数倒序`, default `人数倒序`; tap clears the list, resets cursor and end-flag, cancels the in-flight request, re-requests. 3. Card list — gap 8, h-pad 16, bottom spacer `navBar + 24`, pagination at `count - 2`. 4. Share overlay top-trailing. 5. Comment overlay bottom-trailing — only when `enableCourseComment`; anonymous tap opens the login dialog, otherwise opens Course detail by `courseTableId`.

**Values:**

| Element | Value | Note |
| --- | --- | --- |
| Card | pad 16 uniform, r16 | Android pad 0 + 12 insets |
| Name / instructor / gaps | name 16 bold, max 2 lines, ellipsis; instructor `.caption` 12; 8 below the name, 4 between range rows | iOS 16 / Android 2 |
| Count | `%d位同学的成绩`, `.caption` | |
| Range label | `"\(from)-\(to)"`, `.caption2`, width 48, leading | iOS 45 |
| Bar | max 180, h4, r2, trailing pad 4, width `180 * size / total`, **omitted when `size == 0`** | iOS result card h6 r3, renders a zero-width bar |
| Bar colour | server `range.color` → CCKV `colorMap["from-to"]` → brand blue | iOS has no colourMap |
| Divider / stat column | h120, 16 top-bottom / width 64, spacing −2 | iOS 130 |
| `均分` / average | `.caption` label; value `%.1f`, 28 bold rounded | Android 24sp plain Bold |
| Median | `中位数: %@`, `.caption2`, **only when `median > 0`**; stat column always rendered | iOS always shows the median but hides the column when `total <= 2` |
| Share / comment | top- / bottom-trailing, 32×32 r8 gray 0.2, glyph 16, inset 12 | Android glyph 20 |
| Chip | `.caption`, pad 6 h / 4 v, r6, `color @ 0.1`; selected brand, unselected gray | |

**Strings:** `人数倒序` · `均分倒序` · `中位数倒序` · `%lld位同学的成绩` · `均分` · `中位数: %@` · `分享到`. Share payload: title `%@-%@的给分数据`, summary `来自Ham` — do not hand-roll a `" | "` description.

**States:** *empty* → centred empty view **with copy** (neither platform has one today). *loading* → no spinner. *error* → toast only. *end of list* → no footer; stop paging when a page returns zero items. *not logged in* → comment bubble still shown, tap opens login.

**Share:** system sheet with a signed web URL — `{DocHost}/whu-ham/inner/course-score/result?name=&instructor=&data=&dataTime=&sign=&v=2`, `sign = md5("v2" + name + instructor + json + dataTime + salt).lowercase()`. The salt is a compile-time secret duplicated on both platforms; read the host from config, never hardcode it. Android currently offers QQ only, at a 0.4 sheet-height ratio.

---

#### Course detail (课程详情)

**Purpose:** Everything about one course — grade distribution, rating distribution, want/review actions, comment preview, own review. **Entry:** a result card's comment bubble (by `courseTableId`), a course-center card, or the match flow. **Platforms:** both.

**Layout:**

```
┌────────────────────────────────────┐
│ <  课程详情                         │
│ ┌─ ① 成绩统计 ────────────────────┐ │  pad 0; h-pad 16, v-pad 12
│ │ 课程名 教师   ┃ 均分 84.5        │ │  divider 120, stat col 64
│ │ 128位同学的成绩┃ 中位数: 90       │ │  bar max 180 h4 r2
│ │ 0-59 ▓▓▓ 12  ┃                  │ │
│ │ ── [全部] ┃ [2024-1学期] … ────  │ │  semester row
│ └────────────────────────────────┘ │
│ ┌─ ② 评分 ───────────────────────┐ │  watermark star 128 @ (24,24)
│ │ ★★★★★ ▓▓▓▓▓▓▓ 40          4.0  │ │  track 144×4 r2, avg col 84
│ │ ★★★★☆ ▓▓ 12               平均  │ │  67位同学的评分
│ └────────────────────────────────┘ │
│ ┌─────────────┐ ┌─────────────┐    │  ③ gap 8, r12, v-pad 8
│ │ ♥ 想上       │ │ ✎ 写评价     │    │  bg color @ .1
│ │   36人想上…  │ │   说点什么…  │    │
│ └─────────────┘ └─────────────┘    │
│ ┌─ ④ 评论 24 ────────────────────┐ │
│ │ (av32) name ★★★★★          👍 8 │ │  gap 12, divider per row
│ │ body · 2026-01-05 12:30         │ │  查看全部 > (centred, brand)
│ └────────────────────────────────┘ │
│ ┌─ ⑤ 你的评价 ───────────────────┐ │  iff rated or commented
│ │ ★★★★★        收获8个 👍         │ │  star 24 orange
│ │ ──────────────────────────────  │ │  divider iff both
│ │ 讲课很清楚 · 2026-01-05 12:30    │ │
│ └────────────────────────────────┘ │
└────────────────────────────────────┘
 gap 16 between blocks, h-pad 16, bottom pad 32, bg ham_bg_b1
```

**Content blocks, in order:** ① Grade-stat card (when `course_grade_stat_info` exists) · ② Rate card (`rate_info`) · ③ Function row (`create_review_info`) · ④ Comment card (`course_comment_info`, three-way on `enable_state`) · ⑤ Self-review card (only when `rate_info` or `comment_info` exists). **Render every block conditionally** so a sparse payload degrades instead of showing empty cards.

**Values:**

| Element | Value | Note |
| --- | --- | --- |
| Container | list gap 16; padding 16 h, 0 top, **32 bottom**; bg `ham_bg_b1`; card pad 16 r16, title 16 bold; refetch on the create-review success notification | |
| ① Padding | card 0; content h-pad 16, v-pad 12 | Android 12 all sides |
| ① Name / count | name 16 bold, max 2 lines; instructor `.caption` 12; `%lld位同学的成绩` `.caption`; 8 below the name, 4 above the bars | iOS name unbounded; Android gaps 2 |
| ① Range label | `"\(from)-\(to)"`, `.caption2`, width 48, leading | iOS 45 |
| ① Bar | max 180, h4, r2, gap 4, width `180 * range.total / total`, **omitted when `range.total == 0`**; count suffix `.caption2`, pad 4 when non-zero | |
| ① Bar colour | server `range.color` else gray | Android: server → colourMap → brand |
| ① Divider / stat column | h120, 16 top-bottom / width 64, spacing −2 | iOS 130 |
| ① Average / median | `均分` `.caption` + `%.1f` 28 bold rounded / `中位数: %@` `.caption2`, **only when `median > 0`** | Android 24sp plain; iOS always shows the median |
| ① Semester row | `全部` chip · 24-tall vertical divider with 8 leading pad · horizontal scroller gap 6 pad 8; chips `.caption`, brand text, pad 4, r6, selected bg brand @ 0.15 else clear; label `%@-%@学期` | Android chips have no text style (render at 16) and selected bg brand @ 0.1 |
| ① Semester refetch | dim the card with black 0.35 + a 48/4 spinner that swallows taps; guard re-entrancy; on error toast and **leave selection and data untouched** | Android has no re-entrancy guard |
| ② Card / stars | title `评分`; watermark `star.fill` **128**, gray @ 0.1, offset (24, 24); exactly 5 rows 5→1, glyph 10, filled gray / empty gray @ 0.2, gap 2 | Android watermark 172 @ offsetY 24 and renders `max(star)` glyphs per row |
| ② Track / row count | 144 × 4, r2, track gray 0.2, fill gray, 8 leading pad / `%.0f`, `.caption` t2, 8 pad | Android track gray 0.1 |
| ② Average column | width 84, trailing; `%.1f` at 28 bold rounded; label `平均` at 10; footer `%lld位同学的评分` `.caption` 4 top pad | **Android renders `%.2f`** (4.00 vs 4.0) |
| ③ Row | two equal-weight buttons, gap 8, r12, v-pad 8, bg `color @ 0.1` over the card bg | Android stacks `ham_bg_b1` then the tint |
| ③ Icons / copy | heart (want) + **pencil** (write review), 24; icon + column gap 8; title 16 bold; subtitle `.caption` 12; leading alignment | iOS uses `paperplane.fill`; Android icons 28, subtitle 11, centred |
| ③ Want / review state | want: brand → gray and disabled once recorded; review: brand when server `enabled` else gray and disabled | |
| ③ Want tap | optimistic `total += 1` + `recorded = true`, roll back on error, guard re-entrancy | |
| ③ Review tap | push Create review with `courseTableId`, current `star`, current comment text | iOS route arguments; Android SavedStateHandle |
| ④ Title / body | `评论 %lld` / leading column gap 12, divider after each row | |
| ④ Empty / footer | `暂无评论`, body, t1, suppress the footer / centred `查看全部` + chevron, gap 2, brand → Comment thread | iOS chevron 12, Android 24 |
| ④ Disabled | icon 64 brand @ 0.65, pad 12, gap 16, then the server `reason` in bold; **no card, no title** | Android wraps it in a titled `评论` card, 72 icon, fallback `发表课程评论后，才可以查看其它评论哦` |
| Row avatar | 32 circle, crop, 0.5 s fade; no URL → gray 0.2 circle with 8pt `avatarIDText`, else person glyph 20; 8 gap to body | Android has explicit loading/error placeholders |
| Row text | body column gap 8; username `.caption2` 11, 1 line; stars `star.fill` **12** t1, 4 gap; body 14 t1; time `yyyy-MM-dd HH:mm` `.caption` t2 | Android username `.caption` 12; iOS star 10, Android 16 untinted |
| Row like column | width 32, gap 4; filled/outline thumbs-up 16 + count `.caption`; brand when liked, gray otherwise; **disabled on your own comment**; optimistic ±1 with rollback and re-entrancy guard | |
| ⑤ Title / gap / stars | `你的评价`; column gap 12; `star.fill` **24** orange, gap 2 | Android gap 8, star 32 |
| ⑤ Title / gap / stars | `你的评价`; column gap 12; `star.fill` **24** orange, gap 2 | Android gap 8, star 32 |
| ⑤ Not rated / likes / comment | `未评分` 28 / `收获%lld个` `.caption` + thumbs-up 12, gap 4 / divider only when there is both a rating and a comment, then body text and `yyyy-MM-dd HH:mm` `.caption` t2, gap 8 | iOS likes gap 0; Android comment gap 0 |

**Strings:** `课程详情` · `%lld位同学的成绩` · `均分` · `中位数: %@` · `%@-%@学期` · `全部` · `评分` · `平均` · `%lld位同学的评分` · `%lld人想上这门课` · `评论 %lld` · `暂无评论` · `查看全部` · `你的评价` · `未评分` · `收获%lld个` · `评论` · `发表课程评论后，才可以查看其它评论哦` · `尴尬了，网络请求失败了` · `试试重新请求呢` · `重试`. Want/review titles and the disabled reason are server data.

**States:** *initial loading* → centred spinner replacing the content. *error* → full-screen view: 🥲 at 96, title `尴尬了，网络请求失败了`, subtitle = the caller-supplied message or `试试重新请求呢`, full-width `重试` (r12, brand @ 0.1 fill, pad 8 h / 12 v) that re-fetches. *empty comments* → `暂无评论`. *comments disabled* → icon + reason. *semester refetch* → dimmed overlay. *not logged in* → no local gating; the server drives `enabled`.

---

#### Comment thread (课程评论)

**Purpose:** Full paginated list of every comment on a course. **Entry:** the detail screen's `查看全部` footer. **Platforms:** both.

**Layout:**

```
┌────────────────────────────────────┐
│ <  课程评论 24                      │
│ (av32) name ★★★★★              👍 8│  h-pad 16, bottom 64, gap 12
│  body · 2026-01-05 12:30 ─────────  │  divider between rows
│ (av32) name …                       │
│       ⟨spinner, 1s delay⟩           │  after last item only
└────────────────────────────────────┘  bg ham_bg_b1
```

**Content blocks, in order:** 1. Title `课程评论 %@` — while the total is unknown render the **bare title**, never a format string with an empty substitution. 2. First-page spinner, only while the list is empty. 3. Rows — the shared comment row (detail, "Row avatar" / "Row text" / "Row like column"). 4. Divider between rows, none after the last. 5. Next-page spinner, after a **1000 ms** delay if still loading. 6. Pagination at `count - 2`.

**Values:** padding 16 h, 16 top (nav-managed), **64 bottom** (iOS 32); row gap 12; bg `ham_bg_b1`.

**Strings:** `课程评论 %@`.

**States:** *first-page loading* → centred spinner. *next-page loading* → delayed footer spinner. *error* → full-screen error view with retry **only while empty**, otherwise a toast. *empty* → no state today; add a centred empty view. *end of list* → no footer text. *not logged in* → no gating (like is per-row).

The legacy name/instructor comment page (with its own input bar) is dead on both platforms — do not build it.

---

#### Create review (创建评价)

**Purpose:** Rate a course 1–5 stars and/or write a review, with server-driven length validation. **Entry:** the detail screen's 写评价 button. **Platforms:** both.

**Layout:**

```
┌────────────────────────────────────┐
│ <  创建评价                  [发布] │  chip r6, inline spinner
│       点击/滑动评分                 │  16
│     ★    ★    ★    ★    ★          │  40 each, centred
│  ⚠ 你给这门课程评过分了… (red)      │  iff !canRate
│ ────────────────────────────────── │  divider, 12 top pad
│ ┌────────────────────────────────┐ │
│ │ 发布一条课程评价吧～  (t2 @ .3)  │ │  minH 80, maxH 180
│ └────────────────────────────────┘ │
│ 评论需满足20-100字，当前0字          │  .caption; red if out of range
└────────────────────────────────────┘
 success → SuccessView(发布成功, 已成功发布课程评分与评论) → dismiss
```

**Content blocks, in order:** 1. Trailing `发布` chip — hidden once success, disabled while submitting with an inline 16 spinner. 2. Prompt `点击/滑动评分`. 3. Five stars — tap **and** horizontal drag set the value (drag threshold 5); **both** paths must respect `canRate`; blur the field on change. 4. Already-rated warning, red `.caption`, 8 top pad. 5. Divider. 6. Editor — multiline, top-leading, min 80 / max 180, 6 visible lines, `Done` clears focus, **strip newlines**. 7. Placeholder when empty. 8. Then either the already-commented warning (red) or the length counter. 9. Success — replace the content with the shared success view, dismiss, notify the detail screen to refresh.

**Values:**

| Element | Value | Note |
| --- | --- | --- |
| Root | pad 16, gap 16 | |
| Star glyph / row | **40**, orange / gray 0.2; intrinsic width, centred | iOS 32 / Android 56; iOS pins 32×5 + 16×4 = 224 |
| Editor | min 80 / max 180, body 16, `lineLimit 6` | |
| Placeholder | `ham_text_secondary @ 0.3`, shown when the text is empty | Android also requires unfocused; iOS alpha 0.3, Android full secondary |
| Counter | `.caption`, t2 in range / red out of range; **always rendered** | Android hides it when config is null |
| Min / max / hint | from config, defaults **20 / 100 / `发布一条课程评价吧～`**; read at render time | iOS freezes them as `let` at init; Android skips validation entirely when config is null |
| Submit chip | r6, pad 16 h / 4 v, `color @ 0.1` fill, brand text | iOS is a plain text bar item with no spinner |
| Initial state / validation | `star = initStar > 0 ? initStar : 5`; `canRate = initStar == 0`; `canComment = initComment.isEmpty`; block when `canComment` and the length is outside min…max; toast `评论字数未符合要求` | |
| Post success | notify the detail screen to refresh **unconditionally** | Android notifies only when a flag flipped |

**Strings:** `创建评价` · `发布` · `点击/滑动评分` · `你给这门课程评过分了，不如去填写评价呢` · `你评论过该课程了，不如去评下分呢` · `评论需满足%lld-%lld字，当前%lld字` · `发布一条课程评价吧～` · `评论字数未符合要求` · `发布成功` · `已成功发布课程评分与评论`.

**States:** *submitting* → chip disabled + spinner. *already rated / already commented* → that sub-editor disabled + red caption. *invalid length* → blocked with a toast, counter red. *success* → success view then dismiss. *error* → toast, form kept. *not logged in* → inherited from the detail screen's `enabled` flag.

---

#### Course center (我的数据)

**Purpose:** The user's own footprint — profile header plus up to three summary cards, each with a `查看全部` jump. **Entry:** the home toolbar `我的数据` button (gated on `EnableCourseCenter`). **Platforms:** both.

**Layout:**

```
┌────────────────────────────────────┐
│ <  我的数据                         │
│ ┌────────────────────────────────┐ │  ① brief, tinted, tappable
│ │ (av64) Nickname 20bold · desc  │ │  3 lines, gap 16 to text
│ └────────────────────────────────┘ │
│ ┌─ ② 成绩排行 ───────────────────┐ │  card gap 16, inner gap 16
│ │ 高等数学 张三            >      │ │  name/instructor 1 line
│ │  0-59 ┃▓▓▓▓   你的分数  98     │ │  bar column 210
│ │  60-69┃▓▓              前 3.2% │ │  band row 16, gap 4
│ │ ──────── 查看全部 > ──────────  │ │  divider iff list non-empty
│ └────────────────────────────────┘ │
│ ┌─ ③ 想上 ───────────────────────┐ │  ♥ pink 28, gap 8
│ │ 高等数学 张三        >  2026-01 │ │  name/instructor 3 lines
│ └────────────────────────────────┘ │
│ ┌─ ④ 我的评价 ───────────────────┐ │
│ │ ▌6×12 高等数学 >      ★★★★★   │ │  stars 16 orange, col 80
│ │ 评论正文 · 2026-01-05 12:30     │ │  body 16, time .caption
│ └────────────────────────────────┘ │
└────────────────────────────────────┘  h-pad 16, bg ham_bg_b1
```

**Content blocks, in order:** ① profile (if `brief_info`) · ② rank (if `score_rank_info`) · ③ want (if `want_info`) · ④ comment history (if `course_comment_info`). Each of ②③④ is a titled card (title is server data) with gap 16, an item separator, an empty line when the list is empty, and a `查看全部` footer when `show_more`.

**Values:**

| Element | Value | Note |
| --- | --- | --- |
| Card gap / padding | **16** / 16 | iOS 0 — cards touch |
| ① Container | card, bg = server `background_color` else brand @ 0.1; the whole row is tappable → user centre | iOS is a bare untappable row with no bg |
| ① Avatar / gap | 64 circle, crop, fade; loading/error → gray 0.2 + person 32; 16 to text | |
| ① Nickname / bio | 20 bold primary, gap 4 to bio / `.caption` t2, 3 lines, tail truncation, hidden when empty | iOS `.title2` 22 regular; iOS bio uses `Color.gray` |
| ②③④ gap / empty | 16 inner gap, separator per item / `没有已上传的成绩记录` · `没有想上课程记录` · `没有课程评论历史记录`, body t2 | **empty copy absent on iOS** |
| Footer | centred `查看全部` + chevron, gap 2, **brand on all three cards**; preceding divider only when the list is non-empty | iOS tints only the rank footer and always inserts a divider |
| Rank item | name 16 bold + instructor `.caption`, **1 line each**, chevron 16; bar row gap 4 | iOS unbounded lines, gap 8 |
| Rank bars | band row 16; label `"\(from)-\(to)"` `.caption2`; vertical divider 4 leading pad; bar column width **210** | |
| Rank fill | self-fill = `selfProgress / bucket.progress`, where `selfProgress = progress − Σ(progress where score > scoreTo)`; clamp to 0.01…1; trailing corner radius **8** (= itemHeight/2); track = colour @ 0.15 | **iOS divides by `maxProgress`, not the bucket progress**; Android radius 6 |
| Rank score / percentile | label `你的分数` `.caption2`; value 28 bold rounded, right-aligned, 32 leading pad; then `TOP1` bold when `progress >= 1`, else `前` + `%.1f%%` of `(1 − progress) * 100` | Android 24sp plain + `offset(y:4)`; iOS composes the percentile as two weighted runs |
| Want item | pink heart (1, 0.41, 0.71) 28, gap 8; name 16 bold 3 lines, instructor `.caption` 3 lines, chevron; **timestamp `yyyy-MM-dd HH:mm` `.caption` t2** | Android renders **no timestamp** |
| Review item | identifier bar r3 6×12, server `identifier_color` else brand; name 14 1 line + chevron 16 (gap 2); stars 16 orange in an 80-wide trailing column with 12 leading pad; body 16 with 8 top pad; timestamp `.caption` t2 | iOS name is `.headline` 17 semibold |

**Strings:** `我的数据` · `查看全部` · `你的分数` · `前` · `TOP1` · `请求失败` · `重试` · `服务器没有返回数据` · `没有已上传的成绩记录` · `没有想上课程记录` · `没有课程评论历史记录`.

**States:** *loading* → centred spinner replacing the content. *error* → error view `请求失败` + retry, showing the server message when there is one and `服务器没有返回数据` when the response itself is null. *empty sub-list* → the per-card empty line. *not logged in* → unreachable (the entry point redirects to login). *partial payload* → omit missing cards.

---

#### Course-center sub-views (评论历史 / 想上历史 / 成绩排行)

**Purpose:** The full paginated lists behind the three course-center cards. **Entry:** each card's `查看全部` footer. **Platforms:** both. One shared template.

**Layout:**

```
┌────────────────────────────────────┐
│ <  评论历史 24                      │  title + count
│ ┌────────────────────────────────┐ │
│ │ <item view>                     │ │  no card wrap, gap 16
│ └────────────────────────────────┘ │  h-pad 16
│ ──────────────────────────────────  │  divider between items
│ ┌────────────────────────────────┐ │
│ │ <item view>                     │ │
│ └────────────────────────────────┘ │  ⟨spinner⟩ after last
└────────────────────────────────────┘
```

**Content blocks, in order:** 1. Title **with the count** — `评论历史(%d)` / `想上历史(%d)` / `成绩排行(%d)`, falling back to the uncounted form while the total is unknown. 2. Fetch on appear. 3. First-page spinner while the list is empty. 4. Items — reused verbatim from the course-center cards, **never wrapped in an extra card**. 5. Divider between items only. 6. Next-page spinner after the last item while loading. 7. Pagination at `count - 2`.

**Values:**

| Element | Value | Note |
| --- | --- | --- |
| Item gap / padding | 16 / 16 h, 16 bottom, top from the nav bar | |
| Background | page bg `ham_bg_b1` | iOS uses `ham_bg_b2` (card white) |
| Spinners | first page: centred; next page: centred after the last item, no divider | iOS pins the first-page one to the top and prepends a divider |

**Strings:** `评论历史` / `评论历史(%d)` · `想上历史` / `想上历史(%d)` · `成绩排行` / `成绩排行(%d)`.

**States:** *loading* → spinners as above. *empty* → **add a centred empty view** (today Android renders a blank screen, iOS an empty list). *error* → **add a retry view** (absent on both). *end of list* → no footer text.

---

#### Course detail — match variant (课程详情 · 匹配)

**Purpose:** Resolve a course by (name, instructor) when no `courseTableId` exists, then render the identical detail content. **Entry:** pushed from the Score module's single-score cell. **Platforms:** both.

**Layout:**

```
┌────────────────────────────────────┐
│ <  课程详情   ·   ⟨spinner⟩         │  loading
│    ❓ 64 gray + <server message>    │  not found, gap 8, centred
│    <full detail content, § above>  │  resolved
└────────────────────────────────────┘
```

**Content blocks, in order:** 1. Loading → centred spinner. 2. Resolved → the exact same content view as Course detail, parameterised by the resolved `courseTableId`. 3. Not found (`success == false`) → centred question glyph 64 gray + the server message, gap 8. 4. Network error → the shared detail error view with retry → refetch.

**Values:**

| Element | Value | Note |
| --- | --- | --- |
| Not-found icon / copy | `questionmark.circle.fill` 64 gray / body 16 t2, gap 8 | Android `Help` 72, gap 12 |
| Precedence | not-found and error are mutually exclusive branches; not-found wins | Android's are independent conditions |
| Guards / telemetry | early-return when both id and name/instructor are missing; re-entrancy guard on refetch; log under a **distinct** page name from the canonical detail screen | both log `CourseScoreCourseDetailView` |

**Strings:** `课程详情`; the not-found message is server data. Do not ship a hardcoded `暂未收录该课程`.

**States:** *loading* · *resolved* · *not found* · *network error* (retry). Render nothing when neither an id nor a name+instructor pair is supplied.

---

##### Divergences

- **Gate:** panel 350/r10/gray .1 vs 320/r12/gray .2; iOS embeds `**bold**` in one localized string per bullet, Android splits each bullet into two resources with a hardcoded `"- "`; lock glyph blue vs gray; Android also gates on `permitUploadScore`.
- **Home:** brand icon 48 vs 64; search label 28 vs 24; inner pad 16/10 vs 24/10; bg alpha 0.15 vs 0.2; history 4 rows + `prefix(30)` + r8 + pad 8/6 + t1@0.65 + 1 line vs 8 rows uncapped, r10, pad 4, t2, unlimited lines; the `其他服务` card is iOS-only.
- **Suggestions:** placeholder `输入关键词` vs `输入课程名或授课人`; iOS's clear button is conditional and resets results, Android's is always visible and clears only the text; iOS force-unwraps malformed `<strong>` (crash), Android drops the tail; Android copies the suggestion into the field.
- **Result list:** iOS always shows the sort chips, Android hides them behind `CourseDetailConfig.enableCourseScoreResultFilter`; iOS hides the stat column when `total <= 2` but always shows the median, Android does the opposite; bars 6/r3 with a zero-width render vs 4/r2 omitted; label width 45 vs 48; divider 130 vs 120; card pad 16 vs 0+12; iOS's comment tap always opens detail by id, Android forks on `enableCourseDetailEntry` and that legacy `Comment` route is mis-wired to the comment-thread screen.
- **Share:** iOS system sheet with a hardcoded `whu-ham.github.io` host and a hand-built `"name | instructor"` description; Android a QQ-only 0.4-height sheet reading the host from CCKV. The salt is duplicated verbatim in both codebases.
- **Detail:** rate average `%.1f` vs `%.2f`; star rows fixed at 5 vs `max(star)`; watermark 128 @ (24,24) vs 172 @ offsetY 24; function icons heart+paperplane 24 vs heart+pencil 28, leading vs centred; disabled-comment block frameless 64 with no fallback copy vs a titled `评论` card with a 72 icon and a default-reason fallback; comment-row star 10 tinted vs 16 untinted; self-review star 24 vs 32; iOS renders every card unconditionally except self-review, Android gates all five on payload fields; iOS's error view accepts a server message, Android's does not; Android has no semester re-entrancy guard.
- **Thread:** empty title vs `课程评论 ` with an empty substitution; Android has a 1 s-delayed next-page spinner, iOS none; bottom inset 32 vs 64.
- **Create review:** star 32 vs 56 (iOS pins the row to 224×32); plain text `发布` vs an r6 tinted chip with a 16 inline spinner; iOS strips newlines and sets `lineLimit 6`; iOS always shows the counter with 20/100 defaults while Android hides **and skips validation** when config is null; iOS freezes min/max/hint at init; Android's row-level drag ignores `enableRate`.
- **Course center:** card gap 0 vs 16; profile header bare and untappable vs a tinted card opening the user centre; empty copy on all three cards is Android-only; iOS tints only the rank footer and always inserts a preceding divider; rank self-fill divides by `maxProgress` vs the bucket's own `progress`; bar corner 8 vs 6; score 28 rounded vs 24 plain; the want item has a timestamp (and a trailing space in the name) only on iOS.
- **Sub-views:** iOS titles are static, Android appends counts to rank and want; iOS uses `ham_bg_b2` for the page bg; Android wraps want items in a card, iOS wraps none; no empty or error state on either platform.
- **Match variant:** not-found icon/size/gap 64/8 vs 72/12; branch precedence differs; both log the same page name as the canonical detail screen.
- **Systematic:** iOS uses `.system(..., design: .rounded)` for every big number, Android has no rounded face; `title2` is 22 regular vs 20 bold, `headline` is 17 semibold vs 14 normal; Android sets `userScrollEnabled = false` plus a custom `bouncy` modifier on every list and flips `ignoreTopSafeArea` once data arrives on five screens; format verbs differ throughout (`%lld` vs `%1$d`, `%.1f` vs `%.2f`); Android's `CourseCommentItemView.kt:49` tests `avatar_url?.isEmpty() == false`, so a null avatar URL skips the fallback branch; Android's `CourseCommentMainView` is unreferenced mock-only dead code.


---

### My and user center

#### My tab (我的)
**Purpose:** Account-and-module hub for the third tab. **Entry:** third tab of the root tab bar. **Platforms:** both.

    ▓ remote bg image h 250, scaledToFill, offset y = −scrollY, stretches on overscroll ▓
    ┌ collapsed bar (scroll > 50) · h = statusBar + 56 · bg_b1 @0.85 · fade 0.3 s ┐
    │ [16] remote title 20 Bold │ content top gap 112 + statusBar, bottom 80 + safe │
    │ ┌ 1 User-center card r16 pad16 ┐ avatar 48⌀ · 12 · 点击登录/昵称 · divider vPad 12 │
    │ │ (40⌀ icon) 登录信息门户 / 管理信息门户设置 · 使用校内服务的前提 › │ CAS row │
    │ ┌ 2 Grid ┐ h-scroll h=2×64=128, pad 16/8, gap 8, tile 48h r12 │ ┌ 3 Board ┐ ┌ 4 Promotion ┐ ┌ 5 Links ┐ │
    column maxW 400 centred, block spacing 10, card pad/radius 16, press alpha 0.25

**Blocks:** background image, no tap · collapsed nav bar (scroll > 50) · user-center card (tap top →
login or user center; tap CAS row → CAS settings) · module grid · board card · promotion card (**must be dismissible — add a close affordance and a persisted flag**) · link card.
**8 module tiles** (icon SF / Material · brand light/dark · target): 图书馆 books.vertical / Book
#007AFF / #0A84FF → library/home · 运动 sportscourt / SportsVolleyball #34C759 / #30D158 → sport/main ·
成绩 doc.plaintext / Article #FF9500 / #FF9F0A → score · 给分 chart.bar.xaxis / InsertChart #283593 → course-score · E卡 creditcard / CreditCard #BF360C → pay sheet (no navigation) · 校巴 bus.fill / DirectionsBus #A2845E / #AC8E68 → bus · 课程表 tablecells / TableChart #1B5E20 → course/setting · 日程 calendar / CalendarMonth #01579B → schedule/home.
**6 link rows** (icon · guard · target): 自动化 / 添加Siri捷径 `alarm`, always → automatic ·
使用指南 / 查看使用文档 `book.fill` / MenuBook, `guide.visible` (default true) → `https://docs.ham.nowcent.cn/` · 反馈 / 加入 Discord 社区 `paperplane.fill` / Feedback, `feedback.visible` + valid url → `https://discord.gg/GwwGksTDVE` · 设置 / 自动化、小组件、关于等相关设置 `gear` / Settings, always → `my-view/setting` · 关于 / 版本信息与隐私协议 `hexagon`, always → `about` · 调试入口 / Debug settings `wrench.and.screwdriver` / BugReport, debug builds → `my-view/debug`.

| Element | Value | Note |
| --- | --- | --- |
| Collapse threshold / bar / fill | scroll > 50 / `statusBar + 56` / `bg_b1 @0.85`, no blur or border | iOS currently 7; iOS gradient-masks `bg_b2` |
| Content top / bottom / column / block spacing | `112 + statusBar` / `80 + bottom safe` / maxW 400 centred / 10 | iOS 100+statusBar / 100 / — / 15 |
| Card pad / radius / inner pad / base / divider | 16 / 16 / 16 / `bg_b1`, card `bg_b2` / 1dp `#EDEEEF` / `#0F0E0F`, v-pad 4 | iOS board+link 10 / 8; user-center-card divider v-pad 12 |
| Avatar / gap / headline / sub-line | 48⌀ circle, `gray@0.15` + smiley 28, fade 0.5 s / 12 / 16 Bold / 12 secondary | iOS 17 |
| CAS row icon / title | 24 glyph in a 40⌀ circle, `blue @0.15` / 16 Bold | iOS 32⌀/16pt; Android 48⌀/36dp, title 14 |
| Chevron / grid | 16, matching row text colour / h-scroll, 2 fixed rows, height 128, pad 16/8, gap 8/8 | iOS chevron 12 (CAS) / 17; Android 24; iOS grid is a single HStack, rows scroll side-by-side |
| Tile / icon / title / subtitle | 48 h, r12, brand @0.15 over `bg_b1`, h-pad 12 / 24, gap 8 / 16 Bold / 12 | iOS 52h, h-pad 16, icon 17, title 17 |
| Promotion icon / title / subtitle / link icon / overscroll | 40⌀ circle / 16 Semibold / 12 secondary / 24 glyph in a 40⌀ `blue@0.10` circle, icon→text 8, row pad 8 / 2.5 | Android promotion icon 30⌀; iOS title 17; iOS link icon 20pt, Android 25 (settings 30); overscroll Android only |

**Strings:** 我的 · 点击登录 · 不登录也可以使用校内功能哦 · 登录信息门户 · 管理信息门户设置 ·
使用校内服务的前提 · 未登录 · 自动化 · 添加Siri捷径 · 使用指南 · 查看使用文档 · 反馈 · 加入 Discord 社区 · 设置 · 自动化、小组件、关于等相关设置 · 关于 · 版本信息与隐私协议 · 调试入口 / Debug settings (debug-only, hardcoded both) · tile labels 图书馆 / 运动 / 成绩 / 给分 / E卡 / 校巴 / 课程表 / 日程. Bar title, tile titles/subtitles, board, promotion and guide/feedback copy are remote-overridable per locale. **States:** logged-out → placeholder avatar + 点击登录 / 不登录也可以使用校内功能哦 in secondary, tap opens
the login overlay · logged-in → remote avatar + nickname 16 Bold (2-line max), tap → user center · CAS unbound → 登录信息门户 with title/subtitle/chevron in blue · CAS bound → 管理信息门户设置 in primary (all four combinations reachable) · loading → avatar only · error → none · empty config → board and promotion hidden, grid collapses to zero height, user-center card always renders.

#### Settings hub (设置)
**Purpose:** Second-level index of settings destinations. **Entry:** My tab link row 设置. **Platforms:** Android (**build an iOS equivalent** — today iOS has no pushed hub; its entries are rows
in the My-tab link card).

    ◀ 设置 · header 36, bg_b1 @0.75 · title 16 Bold max 300 · back chevron ham_blue m16
    ┌ card A r16 bg_b2 pad16 ┐ 小组件 ›  (16 gap, no divider)  自动化操作 ›  ┐
    ┌ card B ┐ 语言  简体中文 ›  right value 16 primary  ┌ card C ┐ 关于Ham › · pad 16, spacedBy 16
    chevron ChevronRight tint ham_gray · pressed alpha 0.25 · scroll enabled

**Blocks:** card A — 小组件 → widget, 自动化操作 → automation · card B — 语言 with the current language as
the right value → language settings · card C — 关于Ham → About · rows: label `weight(1f)` one line ellipsis, optional right value, trailing chevron.

| Element | Value | Note |
| --- | --- | --- |
| Header | 36, title 16 Bold `text_primary`, max 300 | |
| List padding / spacing / card | 16 / 16 / r16 `bg_b2` pad 16, no elevation | |
| Row title / right value | 16 `text_primary` | right value is primary, not secondary |
| Chevron / pressed alpha / scroll | `ChevronRight` tint `ham_gray` / 0.25 / enabled | Android sets `userScrollEnabled = false`, so card C is unreachable on short viewports |

**Strings:** 设置 · 小组件 · 自动化操作 · 语言 · 关于Ham · 简体中文 · English · 日本語 · 跟随系统. **States:** normal only. No loading, error, or empty state.

#### About (关于Ham)
**Purpose:** App identity, version and channel, privacy copy with an inline link, outbound links, and a
developer panel. **Entry:** Settings hub row 3 (Android) / My-tab link card row 5 (iOS). **Platforms:** both.

    header 36 bg_b1 · [◀16] 关于Ham 16 Bold · scrolls · spacer 32, column gap 8 · logo 96 r12, shadow 16 ham_gray (×10 tap target)
       Ham 20 Bold · 1.2.3 (456) 12 secondary
    ┌ card r16 bg_b2 pad16 ┐ privacy blurb 12 secondary, inline link ham_blue · card inset 16
    ┌ card 2, rows spacedBy 12, 1dp ham_lightGray dividers, cards 32 apart ─────────┐
    │ 版本渠道  [正式版 ▾] pill gray@15% r8 pad v6/h8 · 已是最新版本 ⟨◎20⟩ › │
    │ 前往主页 / 前往App Store › · 来Github找我 › · ≥10 taps: 复制课程回包日志 › · 分享日志 › · token 12 │

**Blocks:** logo — 96 square, r12, shadow 16 `ham_gray`; tap counter, at **10 taps** reveal the developer
rows and the token row, no unlock feedback · name + version · privacy card — remote CCKV `PrivacyText` keyed by locale, falling back to the bundled `privacy_help`; inline `[name](href)` spans `ham_blue` and tappable; when the link name is empty use 链接 · card 2 rows: 版本渠道 pill (dropdown sets and persists the channel then re-runs the update check; selected item shows a `Check`, alpha 1/0, tint `ham_blue`) · update row (检查新版本 / 已是最新版本 / 发现新版本 %s; right slot spinner + chevron, gap 8; tap re-runs the check only) · 前往主页 (`docHost`, appending `/ja` or `/en` for those locales) and 前往App Store (`itms-apps://apple.com/app/id1577896044`) — **ship both rows on both platforms** · 来Github找我 (CCKV `DocGithubUrl`) · 复制课程回包日志 and 分享日志 behind the easter egg · TPNS token behind the easter egg, tap copies + toast.

| Element | Value | Note |
| --- | --- | --- |
| Header / top spacer / column gap | 36, title 16 Bold max 300, back chevron `ham_blue` m16 / 32 / 8 | iOS uses the 44 inline bar with title 关于; Android spacer 64, iOS 16 inside a card |
| Logo | 96 square, r12, shadow 16 `ham_gray` | iOS 120×120 r20 with CoreMotion parallax (image `roll*15/pitch*15`, tile `roll*20/pitch*20`) |
| Easter-egg / name / version | 10 taps / 20 Bold / 12 `text_secondary` | iOS exposes the developer rows and token unconditionally; iOS body 17 / `.caption` 12 |
| Card / inset / gap / row spacing / divider / label | r16 `bg_b2` pad16 / 16 / 32 / 12 / 1dp `ham_lightGray` / 16 `text_primary` | iOS card gap 12 + 16 |
| Channel pill / dropdown | `gray @0.15` r8 pad v6/h8, 16 / item 16 `weight(1f)` one line, container `bg_b2` | container colour unset → Material3 default |
| Spinner | 20, stroke 2, `ham_gray`; gap to chevron 8 | |
| Privacy body / link / toast / pressed alpha | 12 `text_secondary` / `ham_blue` / `ham_green` r12 pad 16 gap 8 icon 32 top gravity 2000 ms / 0.25 | iOS renders native SwiftUI markdown; Android regex-parses `\[.*?]\(.*?\)` |

**Strings:** 关于Ham · 版本渠道 · 正式版 · 预览版 · 已是最新版本 · 发现新版本 %s · 检查新版本 · 前往主页 ·
前往App Store · 来Github找我 · 复制课程回包日志 · 分享日志 · 链接 · 已复制 · 课程回包日志已复制 · token已复制 · `privacy_help` (**with** the markdown link on both platforms). **States:** update idle / checking (spinner, row tap disabled) / up to date / update available / dev
panel locked vs unlocked. No error UI — the update row silently keeps its previous value. > Div: iOS has no version-channel or update rows and no easter egg; Android has no App Store row. > Android's `HamTheme` is not applied on this path, so dropdowns fall back to Material3.

#### Version update sheet (发现新版本)
**Purpose:** Present a newer build with its changelog and download paths. **Entry:** raised when
`newVersionCode > App.VERSION_CODE` (app foreground, throttled 8 h; unthrottled from About). **Platforms:** Android (**add on iOS**).

    ░ scrim black @0.5 × progress, tap → hide ░ · 85 % height, r24 top, bg_b1
      ┌ drag strip 32 — DRAW A GRABBER ┐ over-drag 100; dismiss at offset > height/4
        ⟳ Update 72 ham_blue · 发现新版本 24 Bold · pad top 32, bottom nav-bar height
        当前版本过低，请及时更新 16 Bold ham_red (conditional on VERSION_CODE < minVersionCode)
      ┌ pad 16 ──────────────────────────────┐
      │ 1.2.4 20Bold  2026-01-31 12sec  gap 8, bottom-aligned │ <title> 16Bold · <content markdown> 16 · (48) │

**Blocks:** scrim (tap dismisses) · drag strip with the over-drag and fling physics · icon 72 `ham_blue`
· title 24 Bold · low-version warning (advisory — the sheet stays dismissible) · body — version + date (date only when `updateTime > 0`), changelog title, changelog content as **markdown** and **scrollable** · buttons `spacedBy 8`: 从PlayStore下载 (conditional on the Play flag, URL pinned to `com.android.vending`), 从浏览器下载 (conditional on a non-empty `updateUrl`), 关闭 (always).

| Element | Value | Note |
| --- | --- | --- |
| Sheet / extend / corners / content padding | 85 % / 100 / r24 top-start + top-end / top 32, bottom = nav-bar height | |
| Icon / title / warning | 72 `ham_blue` / 24 Bold / 16 Bold `ham_red` | |
| Version / date / changelog | 20 Bold / 12 secondary / 16 Bold + 16 | body is plain `Text` today — must use markdown |
| Pill button / 关闭 / pressed alpha | r12, `ham_blue @0.15`, pad v16 ≈54, label 16 Bold `ham_blue`; 关闭 gets the same pill / 0.25 | today 关闭 is bare text with no background or clip |

**Strings:** 发现新版本 · 当前版本过低，请及时更新 · 从PlayStore下载 · 从浏览器下载 · 关闭. **States:** normal · low-version · no Play listing · no update URL (if both absent only 关闭 remains) ·
missing changelog fields (each suppressed individually) · null response (render an error fallback, not a blank sheet) · long changelog (must scroll).

#### Widget settings (小组件设置)
**Purpose:** Preview the two home-screen widgets and set the library widget's refresh interval.
**Entry:** Settings hub → 小组件. **Platforms:** Android (iOS leaves refresh scheduling to WidgetKit).

    toolbar 42, bg_b1 @0.95 · [◀ 16] 小组件 16 Bold max 300 · pad 16, spacedBy 32
    ┌① TIP CARD r16 pad16 ┐ 提示 16 Bold · 8 · Ham提供了课程与图书馆小组件… 12
    │ 要正常使用小组件功能，您需要开启本应用"自启动"… 12 → jump to settings │
    │ [Widgets watermark 144 ham_gray@0.25, +16/+16 BottomEnd] │
    32 · 课程表小组件 16 Bold · 8 · ② COURSE PREVIEW h 130: 星期X 16B ham_red · 第1周 12 · 下一节 16B + 4 + 1 row
      │1dp divider│ right: 2 more rows + right-aligned N节被隐藏 12 · no tap targets
    32 · 图书馆小组件 16 Bold · 8 · ③ LIBRARY 200×200: 已预约 16B ham_blue · 118 28B · 📍/📅 16 ham_gray gap 4
      · location 12 maxLines2 · time 12 · [取消预约] h48 r12 ham_red@0.15 (INERT MOCK)
      16 → 更新间隔 16 · 8 · chip 96 r12 ham_gray@0.15 pad v8 → Popup y+24 + HamPicker 80

**Blocks:** toolbar · tip card with the watermark (**make the 自启动 advice a jump-to-settings action**)
· course section — heading then a 130-tall illustrative preview (left: 星期X, 第1周, 下一节 + one course row; 1dp `ham_lightGray` divider; right: two more course rows plus right-aligned `N节被隐藏`) · library section — the 200×200 mock beside, 16 to its right, the interval column · interval chip → `Popup(IntOffset(0, 24))` with a 96-wide r12 `bg_b1` surface holding `HamPicker`; **selecting an option must close the popup**. Options (seconds → label): 60 1分钟 · 120 2分钟 · 300 5分钟 · 600 10分钟 · 1200 20分钟 · 1800 30分钟 · 2700 45分钟 · 3600 1小时 · 7200 2小时. Out-of-range → 20分钟. Persist and reschedule the alarm.

| Element | Value | Note |
| --- | --- | --- |
| Content padding / spacing / card / watermark | 16 / 32 / r16 `bg_b2` pad 16, clipped / 144 `ham_gray@0.25`, +16/+16 `BottomEnd` | |
| Card title→body / heading→preview / tip body | 8 / 8 / 12 `text_primary` | |
| Preview / columns / weekday / week / 下一节 | 130 tall, two `weight(1f)`, extra rows = `(130−16−20)/40` = 2 capped 5 / 16 Bold `ham_red` / 12 / 16 Bold, 下一节→course 4 | |
| Divider / course row | 1dp `ham_lightGray` / colour bar 6×32 r3 `blue@0.15`, gap 6, name 12 B 1 line, period+location 12 1 line, row gap 4 | |
| Library preview | 200×200, inner `spacedBy 4`; 已预约 16 Bold `ham_blue`; seat 28 Bold; icons 16 `ham_gray` gap 4; location 12 2 lines; time 12 | |
| 取消预约 (mock) | h48, r12, `ham_red @0.15`, 16 `ham_red`, inert | |
| Chip / popup / picker | 96 r12 `ham_gray@0.15` pad v8 / popup `bg_b1`, offset y+24, fade+expand / wheel 80 (`halfNumbersColumnHeight` 40), band 32 r8 `ham_gray@0.15`, item pad v8/h20, min alpha 0.3, `exponentialDecay(friction 20)` | |
| Default interval / pressed alpha | 1200 s (20分钟) / 0.25 | |

**Strings:** 小组件 · 提示 · Ham提供了课程与图书馆小组件，您可在桌面进行添加。 ·
要正常使用小组件功能，您需要开启本应用"自启动"，否则小组件可能会出现无法正常更新。 · 课程表小组件 · 图书馆小组件 · 更新间隔 · 未设置 · %d分钟 · %d小时 · 星期%s · 第%d周 · 下一节 · %d节被隐藏 · %d节 / %d-%d节 · 已预约 · 取消预约. **States:** normal, picker open/closed. No loading, error, or empty state; no add-to-home flow exists — the screen previews only. > Div: `feature/my` ships no `values-en` / `values-ja` for these keys; the seat number `118` is hardcoded.

#### Automation / Siri shortcuts (自动化操作)
**Purpose:** Configure unattended automation. **Entry:** Settings hub → 自动化操作; My tab row 自动化. **Platforms:** both — the two platforms currently ship **different features under the same name**.

    header 36, status-bar box bg_b1@0.75 · [◀ 16] 自动化操作 16 Bold · pad 16, spacedBy 16
    ┌① TIP CARD r16 pad 12 ┐ 提示 16 Bold ham_blue · 8 · 通过选定时间，实现自动预约图书馆或完成每日计划。12 · (+8) 要正常使用自动化功能，您需要开启本应用"自启动"… → settings · [Alarm watermark 128 ham_gray@0.25 +16/+16 BottomEnd]
    ┌② LIBRARY CARD r16 pad16 animateContentSize, inner spacedBy 16 ───────┐
    │ 图书馆 16 Bold                                                        │
    │ 自动快速预约 16  1f                                        [HamSwitch] │
    │ ══ AnimatedVisibility(enabled), slide+fade ══                         │
    │ 时间 16  1f                                         ┌ 08:00 ┐ chip   │
    └─────────────────────────────────────────────────────────────┘
    time picker: Material3 AlertDialog + TimePicker, 关闭 / 选择 · exact-alarm sheet (API 31+, app level): 85 % r24, ErrorOutline 72 ham_red, title 24 Bold, body pad 16, 48 to buttons, 去打开 (r12 blue@15% pad v16) + 好 (bare blue)

**Blocks:** header (content scrolls with a bounce, min height `screenHeight + 2`) · tip card, pad 12 ·
library card: section title 图书馆; enable row 自动快速预约 + `HamSwitch` (tap writes the flag and arms or cancels the alarm, **no confirmation dialog**); time row revealed by `AnimatedVisibility(enabled)` with slide + fade · time chip → time-picker dialog seeded from `Calendar.HOUR_OF_DAY` (**not** `Calendar.HOUR`, or afternoon times do not round-trip) · a **single** scheduled task (enabled + exact alarm at the stored time-of-day → smart book of today's or tomorrow's preferred seat, with running / ignored / success / failure notifications on channel 图书馆预约提醒); re-arming must preserve the chosen time-of-day · permissions — exact-alarm gating is app-level only; **add a denied state on the enable row**; `POST_NOTIFICATIONS` and OEM autostart have no runtime path today.

| Element | Value | Note |
| --- | --- | --- |
| Tip card / 提示 heading / body | r16, `bg_b2`, pad **12**, watermark 128 `ham_gray@0.25` / 16 Bold `ham_blue`, heading→body 8 / 12 `text_primary`, lines 8 apart | gap between the two lines is 0 today |
| Library card | r16, pad 16, `animateContentSize()`, inner `spacedBy 16` | |
| Switch / time chip | checked `ham_green` #34C759, unchecked `ham_lightGray`, Material3 dimensions / `ham_gray @0.15`, r8, pad v6/h8, 16, label `HH:mm` (`SimpleDateFormat(..., Locale.CHINA)`) | the chip uses raw `Color.Gray` today |
| Permission sheet | 85 % + 100, r24, scrim `black@0.5 × progress`, drag strip 32, pad top 32 / bottom nav-bar, icon 72 `ham_red`, title 24 Bold, body pad 16, → buttons 48, `spacedBy 8` | |
| Alarm | `setExactAndAllowWhileIdle(RTC_WAKEUP, …)`, requestCode `1000001`, `FLAG_UPDATE_CURRENT or FLAG_IMMUTABLE` | failures swallowed — surface them |
| Defaults / pressed alpha | enabled false, time null → 未设置 / 0.25 | |

**Strings:** 自动化操作 · 提示 · 通过选定时间，实现自动预约图书馆或完成每日计划。 ·
要正常使用自动化功能，您需要开启本应用"自启动"。执行自动化前后，Ham会为您发送提醒。如果您想接收提醒，请开启本应用的通知。 · 图书馆 · 自动快速预约 · 时间 · 未设置 · 选择 · 关闭 · 无法获取设定精确闹钟权限 · 在Android14及以上，设定精确闹钟的权限被默认关闭。… · 去打开 · 好 · notifications 图书馆自动化正在执行 / 图书馆快速预约 / 已忽略今日自动化预约 / 未登录图书馆 / 暂未找到今日与明日的首选座位预约 / 图书馆自动化执行失败 / 图书馆自动化执行成功 / 已预约%1$s. **States:** enabled (switch `ham_green`, time row visible, alarm armed) / disabled (default; switch
`ham_lightGray`, time row hidden, alarm cancelled) / time unset (chip 未设置 and arming does nothing —
**show a warning**) / permission granted or denied (API 31+). No loading or error surface. > Div: iOS `automatic` → `MySiriView` is a 31-line screen with one caption plus one `SiriButtonView` at > `frame(height: 60)` for a single shortcut (`.libraryQuickBookIntent`) — name parity, no feature parity. > Adopt the scheduled-task model on iOS, keeping the Siri donation button as an extra row.

#### Language settings (语言)
**Purpose:** Switch app language. **Entry:** Settings hub → 语言. **Platforms:** Android (iOS follows the
system locale and ships zh-Hans / en / ja catalogs — no screen needed).

    header 36, bg_b1 @0.75 · [◀ 16] 语言 16 Bold · LazyColumn pad 16, spacedBy 16
    ┌① PER-APP CARD (API 33+ only) r16 pad16 ─────────────────────────┐
    │ 修改 Ham 的语言 16 Bold · 8 · 从 Android 13 起… 12 secondary · 前往系统设置 16 ham_blue · 16 · 使用应用内语言设置 16 ham_blue │
    └──────────────────────────────────────────────────────────┘
    ┌② LANGUAGE LIST (when showDefaultSetting) r16 pad16, spacedBy 16 ─┐
    │ 跟随系统 16 1f [✓] · 简体中文 16 1f [✓] · English 16 1f [✓] · 日本語 16 1f [✓] │

**Blocks:** header · per-app card (API ≥ 33): 前往系统设置 → explicit intent `com.android.settings` /
`.localepicker.AppLocalePickerActivity`, data `package:com.nowcent.ham`, `FLAG_ACTIVITY_NEW_TASK` (**surface a fallback on failure** — today `runCatching` swallows it); 使用应用内语言设置 → reveals ② · language list — four rows in fixed order, label `weight(1f)` one line ellipsis, trailing `Icons.Rounded.Check` 24 tint `ham_blue` at alpha 1/0 (the icon always occupies layout space) · apply via `AppCompatDelegate.setApplicationLocales(LocaleListCompat.forLanguageTags(tag))` (empty list for `""`) plus persistence; resolution order app locales → stored value → `""`.

| Element | Value | Note |
| --- | --- | --- |
| Options | 跟随系统 → `""` · 简体中文 → `zh` · English → `en` · 日本語 → `ja`; names `translatable="false"` | |
| List | pad 16, `spacedBy 16` | `userScrollEnabled = false` today — enable it |
| Card / title / subtitle | r16 `bg_b2` pad16 / 16 Bold / 12 `text_secondary`; title→content 8 | |
| Action rows / language row / check / pressed alpha | 16 `ham_blue`, no background, `spacedBy 16` / 16 `text_primary` `weight(1f)` one line / `Check` 24 tint `ham_blue`, alpha 1 or 0 / 0.25 | |

**Strings:** 语言 · 修改 Ham 的语言 · 从 Android 13 起，支持为每个应用单独设置语言。请前往系统设置，为 Ham
选择使用的语言。 · 前往系统设置 · 使用应用内语言设置 · 跟随系统 · 简体中文 · English · 日本語. **States:** API ≥33 default (① only) / API ≥33 after tapping 使用应用内语言设置 (① + ②) / API ≤32 (②
only). Unsupported system locale falls through to 跟随系统. > Div: on API 33+ the OS store and the in-app stored value can disagree; the selected row is computed > once per composition rather than observed as state.

#### User center main (用户中心 / 个人中心)
**Purpose:** Hub for the authenticated account — identity header, grouped entry rows, destructive logout.
**Entry:** My-tab user-center card when logged in. **Platforms:** both.

    toolbar 42 + status bar, bg_b1 @0.95 · [◀ 24 ham_blue m16] 个人中心 16 Bold max 300
    Column pad v32/h16, spacedBy 32 · bottom spacer = nav-bar inset + 80
      ┌ avatar 108⌀ circle, Crop + crossfade 100 ms, placeholder Person on #EDEEEF tint #888888 pad 12 ┐
       nickname 22 Bold, 2-line max · header block spacedBy 8 · tap → edit-info · 32
    ┌ HamCardView r16 bg_b2, inner v2/h16, spacedBy 4 ─────────────────┐
    │ [■28] 个人信息 › · 登录设备 › · 社交账号 › · Passkey管理 › · 授权应用 › · 扫码登录 › │ row pad v4, gap 12, ≈36 · divider 1dp between rows only · label 16 · chevron 24 ham_gray · ≈245
    32
    ┌ card r16, pad v8 ┐ 退出登录 16 ham_red, centred, ≈40 ┐ → CONFIRM DIALOG first

**Blocks:** header — avatar 108⌀ plus nickname 22 Bold; avatar falls back to the placeholder when
`avatar_url` is empty; tap opens edit-info · grouped nav card — six rows, always rendered, divider
**between rows only**; taps: 个人信息 → edit-info, 登录设备 → devices, 社交账号 → social, Passkey管理 →
passkey, 授权应用 → authorized apps, 扫码登录 → scan · logout — full-width red card; tap **opens a confirmation dialog** (title 退出登录, 确定 / 取消), then logs out.

| Element | Value | Note |
| --- | --- | --- |
| Toolbar | 42 + status bar, `bg_b1 @0.95`, title 16 Bold one line max 300 | iOS uses the ~44 system bar and sets **no** title |
| Back chevron / content padding / spacing | 24 tint `ham_blue`, start margin 16 / v32 / h16, `spacedBy 32`, header `spacedBy 8` | iOS spacers 36/8/32/16/32 with `.padding(16)` |
| Avatar / nickname | 108⌀ circle / 22 Bold `text_primary` 2-line max | iOS 120×120; iOS `.title2` 22 regular, Android forces Normal |
| Card | r16 `bg_b2` no elevation, inner pad v2/h16, `spacedBy 4` | iOS `HamCardView(padding: 4)` + inner 8 |
| Row / icon chip / label / chevron / divider | pad v4, `spacedBy 12`, ≈36 / 28×28 r6 bg `#888888`, glyph 24 white / 16 `text_primary` / 24 `ham_gray` / 1dp `ham_lightGray` between rows only | iOS icon 32×32 r8 gray, glyph 20; iOS label 17, chevron 16 |
| Logout label | 16 `ham_red`, pad v8, ≈40 | |
| Pressed alpha / image spinner | 0.25 / 20, stroke 2, `#888888` | |

**Strings:** 个人中心 · 个人信息 · 登录设备 · 社交账号 · Passkey管理 · 授权应用 · 扫码登录 · 退出登录. **States:** logged-in — avatar from `avatar_url`, nickname populated, all rows enabled · logged-out —
**redirect to login** (Android wrongly keeps every row live, including 退出登录) · loading — avatar
spinner only · error — avatar falls back to the placeholder silently. > Div: iOS hides the card behind a remote flag by default, orders rows 个人信息 / 登录设备 / 社交账号 / > Passkey管理 / 扫码登录 / 授权应用, and adds a 更多 row that pushes the logout/deactivate screen.

#### Personal info (个人信息)
**Purpose:** Edit the profile — replace the avatar, edit the nickname, save. **Entry:** user-center row
个人信息; tapping the avatar header. **Platforms:** both.

    toolbar 42 · [◀] 个人信息 16 Bold · 保存 14 accent, end margin 8 · pad v32, spacedBy 32
      ┌ avatar 108⌀ circle ┐ ┌──┴──┐ badge ≈36⌀ bg #888888, glyph 24 white, pad 6, BottomEnd · 32
    ══ 1dp divider, FULL-BLEED ══ · HamTextField bg_b2, pad 8, ≈40, r8 + 1dp ham_lightGray border, body 16, cursor ham_blue
    ══ 1dp divider, FULL-BLEED ══     bottom spacer = nav-bar inset + 80

**Blocks:** avatar editor — the whole avatar is the tap target; opens the system photo picker (images
only); **show a local preview immediately** and again after the server round-trip; validate ≤ 4 MB and extension in png / jpg / jpeg — failures must toast; upload in 2 MB chunks behind a modal spinner · nickname field — the only editable field, hint 昵称, single line, **max 20 characters enforced by an input filter *and* at save, with a visible counter** · save — trailing toolbar text button 14 accent; while in flight disable it and show a spinner; on success toast 保存成功 and **pop** · no read-only identity rows.

| Element | Value | Note |
| --- | --- | --- |
| Toolbar / save / padding / spacing | 42, title 16 Bold, `bg_b1 @0.95` / 14 `ham_blue`, end margin 8 / v32, h0, `spacedBy 32` | iOS uses the automatic (large) display mode; iOS `.padding(.vertical, 16)` + `VStack(spacing: 32)` |
| Avatar / placeholder | 108⌀ circle, Crop, crossfade / `Person` 24 tint `#888888` on `#EDEEEF`, pad 12 | iOS 120×120; iOS `face.smiling` 64 on `gray@0.15` |
| Edit badge | ≈36⌀ circle bg `#888888`, glyph 24 white, pad 6, `BottomEnd` | iOS `pencil` 36×36, pad 8 |
| Field fill / padding / radius / border | `bg_b2` / 8 / 8 / 1dp `ham_lightGray` | Android's `boxModifier` drops both |
| Field text / hint | 16 `text_primary` / 16 `#888888`, `offset(y = -2)` | iOS 17 |
| Nickname / avatar limits | max 20 chars, filter + counter / ≤ 4 MB, png/jpg/jpeg, 2 MB chunks, JPEG q0.5, filename `UUID().jpg` | Android validates the nickname at save only, no counter |
| Pressed alpha | 0.25 | |

**Strings:** 个人信息 · 保存 · 昵称 · 用户名输入有误 · 用户名不能大于20个字符 · 图片大小不能大于4MB ·
保存成功 · 遇到了错误. **States:** loaded · logged-out (**redirect to login**) · loading (avatar modal spinner; save button disabled with a spinner) · error (inline field error for blank or >20 chars; toast on RPC failure) · success (toast 保存成功, then pop). > Div: Android has no loading, disabled, or pop-on-success behaviour; iOS never pops and gives no > feedback during upload.

#### Login devices (登录设备)
**Purpose:** List every session for the account, mark the current device, revoke the others.
**Entry:** user-center row 登录设备. **Platforms:** both.

    toolbar 42 + status bar, bg_b1 @0.95 · [◀ 16] 登录设备 16 Bold · 刷新 · pad 16
    [Loading] centred 20 spinner #888888 · AnimatedContent fadeIn togetherWith fadeOut
    ┌ card r16 bg_b2 pad16, spacedBy 8 ──────────────────────────────┐
    │ [ic 24] 设备名 16 Bold 1f              [下线 chip | 本机 badge] │ ≈56, no vertical padding
    │ device_id (AND/IOS stripped) 12 · 最近上线：yyyy-MM-dd HH:mm 12 │ chip 12 accent r6 bg accent@15%, pad 4
    ├ 1dp divider, between rows only ┤   [Empty] empty view · [Error] error view + 重试

**Blocks:** loading — centred 20/2 spinner · device card — one card, one row per device, `spacedBy 8`,
divider after every row except the last; only the trailing control is tappable · row icon 24, platform inferred from the device id (`IOS` → PhoneIphone, `AND` → PhoneAndroid, else Smartphone); no IP address is shown or required · row text — device name 16 Bold, device id with the prefix stripped 12, last-active time `yyyy-MM-dd HH:mm` 12 · current-device marker `本机` — 12 accent-tinted badge, r6, pad 4 · revoke chip `下线` — tap **opens a confirm dialog**, then revoke; while in flight replace the chip with a 20 spinner; on success toast 设备已删除 and remove the row · error view + 重试 · empty-state view.

| Element | Value | Note |
| --- | --- | --- |
| Spinner / card | 20, stroke 2, `#888888` / outer pad 16, inner pad 16, r16, `bg_b2`, rows `spacedBy 8`, no elevation | iOS `HamCardView(padding: 8)` |
| Divider / row | 1dp `ham_lightGray` between rows only / `spacedBy 8`, centred vertically, no vertical padding, ≈56 | iOS row ≈70 (icon 32×32 r4, glyph 24) |
| Device name / id / time | 16 Bold / 12 / 12 | iOS 17/12/12, label 最近上线：%@ |
| 下线 chip / 本机 badge | 12 `ham_blue`, r6, bg `ham_blue@15%`, pad 4 / 12 accent, r6, pad 4 | iOS chip 12 on `ham_btn_bColor` #EFF2F2/#444444 r8; Android's 本机 is unstyled plain text |
| Refresh / pressed alpha | pull-to-refresh + toolbar 刷新 / 0.25 | Android fetches once in `init`, no refresh or retry |

**Strings:** 登录设备 · 下线 · 本机 · 设备已删除 · 暂无数据 · 重试. **States:** loading · loaded · empty ·
error (+ 重试) · per-row revoking (chip → spinner) · per-row revoke error (chip restored, toast, row retained) · logged-out (redirect to login). > Div: Android has no empty state and no kick confirmation — `else -> {}` renders a blank page; iOS falls > through to the card branch on error, which is visually identical to "zero devices".

#### Social accounts (社交账号)
**Purpose:** Show which OAuth providers are linked and link more. **Entry:** user-center row 社交账号. **Platforms:** both.

    toolbar 42 + status bar, bg_b1 @0.95 · [◀ 16] 社交账号 16 Bold · pad 16
    [Loading] centred 20 spinner · ┌ card r16 bg_b2, outer pad 16, inner v2/h16, spacedBy 4 ────────┐
    │[ic 26] QQ                                   已登录 / › / ◌20  │ row pad v4, gap 12, ≈34
    ├ 1dp divider, between rows only, never after the last ┤ │[ic 26] 微信 · Github · Apple · 自强 … per ValidLoginType, SERVER ORDER │

**Blocks:** state switch `AnimatedContent` `fadeIn() togetherWith fadeOut()` · provider card — one row per
server-supplied `ValidLoginType` entry in **server order** (QQ / 微信 / Github / Apple / 自强); unmapped labels are skipped with their divider · row — icon 26 box r6 inner pad 2 → 12 → name 16 → spacer → trailing: bound → 已登录 12 secondary; unbound idle → `ChevronRight` 24 `ham_gray`; unbound in flight → 20 spinner · taps: unbound starts the bind flow (QQ native SDK; WeChat / GitHub / 自强 via the login sheet or an external browser then a deep link; Apple → toast 设备暂不支持该方式登录, no bind path on Android);
**bound rows must offer unbind — today a no-op on both** · error / unload → error view + 重试 (today both
render a blank page).

| Element | Value | Note |
| --- | --- | --- |
| Card | r16, `bg_b2`, outer pad 16, inner v2 / h16, `spacedBy 4` | iOS `HamCardView(padding: 8)` |
| Row / icon box | pad v4, `spacedBy 12`, ≈34 / 26, r6, inner pad 2 | iOS gap 16, row 28, icon 28×28 r8 glyph 24 |
| Icon fills | QQ #12B7F5 · 微信 #58BE6A · Github `text_primary` black/white · Apple `text_primary` · 自强 `#01579B @10%` bg with `#01579B` glyph | 自强 uses pad 4 on Android — normalise to 2; QQ/微信 glyphs white |
| Name / 已登录 | 16 `text_primary` / 12 `text_secondary` #888888 | iOS 17 / 17 gray ≈#808080 |
| Spinner / chevron / divider / pressed alpha | 20, stroke 2, `ham_gray` / 24 `ham_gray` / 1dp `ham_lightGray` / 0.25 | iOS chevron 16 |

**Strings:** 社交账号 · QQ · 微信 · Github · Apple · 自强 · 已登录 · 设备暂不支持该方式登录 · 登录成功 ·
Github登录 · 微信登录 · 取消. **States:** loading · loaded · error / unload (error + 重试) · empty list (empty card) · row bound / unbound-idle / binding / bind-failed (falls back to the chevron plus an error toast) · Apple permanently unbindable on Android. > Div: neither platform refreshes the list after a successful bind (the row keeps its chevron until > reopened); iOS's `passkey` login type matches no row builder and emits no row; iOS's Apple error handler > is an empty closure; iOS WeChat binding round-trips through `UIPasteboard` and decrypts with a hardcoded > AES-128 ECB key — replace with a proper callback.

#### Passkey config (Passkey管理)
**Purpose:** Explain Passkey, register a credential, list and delete registered passkeys.
**Entry:** user-center row Passkey管理. **Platforms:** both.

    toolbar 42 · [◀] Passkey管理 16 Bold · pad h16 / top16, spacedBy 16 · no enclosing card
    A  explainer 12 text_secondary, spacedBy 8 · 支持设备：… [点击这里] 前往密码管理设置。
    B  [ ➕24 4 添加Passkey ]  r12 ≈56, fillMaxWidth, bg_b2, pad v16, icon+label ham_blue
    C  已注册的Passkey 16 Bold (always) · 16 · [Loading] 20 spinner left-aligned · [Empty] 暂无数据 12 · [Rows]
       ┌ r12 bg_b2 pad16, spacedBy 16 ┐ name 12 2-line max ellipsis 1f · 删除 16 ham_red
       │ 2024-10-15 01:34:22创建 12 secondary │ rows 8 apart, no dividers

**Blocks:** explainer — intro paragraph 12 `text_secondary`, then the 支持设备 block with a tappable
点击这里 span (`ham_blue` + underline) opening the system password-management settings · add button — always visible; tap → register · registered list — header (always shown) then the state-gated body · passkey row — name (12, 2-line max, ellipsis, `text_primary`, `weight 1f`) above the created date (`%s创建`, `yyyy-MM-dd HH:mm:ss`, 12 `text_secondary`); trailing 删除 16 `ham_red`; tap → **confirm dialog**, then delete.

| Element | Value | Note |
| --- | --- | --- |
| Root / explainer | pad h16 / top16 / bottom 0, `spacedBy 16` / inner `spacedBy 8` | iOS wraps everything in `.padding(16)` on a ScrollView |
| Body text / warning spans / link | 12 `text_secondary` #888888 / `ham_red` #F44336 / `ham_blue` + underline | Android resolves `ham_red` from `R.color.red`, not the themed token |
| Add button | r12, `bg_b2`, fillMaxWidth, pad v16 ≈56; icon 24 `ham_blue`; gap 4 | iOS 17 with a default-size `plus` |
| Section header | 16 Bold `text_primary` | iOS 17 bold |
| List spinner / empty / row | 20, stroke 2, `ham_gray`, **left-aligned** / 暂无数据 12 `text_secondary` / r12, pad 16, `spacedBy 16`, 8 apart, ≈64, no divider | iOS spinner is centred |
| Name / date | 12, 2-line max, ellipsis / 12 `text_secondary` | iOS name 14, date 12 primary (not dimmed) |
| 删除 | 16 `ham_red`, **min 48 hit target** | iOS 17 red with no hit target |

**Strings:** Passkey管理 · the Passkey explainer paragraph · 支持设备： · 1. 带GMS服务的设备 … ·
点击这里 · 前往密码管理设置。 · 2. 搭载鸿蒙系统的设备… · 3. 搭载ColorOS14系统的设备… · 添加Passkey · 已注册的Passkey · 暂无数据 · %s创建 · 删除 · 注册Passkey失败 · 已删除Passkey. **States:** list loading (left spinner) · loaded-empty (暂无数据) · loaded · error / unload (**show an
error view with retry — today both render blank**) · register success (refetch) · register failure (toast; user cancellation must be **silent**) · delete success (row removed) · delete failure (toast, row retained).
**> Div:** iOS's add button has no loading or disabled state, so repeated taps fire repeated requests.

#### Logout / deactivate account (退出登录 / 注销该账号)
**Purpose:** End the session; separately, permanently delete the account. **Entry:** user-center logout
row (→ logout); the dedicated 退出登录 screen (→ deactivate, iOS route `userCenterLogout`). **Platforms:** both.

    logout = a confirm dialog: ┌ 退出登录 24 Bold ┐ 确定要退出登录吗？ 16 ┐ 取消 · 确定 (ham_red) ┐
    deactivate screen:
      toolbar 42 · [◀] 退出登录 16 Bold · pad v8/h16
      ┌ 注销该账号 16 ham_red 1f        [◌25 | ▶ rot 0°↔90°] ┐ ≈40
      ══ AnimatedVisibility(fade + expand) ══
      │ 注销该账号后，您将无法使用同步功能，您的一切个人信息将被清除，是否继续？ 12 │
      │                          [◌25 | 确定]  r8, red@10%, pad h16/v2, min 40 │ panel ≈88

**Blocks:** logout row — full-width card, red 16 label, pad v8, centred, ≈40; tap → **confirmation dialog**
(title 退出登录, message, 取消 / 确定 with 确定 in `ham_red`), then log out · deactivate row — 16 `ham_red`, trailing 25 spinner while loading or a chevron rotating 0° ↔ 90°; tap toggles the disclosure — nothing destructive fires yet; guard re-taps while in flight · warning panel — the 2–3-line warning at 12 `text_primary`, then a right-aligned action area holding **either** the 25 spinner **or** 确定 (mutually exclusive, so double-submit is impossible) · 确定 → deactivate; on success toast 成功 / 已注销账号 and leave the session.

| Element | Value | Note |
| --- | --- | --- |
| Logout label | 16 `ham_red` #F44336 | iOS destructive #FF3B30; Android's dead-code path uses raw `0xFFFF0000` — three reds in play |
| Logout row / deactivate row | pad v8, centred, ≈40, card r16 `bg_b2` / pad h16 / v8, ≈40 | |
| Warning text | 12 `text_primary`, panel pad v8 / h16 | |
| Spinner / 确定 / pressed alpha | 25 `CircularProgressIndicator` / r8, bg `red @0.1`, pad h16 / v2, min height 40, 14 Bold `ham_red` / 0.25 | |

**Strings:** 退出登录 · 确定要退出登录吗？ · 注销该账号 ·
注销该账号后，您将无法使用同步功能，您的一切个人信息将被清除，是否继续？ · 确定 · 取消 · 成功 · 已注销账号. **States:** idle · logout confirming · logout in flight (25 spinner) · deactivate in flight (spinner
replaces 确定) · error (toast, return to idle) · deactivate success (toast, then leave the session) · logout success (session cleared). > Div: **Android has no deactivation UI at all** — its `SyncLogoutView.kt` is unreachable dead code (no > route in `UserCenterPath`), so 注销该账号 and its copy are defined but never read. iOS's 确定 currently > performs an ordinary logout rather than the promised permanent deletion and never resets its loading > state. Android's shipped logout calls `logout()` directly: no dialog, spinner, toast, or error handling.

#### Authorized apps (授权应用)
**Purpose:** List third-party apps granted SSO access, with scopes and grant date, and revoke them.
**Entry:** user-center row 授权应用. **Platforms:** both.

    toolbar 42 + status bar, bg_b1 @0.95 · [◀] 授权应用 16 Bold · pad 16, items spacedBy 16
    [First load] centred 20 spinner, full size · [Empty] centred 暂无授权应用 16 secondary
    ┌ card r16 bg_b2, pad 16 + 12 ────────────────────────┐ content = screen − 32
    │[ic48] App name 16 Bold │ icon r10, Crop, crossfade │ 12 描述 (12, 2-line max) · [scope][scope] 11 chips, FlowRow 4×4 │ bg gray@0.15 r4 pad h6/v2
    │ 授权于 2026-04-16 12 gray                     │ 6 above
    │ 取消授权 12 ham_red [◌14] │ 8 above, ≈104–183
    └ tail paging spinner 20 · bottom 16 + bottom safe ┘ page size 10
    revoke dialog: 取消授权 24 Bold · 确定要取消对该应用的授权吗？16 · 取消 · 取消授权 (destructive)

**Blocks:** first-load spinner (empty list + loading) · empty state · list — one card per app,
`spacedBy 16`, page size **10**, prefetch when the 2nd-to-last item composes · app card — icon 48 r10 (placeholder `Apps` 28 on `ham_gray@0.2`) · name 16 Bold one line · description 12 2-line max (conditional) · scope chips 11 (conditional) · grant date `授权于 %s` in `yyyy-MM-dd`, 12 secondary (conditional) · 取消授权 12 `ham_red` (8 above) · revoke — tap opens the confirmation dialog; it does not revoke directly; confirm → revoke, replacing the row label with a 14 spinner; success removes the row, decrements the total, toasts 已取消授权 on `ham_green`; failure restores the label and toasts · tail paging spinner 20, full-width centred.

| Element | Value | Note |
| --- | --- | --- |
| Card | r16, `bg_b2`, pad 16 + row pad 12 = 28 all sides, no dividers | iOS uses a flat list: row pad h16/v12, icon 44 r10, divider after every row |
| Icon | 48, r10, `ContentScale.Crop`, crossfade | iOS 44, r10, fade 0.3 s |
| Name / description / date | 16 Bold one line / 12 `text_secondary` 2-line max / 12 secondary | iOS date uses the system abbreviated format |
| Scope chip | 11, `ham_gray @0.15`, r4, pad h6/v2, FlowRow 4×4, not tappable | hardcoded 11.sp — use the `caption2` token |
| 取消授权 | 12 `ham_red`; row spinner 14, stroke 2, `ham_red`, gap 6 | iOS: outlined pill 12, r8, 1dp `red@0.3` stroke; Android hardcodes 13.sp |
| Spinners | 20, stroke 2, `ham_gray` | |
| Dialog | r16 card treatment, no elevation; title 24 Bold; body 16; confirm `ham_red` | Android's is a stock M3 `AlertDialog` (r28, elevation 6, min 280 / max 560, scrim `black@0.32`) — `HamTheme` is not applied |
| Bottom safe area / page size / pressed alpha | 16 content padding **plus** the safe-area inset / 10 / 0.25 | Android omits the safe-area spacer |

**Strings:** 授权应用 · 暂无授权应用 · 取消授权 · 确定要取消对该应用的授权吗？ · 已取消授权 · 授权于 %s ·
确定 · 取消. **States:** loading · paging (tail spinner, list stays) · loaded · empty ·
**loaded-with-error (mid-list: toast, list retained, no inline retry; on first load render an error state
with a retry — today no branch matches and the page is blank and unrecoverable)** · revoking (row spinner, tap suppressed, single-flight) · revoke error (toast, row retained) · revoked (row removed, success toast, empty state if last) · dialog dismissed (nothing revoked). > Div: Android uses the framework string `android.R.string.cancel` for 取消, so it is not app-localisable.

#### SSO authorization sheet (授权请求)
**Purpose:** Review and grant the scopes a third-party app requests, then hand an auth code back through
`redirect_uri`. **Entry:** inbound `sso-authorize://` / `https://ham.nowcent.cn/sso-authorize?…` deep link. **Platforms:** both.

    NAV 授权请求 16 Bold centred, 36 (Android) / 44 (iOS) · trailing ✕ → dismiss · 85 %, r24 top
    scroll column pad top 32 / h16, spacedBy 32 · icon 64×64 r14 Crop+crossfade (ph gray@15% + Apps 32)
    8 · app name 20 Bold 1 line · 8 · description 12 #888888 centred ≤5 lines (omit when empty)
    32 · 请求以下权限：12 #888888 · 4 · ┌ scope card gray@8% pad 12 r12, rows gap 8 ┐ ☑(必选) … ☑(已授权) … ☑ … │ checkbox 20, gap 8, text 12 2-line
    200 spacer → ░ gradient bg_b1 0→.8→.9→1, pad 32/32 ░ · 下次自动授权 12 + switch, pad h20/v6
    ┌ 授权 16B r12 fillMaxWidth pad v12, blue@15% on / gray@10% off ┐ 16 → 取消 16 blue, no bg · action pad h20/v12, bottom 32

**Blocks:** nav bar (every variant) · app header (icon, name, description — the description block is
omitted when empty) · 请求以下权限： label · scope card — one row per scope in server order, no dividers, no per-scope icon or name; every scope starts **selected** · scope row — checkbox 20, gap 8, text 12 2-line max; `isLocked = isRequired || alreadyGranted` → checkbox disabled, text `text_secondary`; prefix tags `(必选)` / `(已授权)` · 200 spacer so the last row clears the gradient · bottom gradient (floats over the scroll content, swallows touches) · auto-authorize switch (writes the preference **immediately**, even if the user then cancels) · 授权 — enabled iff `hasSelectedScopes && hasScrolledToBottom` (short content auto-passes) · 取消 — dismisses, never notifies the third party.
**Variants** (`canAutoAuthorize` = server flag; `autoAuthorizeEnabled` + `lastAuthorizeTime` = a local
preference keyed by `client_id`): **A full** — `canAutoAuthorize == false` **OR** `autoAuthorizeEnabled == false` · **B simplified** — both true **AND** last authorize > 24 h ago or never (`Spacer(1f)` → header → `CheckCircle` 48 `ham_green` → 12 → 你已授权过该应用，是否继续授权？ 16 centred pad h32 → `Spacer(2f)` → gradient → 授权 always enabled + 取消; no scope list, no switch) · **C silent** — both true **AND** last authorize within 24 h (no content; `authorize()` fires immediately and the sheet self-dismisses).

| Element | Value | Note |
| --- | --- | --- |
| Sheet / corners / scrim / drag strip | 85 % + 100 over-drag / r24 top-start + top-end / `black @0.5 × modalProgress` / 32, **draw a visible grabber** | iOS declares no `presentationDetents` — set them |
| Dismiss / resistance | release hides when `target > sheetHeight / 4`; `yDelta/2` below rest, `yDelta×2/(|current|+1)` above | |
| Nav bar / app icon | 36, `bg_b1` opaque, status-bar inset suppressed, title 16 Bold max 300 one line / 64×64 r14 Crop + crossfade, placeholder `gray@0.15` + `Apps` 32 | iOS nav bar 44 |
| App name / description | 20 Bold one line / 12 `#888888` centred ≤5 lines | **iOS uses `.title3.bold()` ≈17 — the largest text divergence in the surface** |
| Scope card / checkbox | `ham_gray @0.08`, r12, pad 12, row gap 8 / 20, gap 8, text 12 2-line max | iOS uses `ham_text_t2Color @0.08`; iOS checkbox is an SF `checkmark.square.fill` / `square` |
| Bottom gradient / action column | `bg_b1` at 0 / 0.8 / 0.9 / 1, pad 32 / 32 / pad h20 / v12, `spacedBy 16` | iOS uses an opaque bar with a 32 gradient spacer |
| 授权 / 取消 | 授权: r12, fillMaxWidth, pad v12, 16 Bold; enabled `ham_blue` on `ham_blue@0.15`, disabled `ham_gray` on `ham_gray@0.10` · 取消: 16 `ham_blue`, no background or border | iOS uses system `accentColor` |
| Loading / guards / tags | spinner 40, stroke 4, `ham_blue`, centred · redirect allow-list `http`/`https`/`ham` · nonce-invalid auto-retries the fetch **once** (consent choices lost) · missing `redirect_uri` aborts **without showing the sheet** · prefix tags `(必选)` / `(已授权)` must be localised | iOS spinner unsized; Android concatenates the tags into the description and loses the styling |

**Strings:** 授权请求 · 请求以下权限： · 必选 · 已授权 · 下次自动授权 · 授权 · 取消 ·
你已授权过该应用，是否继续授权？ · 网络错误，请稍后重试 · 无法打开回调链接 · 请至少选择一项权限. **States:** idle / redirecting / error → nothing (sheet dismisses) · loading and authorizing → spinner ·
A · B · C · no scopes selected → silent no-op · not scrolled to bottom → 授权 disabled with **no explanation (add one)** · not logged in → sheet force-hidden, global login prompt shown. > Div: iOS adds a trailing `✕` close; Android relies on `showBackButton = false` plus swipe. iOS has an > authorized-apps screen and `SSOAuthPreferenceManager`; Android has neither a revoke/forget UI nor > account selection (`removePreference()` is never called).

#### Scan code (扫一扫)
**Purpose:** Continuous barcode / QR capture for login, palette sharing, and general scanning.
**Entry:** user-center row 扫码登录; the course-theme editor row 扫码导入自定义配色. **Platforms:** both.

    ▓ ham_black, safe area ignored · toolbar transparent, action-bar area ignored, back hidden ▓
           <title> 17 Bold white · 24 above the scan-window top
    │ HMS ScanKit preview fillMaxSize, continuous scan on · ▂▂▂▂ sweep band ▂▂▂▂ │ scan frame 300 tall: top = (screenHeight−300)/2
    │ ● 32 ham_blue at result │ [⚡torch 44] bottom-left · [🖼album 44] bottom-right · glyph 24, inset 16 │ · denied: icon + 相机权限未开启 + 请前往“设置”开启 + 去设置 (replaces the preview)

**Blocks:** preview — full-bleed, `ham_black`, safe area ignored, continuous scan on; results arrive by
delegate so **consumers must dedupe** · mask / window / reticle — centred transparent window 300 tall, horizontal extent = screen width · hint caption, caller-supplied · sweep band — radial gradient `Color.blue` at 0.9 / 0.7 / 0.5 / 0 from centre, base 200×200, x-scaled to `(screenWidth − 32)/200`, y-scaled 0.2, clipped to 20 tall; loop: fade in 1 s linear → sweep 3 s ease-in-out (travel 450) → hold 2 s → fade out 1 s linear → gap 1 s → repeat · torch bottom-left, album bottom-right, both 44×44 with a 24 glyph and 16 inset; album opens the photo picker and scans a still · permission denied → persistent state, preview hidden, sweep stopped · accept only `ham://` payloads; toast otherwise.

| Element | Value | Note |
| --- | --- | --- |
| Background / scan frame | `ham_black` alpha 1 / 300 tall, centred, full width | iOS sets no frame; **Android's `right` comes from `screenHeight` — copy-paste bug** |
| Sweep / loop | fillMaxWidth × 300 / 1 s in, 3 s sweep, 2 s hold, 1 s out, 1 s gap | Android uses HMS `ScanDrawable`; iOS-only animation |
| Result marker | 32 `ham_blue` circle at the detected centre | iOS none |
| Toolbar | transparent, action-bar area ignored, back hidden, title 17 Bold white | |
| Torch / album | 44×44, glyph 24, inset 16 | **iOS omits both** |
| Startup / lifecycle | `onPause(); onStop(); delay(100); onResume(); onStart()` / map host lifecycle to `onStart…onDestroy`, pause off-route | Android only |
| Unsupported device | no ARM ABI → titled screen with a 16-padded message | iOS has no fallback |
| Permission toast | bg #FF0000, fg white, 12 caption, 2-line max, full width, dismiss 3.3 s | |

**Strings:** 扫描登录二维码 · 扫描配色分享二维码 · 相机权限未开启 · 请前往“设置”开启 · 去设置 · 相册 ·
扫码登录 · 扫码导入自定义配色 · 成功从二维码导入配色. **States:** authorized · permission undetermined · denied · scan success (`ham://` → push QR login, deduped on route equality; other → post with the caller `id`) · unrecognised payload (toast).

#### QR code login (二维码登录)
**Purpose:** Confirm a desktop sign-in after the phone scans a desktop QR. **Entry:** deep link
`ham://qrcode-login?ticket=…` pushed from the scanner. **Platforms:** both.

    toolbar 42 + status bar, bg_b1 @0.95 · [◀16 ham_blue] 二维码登录 16 Bold max 300 · pad h16, spacedBy 8, centred, non-scrolling · bottom spacer = nav-bar inset + 80
      64 · ✓ 64 (success: circle ham_blue, pad 8, glyph 48 white) / 🖥 72 (confirm: tint text_primary)
      登录成功 24 Bold / 确定在电脑上登录Ham吗 (server message, 3-line max, centred) 16
    40 (8 spacing + 32 padding) · countdown 二维码将在 m:ss 后失效 · loading: spinner + skeleton
    ┌ 返回 / 确认登录 16 Bold, r12, pad v12, fillMaxWidth, accent@0.15 ┐ + 取消 alongside 确认登录

**Blocks:** toolbar, back pops the stack · top spacer 64 (both states) · state icon · message — success
登录成功 24 Bold; confirm the server `message` verbatim, falling back to 确定在电脑上登录Ham吗 · countdown — remaining validity; on expiry auto-fetch a new ticket and re-render, and offer an explicit 刷新 (**neither platform does this today**) · 确认登录 — only when the server state is `REQUEST_CONFIRM` (2); tap → confirm, disable the button and show an in-flight spinner, then swap to the success block · 取消 — always present alongside 确认登录, pops · 返回 — on the success block and on any non-confirmable state, pops to the user-center root (**not** back into the live scanner) · loading — centred 20 spinner plus skeleton; errors render inline with a retry.

| Element | Value | Note |
| --- | --- | --- |
| Confirm / success icon | 72 `Computer`, explicit tint / 64 circle `ham_blue`, pad 8, 48 white glyph | Android's confirm icon has no tint and is invisible in dark mode; iOS's success check is white-on-green, i.e. white-on-white in light mode |
| Message / success title | 16 / 24 Bold | iOS `.title` 28 |
| Button gap / button / feedback | 40 (8 + 32) / fillMaxWidth, r12, `accent @0.15`, pad v12, 16 Bold accent / disabled tint + in-flight spinner | iOS caps width at 350, r8, `.padding()` 16, raw `Color.blue` at 0.1; no feedback today — the button can be spam-tapped |
| Expiry / countdown | poll the check RPC, show `m:ss`, auto-refresh on expiry | `CheckQrCodeLogin` is never called on either platform |
| Fetch RPC / server states / dismiss | fetch ticket info on appear, cancellable / 0 none · 1 success · 2 requestConfirm · 3 fail · 4 expired / success pops to the user-center root | iOS fires it in `init`, uncancellable; iOS pops one level back into the live scanner, which re-reads the same QR |

**Strings:** 二维码登录 · 确定在电脑上登录Ham吗 · 确认登录 · 取消 · 登录成功 · 返回 ·
二维码将在 %@ 后失效 · 遇到了错误. **States:** loading · awaiting confirmation · confirm in flight · success · not confirmable (icon + message + 返回) · error (inline + retry) · expired (auto-refresh or 刷新). > Div: iOS's fallback message omits the trailing full-width question mark (吗 vs 吗？). Android renders a > completely blank screen while loading and discards the returned `LoginTokenInfo`.

#### Login (登录Ham)
**Purpose:** Gate non-campus features behind an account; offer Passkey and social providers, gated on
privacy consent. **Entry:** a centred overlay above the nav host, shown by the account manager. **Platforms:** both.

    ░ scrim black @0.5, tap → dismiss (plus a back handler) ░
    ┌ card: margin 24, r16, pad 16, bg_b2, no elevation ──────────────┐
    │ 登录Ham 16 Bold  1f                                     [✕ 24]  │ → dismiss (no sign-in)
    │ 登录Ham后，你可以使用校内功能以外的其他功能 12 secondary (remote-overridable) │
    │ [☐/☑]16 4 我已阅读并同意 用户隐私协议 12, link ham_blue+underline │ (column spacedBy 4) │ ── 20 ── · ╭ 通过Passkey登录 16B, r12, h56, gray@0.15, icon 24, gap 4 ╮ │
    │ ── 20 ── · 或者选择以下登录方式 12 secondary (if providers) │ ●QQ ●Apple ●WeChat ●GitHub ●自强 32⌀, spacedBy 16 │
    └ loading: scrim black@0.75 + 64×64 r16 bg_b2 plate + spinner ┘ · providers: QQ `QQ-2` pad5→22 on #12B7F5 (iOS hides unless QQ installed) · Apple `applelogo` 22 on text_primary (**iOS only — add on Android**) · WeChat `login/wechat` pad5→22 on #58BE6A · GitHub `login/github`(/`_light` dark), no circle · 自强 `login/ziqiang` pad5→22 on `ham_darkBlue @0.15` (Android uses `ham_lightGray`)

**Blocks:** scrim (tap and back dismiss) · card · title row — 登录Ham 16 Bold plus a trailing close;
closing does **not** sign in and there is no skip or guest path · subtitle, remote-overridable · consent row — checkbox 16, gap 4, 我已阅读并同意 plus an underlined `ham_blue` 用户隐私协议 link at 12; tapping the link opens the terms screen; any provider tap with the box unticked toasts 请阅读用户隐私协议; **persist the checkbox state** · passkey button — always visible, not gated on the provider list · 或者选择以下登录方式 — conditional on a non-empty provider list · provider circles in order QQ → Apple → WeChat → GitHub → 自强; when none is configured show 该版本暂不支持登录 · provider web flows — a sheet-hosted webview to the provider URL returning via `ham://login-server/open/<provider>/redirect` · loading — scrim `black @0.75` plus a 64×64 r16 `bg_b2` plate with a spinner.
**Login webview sheet:** 85 % height, r24 top corners, 100 over-drag, scrim `black@0.5 × progress`, 32
invisible drag strip (**draw a grabber**); header with 关闭 (start) and 刷新 (end), both 14 `ham_blue` at pad 16; webview below; when SSL errors are ignored show a red banner (`ham_red @0.7`, pad 8, gap 4, warning icon 24 white, text 12). **Install a back handler and add a progress bar.**

| Element | Value | Note |
| --- | --- | --- |
| Scrim / card margin / radius / padding | `black @0.5` / 24 / 16 / 16 | **iOS scrim 0.3, Android 0.75**; iOS margin 16, r24, 468 width cap, Liquid Glass on iOS 26+ |
| Title / subtitle / consent row | 16 Bold / 12 `text_secondary` / 12, checkbox 16, gap 4, link `ham_blue` + underline | iOS 17 Bold, cloud-overridable tip; iOS checkbox is a default-size SF symbol in an `HStack(spacing: 0)` |
| Gaps | consent→passkey 20; other-methods 16 above then 8 to the row | iOS 8; Android 12 |
| Passkey button | fillMaxWidth, r12, h56, `ham_gray @0.15`, icon 24, gap 4, label 16 Bold `text_primary` | Android: r10, `ham_gray @0.3`, pad v8, label 16 **normal**, icon 24 offset y +1 |
| Provider circle / gap | 32 / 16 | iOS 35 / 16 |
| Loading / entry / pressed alpha | scrim `black @0.75` + 64×64 r16 `bg_b2` plate / fade 0.35 s / 0.25 | Android has no login loading state |

**Strings:** 登录Ham · 登录Ham后，你可以使用校内功能以外的其他功能 · 我已阅读并同意 · 用户隐私协议 ·
通过Passkey登录 · 或者选择以下登录方式 · 该版本暂不支持登录 · 请阅读用户隐私协议 · 关闭 · 刷新 · 该网页存在SSL证书错误，请注意甄别。. **States:** hidden · normal · loading · provider list empty · consent unticked (toast; do not disable the buttons) · provider web sheet open · success · error (toast; cancellations silenced). **> Div:** iOS has an Apple login and a QQ-installed check, Android neither; the privacy URL differs — `https://whu-ham.github.io/privacy/` (iOS, Android `PrivacyView`) vs `https://orangeboychen.github.io/whu-ham/privacy/` (Android `common_privacy_agree_text`).

#### CAS settings (信息门户设置)
**Purpose:** Report whether the stored CAS (信息门户) credential is present and still valid, re-run the
portal login, and unbind. **Entry:** My-tab user-center CAS row; the CAS error card in the status module. **Platforms:** both.

    toolbar 42 + status bar, bg_b1 @0.95 · [◀] 信息门户设置 16 Bold · pad h16, spacedBy 8
    ┌ card r16 bg_b2 pad16 ──────────────────────────────────────┐
    │ 登录状态 16 Bold · 信息门户的登录状态 12 secondary · 8 · status slot · 8 │
    │ [AnimatedContent] ◌20/2 | 已登录 12 | 登录状态失效 12 ham_red | 未登录 12 │ 重新登录 / 登录 16 ham_blue │
    ┌ card r16 — ALWAYS shown ── 其它设置 16 Bold · 8 · 退出登录 16 ham_red ─┐
    login sheet 85 % r24: NAV 信息门户 + 关闭 14 ham_blue pad h16 → CAS webview / RN module

**Blocks:** toolbar · login-state card — title + subtitle + an `AnimatedContent` state slot (unbound:
nothing rendered and the button reads 登录; validating: 20/2 spinner; bound and valid: 已登录; bound and expired: 登录状态失效 in `ham_red`; initial: empty) · re-login button 重新登录 when bound, 登录 when not, 16 `ham_blue` → shows the login sheet · other-settings card — **always shown**; 其它设置 + 退出登录 16 `ham_red`; tap → confirm, then clear the credential and re-validate · login sheet — 85 %, r24 top, 100 over-drag, scrim `black@0.5 × progress`, 32 drag strip; nav title 信息门户 with a trailing 关闭 (14 `ham_blue`, pad h16); body is the CAS webview or the `RNCasMobileLogin` RN module when the remote config enables it; **install a back handler** · validity probe — `CasClient(service: "https://bus.whu.edu.cn/mobile/%23%2F").fastLogin()`, a `ticket` in the redirect means valid — **use a CAS-specific endpoint, not the bus service** · login capture — require a `cas.whu.edu.cn` cookie containing both `CASTGC` and `JSESSIONID`; store the cookie, username and password for downstream modules;
**do not persist the plaintext password** · success — hide the sheet, refresh the state, toast 登录成功.

| Element | Value | Note |
| --- | --- | --- |
| Toolbar / padding / card gap / card | 42 + status bar, `bg_b1 @0.95` / 16 / 8 / r16 `bg_b2` pad 16, no elevation, title→content gap 8 | iOS sets **no** nav title |
| Card title / subtitle / status | 16 Bold / 12 `text_secondary` / 12, expired in `ham_red` | iOS builds the header by hand at 17 / 12 and uses stock `.red` |
| Loading indicator / transition | 20 / 2 / `ham_gray` / `AnimatedContent` | iOS uses an unstyled `ProgressView()` + `withAnimation` |
| Actions | 16 `ham_blue` (login) / 16 `ham_red` (logout), no background | |
| Other-settings visibility | always | iOS gates it behind `enable` |
| CAS URL | `https://cas.whu.edu.cn/authserver/login?service=https%3A%2F%2Fcas.whu.edu.cn%2Fauthserver%2Fmobile%2Fcallback%3FappId%3D985180443&login_type=mobileLogin` | hardcoded on both |
| WebView | non-persistent store, cookies cleared on every open, JS + DOM storage on, background `bg_b1`, force-dark by theme, **progress bar required** | iOS's `updateUIView` reloads on every redraw, wiping typed input |
| JS bridge | hook `#mobileUsername`, `#mobilePassword`, `#load`; remove `.social-aut-login`; student-ID length 13 | |
| Pressed alpha | 0.25 | |

**Strings:** 信息门户设置 · 登录状态 · 信息门户的登录状态 · 已登录 · 登录状态失效 · 未登录 · 重新登录 ·
登录 · 其它设置 · 退出登录 · 关闭 · 信息门户 · 学号 · 请输入正确的学号 · 登录成功. **States:** unbound · validating · bound and valid · bound and expired · login sheet open · login success
· signed out · SSL error (the warning string exists but is not wired into the webview component). > Div: iOS has a native webview fallback behind a CCKV flag; Android appears RN-only. Android's logout is > immediate with no confirmation and does not clear the Chaoxing identity fields. A cancelled sheet leaves > a stale status label — refresh on dismiss.

#### Debug (Debug)
**Purpose:** Developer-only panel repointing the HTTP and gRPC clients at arbitrary hosts.
**Entry:** the debug row of the My-tab link card / settings. **Platforms:** both — **iOS is `#if DEBUG`
gated; gate Android too** (today it is ungated).

    toolbar [◀] Debug, inline display mode · ScrollView pad h16, spacedBy 16, no vertical padding
    ┌ card r16 bg_b2 pad16 ┐ Environment 16 Bold, 8 bottom padding ┐
    │ ┌ inner spacedBy 12 ┐ Production / Test 16 Bold 1f [toggle] · 12 secondary helper (stack 4) │
    ══ AnimatedVisibility(!useProductionEnv) ══ ┌ HTTP Server ┐ helper 12 · Base URL 16B · [field r8, URL kbd] · 4 · Save 16B ham_blue
    ┌ gRPC Server ┐ helper 12 · Base URL 16B · [field r8] · Use TLS (mTLS) 16 regular [toggle] · 4 · Save 16B ham_blue │ inner spacedBy 12

**Blocks:** environment card — a toggle bound to the production flag with the label flipping
Production ↔ Test and a 12 `text_secondary` helper; when on, both server cards are hidden and their stored values ignored · HTTP server card — helper, `Base URL` label, a single-line field (placeholder `https://api.ham.nowcent.cn`, `.keyboardType(.URL)`, no autocapitalisation or autocorrection), 4 gap, Save; Save trims whitespace and persists, taking effect on the **next** request · gRPC server card — same shape with placeholder `https://api.ham.nowcent.cn:4443` plus a `Use TLS (mTLS)` toggle, then Save; the URL is split into host and port and a URL with no explicit port falls back to 4443 · persistence — `debug_use_production_env` (default true), `debug_http_base_url`, `debug_pb_base_url`, `debug_pb_use_tls` (default true); seed the edit buffers on appear and discard unsaved edits on exit.

| Element | Value | Note |
| --- | --- | --- |
| Padding / root + card spacing | horizontal 16, no vertical / 16 | |
| Card / header / body spacing | r16, `bg_b2`, pad 16 / 16 Bold `text_primary`, 8 bottom padding / 12 | iOS header 17 Semibold; Android's Environment card uses 8 |
| Toggle label / helper | 16 Bold / 12 `text_secondary`; label stack spacing 4 | |
| Toggle tint | `ham_blue` | iOS AccentColor is unset |
| Field | r8 radius, 1dp border, single line, URL keyboard | Android uses `OutlinedTextField` r12 with **no** IME configuration |
| Save / pre-button spacer | 16 Bold `ham_blue` / 4 | iOS has no spacer |
| Nav title | `Debug`, inline display mode | iOS uses the large-title bar, unlike the rest of the app |
| Build gating | `#if DEBUG` on both | Android is comment-only |

**Strings:** all English and intentionally unlocalised on both platforms — Debug · Environment ·
Production · Test · Using production server config. HTTP/gRPC settings are locked. · Using custom server config. You can edit HTTP/gRPC settings below. · HTTP Server · Changes take effect on the next HTTP request. · Base URL · Save · gRPC Server · Changes take effect on the next gRPC request. · Use TLS (mTLS). **States:** enabled (only the Environment card) / editable. Add validation and an error toast — today an invalid or empty URL is accepted silently and falls back to production. > Div: iOS uses a plain `if` where Android animates the reveal; the two files were generated separately > and agree on copy but not on animation, spacing, field style, IME config, or build gating.

#### Route table (Android user-center graph)
| Route | Destination | Notes |
| --- | --- | --- |
| `user-center` | `userCenterNavGraph` | nested graph, start `user-center/main` |
| `user-center/main` | `UserCenterMainView` | |
| `user-center/device` | `UserCenterDeviceView` | navController passed but unused |
| `user-center/social-account` | `UserCenterSocialAccountView` | |
| `user-center/edit-info` | `UserCenterInfoView` | |
| `user-center/passkey` | `UserCenterPasskeyConfigView` | |
| `user-center/authorized-apps` | `AuthorizedAppsView` | |
| `QrCodeRoute.Scan(handlerType, autoPopBack)` | `QrCodeScanView` | the only typed route in the module |
| `user-center/logout` | **absent** | logout is inline in the hub's ViewModel |

All destinations use the string `composable(route)` overload — no typed routes, no arguments, no deep links. **No graph-level auth gate**: on a null user the login layer pops the whole stack back to `MainRoute` when the current route contains `user-center`. No `popUpTo` / `launchSingleTop` / `restoreState` on any user-center navigation. |

##### Divergences

- **Settings hub is Android-only.** iOS `Route` has no `setting` case; the settings surface is a card in the My tab. The hub should exist on both.
- **Widget settings and language settings are Android-only** — correct as-is for iOS (WidgetKit owns refresh; iOS follows the system locale), but the About row set must be reconciled.
- **Logout is iOS-only as a screen; Android's is an unconfirmed one-tap destructive row.** Android's `SyncLogoutView.kt` is unreachable dead code, so **Android has no account-deactivation UI at all** and 注销该账号 / 确定 / 成功 / 已注销账号 are defined but never read.
- **iOS `debug` is `#if DEBUG` gated; Android's is ungated.**
- **iOS "automatic" is not Android's automation screen** — Siri Shortcut donation vs a scheduled library-booking alarm. Name parity, no feature parity.
- **Android function-grid tiles render with blank titles on a cold install** — the config declares `@SerialName("title-content")` but the bundled default JSON supplies `"title"` as a plain string.
- **`HamTheme` is not applied on the Android user-center path** (`MainActivity` sets no theme), so the revoke `AlertDialog` renders with stock Material3 defaults (r28, elevation 6, min 280 / max 560).
- **Blank-screen failure modes on Android:** authorized-apps and login-devices both fall to `else -> {}` on error with an empty list — a blank page with no retry. iOS falls through to a card that looks identical to "zero devices".
- **Three destructive reds in play:** iOS #FF3B30, Android themed `ham_red` #F44336, and raw `androidx…Color.Red` `0xFFFF0000` in the dead-code logout path.
- **iOS has 38 hardcoded strings across 8 files vs 1 on Android.** Four are genuine non-localising bugs (a ternary or a `String`-typed parameter defeats `LocalizedStringKey`): the CAS card label and the three `QrCodeLoginView` strings. `CasSettingView` is entirely unlocalized.
- **Android `my`'s default-locale file is `string.xml` (singular)** while en/ja are `strings.xml`.
- **Android authorized-apps uses `android.R.string.cancel`**, which is not app-controllable.
- **iOS About exposes the developer rows and the TPNS token unconditionally;** Android hides them behind a 10-tap easter egg. iOS animates its logo with CoreMotion parallax; Android's is static (96 vs 120).
- **SSO sheet:** iOS's app name is `.title3` (~17) vs Android's 24sp — the largest single text-size divergence in the surface. iOS adds a `✕` close; Android hand-composes the `(必选)`/`(已授权)` tags into the description string and loses the styling. iOS has an authorized-apps screen and a preference store; Android has neither, and no account selector exists on either.
- **Login scrim is 0.3 on iOS and 0.75 on Android;** iOS has an Apple button and a QQ-installed check, Android neither; Android has no login loading state.
- **Neither platform polls QR-login expiry** — `CheckQrCodeLogin` is never called.
- **Android's scan frame sets `right` from `screenHeight`** — a copy-paste bug.
- **iOS's CAS validity probe hits the bus service URL**, not a CAS endpoint.
- **iOS's WeChat binding flow round-trips through `UIPasteboard`** and decrypts with a hardcoded AES-128 ECB key.


---

### Shared screens

#### Intro / connect screen (连接页)
**Purpose:** Gate shown before a module has credentials. **Entry:** sheet when a module reports
not-connected. **Platforms:** both.

    [关闭] 14 ham_blue  ·  padTop 12  ·  screen padding 16, top-aligned
     ┌────┐ 🔗 ┌──────┐   icons 48, gap 12; logo r8; module icon tint @0.25
      获取课程 24 Bold  ·  副标题 16  ·  gaps 32 / 4 / 16 down to the card
    ┌ card ham_lightGray r12 pad16, rows gap 8 ─────────────┐
    │[24] 从信息门户验证 ›   row: icon 24 ham_blue, gap 8, 16B/12, chevron │

**Blocks:** dismiss top-left 14 `ham_blue` · logo → link glyph (`text_primary`) → module icon · title ·
subtitle (if any) · choice card · rows as drawn · CAS row always present and last → CAS mobile login.

| Element | Value | Note |
| --- | --- | --- |
| Logo / icon / gaps | 48 r8 / 48 tint @0.25 / gap 12; icons→title 32, title→sub 4, sub→card 16 | iOS logo 56 r12, gap 16, title→subtitle 8 |
| Card fill / radius / pad / row gap | `ham_lightGray` / 12 / 16 / 8 | iOS `.gray@0.15` / 8 / 16 / **24** |
| Dismiss label | 关闭 | iOS 取消 |

**Strings:** 关闭 · 从信息门户验证 · 将进入武汉大学信息门户网页验证你的身份 · per-module title/subtitle
(获取课程 / 将按照设定的开学日期获取课程 · 连接成绩 · 获取成绩 · 连接校巴 · 连接珞珈E卡 · 图书馆).
**States:** the screen *is* the not-connected state; no internal loading or error state.
> Div: iOS hosts a `NavigationView` stack so rows can push; Android hosts a `HamNavHost` in the sheet.

#### Error view / Success view / Empty view
**Purpose:** Terminal failure, terminal success, and no-content feedback. **Entry:** swapped in by any
screen whose load state ends in failure / success / empty. **Platforms:** both.

         ⊗ / ✓  64 box, 8 pad, white glyph on filled circle    pad 16, centred
        更新失败 28 Bold · <hint> 17 (opt) · <injected content> (Success only) · spacer 32
      ┌ fillMaxWidth h48, accent@0.15, r12 ──────────────┐
      │ 返回 · 16 Bold accent · slide-in from half-height + fade after 200 ms │

**Blocks:** status icon · title · hint (if non-empty) · injected content (Success only) · button →
`backAction` if supplied, else dismiss. Error = red circle, Success = green circle.

| Element | Value | Note |
| --- | --- | --- |
| Icon box / pad / fills | 64 / 8 / error `ham_red`, success `ham_green` | iOS `.red` #FF3B30; **Android Success `ham_blue` — inverted** |
| Title / hint / stack gap | 28 Bold / 17 / 8; icon→button 32 | iOS `.title` + 12, gap 4, button gap 16 |
| Button h / radius / fill / text | 48 / 12 / `accent @0.15` / 16 Bold accent | iOS height implicit, Success fill @0.1 |
| Padding / delay / haptic / confetti | 16 / 0.5 s / heavy impact / Lottie `lottie_congrats.json`, full bleed, speed 1 (Success) | iOS none (Error) or 32 (Success) / 0.8 s |

**Strings:** button 返回 (Android `common_done` 完成); titles caller-supplied: 更新失败 · 请求失败 ·
预约失败 · 验证失败 · 更新成功 · 验证成功 · 发布成功.
**States:** static; only the hint varies. **Empty:** **no shared empty component exists on either
platform** — specify one now: icon 64 `ham_gray`, message 17 `text_secondary`, optional action as the standard 48 pill. **> Div:** iOS supports injected Success content, Android does not; Android always renders the message.

#### Loading views
**Purpose:** Indicate in-flight work. **Platforms:** both.

    ░ scrim black @0.65, tap-absorbing, fillMaxSize ░
        ┌──────────┐
        │    ◌     │   72 box, r12, ham_bg_b1
        └──────────┘

**Blocks:** inline spinner · optional label below it (pair text with the spinner whenever the wait
exceeds ~1 s) · blocking modal for non-cancellable work · skeleton for list-shaped content.

| Element | Value | Note |
| --- | --- | --- |
| Spinner size / stroke / colour | 20 / 2 / `ham_gray` | iOS has **no** shared spinner — bare `ProgressView()` at ~30 sites |
| Modal scrim / box | black @0.65 / 72, r12, `ham_bg_b1` | iOS has no equivalent |
| Skeleton | `LazyVGrid` adaptive min 70, spacing 10, 48 cells, r8, `gray @0.2`, v-pad 4 | iOS-only today |
| Determinate variant | `ProgressView(value:)`, `nil` → indeterminate | iOS booking flow only |

**States:** loading only. Error → Error view; empty → Empty view.

#### Toast
**Purpose:** Transient top-anchored notification. **Platforms:** both.

    ←status-bar inset→ · Surface r12, pad 16, fillMaxWidth, slide+fade in / fade out
     │ ⊙ 32  标题 16 Bold  ·  内容 12, 2 lines max  ·  gap 8 │

**Blocks:** status-bar-safe top inset · type icon 32 · title (optional) · content (optional, 2-line
max) · tap or drag-up (>10) → dismiss; auto-dismiss 3 s.

| Element | Value | Note |
| --- | --- | --- |
| Icon / gap | 32 / 8 | iOS 36 / 5 |
| Title / content | 16 Bold / 12, 2-line max | Android 16/16; iOS 17 Sem / 12 |
| Radius / padding | 12 / 16 (top = status-bar inset) | iOS 0, full-bleed rectangle |
| Auto-dismiss / animation / gestures | 3 s / slide from top + fade / tap + drag-up | Android 2000 ms, fade only, tap only |
| Types | info / warning / success / error / normal | Android has only Normal / Success / Error |
| Palette | normal `text_primary` on #EDEEEF · success white on `ham_green` · error white on `ham_red` · info white on `ham_blue` · warning white on yellow | |
| Icons | success `checkmark.circle.fill` · error `xmark.circle.fill` · info+normal `exclamationmark.circle.fill` · warning `exclamationmark.triangle.fill` | |

**Strings:** caller-supplied; drop when title and content are both empty. **States:** single transient
state; no legacy "了解更多" action.

#### Bottom sheet
**Purpose:** Primary modal container (Intro, Pay, SSO, changelog, privacy). **Platforms:** both.

    ░ scrim black @(0.5 × progress), tap → dismiss ░
       ╭ top radius 24, bg_b1, height 85 % of screen ╮
       │ ▭ drag strip 32 — DRAW A GRABBER ▭ │
       │ content │   over-drag 100; dismiss at offset > height/4
      below rest: offset += delta/2 · above rest: offset += delta×2/(|current|+1)

**Blocks:** scrim (tap dismisses) · surface `ham_bg_b1`, top-only radius 24 · drag strip 32 · caller
content — **callers must supply their own 关闭/取消 control; the sheet ships none.**

| Element | Value | Note |
| --- | --- | --- |
| Height | 85 % of screen (large detent) | iOS uses stock `.sheet` with **no** `presentationDetents` |
| Top radius / background | 24 / `ham_bg_b1` | |
| Scrim / drag strip / over-drag | black @(0.5 × progress) / 32 / 100 | |
| Dismiss / animation | offset > height/4 / `Animatable.animateTo(0 / sheetHeight)` under a mutex | |
| Content when hidden | not rendered | |

**States:** shown (offset 0) / dragging / hidden (offset = height).

#### Text field
**Purpose:** Single- or multi-line text entry. **Platforms:** both.

    ┌ r8, fill ham_bg_b2, 1 border ham_lightGray, inner pad 8, body 17 ┐ fillMaxWidth

| Element | Value | Note |
| --- | --- | --- |
| Radius / fill / border | 8 / `ham_bg_b2` #FFF9F9F9 / 1 `ham_lightGray` #EDEEEF | iOS 4/6, no fill |
| Inner padding / text | 8 / 17 | iOS 6 / 9; Android 16 |
| Hint / cursor | `ham_text_secondary` #888888 / `ham_blue` | |
| Lines | single-line; `isPassword` → dot transformation | |
| Multi-line | plate `gray @0.3` r12, editor minHeight 30, placeholder `gray` @0.8 | iOS-only today |

**States — all required:** empty (hint) · focused (border `ham_blue`) · error (border `ham_red` +
caption-12 message) · disabled (fill `ham_lightGray`, text `text_secondary`). Only empty/non-empty exist today. **Strings:** hint is caller-supplied.
> Div: iOS's `TextEdit` and `TextEditorApproach` are dead code — only Android's `HamTextField` is live.

#### Segmented picker
**Purpose:** Switch between 2..n mutually exclusive options. **Platforms:** both.

    ┌ track ham_gray@0.4, r6, pad 2, Box fillMaxWidth ─────────┐
    │ ┌ thumb = (measuredWidth/count) − 2, ham_lightGray, r5 ┐  label 1  label 2 │
    └── animateDpAsState on x-offset; segments weight 1f each ─┘

| Element | Value | Note |
| --- | --- | --- |
| Track fill / radius / padding | `ham_gray @0.4` / 6 / 2 | |
| Thumb width / fill / radius | `(measuredWidth/count) − 2` / `ham_lightGray` / 5 | radius 5 vs track 6 — align |
| Thumb height / animation | full measured track height / `animateDpAsState` on x-offset | |
| Segment weight / selection | 1f each / **by value or key** | iOS component selects by index, dead code |
| Flags | optional `isScroll` horizontal mode; optional sliding indicator | |

**States:** selected / unselected. No loading, empty, or error state. **Strings:** caller-supplied.

#### Divider
**Purpose:** Separate rows or sections. **Platforms:** both. Horizontal → full width × 1; vertical →
`IntrinsicSize.Max` height × 1. **Values:** thickness **1**, colour **`ham_divider` #F4F4F4 / #0C0C0C**.
> Div: Android's `HamDivider` uses `ham_lightGray` #EDEEEF and leaves the `ham_divider` token unused;
> iOS calls stock SwiftUI `Divider()` with a system colour. Three values disagree.

#### Bus (校巴)
**Purpose:** Webview of `https://bus.whu.edu.cn/mobile/#/` opened with a CAS ticket, behind an Intro gate.
**Entry:** My-tab / module link. **Platforms:** both.

     ◀ 校巴 · toolbar 42 + status bar · bg_b1 @0.95
     [WebView fillMaxSize, bounce off, geolocation shim injected on finish]
     ┌ overlay sheet when CAS not authorised ──────────────────┐
     │ 连接校巴 · brand .brown · icon DirectionsBus · 500 ms delay │
     │ [🎓] 从信息门户验证 › │

**Blocks:** webview fillMaxSize, bounce off, inject a `navigator.geolocation` shim on page finish ·
toolbar 42 + status bar, title 校巴 · intro gate after 500 ms when CAS is not authorised (连接校巴, 使用前，Ham需要验证你的在校信息, brand `.brown`, single CAS row) · ticket via `CasClient(service: "https://bus.whu.edu.cn/mobile/%23%2F").fastLogin()`, accept only if the redirect contains `ticket`.

| Element | Value | Note |
| --- | --- | --- |
| Toolbar | 42 + status bar | iOS has none — bare webview |
| Intro icon / brand / delay | `bus.fill` / `DirectionsBus` / `.brown` / 500 ms | iOS delay 300 ms |
| Success copy | 验证成功 / 你可以开始使用**校巴**了 | **BUG: Android ships 你可以开始使用图书馆了** |

**Strings:** 校巴 · 连接校巴 · 使用前，Ham需要验证你的在校信息 · 从信息门户验证 ·
将进入武汉大学信息门户网页验证你的身份 · 验证成功 · 验证失败.
**States:** not-authorised → Intro · authorised → webview · ticket failure → error toast (iOS) / Error
view in the Intro sheet (Android). No loading indicator.

#### Pay (珞珈E卡)
**Purpose:** Campus E-card web app behind an Intro gate. **Entry:** My-tab / module link. **Platforms:** both.

    ┌ HamSheet 85 % ──────────────────────────────┐
    │ ◀ E卡 · toolbar 42, status-bar height ignored │
    │ [WebView fillMaxSize, bounce off, back + close affordances] │
    ├ not logged in ─────────────────────────────┤
    │ IntroView: 连接珞珈E卡 · brand #BF360C · icon creditcard.fill · CAS row │

**Blocks:** intro gate when not logged in (连接珞珈E卡, 使用前，Ham需要验证你的在校信息, brand
`ham_brand_pay` #BF360C, single CAS row; success → Success view 验证成功 / 你可以开始使用E卡了, failure → Error view 验证失败) · webview sheet with title E卡 plus back and close affordances, fillMaxSize · safe area: one rule — ignore the status-bar height, respect the bottom.

| Element | Value | Note |
| --- | --- | --- |
| Presentation | `HamSheet` 85 % | iOS branches inline inside a `VStack` |
| Toolbar / title | 42, status-bar height ignored / E卡 | iOS inline nav bar, ignores the bottom inset |
| Intro icon / brand | `creditcard.fill` / `Icons.Rounded.CreditCard`, #BF360C | |
| Intro subtitle | 使用前，Ham需要验证你的在校信息 | Android shows none |

**Strings:** E卡 · 连接珞珈E卡 · 使用前，Ham需要验证你的在校信息 · 从信息门户验证 ·
将进入武汉大学信息门户网页验证你的身份 · 验证成功 · 你可以开始使用E卡了 · 验证失败.
**States:** not-logged-in → Intro · logged-in → webview · fetch failure → Error view (Android) / error
toast (iOS). No loading state.

#### Privacy (隐私协议)
**Purpose:** First-run blocking consent gate. **Entry:** cold start when consent is not granted.
**Platforms:** both.

    ░ fillMaxSize bg_b1 + black@0.5 scrim ░
      ┌ card 300 wide, r10, bg_b2, pad 16 ────────────────────┐
      │ 隐私协议 24B · 您需要同意隐私协议才能继续使用Ham。17 · Ham不会… 17 │
      │ 总的来说，Ham会获取下列信息：17 · [必须] 设备标识… · [可选] 查给分… │
      │   [查看隐私协议] ham_blue, end-aligned │
    ┌ policy sheet 85 % · toolbar pad 8, SpaceBetween ───────────────┐
    │ 拒绝 ham_red (dismiss only)        接受 ham_blue (dismiss + consent) │
    │ [WebView: {host}/privacy/] │  gate: 退出Ham ham_red / 同意 accent │

**Blocks:** scrim + centred card · title · lead sentence · legal paragraph · `[必须]` / `[可选]`
data-collection list · view-policy link (end-aligned, `ham_blue`) → 85 % policy sheet · policy-sheet toolbar: 拒绝 (`ham_red`) dismisses only, 接受 (`ham_blue`) dismisses **and** records consent · gate actions 退出Ham (`ham_red`, suspends the app) / 同意 (accent).

| Element | Value | Note |
| --- | --- | --- |
| Card width / radius / fill / padding | 300 / 10 / `bg_b2` / 16 | iOS is a full-screen `VStack`, no card |
| Scrim | black @0.5 | iOS none |
| Title / body / paragraph gap | 24 Bold / 17 / 8 | iOS `.title`+Semibold, legal text 12, gap 15 |
| Link | 17 `ham_blue` | iOS 12, system blue, leading-aligned |
| Actions | 拒绝 `ham_red` / 接受 `ham_blue` | iOS labels 退出Ham / 同意 |
| Policy URL | CCKV `DocHost` + `/privacy/`, fallback `https://whu-ham.github.io` | iOS hardcodes it |

**Strings:** 隐私协议 · 您需要同意隐私协议才能继续使用Ham。 ·
Ham不会在未经允许的情况下收集您的任何个人信息。但根据《中华人民共和国个人信息保护法》规定，任何应用都需要隐私协议才可提供服务。 · 总的来说，Ham会获取下列信息： · [必须] 设备标识，用于推送服务、统计数据与记录崩溃数据 · [可选] 查给分时上传的成绩 · 查看隐私协议 · 拒绝 · 接受. **States:** agreed / not-agreed.
> Div: iOS hardcodes all 12 strings; Android centralises them. iOS shows the `[必须]`/`[可选]` list,
> Android omits it. **URL inconsistency:** `whu-ham.github.io` (iOS, Android `PrivacyView`) vs
> `orangeboychen.github.io/whu-ham/privacy/` (Android `feature/auth`).

#### Version changelog (更新日志)
**Purpose:** Post-update "what's new" sheet, shown once after an update. **Entry:** app-level sheet on
first launch after an update. **Platforms:** both.

    HamSheet 85 % · padTop 32 · padBottom = nav-bar height · 📄 / ✧ 72 ham_blue · 更新日志 28 Bold
      ┌ pad 16 ────────────────────────────────────┐
      │ 1.2.3 22 Bold      2026-01-31 12 secondary   gap 8, bottom-aligned │
      │ <changelog title> 16 Bold · <content, markdown> 17 · scrollable │
      │ (48)  ┌ 好 · fillMaxWidth · accent@0.15 · r12 · pad v16 ┐ → dismiss+read │

**Blocks:** header icon 72 `ham_blue` · title · version + date row (hide the date when
`updateTime <= 0`) · changelog title · changelog content, markdown, scrollable · confirm → dismiss and mark read.

| Element | Value | Note |
| --- | --- | --- |
| Header icon / colour | 72 / `ham_blue` | iOS `wand.and.stars` 56, `.blue` |
| Title | 28 Bold | iOS `.title.bold()`; Android 24 Bold |
| Version / date | 22 Bold / 12 secondary | iOS shows neither |
| Body container | pad 16, scrollable, no fill | iOS `gray @0.1` r10, pad 12 |
| Button gap / fill / text / radius / pad | 48 / `accent @0.15` / accent / 12 / v16 | iOS `Spacer()`, opaque blue + white text, r10 |
| Sheet top / bottom padding | 32 / nav-bar height | iOS 16 |

**Strings:** 更新日志 · 未知版本 · 好 · iOS 本次更新日志 / 去使用. **States:** render nothing when the
changelog body is null. No loading or error state.
> Div: iOS shows a raw markdown string with no version/date and always renders (empty → empty grey box).
> Android also has a separate `VersionUpdateSheet` prompt, distinct from this sheet.

#### Webview screens
**Purpose:** Host web content in three forms — in-app page, full-text reader, external browser.
**Platforms:** both (the full-text reader is iOS-only today).

    navTitle = <page title | 加载中>   ·   inline display mode
    ▬▬▬▬ progress bar (required) ▬▬▬▬   ·   2, ham_blue
    [WebView fillMaxSize]

**Blocks:** nav title — the reported page title, 加载中 while loading, or the supplied fixed title ·
progress bar — **required, absent on both today** · webview — fillMaxSize, JavaScript on, JS-can-open-windows on, DOM storage on, load images on, media playback without user gesture, background `ham_bg_b1`, force-dark by system theme, back/forward swipe gestures on · error state with retry —
**required, absent on both today**.

| Element | Value | Note |
| --- | --- | --- |
| Background / force dark | `ham_bg_b1` / follow system theme | iOS sets neither (white flash) |
| Cookies / cache policy | cleared on create for CAS/education flows, `.nonPersistent()` for CAS login / `.returnCacheDataElseLoad` on update | Android uses the default persistent store and has no update reload |
| Progress bar / error-retry | 2 `ham_blue` / inline error + retry | absent on both |

**Full-text reader:** `VStack` spacing 4, padding 16, `ScrollView`, top-leading — title `headline`
2-line max; date `body` 17 `text_secondary` 1-line; gap 4; markdown content; bottom-trailing 查看详情 ↗ in accent at offset (−16, −4). Render title and date only when non-empty. **Android lacks this screen.**
**External links:** iOS uses an in-app `SFSafariViewController`; **Android must use a Custom Tab** (today a
raw `ACTION_VIEW` hand-off). **Strings:** 加载中. **States:** loading / loaded / error (retry).
> Div: Android has one themed, centrally-configured `WebViewCompose`; iOS builds ad-hoc `WKWebView`s per
> screen, and its `InnerWebView` back/close toolbar is commented out.

#### React Native container screens
**Purpose:** Host RN-implemented screens in a native container. **Platforms:** both.
**Modules:** `RNCommon` (headless bootstrap, app root) · `RNCasMobileLogin` · `RNFetchCourseView` ·
`RNFetchScoreView` · `RNScoreCalcView`.

    [RN root view fillMaxSize]
     loading overlay: spinner 20/2 ham_gray + label   REQUIRED
     failure overlay: Error view + retry              REQUIRED

**Blocks:** container fillMaxSize, no styling of its own · loading overlay — **required**; today iOS
renders an empty `ZStack` and Android an empty composable · error overlay — **required**; no error boundary exists on either platform, so a failed bundle load is a silent blank screen · remote kill-switch — gate each module on CCKV `rnComponentConfig` and fall back to the native path (e.g. the native captcha view) when disabled or when RN fails to load.

**Values:** fillMaxSize only; iOS `ScoreJsCalcView` also ignores the top and bottom safe areas.
**Strings:** 正在更新 · 更新成功 · 更新失败 · 选择计算方式.
**States:** loading (spinner + label) / loaded / error (retry) / RN-disabled (native fallback).
> Div: iOS initialises RN synchronously (async variant available) with a prebuilt `main.jsbundle` +
> HotUpdater OTA; Android initialises on `Dispatchers.IO` and dereferences `delegate.reactRootView!!`.

#### Image crop (裁剪图片)
**Purpose:** Crop/rotate an image to a fixed aspect ratio before upload. **Platforms:** both.

    ┌ 返回 · Color.secondary bar · 完成 ─────────────────────┐
    │  ↻ ⟷ ↕ ✂  options menu: rotate / flip-H / flip-V / crop │
    │ ┌ CropImageView, guidelines ON, fixed (ratioX, ratioY) ─┐ │
    │ │ frame stroke white 2 · surround masked black @0.5 eoFill │ │
    │ └──── full screen, safe area ignored ──────────────────┘ │

**Blocks:** toolbar — 返回 (cancel) leading, 完成 (confirm) trailing · options menu — rotate +90, flip
horizontal, flip vertical, crop (**iOS ships only reset — adopt all four**) · crop surface · confirm → crop to the frame and return the result.

| Element | Value | Note |
| --- | --- | --- |
| Guidelines | ON | iOS Mantis default; the legacy iOS cropper has none |
| Mask / frame stroke | black @0.5 eoFill antialiased, hit-test off / white 2 | |
| Ratio / shape | fixed from caller `ratioX`/`ratioY` / rect, circle for avatars | iOS Mantis multi-preset; Android rect only |
| Zoom | clamp [0.01, 2]; tap-to-zoom +0.1 per tap; drag clamped to ±(imageSize·scale − cropSize)/2 | |
| Output / failure | PNG quality 100 to a temp file → URI; iOS returns `UIImage` / error toast | iOS delegate methods empty; Android swallows IO exceptions |

**Strings:** 返回 · 完成 · `image_crop_title`. **States:** loading (show an indicator — Android's async
image load has none) / error (toast) / missing params (cancel immediately).
> Div: different third-party croppers (Mantis vs CanHub). Android is an XML `Activity` while the rest of
> the app is Compose; iOS has two competing implementations, one hand-rolled with debug `print`s.

#### Share sheet (分享)
**Purpose:** Offer QQ share and the system share sheet for a URL. **Entry:** `.ham_share` notification
with `type == "url"`. **Platforms:** iOS (**add an Android equivalent**).

    ┌░ scrim black @0.5, tap → dismiss ░────────────────────┐
    │ ┌ surface r16 top corners, pad 16 + 56 bottom ──────┐ │
    │ │ 分享到 17                                          │ │
    │ │ ┌────┐ ┌────┐  56 circles: QQ #12b7f5 · system gray@0.25 │ │
    │ │ │ QQ │ │ ⬆  │  glyph 24, QQ inner pad 8             │ │
    │ └────────── .move(edge: .bottom) ────────────────────┘ │

**Blocks:** scrim black @0.5, ignores safe area, tap dismisses · sheet — surface fill, r16 top corners
only, pad 16 + 56 bottom, leading-aligned · title 分享到 17 · QQ — 56 circle `#12b7f5`, asset `QQ-2` fitted with 8 inner padding, **render only when `canOpenURL("mqq://")`** · system — 56 circle `gray @0.25`, glyph `square.and.arrow.up.fill` 24 in `text_primary` · dismiss after either.

| Element | Value | Note |
| --- | --- | --- |
| Scrim / fill / radius / padding | black @0.5 / `surface.primary` (adaptive) / 16 top / 16 + 56 bottom | **iOS hardcodes `Color.white` — broken in dark mode** |
| Icon container | 56 circle | |
| QQ fill / glyph | `#12b7f5` / `QQ-2`, 8 inner padding | |
| System fill / glyph | `gray @0.25` / `square.and.arrow.up.fill` 24 | glyph colour not adaptive either |

**Strings:** 分享到. **States:** shown / hidden. No loading, error, or empty state.

#### Scan code (扫一扫)
**Purpose:** Continuous barcode / QR capture for login, palette sharing, and general scanning.
**Entry:** scan button or the 扫码 route. **Platforms:** both.

    ▓ ham_black, safe area ignored · toolbar transparent, action-bar area ignored ▓
            <title> 17 Bold white  ·  24 above the scan-window top
     │ HMS ScanKit preview fillMaxSize · ▂▂▂▂ sweep band ▂▂▂▂ · scan frame 300 tall, centred │
     │ ● 32 ham_blue at the result │  [⚡ torch 44] [🖼 album 44] 24/16 │

**Blocks:** preview full-bleed, `ham_black`, safe area ignored, continuous scan on — results arrive by
delegate so **consumers must dedupe** · mask/window/reticle centred, window 300 tall (`top = (screenHeight − 300)/2`) · hint caption, caller-supplied · sweep band · torch bottom-left and album bottom-right, both 44×44, glyph 24, inset 16 · permission denied → icon + 相机权限未开启 + 请前往“设置”开启 + 去设置 · accept only `ham://` payloads, toast otherwise.

| Element | Value | Note |
| --- | --- | --- |
| Background | `ham_black` alpha 1 | |
| Scan frame | 300 tall, centred, full width | iOS sets none; **Android's `right` comes from `screenHeight` — copy-paste bug** |
| Sweep / loop | fillMaxWidth × 300; gradient `Color.blue` @0.9/0.7/0.5/0 from centre, base 200×200, x-scaled `(screenWidth−32)/200`, y-scaled 0.2, clipped 20 tall; 1 s fade-in → 3 s sweep (travel 450) → 2 s hold → 1 s fade-out → 1 s gap | Android uses HMS `ScanDrawable` |
| Result marker | 32 `ham_blue` circle at the detected centre | iOS none |
| Toolbar | transparent, action-bar area ignored, back hidden, title 17 Bold white | |
| Torch / album | 44×44, glyph 24, inset 16 | **iOS omits both** |
| Startup / lifecycle | `onPause(); onStop(); delay(100); onResume(); onStart()` / map host lifecycle to `onStart…onDestroy`, pause off-route | Android only |
| Unsupported device | no ARM ABI → titled screen with a 16-padded message | iOS has no fallback |
| Permission toast | bg #FF0000, fg white, 12 caption, 2-line max, full width, dismiss 3.3 s | |

**Strings:** 扫描登录二维码 · 扫描配色分享二维码 · 相机权限未开启 · 请前往“设置”开启 · 去设置 · 相册.
**States:** authorized / permission undetermined / denied / scan success / unrecognised payload.

##### Divergences

- **Dead iOS components** — `Banner`, `TextEdit`, `TextEditorApproach`, `SegmentedPicker`, `NotificationService` have zero production call sites. Do not spec them as live; revive or delete.
- **No shared empty-state component on either platform** — every module authors its own.
- **No shared full-screen loading screen on iOS** — bare `ProgressView()` at ~30 sites; only the Android print module pairs a spinner with a text label.
- **Success/error icon colour semantics are inverted** — iOS Success green / Error red; Android Success `ham_blue` with green reserved for the toast. Same inversion in QR login.
- **The shared "accent pill" has a triple standard** — iOS Error/Success r12 vs r8, QR-login r8 / pad 16; Android r12 / 12 or 16 vertical. iOS also caps width at 350 where Android uses `fillMaxWidth`.
- **Two competing radius families** — 8 vs 12 recurs in the intro card, error/success buttons, text field, changelog and login social buttons.
- **iOS strings are hardcoded** in `PrivacyView` (all 12), `SuccessView`'s 返回, `ShareView`'s 分享到, image-crop 返回/完成, all five `QrCodeLoginView` strings, all `DebugView` strings, `InnerWebView`'s 加载中. Android centralises everything in `strings.xml`.
- **Copy bug** — `cas_bus_success_message` reads 你可以开始使用图书馆了 (library) on the **bus** success screen (`feature/cas/.../strings.xml:23`).
- **Privacy URL inconsistency** — `whu-ham.github.io` vs `orangeboychen.github.io/whu-ham/privacy/`.
- **RN containers have no loading, error, or fallback UI on either platform.** iOS mitigates with a CCKV kill-switch plus a native captcha fallback; no Android equivalent found.
- **Dark-mode bug in the iOS share sheet** — `Color.white` background, non-adaptive foreground.
- **Same name, different feature** — "Automatic" is Siri Shortcuts on iOS and a scheduled library-booking alarm on Android; "WebView" is per-screen ad-hoc `WKWebView` on iOS and one themed `WebViewCompose` on Android; "CAS login" has a native webview fallback on iOS (CCKV-gated) but appears RN-only on Android.
- **Two AI-generated, near-identical Debug screens** agree on copy and structure but differ in animation, spacing, field style, IME configuration and build gating (iOS `#if DEBUG`, Android ungated).
- **Platform-only surfaces** — iOS-only: Banner/TextEdit/TextEditorApproach/SegmentedPicker (all dead), share sheet, Siri/Automatic, authorized apps, passkey config, social-account binding, full-text reader. Android-only: settings hub, language settings, widget settings, print flow, `HamTextField`, `HamHorizontalPicker`, `HamDivider`, `LoadingModalView`, `HamLoadingProgressBar`, `VersionUpdateSheet`.


---

## 5. Shared flows

Four sequences recur across modules. They describe **order and state transitions**; the visual
spec for each screen involved lives in [§4.9 Shared screens](#shared-screens).

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
**Screen spec:** [Intro / connect screen](#intro--connect-screen-连接页).

### 5.2 Captcha

The education and sport flows route a CAPTCHA through a bundled local HTML page
(`education-captcha-page.html`, `sport-captcha-page.html`) and receive the token back through a
platform bridge. Include a 刷新 action. **Screen spec:** [Webview screens](#webview-screens).

### 5.3 Sign-in

A shared login screen offers the available providers. Brand-coloured social buttons use their
own colours, not the app palette. **Screen spec:** see the login screen under
[§4.8 My and user center](#my-and-user-center).

### 5.4 Web content

Remote-config banners and announcements render three ways: an in-app webview push, an external
browser, or a full-text view rendered from a string. Pick by the config entry's action type.
**Screen spec:** [Webview screens](#webview-screens).

---

## 6. Platform rules

These **must** differ and are not defects:

| Area | iOS | Android |
| --- | --- | --- |
| Status bar / safe areas | safe-area insets | `statusBarsPadding()` |
| Back navigation | edge swipe | system back |
| Navigation chrome | `UINavigationController` | in-Compose header |
| Native controls | SwiftUI `Toggle` / `Slider` / `ProgressView` | Material3 |
| System pickers | platform-supplied | platform-supplied |
| Widget configuration | none — WidgetKit owns refresh | in-app update-interval picker |
| Language | follows system locale | in-app locale picker |
| Type scaling | Dynamic Type | sp |
| Dark mode | asset catalogs | `values-night` |

Specify the **result** (content begins below a 42-tall header plus the status bar), not the
implementation.

---

## 7. Undecided

Needs a maintainer call before implementation:

1. **Body 17 vs 16.** iOS-as-baseline says 17; Android's 16sp is tokenised across 240 call
   sites, and 10 of iOS's own raw size declarations are already at 16.
2. **Chevron size.** Spec says `icon.xs` (12). iOS currently writes 8 in 22 places, Android
   uses the Material default 24.
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
