# Ham design system

How to build the Ham native apps. This is a specification, not a comparison — read it as
"build it this way". Values are measured from the existing code; where the two clients
disagreed, **iOS is the baseline**.

Current divergences between the shipped clients are recorded in
[§8 Current divergences](#8-current-divergences) at the end, so this document stays readable
as a spec.

## Contents

- [1. Overview](#1-overview)
- [2. Foundations](#2-foundations)
- [3. Components](#3-components)
- [4. Screens](#4-screens)
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

### 4.1 状态 — Status dashboard

**Purpose:** one scrolling column of live module cards, ranked by relevance.
**Entry:** default tab.
**Module colour:** none of its own — each card carries its module's brand colour.

**Layout:**

```
┌─────────────────────────────────────────────┐
│  状态                          ← large title│  34 / Bold
│  (daily photo behind, 200 tall)             │
├─────────────────────────────────────────────┤
│  ┌───────────────────────────────────────┐  │
│  │▓ 天气                              ▸ ▓│  │  #FF9500
│  └───────────────────────────────────────┘  │
│  ┌───────────────────────────────────────┐  │
│  │▓ 图书馆                            ▸ ▓│  │  #007AFF
│  └───────────────────────────────────────┘  │
│  ┌───────────────────────────────────────┐  │
│  │▓ 课程                                ▓│  │  #1B5E20
│  └───────────────────────────────────────┘  │
│                  ⋮                          │
└─────────────────────────────────────────────┘
```

**Top:** large title 状态 at 34 / Bold over a daily photo banner 200 tall. On scroll the title
collapses into a condensed nav-bar title. Title colour flips black/white by sampling the photo's
brightness.

**Cards, in relevance order.** Each card publishes a score; cards sort descending and hide when
negative. Order changes are debounced ~5s to stop jitter.

| Card | Colour | Contains | Shows when |
| --- | --- | --- | --- |
| CAS alert | `feedback.error` | 信息门户登录失败 + 重新登录 | CAS session invalid |
| 天气 | `#FF9500` | Temperature 36 rounded, description, 7-day forecast grid | always |
| 图书馆 | `#007AFF` | One card per active booking: status, seat number large, location, time range, countdown | has a booking |
| 课程 | `#1B5E20` | Next-class tip, progress bar if in class, upcoming rows, 周视图/日视图 toggle | has courses |
| 日程 | `#01579B` | Pending count, nearest item hero with large countdown, next 3 rows | has schedules |
| 校车 | `#A2845E` | Nearest stop, distance, collapsible per-line rows | CAS logged in, within range |
| 运动 | `#34C759` | One card per booking: status, area badge, type, time, pay-by deadline + 去支付 | has a booking |

**States:** there is no screen-level empty, loading, or error state. Each card handles its own
inline — a `ProgressView` while loading, a red strip with 重试 on failure, a text message when
empty. With every card hidden, the page renders title + photo + background only.

**Refresh:** pull to refresh, plus refresh on returning to the foreground.

**Navigates to:** each module's home, plus sport pay and CAS settings.

**Note:** the CAS alert card is **not** a status card. It is a solid `feedback.error` pill with
white text, pinned first.

---

### 4.2 课程表 — Course timetable

**Purpose:** a week-by-week grid of class periods.
**Entry:** tab 1, or the function-grid tile.
**Module colour:** `#1B5E20` — identity only. All controls here use `accent`.

The screen follows the standard rule: **every interactive element here is `accent` blue** —
前往本周, the week picker, the today highlight, selected chips, 添加, 编辑. Course green appears
only in the three identity places: the 状态 course card, the function-grid tile, and the
获取课程 intro. This is correct and intentional, not an oversight.

**Layout — fixed grid, not scrolling cards. The week changes in place.**

```
┌────┬──────────────────────────────────────┐
│    │  一     二     三     四     五      │  ← weekday header, 36 tall
│    │ 9-15   9-16  9-17   9-18  9-19      │     today = surface.tint fill + accent
├────┼──────────────────────────────────────┤
│  1 │ ┌────┐ ┌────┐                       │
│  2 │ │高清│ │线代│                       │  ← grid cells
│  3 │ │-教三│ │-教四│                       │     radius 10, padding 2
│  4 │ └────┘ └────┘                       │
│  ⋮ │                                      │
│ 13 │                                      │
└────┴──────────────────────────────────────┘
 42   ↑ period gutter, 13 rows, tap to toggle
      period numbers ⇄ start/end times
```

| Element | Value |
| --- | --- |
| Week selector bar | status bar + 50 tall |
| Weekday header | 36 tall |
| Period gutter width | 42 |
| Period rows | 13 |
| Columns | 5 or 7, by the 显示周末 setting |
| Grid cell radius | 10 |
| Grid cell padding | 2 |
| Grid gutter | 2 |
| Bottom spacing | tab-root rule |

**Week selector** (centre of the top bar):

| State | Line 1 | Line 2 |
| --- | --- | --- |
| Viewing current week | 本周 | 第N周 |
| Viewing another week | 第N周 | 前往本周 button |
| Before term starts | 放假中 | 前往第一周 button |

**Navigation:** `‹` / `›` buttons, plus horizontal swipe on the grid. Tapping the week label
opens a week picker (1–20).

**Grid cell — course:** three lines, top-leading: course name (bold), instructor, classroom.
Cell fill resolves grid colour → course colour → a deterministic hash of the course ID into an
18-colour pastel palette; text colour auto-contrasts. Course blocks span multiple rows for
consecutive periods.

**Interactions:**

| Gesture | Action |
| --- | --- |
| Tap a course | Opens the course detail |
| Tap an empty cell | Context menu: 添加, 粘贴 |
| Long-press a course | Context menu: 编辑, 复制, 剪切, 删除… |
| Drag after long-press | Moves the block to another empty cell |
| Tap the period gutter | Toggles numbers ⇄ times |

**Course detail:** info card (colour dot, name, credit, instructor, location) → 给分 card
(gated on a remote flag and on the course not being user-created) → 关联日程 card (future rows
then past rows). Top-trailing actions: edit, add schedule, dismiss.

**Background image:** optional user-set photo behind the grid, with two opacity sliders
(背景 0–0.85, 课程 0.15–1). Grid cells fade toward transparent when a background is set.

**Empty state:** when no term start date is configured, replace the grid with a centred block:
title + 请设置开学日期后再使用课程表. The top bar still renders.

---

### 4.3 日程 — Schedule

**Purpose:** a to-do list of schedules with countdowns, filterable by group.
**Entry:** the function-grid tile, or the 状态 schedule card.
**Module colour:** `#01579B` — identity only. All controls here use `accent`.

**Layout:** hero card on top, pinned group tab bar, scrolling list below.

| Element | Value |
| --- | --- |
| Hero card | radius 16, `surface.secondary`, padding 16 |
| Group tab bar | 60 tall, horizontally scrolling |
| Countdown (hero) | 50 / Bold, white, on a `#01579B` panel |
| Countdown (row) | 28 / Bold |
| Countdown (detail) | 72 / Bold |
| List row card | radius 16, `surface.secondary` |
| Row spacing | 16 |

**Content:**

1. **Hero card** — the nearest upcoming item: name, begin time, location, related course; and a
   large countdown on a `#01579B` panel. The panel turns `#FF9500` once the item has passed.
   Hides on scroll when the list is long.
2. **Group tab bar** — a leading 全部 chip, then one chip per group with a count badge.
   Selected chip is marked by a 50 × 6 rounded underline that slides between chips. A trailing
   chevron opens the group manager.
3. **Group manager** — drops down over a scrim. Header: + (new group) and 编辑位置. Body: one
   row per group with icon, name, count, and the group's first upcoming schedule.
4. **List** — future items first, ascending; a divider; then past items, descending. Each row:
   name, begin time, related course, and a right-aligned countdown.

**Countdown units:** below 1 minute floors to 1 分钟; then 分钟 → 小时 → 天.

**Detail:** a 300-wide card over a scrim. Header shows the countdown at 72 / Bold on the same
`#01579B` (or `#FF9500`) panel. Body: name, time range, location, note, related course.
Actions: edit and delete.

**Empty state:** 无日程 in the list; 无群组 in the group manager.

---

### 4.4 图书馆 — Library

**Purpose:** seat and room booking.
**Entry:** the function-grid tile, or the 状态 library card, or the `ham://library` deep link.
**Module colour:** `#007AFF` — identity only. All controls here use `accent`.

**Home layout — scrolling column, in this order:**

1. **Banner** — 200 tall, radius 16. A paged carousel: page 1 is always 图书馆公告 with a 72
   icon over a tiled watermark grid; further pages come from remote config and link to web
   content or a full-text view. 5s auto-advance, page dots when there is more than one page.
2. **Retry-login card** — solid `feedback.error`, white text, 登录信息过期 + 点击重新登录.
   Only when the session token has expired.
3. **Analytics card** — gated on two flags (a remote flag **and** a local opt-in). Two states:
   a consent prompt (同意 / 不同意), and once consented, one row per request group with a bar
   strip (orange below 50%, green above) and a success rate.
4. **Current-booking cards** — one per active booking. A full-width status band, then the seat
   number large, location, and time range. Expanding reveals 刷新, 在地图打开, 变更或取消预约.
5. **Divider** — only when there is a booking.
6. **Quick-book card** — only when a seat resolves (preferred seat wins, else last booking).
   Seat row → seat picker; time row → time picker; then a full-width 预约 button.
7. **Function card** — 150 tall, two columns. Left, full height: 查看房间 with a 64 icon.
   Right, two stacked halves: 历史预约 and 设置, each with a 32 trailing icon.
8. **Print card** — 打印 / 在图书馆公共打印机打印. Full width, below the function card.

**Other screens:**

| Screen | Purpose | Primary action |
| --- | --- | --- |
| 连接图书馆 | CAS verification gate, shown when not authenticated | Complete verification |
| 查看房间 | Pick a building, room, and time; browse the seat grid | 预约 |
| 选择座位 | Reusable seat picker returning a seat | 选择 |
| 快速预约 | Fires the booking automatically on entry | none (automatic) |
| 变更预约 | Change time, or swipe to cancel | 确定 / swipe |
| 历史预约 | Read-only grouped list | none |
| 设置 | Account, captcha, preferred seat, analytics, local data | none |
| 首选座位 | Seat + time + weekday rules, with 电源 / 靠窗 preferences | Add a rule |
| 图书馆公告 | Renders the notice HTML | none |
| 打印 | Pick a file, choose options, submit to a campus printer | 提交任务 |

**States:** empty blocks are omitted rather than shown as placeholders. Errors surface as a
toast, plus the red retry-login card on token expiry.

---

### 4.5 运动 — Sport

**Purpose:** sports venue booking.
**Entry:** the function-grid tile, or the 状态 sport card.
**Module colour:** `#34C759` — identity only. All controls here use `accent`.

**Home layout — scrolling column, in this order:**

1. **Banner** — 200 tall, radius 16. Page 1 is always 场馆预定公告 with a 72 icon over a tiled
   ball grid → the bulletin list. Further pages are bulletins: title, date, excerpt, 查看详情.
   5s auto-advance.
2. **Current-order cards** — one per booking. A blue status strip, an area badge, type and
   venue, time range; when unpaid, a divider plus either a red 请于X前完成支付 or
   当前未处于可支付时间, and a 去支付 button.
3. **Divider** — only when there is an order.
4. **Quick-order card** — only when a venue resolves (starred venue, else last order). Venue
   row with an orange 收藏 pill when it matches the starred venue; a 今天/明天 segmented
   control; a full-width 预定 button.
5. **Function card** — 150 tall, two columns. Left, full height: 查看场馆 with a 64 icon.
   Right, two stacked halves: 订单中心 and 设置, each with a 32 trailing icon.

**The area/order screen** is the core transaction:

- **Pinned header:** sport-type chip strip, and a date row defaulting to tomorrow after 18:00.
- **Scrolling body:** one collapsible card per venue — image 85 square, title, address, red
  已闭馆 when closed, `¥{lowPrice}起`, a collapsed time-summary strip, and when expanded one
  row per court expanding to a slot grid. Slot tile: selected → start/end + 已选择 in white on
  green; unselected → start/end, price, remaining capacity, and a 预定 strip.
- **Pinned footer:** the current selection and the 预定 button → captcha → create order.

**Other screens:** 连接体育场所预定 (CAS gate), 选择 (returns a selection instead of ordering),
输入验证码, 支付, 快速预约, 收藏预定设置, 订单中心 (a webview), 公告 list and detail, 运动设置,
预定成功, 预定失败.

**States:** unload (form) / loading (centred spinner) / error (with 返回) / success.

---

### 4.6 成绩 — Score

**Purpose:** grades, GPA, and F2 calculation.
**Entry:** the function-grid tile. There is no score card on 状态.
**Module colour:** `#FF9500` — identity only. All controls here use `accent`.

**Home layout — scrolling column, in this order:**

1. **Stat card** — 成绩概览, GPA as a hero value, average score, then a divider and one row
   per academic year (大一…大五). Crown watermark at 128 @ 0.12.
2. **Function card** — three equal buttons: 获取成绩, 选择, 设置.
3. **Semester cards** — one per term, sorted descending. Header: `{year}-{year+1} 第N学期`,
   average and GPA; trailing control is a collapse chevron normally, a 全选/全不选 toggle in
   select mode. Body: score rows, 8 apart.
4. **Score row** — a 6 × 28 colour bar, then two lines (name + instructor on the first,
   course type + credit on the second), then the score at 22 / Bold. `--` when absent,
   strikethrough when excluded from the F2 selection. Trailing: a comment button normally, a
   checkbox in select mode.
5. **Pinned selection header** — appears only in select mode: 综测成绩, 平均成绩, GPA, credit
   breakdown, and a 退出 button.
6. **Privacy blur overlay** — a full-screen blur applied when the app backgrounds, so grades
   are hidden in the app switcher.

**Other screens:** 获取成绩 (update sheet: CAS login → captcha → 正在更新 → 获取成功/失败),
成绩设置, 成绩启用, 连接成绩 intro, Face ID enable, JS calc picker (an RN marketplace) and
detail, Face ID error.

---

### 4.7 课程评分 — CourseScore

**Purpose:** course reviews and grade distributions.
**Entry:** the function-grid tile, or the comment button on a score row.
**Module colour:** `#283593` — identity only. All controls here use `accent`.

| Screen | Purpose |
| --- | --- |
| Search home | Search bar, search history card, external-service card, and a 我的数据 entry |
| Search results | Ranked matches with filter chips |
| Course detail | Rating, grade distribution, comments, my review |
| 我的数据 / Course center | Three tabs: 我的评价, 想上, 排行榜 |
| Create review | Star rating + comment |
| Comments | Full comment thread |

**Course detail** is the densest screen: a rate card (overall score as a hero value), a
grade-distribution card with one weighted bar per band, a function card, the user's own review
card if any, and the comment thread.

---

### 4.8 我的 — My

**Purpose:** account hub and navigation.
**Entry:** tab 3.
**Module colour:** none — the page is neutral; each function tile carries its own brand colour.

**Layout — scrolling column:**

1. **Collapsing header** — a background image with a large title that shrinks into the nav bar
   on scroll. The title comes from remote config; when the user is signed in it is replaced by
   their avatar and nickname, falling back to 未登录.
2. **User-center card** — shown when the account feature is enabled. Avatar, nickname, and the
   sign-in / account action.
3. **Function grid** — the module shortcuts from [§1.2](#12-navigation-hub-the-function-grid).
   Horizontally scrolling, each tile tinted with its module colour.
4. **Board card** — remote-config announcement with markdown content. Only when configured.
5. **Promotion card** — remote-config promotional entries.
6. **Settings card** — a flat list: 自动化 / 添加Siri捷径, 使用指南, 反馈, 关于, plus a
   debug entry in debug builds.

**Settings hub** (a pushed screen, not the card): 小组件, 自动化操作, 语言, 关于.

**About:** logo, version, changelog, and actions (前往 App Store, 复制课程信息, 来 Github 找我,
分享日志).

---

## 5. Shared flows

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

### 5.2 Captcha

The education and sport flows route a CAPTCHA through a bundled local HTML page
(`education-captcha-page.html`, `sport-captcha-page.html`) and receive the token back through a
platform bridge. Include a 刷新 action.

### 5.3 Sign-in

A shared login screen offers the available providers. Brand-coloured social buttons use their
own colours, not the app palette.

### 5.4 Web content

Remote-config banners and announcements render three ways: an in-app webview push, an external
browser, or a full-text view rendered from a string. Pick by the config entry's action type.

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
- [ ] Every colour has a light and a dark value and is an adaptive resource.
- [ ] Text styles are explicit — no relying on an inherited default size.
- [ ] Interactive elements meet the 44 minimum tap target.
- [ ] `maxLines` is paired with `Ellipsis` on Android.
- [ ] The same screen exists on the other platform, or the divergence is listed in
      [§6](#6-platform-rules) or [§8](#8-current-divergences).
- [ ] Verified in light and dark mode.
- [ ] Verified with a populated account — empty states hide spacing and type differences.
