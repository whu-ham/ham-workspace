# Ham screen catalog

Per-screen specification for the native apps. This is the companion to
[`design-system.md`](design-system.md) — that document defines the tokens and components,
this one shows how each screen is assembled from them.

Every screen is measured from source, on both platforms. Status: **proposed** — the values
here are the target, not necessarily what ships today.

## How to read a screen section

Each screen section has the same eight fields:

| Field | Meaning |
| --- | --- |
| **Purpose** | What the screen is for, in one line |
| **Entry** | How the user gets here |
| **Layout** | ASCII diagram with region heights; pinned regions are marked |
| **Blocks** | Ordered content blocks, each with its condition and tap behaviour |
| **Values** | Element-by-element table: **iOS** and **Android** columns are *evidence*, the **normative** column is the requirement |
| **Strings** | The important Chinese labels and CTAs |
| **States** | Empty, loading, error |
| **Divergence** | Where the two clients differ today — a migration task, not part of the spec |

Two conventions worth knowing:

- **Where the platforms drift and neither value is functionally required, the normative
  value takes the tokenised or explicit one.** Where one platform omits a treatment the
  other has, the platform that has it wins.
- **Screen counts differ per platform.** A screen marked `platforms: iOS only` is a real
  gap on Android, and vice versa.

## Units and tokens

iOS **pt** and Android **dp** are 1:1, as are **pt** and **sp** for type. Tokens are defined
in [`design-system.md`](design-system.md) §2:

| Group | Values |
| --- | --- |
| radius | `1` 2 · `2` 4 · `3` 6 · `4` 8 · `5` 10 · `6` 12 · `card` 16 |
| spacing | `1` 2 · `2` 4 · `3` 8 · `4` 12 · `5` 16 · `6` 24 · `7` 32 |
| icons | `xs` 12 · `sm` 20 · `md` 24 · `lg` 32 · `xl` 64 · `hero` 72 · `watermark` 128 · plus the named containers `plate` 40 · `badge` 32 · `tile` 64 · `empty` 64 · `banner` 72 |
| type | `largeTitle` 34 · `title` 28 · `title2` 22 · `title3` 20 · `headline` 17 · `body` 17 · `bodyBold` 17 · `callout` 16 · `subheadline` 15 · `footnote` 13 · `caption` 12 · `captionBold` 12 · `caption2` 11 |
| surface | `primary` #F9F9F9 · `secondary` #FFFFFF · `tertiary` #EDEEEF · `tint` #E6F1FF |
| text | `primary` · `secondary` #8E8E93 · `tertiary` @60% · `link` · `danger` #FF3B30 |
| accent | #007AFF — the app-wide interactive colour, the same on every module |
| brand | `course` #1B5E20 · `schedule` #01579B · `library` #007AFF · `sport` #34C759 · `score` #FF9500 · `coursescore` #283593 · `bus` #A2845E · `pay` #BF360C |

**Bold is the only emphasised weight.** Every icon is explicitly sized.

## Contents

1. [Status dashboard (状态 Status)](#1-status-dashboard-状态-status) — 10 screens
2. [Course timetable (课程表 Course)](#2-course-timetable-课程表-course) — 7 screens
3. [Schedule (日程 Schedule)](#3-schedule-日程-schedule) — 5 screens
4. [Library (图书馆 Library)](#4-library-图书馆-library) — 12 screens
5. [Sport (运动 Sport)](#5-sport-运动-sport) — 13 screens
6. [Score (成绩 Score)](#6-score-成绩-score) — 12 screens
7. [CourseScore (课程评分 CourseScore)](#7-coursescore-课程评分-coursescore) — 8 screens
8. [My tab (我的 My)](#8-my-tab-我的-my) — 10 screens
9. [User center (用户中心)](#9-user-center-用户中心) — 21 screens
10. [Auth and sign-in (登录与授权)](#10-auth-and-sign-in-登录与授权) — 8 screens
11. [Shared components (共享组件)](#11-shared-components-共享组件) — 15 screens
12. [Standalone screens (独立页面)](#12-standalone-screens-独立页面) — 8 screens

---


## 1. Status dashboard (状态 Status)

### 1.1 Page shell

```
├ daily photo      top-aligned · h 200 · aspect-fill · parallax −max(0,y) · stretch +max(0,−y)
├ surface.primary  page fill (set explicitly)
└ ScrollView
   ├ Spacer  statusBar + 28
   ├ large title   状态 · 34 / Bold · colour flips with photo luminance
   ├ Spacer  30
   ├ card column   h-pad 16 · gap 8 · reorder cross-fades
   └ Spacer  96 + safe-area inset
overlay: collapsed nav title (alpha ramp) · pull-to-refresh indicator
```

| Property | iOS | Android | Normative |
| --- | --- | --- | --- |
| Title text / type | `NOW`→`状态` `IOS/StatusView.swift:25`, `LS:9` / `.largeTitle.bold()` 34 `IOS/component/view/StatusTitleView.swift:18` | `status_title` `AOS/StatusView.kt:137`, `STR:4` / `32.sp` Bold `AOS/StatusContainerView.kt:106-107` | `状态` · **34 / Bold** |
| Top inset | `statusBar + 28` `StatusView.swift:24` | `64.dp` `StatusContainerView.kt:98` | `statusBar + 28` |
| Title → cards | `30` `StatusView.swift:26` | `10`+`32`=42 `StatusContainerView.kt:115`, `AOS/StatusView.kt:175` | **30** |
| Title colour | flips white/black by luminance `StatusView.swift:39-44` | fixed; hook never passed `StatusContainerView.kt:53-55,108` | **flips with luminance** |
| Collapsed title | `ToolbarItem(.title)`, iOS 26+, `scrollY > 7` `StatusView.swift:105,134-142` | custom bar, alpha `(1+(y−dY)/dY)`∈0…1, 20/Bold `StatusContainerView.kt:120-140` | **17 / Bold**, alpha ramp, platform chrome (`design-system.md` §6) |
| Top scrim | `ham_bg_b2` masked gradient, `statusBar+45`, iOS<26, iPad 400 `StatusView.swift:53-68` | solid `ham_bg_b1`@0.7 `StatusContainerView.kt:126` | `surface.primary`@0.7 · `statusBar+45` · all versions |
| Page background | `ham_bg_b1Color` `StatusView.swift:50` | **none** → `0xFFFFFBFE` `StatusContainerView.kt:72`, `AOSUI/common/ui/theme/HamTheme.kt:44` | **`surface.primary`** |
| Bottom | spacer `96` + 85 scrim @0.75 (iOS<26) `StatusView.swift:28,77-83` | `64.dp` + `HamBottomSpacer` `AOS/StatusView.kt:211-212`, `AOSUI/component/Spacer.kt:28-30` | spacer **96** + safe-area · **no scrim** |
| H-padding | `16` `StatusView.swift:30` | `16.dp` `AOS/StatusView.kt:176` | 16 |

**Daily photo** — base height **200** (iOS `200` `StatusView.swift:47`; Android `300.dp`
`StatusContainerView.kt:76`) · aspect-fill + 0.5 s cross-fade
(`IOS/component/view/StatusBackgroundView.swift:33-34`, `AOS/StatusView.kt:145-148`) · source =
server list, one random pick **per day**, sticky for the session: keep the cached URL while it is
still offered and wait **10 s** before swapping on a session's first load
(`IOS/component/vm/StatusBackgroundViewModel.swift:63-76`; Android re-randomises per process,
`AOS/StatusViewModel.kt:60`) · **not tappable** · failure is silent, no error chrome.

**Pull-to-refresh** — threshold **128** (iOS `IOS/component/view/StatusUpdateView.swift:15,57`;
Android `300f` px `StatusContainerView.kt:158,170`) · indicator is platform-native: iOS keeps the
pill (h 36, r 18, `surface.secondary`, shadow black@0.1 r5 y5, `arrow.clockwise`, `accent`
foreground, label `已请求刷新`, light haptic, 2 s hold then 0.3 s fade — `StatusUpdateView.swift:22-49,61`);
Android uses Material3 `PullToRefreshDefaults.Indicator` (`StatusContainerView.kt:179-183`) ·
action = fan out `onRequestUpdate()` to every card VM (`StatusView.swift:73-75`) **without
resetting scores first** · triggers = **pull-to-refresh + return-to-foreground** only
(`StatusView.swift:89-93`); **no page-level polling timer** (drop Android's 2-minute loop,
`AOS/StatusView.kt:121-126`; per-card cadence belongs to the card).

**Screen states:** none. No page-level empty, loading or error. With every card hidden the page
renders title + photo + background only.

### 1.2 Card scoring and ordering

| Property | iOS | Android | Normative |
| --- | --- | --- | --- |
| Card types | 7 `IOS/StatusContentViewModel.swift:25-33` | 5 `AOS/utils/StatusViewCardScoreManager.kt:25-31` | **7**: casAlert · weather · library · course · schedule · bus · sport |
| Invisible sentinel | separate `Set` `:61` | score `−1` `StatusViewCardScoreManager.kt:59` | **score `−1`** |
| Sort | `sorted { $0.value > $1.value }`, no tie-break `:154-160` | `sortedBy { -it.score }`, no tie-break `:87-97` | descending; **ties broken by declaration order** |
| Filter | invisible set removed `:154-160` | `filter { score >= 0 }` `:90` | `score >= 0` |
| Reset value | 999/5/4/3/2/1/0 `:52-60` | `0` for all `:80-85` | **0** for all |
| Debounce | fast 10 s / quiet 5 s `:65,186-211` | fast 10 s / quiet 5 s `:67-68,116-141` | first change, or >10 s since the last, applies immediately; else 5 s after the last |
| Persistence | `LocalStorageHelper .statusCardOrderCache` `:94-98` | none | **persisted** |
| Reorder animation | `withAnimation` `:94` | `AnimatedContent` fade `AOS/StatusView.kt:178-184` | cross-fade |
| CAS alert | score 999 but starts invisible `:53,61` | unscored, rendered first `AOS/StatusView.kt:189` | **unscored, pinned first** |

**Published scores** (normative; re-publish on every state change, the debouncer absorbs bursts)

| Card | Condition | Score | Cite |
| --- | --- | --- | --- |
| weather | loaded, commute window 06:00–09:00 / 13:00–14:00 / 16:30–18:30 | 120 | `IOS/card/weather/StatusWeatherCardViewModel.swift:160` |
| weather | loaded otherwise | 10 | `:162`; `AOS/component/weather/WeatherCardViewModel.kt:84` |
| weather | no location permission, no data | 100 | `…StatusWeatherCardViewModel.swift:80` |
| weather | fetch failed, no data | 60 | `:128` |
| library | active booking (reserve / checkIn / away) | 60 | `IOS/card/library/StatusLibraryCardViewModel.swift:96`; `AOS/component/library/LibraryCardViewModel.kt:67` |
| library | booking ended today | 20 | `StatusLibraryCardViewModel.swift:111` |
| library | request error | 30 | `:88` |
| course | week has courses | 50 | `IOS/card/course/StatusCourseCardViewModel.swift:136` |
| schedule | has an upcoming item / none | 60 / 5 | `IOS/card/schedule/StatusScheduleCardViewModel.swift:65` |
| bus | stop resolved | 2 | `IOS/StatusContentViewModel.swift:57` |
| sport | has an order | 60 | *new — see Divergence* |
| any | nothing to show | −1 | `StatusLibraryCardViewModel.swift:114`; `StatusCourseCardViewModel.swift:132-133`; `IOS/card/bus/StatusBusCardViewModel.swift:69,163-169` |

`Divergence:` Android publishes nothing for course (`AOS/component/course/CourseCardViewModel.kt:76`),
weather-on-failure (`WeatherCardViewModel.kt:84`) or bus
(`AOS/component/bus/StatusViewBusCardViewModel.kt:199-202`), so those cards sink instead of
surfacing; iOS sport never publishes (frozen at 1, `IOS/card/sport/StatusSportCardVM.swift`). Both
tie-breaks are unstable. Android's bus cut-off is 100 km vs iOS 20 km
(`AOS/component/bus/StatusViewBusCard.kt:70-72`).

### 1.3 Shared card container

```
┌──────────────────────────────────────────────┐
│▓ [icon] 图书馆                          ▸   ▓│ band: brand @0.15 · pad 12 · icon 24 · gap 4
├──────────────────────────────────────────────┤ title 17 / Bold, brand colour
│  <body>                          padding 12   │
└──────────────────────────────────────────────┘ radius 16 · surface.secondary · flat
```

| Property | iOS | Android | Normative |
| --- | --- | --- | --- |
| Radius / background | `16` `IOS/card/CommonStatusCard.swift:80` / `ham_bg_b2Color` `:79` | `12.dp` `AOS/component/CommonStatusCard.kt:49` / `ham_bg_b2` `:50` | **16** / `surface.secondary` |
| Band fill / padding | `color.opacity(0.15)` `:74` / `12` all `:71` | `color.copy(alpha=0.15f)` `:66` / v12 h16 `:67-68` | `brand`@0.15 / **12 all** |
| Band icon / title | `Image(systemName:)`, no size `:52` / `.bold()`, no size → 17 `:53-54` | `size(24.dp)`, tinted `:72-75` / `bodyBold` 16.sp `:76-78` | `icon.md` 24 / **17 / Bold**, both brand colour |
| Chevron | `chevron.right`, no size `:61,66` | `ChevronRight` in `HamButton` `:82-86` | `icon.xs` 12, brand; **only when the card navigates**; min 44 target |
| Body padding | `12` param `:22,76` | `16.dp` `:53` | **12**, overridable (bus passes 0) |
| Inter-card gap | `VStack` with no spacing `IOS/StatusView.swift:149` | `spacedBy(8.dp)` `AOS/StatusView.kt:187` | **8** |
| Size change | none | `animateContentSize()` `CommonStatusCard.kt:47` | animate content-size changes |

Band colours — library `IOSSH/Color+Ham.swift:36` · `AOSUI/common/ui/config/Color.kt:75`; course
#1B5E20 `Color+Ham.swift:41` · `Color.kt:90-91`; schedule #01579B `Color+Ham.swift:42` ·
`Color.kt:93-94`; bus #A2845E `Color+Ham.swift:40` · `Color.kt:87-88`; sport #34C759
`Color+Ham.swift:37` · `Color.kt:78-79`; weather #FF9500
`IOS/card/weather/StatusWeatherCard.swift:15` · `AOS/component/weather/WeatherCardView.kt:90`.

`Divergence:` iOS writes stock UIKit colours (`Color.blue` / `.brown` / `.green` / `.orange`) for
library, bus, sport and weather instead of the brand tokens — promote all four to named tokens.

### 1.4 CAS alert (信息门户登录失败)

Not a status card — a solid error pill pinned above the scored list.

```
┌──────────────────────────────────────────────┐
│ ⚠  信息门户登录失败                 [ 重新登录 ]│
└──────────────────────────────────────────────┘
   feedback.error fill · radius 12 · padding h12 v8 · all text white
```

**Blocks** — 1. **warning icon** (always; filled triangle-exclamation, `icon.md` 24 —
`IOS/card/cas/StatusCasAlertCard.swift:13`, `AOS/component/cas/CasErrorCardView.kt:53`) ·
2. **message** `信息门户登录失败` 17 / Regular (`:14`) · 3. **spacer** (`:18`) ·
4. **action pill** `重新登录` — 12 / Bold, `surface.secondary` text on a white@0.70 pill, r6,
pad h8 v4; **tappable** → CAS settings (`:19-24`, `CasErrorCardView.kt:45-48`).

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Fill / radius | `Color.red` / `12` `:29-30` | `ham_red` / `16` `CasErrorCardView.kt:44`, `AOSUI/common/ui/component/Card.kt:53` | `feedback.error` / **12** |
| Padding / icon→text gap | h8 / v8 `:27-28` / `4` `:12` | `16` `Card.kt:48` / `4.dp` `:50` | **h12 / v8** / 4 |
| Message | body, `.white` `:14,16` | `caption` 12.sp `:54` | **17 / Regular, white** |
| Tap target | pill only `:19` | whole card `:45-48` | **whole card** |

**Strings:** `信息门户登录失败` (`LS:556`) · `重新登录` (`LS:370`).
**States:** CAS valid → absent; CAS invalid → present. No loading or empty state.

`Divergence:` Android folds message and action into one string `请重新登录信息门户` (`STR:5`) with
no pill. iOS scopes `.white` to the outer `HStack` (`:16`), so the pill label is white-on-white —
a contrast defect. Android's icon has `contentDescription = null`.

### 1.5 Weather (天气)

```
┌──────────────────────────────────────────────┐
│☀ 天气                                        │ band brand.score @0.15 · no chevron
├──────────────────────────────────────────────┤
│┌────────────────────────────────────────────┐│ ← error only: feedback.error, r8, pad h12 v8
││ <error message>                   [ 重试 ] ││
│└────────────────────────────────────────────┘│
│  28.5°                                    ⟳ │ 36 / rounded · tabular
│  晴                                          │ 17 / Regular
│  武昌区 · 14:32                              │ 12 / Regular · text.secondary
│┌────────────────────────────────────────────┐│ surface.tertiary · r8 · pad h16 v12 · 5 rows
││ 今日  晴      ☀31.0°   ☾22.0°               ││ gap 12 · icons 12 · column gap 8
││ 明日  多云    ☀29.0°   ☾21.0°               ││
│└────────────────────────────────────────────┘│
│                              <attribution>    │ 11 / Regular · text.tertiary
└──────────────────────────────────────────────┘
```

**Blocks** — 1. **header band** (always; `sun.max.fill`, `天气`; **not tappable** —
`StatusWeatherCard.swift:15`, `WeatherCardView.kt:87-91`) · 2. **error banner** (on failure or
missing permission; message 17 / Regular white 1 line + `重试` chip; **tappable** → retry —
`:17-21`, `WeatherCardView.kt:113-145`) · 3. **loading indicator** (block spinner with no data;
inline spinner beside the temperature once data exists — `:24-29`, `:83-86`,
`WeatherCardView.kt:147-166`) · 4. **brief block** (when data exists; gap 2 — `:81-93`, `:158-197`) ·
5. **forecast grid** (when the list is non-empty; **5 rows**, 3 columns — `:100-137`,
`WeatherCardView.kt:201-285`; `.prefix(5)` / `.take(5)` at `StatusWeatherCardViewModel.swift:177`,
`WeatherRequestHelper.kt:78`) · 6. **attribution** (whenever the body renders — `StatusWeatherCard.swift:30-37`).

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Error fill / radius / message | `Color.red` `:66` / `8` `:67` / body white 1 line `:51-53` | `ham_red` `:119` / `8.dp` `:117` / Bold white `:123-134` | `feedback.error` / **8** / 17 / Regular white 1 line |
| `重试` | pad 6, white@0.7, r8 `:58-61` | bare `HamButton` `:136-143` | chip r6, pad h8 v4, white text |
| Temperature | `.system(36,.rounded)` `:82` | `36.sp`, not rounded `:161` | **36 / rounded, tabular** |
| Condition / `city · time` | body `:90` / `.caption` 12, `Color.gray` `:92-93` | `body` 16.sp `:187` / `caption` 12, secondary `:195-196` | 17 / Regular / 12 / Regular `text.secondary` |
| Forecast fill / radius / padding | gray@0.1 `:142` / `8` `:143` / h16 v12 `:140-141` | `ham_gray`@0.1 `:206` / `8.dp` `:204` / h16 v8 `:207` | `surface.tertiary` / **8** / **h16 v12** |
| Row gap | `12` (Grid) `:113` | `16` px `:264,280` | **12** |
| Day / night icons / column gap | `sun.max.fill` / `moon.fill` @12 `:121-132` / `8` `:126` | `WbSunny` / `ModeNight` @12 `:325-348` / `8.dp` `:338` | `icon.xs` 12 / 8 |
| Forecast temperatures | body `:124,133` | Bold 16.sp `:333,350` | 17 / Bold, tabular |
| Attribution | `.caption2` 11, gray, WeatherKit URL `:34-35` | absent | 11 / Regular `text.tertiary`; **URL follows the data source** |
| No-permission state | reuses the error banner `StatusWeatherCardViewModel.swift:77` | own body `WeatherCardView.kt:359-371` | dedicated body: message 12 / Regular + `去授权` 17 / Regular `accent`, top pad 4 |

**Strings:** `天气` (`LS:107`/`STR:6`) · `重试` (`LS:298`/`STR:7`) · `未获取地理权限` (`STR:8` — iOS
`未获取地理位置权限` `LS:715`) · `去授权` (`STR:9`) · `获取地理位置失败` (`STR:15` — iOS
`获取地理位置遇到了错误` `LS:786`) · `获取天气数据时遇到了异常` (`STR:14`) ·
`今日`/`明日`/`昨日`/`周%1$s` (`LS:526,691,695,608`; `STR:10-13`) · ` Weather` (hardcoded
`StatusWeatherCard.swift:33`) — localize and drop the glyph.

**States:** cold start → render the persisted cache, else block spinner · loading → block or inline
spinner · loaded → brief + forecast + attribution · no permission → permission body · location or
fetch error → error banner · empty → attribution row only.
**Cadence:** refresh on appear if the last update is older than 1 h; on pull-to-refresh; self-poll
every **5 min** (`StatusWeatherCardViewModel.swift:58-69`); persist the last response.

`Divergence:` iOS sources Apple WeatherKit, Android 中国天气 via AMap (`WeatherRequestHelper.kt:73`).
iOS's sub-line drops the city when `subLocality` is nil — `??` binds looser than `+`
(`StatusWeatherCardViewModel.swift:138`). Android has no attribution, no cache, and a 30-min
refresh guard (`WeatherCardViewModel.kt:58-66`) instead of a 5-minute poll.

### 1.6 Library (图书馆)

One full status card **per active booking**.

```
RESERVE                        CHECKED-IN / AWAY
┌────────────────────────────┐ ┌────────────────────────────┐
│▤ 图书馆                  ▸ │ │▤ 图书馆                  ▸ │
├────────────────────────────┤ ├────────────────────────────┤
│ 08:00 开始                 │ │ 14:30离开      ← AWAY only │
│ 剩余2小时                  │ │ 已学习1小时30分钟          │
│ ────────────────────────── │ │ ▓▓▓▓▓▓▓▓░░░░░░░  bar 4/2  │
│ A-118                      │ │ 08:00            22:00     │
│ 信息学部图书馆 3F 自习区    │ │ ────────────────────────── │
│ 2026-09-18 08:00-22:00     │ │ A-118 / 信息学部… / 2026-… │
│              [ 变更预约 ]   │ │              [ 变更预约 ]   │
└────────────────────────────┘ └────────────────────────────┘
```

**Blocks** — 1. **shell + band** (always; `books.vertical.fill`/`Book`, `图书馆`; chevron
**tappable** → library — `IOS/card/library/StatusLibraryCard.swift:51`,
`AOS/component/library/LibraryCard.kt:105,133-135`) · 2. **away banner** (`.away` only;
`{time}离开` — `:53-57`, `LibraryCard.kt:136-143`) · 3. **reserve block** (`.reserve` only; begin
time + `开始` + countdown — `StatusLibraryCardReserveInfoView.swift:19-41`,
`LibraryCard.kt:262-277`) · 4. **checked-in block** (`.checkIn || .away` only; `已学习{time}` + bar
+ begin/end labels — `StatusLibraryCardCheckInInfoView.swift:18-33`, `LibraryCard.kt:313-332`) ·
5. **divider** (always; 1 px `surface.tertiary`, v-pad 4 — `StatusLibraryCard.swift:65`,
`LibraryCard.kt:153`) · 6. **seat block** (always; seat number, location, `date begin-end`, gap 0 —
`:67-75`, `LibraryCard.kt:154-171`) · 7. **`变更预约` chip** (always; trailing; **tappable** →
modify booking — `:77-90`, `LibraryCard.kt:173-186`). Rows 2–4 are mutually exclusive; nothing
else is tappable.

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Away banner | `.subheadline` 15, gray `:54-56` | `body` 16, primary, start pad 4 `LibraryCard.kt:139-141` | **15 / Regular, `text.secondary`** |
| Begin time / `开始` | `.title2` 22 Bold, `.blue` `…ReserveInfoView.swift:19-21` / 22 `:22,25` | `20.sp` Bold `ham_blue` `LibraryCard.kt:262-267` / `20.sp` normal `:268-272` | **`title2` 22 / Bold `brand.library`** / `title2` 22 / Regular `text.primary` |
| Countdown | `.caption` 12; red < 30 min `:37-39` | `caption` 12; red ≤ 30 min `:274-277` | 12 / Regular tabular; `text.danger` at ≤ 30 min |
| `已学习{time}` | 17 Bold `…CheckInInfoView.swift:18` | 16 Bold `LibraryCard.kt:313-316` | 17 / Bold |
| Seat number | `.title` **28** Bold `StatusLibraryCard.swift:69-70` | `24.sp` Bold `LibraryCard.kt:155-160` | **`title` 28 / Bold** |
| Location / date | `.caption` 12 `:71-74` | `caption` 12 `:161-170` | 12 / Regular, `text.secondary` |
| Progress bar | `ProgressView(value:)`, tint green/orange `…CheckInInfoView.swift:27-28` | h4 r2, `ham_blue` both states `LibraryCard.kt:324-326` | h 4, r 2, track `surface.tertiary`; fill `brand.sport` checked-in, `brand.score` away |
| Progress value / bar→labels | clamped at 0 only `:23-27` / `4` `:21` | unclamped `:293-296` / 0 | **`clamp(0,1)`** / 4 |
| `变更预约` chip | 12, gray on gray@0.2, pad h4 v2, r6 `StatusLibraryCard.swift:81-88` | 16.sp `ham_blue`, **no chip** `LibraryCard.kt:180-185` | chip r6, pad h6 v4, 12 / Bold, `accent` on `accent`@0.10 |
| Content gaps / inter-card | outer `10` `:52` / 0 (multi-booking) `:50` | 8 after away, 8 before bar `:145,317` / `8.dp` `:87-91` | outer **8** / 8 |
| Error title / message | `加载时遇到了错误` + 12 `:37-39` | 16 Bold + 12 `:115-119` | 17 / Bold + 12 / Regular `text.secondary` |

**Strings:** `图书馆` (`LS:152`/`STR:63`) · `变更预约` (`LS:129`/`STR:66`) · `%@离开`
(`LS:474`/`STR:65`) · `开始` (`LS:102`/`STR:67`) · `不久后` (`LS:508`) · `加载时遇到错误`
(`STR:64` — iOS `加载时遇到了错误` `LS:101`) · `超过%1$d分钟` (`LS:817`/`STR:68`) ·
`超过不到1分钟` (`LS:89`/`STR:69`) · `还有不到一分钟` (`LS:11`/`STR:70`) · `还有%1$d分钟`
(`LS:586`/`STR:71`) · `还有%1$d小时` (`LS:587`/`STR:72`) · `已学习%1$s` (`LS:639`/`STR:75`) ·
`%1$d分钟` (`LS:93`/`STR:74`) · `已结束` (`LS:644`) · `累计学习%1$d分钟` (`LS:768`).

**States:** cached booking list rendered before the first network call · loading spinner · error
(title + message) · empty → hidden (score −1) · **ended** (today's bookings all finished: `已结束`
17 / Bold + `累计学习{n}分钟` 12 / Regular; minutes = Σ(`end`−`begin`)/60 — `StatusLibraryCard.swift:20-28`,
`StatusLibraryCardEndInfoView.swift:11-17`; score 20) · no library token → hidden, score −1, no request.
**Countdown:** reserve counts **down to start**, checked-in counts **up from start**; tick **1 s**;
buckets `< −60s` → `超过{n}分钟` · `[−60,0]` → `超过不到1分钟` · `(0,60]` → `还有不到一分钟` ·
`(< 3600)` → `还有{n}分钟` with `n = ceil(left/60)` · `≥ 3600` → `还有{n}小时`
(`StatusLibraryCardReserveInfoView.swift:57-78`).

`Divergence:` Android has no ended state (`STOP` filtered out, `LibraryCardViewModel.kt:58-63`),
ticks every 10 s, floors minutes and switches to hours at 2 h (`LibraryCard.kt:207-250`), has an
unreachable loading branch (`:111-112`), and scores errors 10 vs iOS 30.

### 1.7 Course (课程)

```
┌──────────────────────────────────────────────┐
│▤ 课程                                        │ band brand.course @0.15 · no chevron
├──────────────────────────────────────────────┤
│ 正在上高等数学                                │ tip 17 / Bold
│ 1-2节 A101                                    │ 12 / Regular
│ ▓▓▓▓▓▓▓░░░░░░░  08:00            08:45       │ in-class bar 4/2 + from/to times
│ ──────────────────────────────────────────── │ divider · v-pad 8
│ ▁▁▃▁▁▁▁▁▁▁▁▁▁▁▁▁▁   ▲                        │ week-grid bar 4/2 · week view only
│ ┃ 高等数学                                    │ colour bar 6 wide r3 · gap 6
│ ┃ 3-4节 B203                                  │ 12 / Regular
│ ┃ 大学英语                                    │
│ ┃ 5-6节 C301                                  │
│ 没有其它课程                                  │ when the remaining list is empty
│                              切换到日视图      │ 12 / Bold · accent
└──────────────────────────────────────────────┘
```

**Blocks** — 1. **header band** (always; `tablecells.fill`/`School`, `课程`; **not tappable** —
there is no timetable route — `IOS/card/course/StatusCourseCard.swift:24`,
`AOS/component/course/CourseCard.kt:81-86`) · 2. **tip block** (always; 10 variants, below —
`StatusCourseCardHeaderTipView.swift:21-103`, `CourseCard.kt:159-343`) · 3. **divider** (always;
v-pad 8 — `StatusCourseCard.swift:36`, `CourseCard.kt:89`) · 4. **week-grid bar** (week view only;
`13 × weekdayTotal` = 65/91 cells, each the occupying course's colour or clear, `▲` at
`width × timeProgress` — `IOS/card/course/StatusCourseCardProgressView.swift:34-71`) ·
5. **`没有其它课程`** (when the remaining list is empty; **after** the rows —
`CourseCard.kt:361-367`) · 6. **course rows** (one per remaining entry, skipping the focused
course — `StatusCourseCard.swift:54-56`, `CourseCard.kt:355-359`) · 7. **view toggle** (always;
right-aligned; **tappable** → week/day — `StatusCourseCard.swift:64-88`, `CourseCard.kt:371-384`).

Tip variants: `%@后上%@` · `正在上%@` (+ in-class bar and from/to times) · `明日早八` ·
`明日第一节课将在%@后开始` · `下周无课程` · `本周无课程` · `本周课程已上完` · `今日课程已上完` ·
`今日无课，好好休息` · `明日无课，好好休息`.

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Tip primary / secondary | 17 Bold `…HeaderTipView.swift:38` / `.caption` 12 `:52,62` | 16 Bold `CourseCard.kt:199` / `caption` 12 `:275-299` | **17 / Bold** / 12 / Regular |
| Tip stack gap | `4` `:54` | `4.dp` `:267,281` | 4 |
| In-class bar / value | `ProgressView`, tint `displayColor` `:55-56` / unclamped `:175` | h4 r2, course colour `:286-288` / `min(1f,…)` `CourseCardViewModel.kt:156` | h 4, r 2, track `surface.tertiary`, course-colour fill / **clamped to 1** |
| Week-grid bar | h4 r2, track gray@0.15, `▲` @12 gray@0.8 `…ProgressView.swift:56-71` | absent (`weekProgress` never read, `CourseCardViewModel.kt:135`) | **h 4, r 2, track `surface.tertiary`** |
| Row: colour bar / gap | `6 × ∞`, r3 `StatusCourseCardCourseItem.swift:43-45` / `6` `:42` | `6.dp × ∞`, r3 `CourseCard.kt:137-141` / `4.dp` `:149` | 6 wide, **r 3** / **6** |
| Row: name / desc | 17 Bold / 12 `:52-55` | 16 Bold / 12 `:152-153` | **17 / Bold** / 12 / Regular |
| Row: text active / ended | `Color.primary` / `Color.gray` `:58-59` | `ham_text_primary` / `ham_text_secondary` `:151` | `text.primary` / `text.secondary` |
| Row: ended bar | `Color.gray` `:47` | `ham_lightGray` #EDEEEF `:141,145` | `surface.tertiary` |
| Period text | `N节`/`N-M节` + optional `周X ` + optional ` `+location `:66-86` | same, prefix appended last `CourseCard.kt:103-125` | `%1$d-%2$d节`, then `周X ` (week view only), then ` ` + location |
| List gap | `10` `StatusCourseCard.swift:25` | `5.dp` `CourseCard.kt:354` | **8** |
| Toggle labels | **current view**: `周视图`/`日视图` `:81` | **action**: `切换到日视图`/`切换到周视图` `:378-380` | **action labels** |
| Toggle type / guard | `.caption` 12, accent `:82` / `!weekList.isEmpty` `:64` | `caption` 12, `ham_blue` `:381` / none `:371` | 12 / Bold `accent` / **always rendered** |
| Toggle action | `withAnimation` + full `doUpdate()` `:77-86` | flag flip + list recompute only `:374-375` | flag flip **+ full refresh** of current class and progress |
| `没有其它课程` | 12, gray, **above** the list `:49-51` | 12, secondary, **below** `:362-366` | 12 / Regular `text.secondary`, **below** the rows |

**Strings:** `课程` (`LS:115`/`STR:44`) · `没有其它课程` (`LS:380`/`STR:59`) · `切换到周视图`
(`STR:61`) · `切换到日视图` (`STR:60`) · `%@后上%@` (`LS:472`/`STR:55`) · `正在上%@`
(`LS:359`/`STR:56`) · `明日早八` (`LS:692`/`STR:57`) · `明日第一节课将在%@后开始`
(`LS:693`/`STR:58`) · `下周无课程` (`LS:374`/`STR:50`) · `本周无课程` (`LS:375`/`STR:51`) ·
`今日课程已上完` (`STR:54`) · `本周课程已上完` → `本周的课程已经全部结束啦～辛苦啦！💪` (`LS:122`) ·
`今日无课，好好休息` (`LS:377`/`STR:52`) · `明日无课，好好休息` (`LS:376`/`STR:53`) · `%1$d节`
(`LS:354`/`STR:45`) · `%1$d-%2$d节` (`STR:46`) · `%1$d小时` (`LS:481`/`STR:47`) · `%1$d分钟`
(`LS:479`/`STR:48`) · `不到1分钟` (`LS:379`/`STR:49`) · `周%1$s` (`STR:13`).

**States:** no loading, error or screen-level empty. Hydrate from a persisted cache before first
paint and re-rank immediately; week empty → hidden (score −1).
**Constants:** 13 periods/day · 45 min/period · 2 h threshold for tip time strings ·
`showNextWeek` cut-off **Saturday 21:00** (`StatusCourseCardViewModel.swift:24,94,97,126,243`;
`CourseCardViewModel.kt:103,123,164,204`). **Cadence:** 15 s self-restarting loop plus the
course-updated notification (`StatusCourseCardViewModel.swift:51-62,199-272`).

`Divergence:` Android has no week-grid bar, inverts the toggle labels, omits the empty-week guard,
never publishes a score, and greys all next-week rows (`CourseCard.kt:128-130`); its cut-off is
21:30. iOS renders `没有其它课程` above the rows.

### 1.8 Schedule (日程)

```
┌──────────────────────────────────────────────┐
│🗓 日程                                      ▸ │ band brand.schedule @0.15
├──────────────────────────────────────────────┤
│ 本周待完成3项日程                             │ 17 / Bold
│┌────────────────────────────────────────────┐│ hero · whole block tappable
││ 高等数学                              3     ││ name 17 / Bold · countdown 48 / rounded
││ 2026-09-19 08:00                小时        ││ time 12 / secondary
││ 教三楼301                                   ││ location 12 / secondary · 1 line
││ 课程内容…                                   ││ body 12 / primary · 3 lines · top pad 8
││ ▌ 高等数学(一)                              ││ bar 6 wide r3 + name 12 / Bold
│└────────────────────────────────────────────┘│
│ ──────────────────────────────────────────── │ divider
│ 大学物理                          2 天        │ up to 4 more rows · 28 / rounded
│ 大学英语                          5 小时      │ each tappable
│                               其它日程        │ 12 / Regular · accent · trailing
└──────────────────────────────────────────────┘
```

**Blocks** (`IOS/card/schedule/StatusScheduleCard.swift`) — 1. **header band** (always; `calendar`,
`日程`; chevron **tappable** → schedule — `:26`) · 2. **summary line** (always; mutually exclusive:
`本周待完成%1$d项日程` `:122` · `待完成%1$d项日程` `:125` · `未添加日程` `:128`; 17 / Bold) ·
3. **hero block** (when `scheduleList.first` exists; **tappable** → schedule `:134`; name `:80-82`,
begin time `:83-85`, location when non-empty `:146-150`, hero countdown `:160-171`, content body
when non-empty `:178-182`, related-course pill when non-empty `:185-201`) · 4. **divider** (always
`:33`) · 5. **`无日程`** (when `weekList.count < 2 || list.count < 2` `:34-37`) ·
6. **rows 2–5** (`ForEach(1..<5)`, each **tappable** `:39-49`) · 7. **`其它日程`** (when
`weekList.count >= 5 || (weekList.isEmpty && list.count >= 5)`; **tappable** `:52-67`).

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Hero countdown / unit | `.system(48)` Bold `:160-161` / body 17, `offset(y:8)` `:171` | *card absent* | **48 / rounded Bold, tabular** / 17 / Regular baseline-aligned |
| Row countdown / unit | `.title` 28 Bold `:92-93` / `.caption` 12, `offset(y:4)` `:103-104` | *absent* | `title` **28 / rounded Bold** / 12 / Regular |
| Countdown colour / gap | `.orange` expired else `ham_darkBlue` `:106,173` / `4` `:90` | *absent* | `brand.schedule`; `brand.score` expired / 4 |
| Item name / time / location | 17 Bold `:80-82` / 12 gray `:83-85` / 12 gray 1 line `:146-150` | *absent* | 17 / Bold / 12 / Regular `text.secondary` / 12 / Regular 1 line |
| Content body | 12, 3 lines, top pad 8 `:178-182` | *absent* | 12 / Regular, 3 lines, top pad 8 |
| Related course | bar 6 wide r3 `:187-192`; name 12 Bold, gap 4, top pad 8 `:186-200` | *absent* | 6 wide r3, course colour; name 12 / Bold |
| `无日程` / `其它日程` | 12 gray `:35-37` / 12 inherited `:62-67` | *absent* | 12 / Regular `text.secondary` / 12 / Regular **`accent`** |
| Row padding / header→hero / content gaps | v `4` `:108` / top pad `8` `:206` / `12` outer `:27` | *absent* | v 4 / 8 / **8** outer |

**Strings:** `日程` (`LS:688`/`STR:42`) · `无日程` (`LS:685`) · `其它日程` (`LS:569`) ·
`未添加日程` (`LS:713`) · `本周待完成%1$d项日程` (`LS:718`) · `待完成%1$d项日程` (`LS:656`) ·
`%@剩余` (`LS:471`) · `%@前` (`LS:470`) · `分钟` (`LS:574`) · `小时` (`LS:633`) · `天` (`LS:628`).

**Data & cadence:** local Realm store, no network. All items whose `end` is nil or in the future,
ascending by `begin`; the week subset is bounded by Monday-start `begin − (weekday−1)` … `+7d`.
Re-query every **10 s** (`StatusScheduleCardViewModel.swift:28-57,76-88`). Countdown buckets:
`≤1` → 1 分钟 · `<120` → n 分钟 · `<2880` → n 小时 · `<14 400 000` → n 天
(`ScheduleUIUtils.swift:19-31`). **States:** no loading (render nothing until the first query
lands) · no error (replace the force-`try!` at `StatusScheduleCardViewModel.swift:45`) · two
empties (summary `未添加日程`, body `无日程`).

`Divergence:` Android's `ScheduleCard.kt` is an empty stub with zero call sites and no `Schedule`
branch in the render `when` (`AOS/StatusView.kt:208`) — the card does not exist there. iOS bugs:
`:125` formats `待完成%lld项日程` from `weekScheduleList` inside a branch reachable only when that
list is empty (always reads `待完成0项日程`); `:34` shows `无日程` even with exactly one schedule;
`:213-242` is a dead duplicate countdown helper.

### 1.9 Bus (校车)

```
COLLAPSED                           EXPANDED (iOS horizontal timeline)
┌──────────────────────────────┐    ┌──────────────────────────────────┐
│🚌 校巴                     ▸ │    │ 1号线                          ⌃ │
├──────────────────────────────┤    │ ⟵──── horizontal ScrollView ───⟶ │
│ 信息学部站                    │    │  当前     3站      已到站        │
│ 距你1234m                    │    │  🚶16     🚌16      🚌16         │
│ ─────────────────────────── │    │ ══════ track 4/2 ═════════════   │
│┌──────┐                      │    │  -      计算机学院   信息学部    │
││1号线 │  信息学部 → 计算机学院 │    │  columns 56 (当前) / 64 (bus)   │
│└──────┘  🚌🚌🚌               │    └──────────────────────────────────┘
│          3 站           ⌄    │    Android expanded: 3 columns instead —
│  当前正前往计算机学院站        │    counts (当前, N站) │ dot rail │ 🚌 N 到达/前往 X
└──────────────────────────────┘
```

**Blocks** — 1. **header band** (always; `bus`/`DirectionsBus`, `校巴`; chevron **tappable** →
campus-bus screen — `IOS/card/bus/StatusBusCard.swift:18`,
`AOS/component/bus/StatusViewBusCard.kt:79-81`) · 2. **error banner** (on failure; message 17 /
Regular white 1 line + `重试`; **tappable** → retry — `StatusBusCard.swift:22-42`; Android shows a
read-only caption, `StatusViewBusCard.kt:100-105`) · 3. **whole-card spinner** (loading with no
stop — `StatusBusCard.swift:43-45`, `StatusViewBusCard.kt:106-110`) · 4. **stop block** (stop
resolved; name, distance, inline spinner while refreshing, Live Activity menu on iOS —
`StatusBusCard.swift:48-63`, `StatusViewBusCard.kt:84-98`) · 5. **divider**
(`StatusBusCard.swift:68-70`, `StatusViewBusCard.kt:112-113`) · 6. **line list** (up to 3 rows —
`StatusBusCard.swift:73-79`, `StatusViewBusCard.kt:114-119`) · 7. **collapsed row** (default;
**tappable** → expand — `StatusBusCardLineInfoView.swift:76-82`, `StatusViewBusCard.kt:133-135`) ·
8. **expanded row** (timeline; not itself tappable — `StatusBusCardLineInfoView.swift:113-186`,
`StatusViewBusCard.kt:166-309`).

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Content padding | `0`, blocks pad h12 `StatusBusCard.swift:20` | `16.dp` `AOS/component/CommonStatusCard.kt:53` | **0**; blocks apply h12 |
| Stop name / distance | 17 Bold `:51-52` / `"{m}m"` no prefix `:53-54` | 16 Bold `:86-90` / `距你%1$dm` 12.sp `:91-98` | 17 / Bold / **`距你%1$dm`** 12 / Regular `text.secondary` |
| Line-name badge | 12 Bold accent on accent@0.2, pad h8 v4, r4 `StatusBusCardLineInfoView.swift:18-27` | plain 16 Bold `ham_darkBlue` `StatusViewBusCard.kt:131` | chip r6, pad h6 v4, 12 / Bold, `accent` on `accent`@0.10 |
| Arrival headline | `title` rounded: `N 站`/`已到站`/`即将到达` `:50-74` | single `N站`/`未发车` 16.sp `:137-154` | `title` **28 / rounded**: `已到站` at 0, `即将到达` at 1, else `%1$d站`; `未发车` in `body` |
| Bus-count glyphs / `当前…站` / route | one `bus` @8 per bus `:37-46` / `.caption2` 11 `:66-69` / 11, gap 3 `:29-35` | absent / absent / absent | **one bus glyph per approaching bus** / 11 / Regular `text.secondary` / 11 / Regular, gap 4 |
| Expanded row height / columns | intrinsic; 12 spacer icon→pill `:130-131,155-156` / 56 and 64 `:145,171` | `28.dp` fixed `:167,180-188` / counts pad-end 6, bus pad-start 6 `:176-270` | **28** / **horizontal track** h 4, r 2, `text.secondary`@0.2 |
| Expanded count label / stop names | black pill r4, pad 2, white 12 `:158-164` / 11, gray, `maxWidth 48` `:139-142,165-168` | plain 12.sp `:180-203` / 16.sp, 1 line, ellipsis `:272-305` | pill r6, pad h6 v4, 12 / Bold white on `text.primary` / 12 / Regular 1 line ellipsis `maxWidth 48` |
| Per-bus fade | `1/(i+1)*0.5+0.5` `:148,172` | none | **opacity `1/(i+1)*0.5+0.5`** |
| Expand chevron / animation | `chevron.up`/`chevron.down`, no size `:76-82` / bare `withAnimation` `:77-79` | `ArrowRight` 0→−90° `:127,156-161` / `AnimatedVisibility` + `animateContentSize` `:166`,`:75` | `icon.xs` 12 chevron rotate 180° / animate content size, no explicit duration |
| Expansion persistence | survives refresh `StatusBusCardViewModel.swift:184-194` | lost (`remember`) `StatusViewBusCard.kt:126` | **survives data refresh** |
| Live Activity button | `Menu` `line.3.horizontal` @14, gray@0.2, r6 `StatusBusCardLiveActivityButtonView.swift:18-44` | absent | iOS only — a platform rule, not a gap |

**Strings:** `校巴` (`LS:731`/`STR:33`) · `重试` (`LS:298`) · `获取地理位置异常` (`LS:785`) ·
`更新校巴信息异常` (`LS:709`) · `站` (`LS:389`) · `%1$d站` (`LS:390`/`STR:36`) · `到达`
(`LS:394`/`STR:39`) · `前往` (`LS:583`/`STR:40`) · `当前%@` (`LS:654`) · `当前` (`LS:653`/`STR:38`) ·
`未发车` (`LS:712`/`STR:37`) · `已到站` (`LS:395`) · `即将到达` (`LS:396`) · `距你%1$dm` (`STR:34`) ·
`加载时遇到了错误` (`STR:35`) · `把%@添加到实况活动` (`LS:669`) · `清除所有实况活动` (`LS:751`).

**States:** loading (whole-card, then inline) · error (banner + `重试`) · location denied (banner) ·
empty (band only) · hidden when logged out or more than **20 km** from campus
(`StatusBusCardViewModel.swift:68-70,163-169`). **Cadence:** init, CAS login-state change,
pull-to-refresh, and a **60 s** self-loop (`StatusBusCardViewModel.swift:53-64` — defined but never
started on iOS; it must be).

`Divergence:` Android hides beyond 100 km (`StatusViewBusCard.kt:70-72`), has no retry, no
`已到站`/`即将到达` branch (renders `0站`/`-N站`), no bus-count glyphs, no `当前…` sub-line, no Live
Activity, self-loops every 5 s, and never publishes a positive score. Its first rail connector
omits `.background` and is invisible (`StatusViewBusCard.kt:209-213`).

### 1.10 Sport (运动)

One card **per order**. The whole card lives inside the loop, so zero orders emits zero views.

```
┌──────────────────────────────────────────────┐
│⛹ 运动                                      ▸ │ band brand.sport @0.15
├──────────────────────────────────────────────┤
│ 待付款                                       │ 17 / Bold
│┌────┐  羽毛球 | 桂园体育馆                    │ chip 32×32 r6 · body 12 / Regular
││ 7  │  2026-09-19 10:00-11:00                │
│└────┘                                        │
│ ──────────────────────────────────────────── │ divider
│ 请于2026-09-18 22:00前完成支付     [ 去支付 ] │ 12 / danger + filled CTA
└──────────────────────────────────────────────┘
```

**Blocks** (`IOS/card/sport/StatusSportCardView.swift`) — 1. **`ForEach(vm.orderList)`**, keyed by
`orderNo` (`:14-15`) · 2. **header band** (`sportscourt`, `运动`; chevron **tappable** → sport
`:16`) · 3. **status line** (always; 17 / Bold `:18-19`) · 4. **court-number chip** (always;
32×32 r6, fill `brand.sport`, label 22 / rounded Bold white `:22-28`) · 5. **venue line** (always;
`{type} | {stadium}` 12 / Regular `:30-31`) · 6. **time range** (always; 12 / Regular `:33-34`) ·
7. **payment block** (`status == .unpaid` only `:38`): divider, then — when `canPay`
(`paymentStartTime == nil || now > paymentStartTime`) a red `请于%@前完成支付` (`:43-44`) plus
`去支付` (**tappable** → sport pay `:53-62`); otherwise `当前未处于可支付时间` 12 / Bold (`:46-47`)
plus `请于%@-%@完成支付` (`:48`).

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Status line | `.bold` body 17 `:18-19` | *card absent* | 17 / Bold |
| Chip box / label | `32×32`, r `6`, **no fill** `:24-28` / `.title2` 22 rounded Bold white `:23,25` | *absent* | 32×32 r6, fill **`brand.sport`** / 22 / rounded Bold white |
| Venue / time | `.caption` 12 `:30-34` | *absent* | 12 / Regular |
| Payment deadline / closed / window | `.red` 12 `:43-44,51` / 12 Bold `:46-47` / 12 `:48` | *absent* | 12 / Regular `text.danger` / 12 / Bold `text.primary` / 12 / Regular |
| `去支付` | white on `.blue`, pad h8 v8, r6 `:56-61` | *absent* | filled: h 28, pad h12 v6, r6, `accent`, white 12 / Bold |
| Content gaps / divider | `VStack` default `:17,21` / inside the unpaid block `:39` | *absent* | **8** / 1 px `surface.tertiary`, v-pad 4 |

**Strings:** `运动` (`LS:829`; Android `sport_title` in `feature/sport` `STR:3`) · `去支付`
(`LS:594`/`STR:38`) · `请于%@前完成支付` (`LS:802`/`STR:40`) · `当前未处于可支付时间`
(`LS:655`/`STR:41`) · `请于%@-%@完成支付` (`LS:801`/`STR:42`) · status labels `未知` `待付款`
`待使用` `使用中` `已使用` `已取消` `已退款` (`LS:883-889`; also hardcoded as lookup keys at
`SportOrderDetail.swift:30-36` — move to typed `String(localized:)` sites).

**States:** the card must be **state-gated, not data-gated** — keep it mounted and render an inline
loading / error / empty state. Publish 60 with an order, −1 without. Refresh on init and on
pull-to-refresh only (`StatusSportCardVM.swift:15-21`); no polling.

`Divergence:` **Retracted 2026-09-24 — Android *does* have a sport status card.** An earlier
revision said `StatusViewCardType` had no `Sport` case; it does
(`AOS/utils/StatusViewCardScoreManager.kt:40`, `Sport(requiresCasLogin = true)`),
`AOS/StatusView.kt:250-252` composes it, and `StatusSportCard.kt` is 212 lines with an order
card, area number and pay row. What remains genuinely divergent: iOS's chip has no fill, so it
is white-on-white in dark mode (`StatusSportCardView.swift:26-28`), and the iOS VM never
publishes a score, so the card earns no place in the ordering.

---


## 2. Course timetable (课程表 Course)

### 1. Timetable main (课程表) — platforms: both
**Purpose:** Show one week of the term as a 13-period × 5/7-day grid; move between weeks, open a course detail, long-press to add/paste a course.
**Entry:** iOS tab 1 — `MainTabView.swift:27`; Android `CourseRoutePath.HOME` — `CourseView.kt:24`.
**Layout:**
```
┌──────────────────────────────────────────────┐  ignoresSafeArea (CourseView.swift:44)
│ STATUS BAR                    Spacer(sbH) :31│
├──────────────────────────────────────────────┤
│ HEADER                                h = 50 │  CourseViewHeaderView.swift:39
│  [‹]      [本周 / 第N周]      [⚙] [›]        │  H pad 10 (:64), thin material (:65)
├──────────────────────────────────────────────┤  VStack spacing 8 (CourseView.swift:32)
│ WEEKDAY ROW                           h = 36 │  CourseViewBodyView.swift:25
│  │ 42 │ 一  二  三  四  五  (六 日)          │  leftWidth 42.0 (:16)
├──────────────────────────────────────────────┤
│ BODY  GeometryReader, 13 equal rows          │  …CourseBodyView.swift:18
│  │ 1  │ ▢ ▢ ▢ ▢ ▢     singleHeight = h/13   │  (:20)
│  │ .. │ ▢ ▢ ▢ ▢ ▢     itemVpad 2.0 (:15)    │
│  │ 13 │ ▢ ▢ ▢ ▢ ▢                           │
├──────────────────────────────────────────────┤
│ Spacer 90 (tab-bar clearance) — :35          │
└──────────────────────────────────────────────┘
 body H pad 2 (:33) · detail overlay (:38-40) · background (:41-43)
```
iOS 26 variant (`CourseViewHeaderView.swift:70-113`): `ZStack`, `.buttonStyle(.glass)` chevrons pinned to the edges, week label centred with `.glassEffect()`, `minWidth 120` (`:99`), height `50` (`:97`), top pad `statusBarHeight` (`:110`), H pad `10` (`:111`) — **no gear**. Android (`CourseMainView.kt`): week bar
`statusBar + 56.dp` (`:190-193`); weekday row offset `y = 64 + statusHeight` (`CourseMainViewWeekdayCell.kt:64-66`); body offset `x = 40` (`:180`), `y = getBodyPaddingTop()+4`
(`:181`); bottom `bottomWithTabBarHeight() + 8.dp` (`:162`); scrim `Black 0.7f` (`:210`); wallpaper `Image(FillBounds).alpha(backgroundAlpha)` (`:141-158`).

**Grid geometry (normative):**

| Property | iOS | Android | normative |
|---|---|---|---|
| columns · rows | `showWeekend ? 0..<7 : 1..<6` — `…WeekdayView.swift:21` · 13, `singleHeight = h/13` — `…CourseBodyView.swift:18,:20` | `if (showWeekend) 0..6 else 1..5` — `…WeekdayCell.kt:53` · 13, `(boxH − top − pad·13)/13` — `CourseMainView.kt:168-171` | 5 (Mon–Fri); 7 with weekend on · 13 |
| period gutter width · weekday header height | `42.0` — `CourseViewBodyView.swift:16` · `36` — `:25` | `32.dp` — `CourseMainView.kt:71` · `48.dp` — `:70` | 42 · 36 |
| row gap · column gap | `VStack(spacing: 2)` (`:19`) + per-item inset `2.0` — `…CourseNumView.swift:20` · body H pad `2` — `:33` | `Arrangement.spacedBy(4.dp)` — `…ClassNumCell.kt:58` · `itemPadding` `4.dp` — `CourseMainView.kt:169-170` | 4 · 4 |
| cell radius · cell inner padding | `10` — `…WeekdayView.swift:81,:85`, `…CourseItemView.swift:43,:142` · `.padding(.horizontal, 2)` + `.padding(.top, 2)` — `:52-53` | `12.dp` — `…WeekdayCell.kt:73,:87`, `…BodyCell.kt:183` · `padding(top 4.dp, start 4.dp)` — `:196-211` | 10 · top 4, start 4 |
| cell width · height | derived from the grid | `(boxW − pad·cols − gutter − 2·hPad)/cols` · `(boxH − top − pad·13)/13` — `:168-171` | derived |

**Period times** — identical on both platforms (`CourseConfigCenter.swift:67-80` `classDurationList`; `CourseConfig.kt:283-296` `classNumTimeArray`, index 0 blank): `1` 08:00–08:45 · `2` 08:50–09:35 · `3` 09:50–10:35 · `4` 10:40–11:25 · `5` 11:30–12:15 · `6` 14:05–14:50 · `7` 14:55–15:40 · `8` 15:45–16:30 ·
`9` 16:40–17:25 · `10` 17:30–18:15 · `11` 18:30–19:15 · `12` 19:20–20:05 · `13` 20:10–20:55.
**Blocks:** 1 **Header / week bar** — label + prev/next; Android renders the inner `ConstraintLayout` only when `semesterBeginDate != null` (`…WeekChoiceView.kt:66`); long-press/tap → week jump (iOS `contextMenu` `1..<max(currentWeek, 20)` — `CourseViewHeaderView.swift:102-107`; Android `DropdownMenu` `(1..20)` with a
check on the current week — `:199-215`). 2 **Weekday row** — 5 or 7 cells, name + `M-d`. 3 **Period rail** — 13 rows; tap toggles index ↔ start/end clock (`…CourseNumView.swift:79-83`, `…ClassNumCell.kt:53-56`).
4 **Course cells** — height `singleHeight × spans + padding`. 5 **Empty cells** — placeholder chips with a long-press menu. 6 **Detail overlay** (§2).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| header h · H pad · bg · chevron · gear | `50` — `CourseViewHeaderView.swift:39` (iOS 26 `:97`) · `10` — `:64` · `VisualEffectBlurView(.systemThinMaterial)` — `:65` · `25×25` legacy — `:45,:59`; `24×32` iOS 26 — `:81,:89` · `gearshape` `25×25` — `:50-53` | `statusBar + 56.dp` — `CourseMainView.kt:192` · prev/next margin `12.dp` — `…WeekChoiceView.kt:71,:177` · `ham_bg_b2` or `(black\|white)@0.75f` — `:56-58` · `ChevronLeft/Right` 24, `ham_blue` — `Button.kt:78` · absent from the timetable | 50 · 10 · translucent material · 24, blue · gear in the header |
| label primary · secondary · links | `本周`/`第N周`/`放假中` `.semibold` — `:137-138,:153-154,:144-145` · `第N周`/`还有N周开学` `.caption` — `:139-142` · `前往本周` — `:158`, `前往第一周` — `:149` | `bodyBold` 16 — `…WeekChoiceView.kt:96,:123,:155` · `本周` `caption` — `:106` · `Color.ham_blue` — `:137,:169` | 16 semi-bold · 12 · blue 16 |
| weekday name · date · chip bg · colour | `.caption` 12 — `:63` · `.caption2` 11 — `:65` · `ham_lightGray`, `ham_lightBlue` today — `:77,:82`; blur r10 with wallpaper — `:84` · `isToday ? .blue : .primary` — `:67` | `caption` 12 — `:100` · `caption` 12 — `:106` · `ham_gray@0.15f`, `ham_blue@0.15f` today — `:74,:88`; empty chip `alpha(0f)` — `…BodyCell.kt:169-177` · today `ham_blue`, else white/black on wallpaper or primary — `:93-97` | name 12 · date 11 · `#EDEEEF`, today `#007AFF` @0.15 · today blue |
| period number · times · colour | `.caption` 12 — `:63` · `.caption` 12 — `:69,:71` · `highlight ? .blue : .primary` — `:76` | `caption` 12 — `:81` · `caption2` 11 — `:78-79` · white/black on wallpaper else primary — `:70-72` | number 12 · times 11 · primary |
| course text · text colour · cell bg | name `.bold` + instructor + location, all `.caption2` 11 — `:46-51` · `ColorUtils.getTitleColorFromBackgroundColor(bg)` — `:35,:50` · `displayColor` + `black@0.25` in dark — `:116-117`; material + `@0.5` + `.opacity(courseOpacity)` — `:120-123` | name `captionBold` 12, others `caption` 12 — `:196-211` · `Color(ColorUtils.getForegroundColor(bg.toArgb()))` — `:193-194` · `getCourseGridColor`; empty `ham_gray@0.15f` — `:184-188` | name 12 bold, rest 12 · auto contrast · theme colour |
| swipe · long-press · bottom | `DragGesture(minimumDistance: 60)` — `…CourseBodyView.swift:53-62` · `contextMenu` — `…CourseItemView.swift:61-108` · `Spacer 90` — `CourseView.swift:35` | `x > 20` prev, `x < -20` next — `…BodyCell.kt:100-108` · `DropdownMenu` with a `1.1f` scaled clone — `…DropDownMenuCell.kt:74-121` · tab bar + `8.dp` — `:162` | 20 threshold · menu with a clone · tab bar + 8 |
**Strings:** `本周` · `第N周` · `还有N周开学` · `放假中` · `前往第一周` · `前往本周` · `编辑` · `复制` · `剪切` · `删除这节课` · `删除该门同时间段的课` · `删除这门课` · `添加` · `粘贴%@` · `WEEKDAY_LONG_*`/`WEEKDAY_SHORT_*` (short when `showWeekend || lang != zh` — `…WeekdayView.swift:24`) ·
`CURRICULUM` tab (`MainTabView.swift:27`). Android: `course_week_format` · `course_week_current` · `course_week_go_current` · `course_week_go_first` · `course_week_until_start` · `course_menu_edit|copy|cut|delete_class|delete_same_time|delete_course|add|paste` ·
`course_weekdays_full`/`_short` (`strings.xml:79-124`).
**States:** Loading — none, the grid fills asynchronously (`CourseViewBodyViewVM.swift:45-83`). No term
start date — iOS renders grey empty chips with no distinct state; Android shows a centred column, `课程表未设置` at `title2` 20 and `请设置开学日期后再使用课程表` at `body` 16, `padding(24.dp)`, `spacedBy(8.dp)` (`CourseMainView.kt:121-139`), with the week bar still drawn (`:189-194`). Term set, no
courses — grey chips; long-press → add. Before term (`displayWeek <= 0`) — iOS `放假中` + `前往第一周` (`:143-151`); Android `还有N周开学` + `前往第一周` (`…WeekChoiceView.kt:150-171`); both skip week 0 (`CourseViewShareVM.swift:43-59`, `CourseMainViewModel.kt:154-166`). Add/paste out of term — hidden unless
`displayWeek > 0` (`…CourseItemView.swift:146`, `…BodyCell.kt:93`). Long-press feedback — Android vibrates (`…BodyCell.kt:94,:114`), iOS does not.
**Divergence:** iOS 26's header has no gear; week-jump 1–19 vs 1–20; `放假中` is iOS-only; Android has a
real "no term date" empty state; date/period-number sizes are inverted (iOS 11/12, Android 12/11); radius 10 vs 12, gutter 42 vs 32, weekday row 36 vs 48; paste label `粘贴%@` vs `粘贴`.

---

### 2. Course detail (课程详情) — platforms: both
**Purpose:** Overlay/sheet for a tapped course cell — basic info, grade distribution, linked schedules.
**Entry:** Cell tap → `shareVM.detailVM.setCourseGrid(...)` (`…CourseItemView.swift:65-67`), rendered as an `.overlay` (`CourseView.swift:38-40`); Android tap → `showCourseDetail/showModal` (`…BodyCell.kt:84-91`), rendered at `CourseMainView.kt:216-227`.
**Layout:** iOS — full-screen `.overlay`, `matchedGeometryEffect` morph, scrim `displayColor@0.3` + material blur (`:38-47`), spacer `72` (`:24`); Android — half-height slide-up sheet (`:110-114`) over a `Black 0.7f`
scrim (`CourseMainView.kt:210`), top bar `statusBar + 8 + 40` with `ham_black@0.45f` (`:238-243`). Content: scroll, H pad `16`, cards — info (dot + name + `type · credit` + instructor + location), score (conditional), linked schedules (conditional); iOS action buttons at `offset(-16, 65)`.
**Blocks:** 1 **Dot + name** (`CourseViewDetailCourseInfoView.swift:49-52`, `CourseMainViewDetailView.kt:139-146`). 2 **Type · credit** — conditional (`:70`, `:149`). 3 **Instructor / location** — iOS always (`:87-88`),
Android per field (`:166-178`). 4 **Grade card** — conditional. 5 **Linked schedules** — conditional on count > 0 (`CourseViewDetailScheduleView.swift:18`, `:192`). 6 **Action bar.**
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| presentation · H pad · card gap | full-screen overlay, morph, spring 0.6/0.8 — `CourseViewDetailViewModel.swift:40,:48` · `16` — `:32` · `VStack(spacing: 0)`, card pad bottom 8 — `:21,:32` | half-height `slideInVertically{it/2}+fadeIn` — `:110-114` · `16.dp` — `:132` · `spacedBy(16.dp)` — `:133` | half-height sheet · 16 · 16 |
| card · dot · gap · name · credit | `RoundedRectangle(10)` `b2` + `.cornerRadius(16)` — `:26-31` · `Circle` `10×10` — `:49` · default — `:48` · `.bold` 17 — `:52` · `.caption` 12, `%.1f` 学分 — `:70-71` | `HamCardView` r 16, pad 16 — `Card.kt:49,:53` · `Box` `8.dp` — `:139-144` · `8.dp` — `:145` · `bodyBold` 16 — `:146` · `caption` 12, `course_credit_format` `%.2f` — `:150-161` | surface, r 16 · 8 · 8 · 16 bold · 12, `%.2f` |
| instructor/location · schedule title · row · divider | body 17, `VStack(spacing: 0)` — `:86-89` · `关联了%lld项日程`, counts **future** items — `…ScheduleView.swift:19,:21` · name `.bold` + `.caption` time, pad v 4 — `:68,:74-80` · `Divider` between future/past — `:29-31` | `body` 16, `Spacer(4.dp)` — `:169` · `course_linked_schedules`, counts **pending** — `:194-199` · name `bodyBold` + time `caption` printed **twice** — `:218,:226`, rows gap 8 — `:201` · none | 16, gap 4 · count future items · 16 bold / 12, gap 8 · divider |
| countdown · actions · container | number `.title.bold()` 28 — `:88-89`, unit `.caption` `offset(y:8)` — `:99-100`, orange when expired else `ham_darkBlue` — `:102` · `IconButton` `.body.bold` 17, `.secondary`, pad 8, `.ultraThinMaterial` circle — `:41-44`, gap 8 — `:17` · `.overlay(.topTrailing)` `offset(-16, 65)` — `:48-52` | none — prints the start date · add-schedule h 32, r 8, `ham_gray@0.75f`, pad h 8 — `:256-275`; close 32 circle — `:279-292` · `Column` h `statusBar+8+40`, `ham_black@0.45f`, pad h 16, gap 8 — `:238-249` | 28 number + 12 unit · 32 high, r 8 · top-end row, pad 16, gap 8 |
| score bars | `CourseScoreSingleCard(showShare: true, …)` — `:21-28` | label w `48.dp` — `:123`; bar h `4.dp` r 2 — `:141-142`; divider h `120.dp` at end `81.dp` — `:168-173`; average `%.1f` at `title` 24 — `:219-236`; comment icon `Forum` 20, bg `ham_gray@0.2f`, pad 4 — `:206-215` | bar 4 high, r 2; average 24 |
**Strings:** `学分` · `关联了%lld项日程` · `%@剩余` · `%@前` · `暂无这节课的成绩统计信息`; courseType is looked up dynamically (`String(localized: String.LocalizationValue(courseType))`, `:70`). Android:
`course_credit_format` · `course_linked_schedules` · `course_add_schedule` · `course_score_total_students` · `course_score_average` · `course_score_empty` · `common_close`.
**States:** Dismiss — iOS `✕` or `vm.dismiss()`, plus auto-dismiss on `ham_courseTimetableNeedUpdate`
(`CourseViewDetailViewModel.swift:29-31`); scroll-to-dismiss is commented out (`:62-73`); Android taps outside or `✕` plus `vm.showModal = false` (`CourseMainView.kt:223-226`). Score hidden — `canUseCourseScore == false` (no education fetch, or the id contains `customize`)
(`…CourseScoreViewModel.swift:39-54`); Android `enableCourseScoreInfo == false` or the id contains `CUSTOM_COURSE_PREFIX = "cust"` (`:183-186`, `CourseEntity.kt:9`). Score loading — `ProgressView` card (`…CourseScoreView.swift:37-43`) vs `HamLoadingProgressBar` (`:243-244`). Score empty —
`暂无这节课的成绩统计信息` `.caption` (`:30-35`) vs `course_score_empty` `body` in `Box(padding 12.dp)` (`:247-252`). Score error — iOS renders **nothing** (`:19-45`); Android renders `vm.errorMessage` (`:255-261`). No schedules — card omitted. Schedule popover — iOS rows are `NavigationLink`s
(`…ScheduleView.swift:54`); Android has a full-screen `Black 0.75f` `CourseSchedulePopover` with a `HamCardView` pad 32 and three icon buttons (`:376-431`). Live countdown — Android re-reads `now` every 10 s (`:86-91`).
**Divergence:** iOS is a full-screen morphing overlay with a countdown and a silently-omitted error card; Android is a half-height sheet that prints the error and the start date twice per row, uses `%.2f` credits and an 8 dp dot.

---

### 3. Course settings (课程表设置) — platforms: both
**Purpose:** Term start date, background image + opacities, weekend toggle, reset, theme entry, help.
**Entry:** iOS gear in the header (`CourseViewHeaderView.swift:50`) or the My-tab route map (`MyViewFunctionCard.swift:41`); Android `CourseRoutePath.SETTING` (`CourseView.kt:36`).
**Layout:** nav `课程设置` (iOS inline) / `课程表设置` → pad `16`; iOS `VStack` gap `24` with three `.caption` section headers (基础信息 / 自定义配置 / 帮助), Android gap `8` with none. iOS order: `开学日期` + computed semester string, then a fetch row (blue circle icon `40×40` + title + subtitle + chevron) → `背景设置`
(preview `200` wide × `200·ratio`, r `10`, two sliders, remove/choose) → `高级设置` (`显示周末` toggle, `重置课表`, `前往配色设置`) → help. Android order: `开学日期` + `显示周末` + `获取课程表` → `个性化` (background) → `配色设置` → `其它` (reset) → help (`CourseSettingView.kt:73-194`).
**Blocks:** 1 **Term start date** + semester string (`CourseSettingViewBasicSection.swift:25-31`, `CourseSettingView.kt:82-105`). 2 **Fetch timetable** → sheet (§7). 3 **Background** — with an image: preview + two sliders + remove; without: choose-image button (`CourseSettingViewUISection.swift:29-130`,
`CourseSettingViewBackgroundCard.kt:79-134`). 4 **Weekend toggle** (`:135`, `:113-120`). 5 **Reset** (`:143-147`, `:155-169`). 6 **Theme entry** (`:150-154`, `:141-150`). 7 **Help** (`CourseSettingViewHelpSection.swift:15-33`, `:174-193`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| nav title · bg · pad · gap · card | `SCHEDULE_PREFERENCES` 课程设置 — `:24-25` · `ham_bg_b1Color` — `:23` · `16` — `:20` · `24` — `:15` · `HamCardView` 16/16, `b2`, title `.semibold`, subtitle `.caption` — `CardView.swift:22-23,:60-68` | `course_setting_title` — `:69` · `ham_bg_b1` — `NavigationView.kt:385` · `16.dp` — `:70` · `8.dp` — `:71` · `HamCardView` 16/16, title `bodyBold`, subtitle `caption` — `Card.kt:49,:53,:81-85` | 课程表设置 · `#F9F9F9` · 16 · 8 · 16/16 |
| section headers · date row · chip | `.caption` 12 + `ham_text_t2Color` — `BasicSection.swift:19-20`, `UISection.swift:21-22`, `HelpSection.swift:16-17` · `DatePicker` `.labelsHidden()` — `:25-28`; semester `.caption` `.gray` — `:29-31` · none | none · `body` 16 label — `:82-86`; semester `caption` 12 — `:96-100` · `Gray@0.15f`, r 8, pad v6/h8 — `DatePickerButton.kt:43-52` | per-card titles · labelled row + chip, r 8 |
| fetch row · preview | icon `arrow.up.right` `.system(20).semibold` — `:43`; circle `blue@0.1` `40×40` — `:44-49`; pad 8 — `:50`; frame 40 — `:51`; title `.semibold` — `:54-56`; subtitle `.caption` `.gray` — `:57-59`; chevron — `:63-64` · `200 × 200·ratio`, r `10` — `UISection.swift:27-28,:36-39`; 3 mock chips 40×120/80/160, r 5 — `:42-76`; top-bar chip 180×40, shadow r5 y2 — `:79-89` | `HamButton` + `body` 16 `ham_blue` — `:128-132` · width `200.dp`, height `width·ratio` — `BackgroundCard.kt:179-190`; chips `(200−16)/5+5` wide ×4/×3/×5, r 8 — `:185-227`; container pad 8 — `:200-205` | icon row + chevron · 200 wide, r 10 |
| sliders · remove · choose | background `0...0.85` — `:95`; course `0.15...1` — `:105` · `REMOVE_BACKGROUND_IMAGE` red — `:121-123` · `CHOOSE_BACKGROUND_IMAGE` blue — `:128` | Material3, thumb+track `ham_blue` — `:104-107,:122-125`; stored inverted `1−alpha`, clamped `<= 0.95` — `:100-103,:118-121` · `course_background_remove_image` `body` `ham_red` — `:130-134` · `course_background_set_image` `body` `ham_blue` — `:83-87` | 0.05…1 each · red · blue, 16 |
| weekend · reset · theme · help | `Toggle(SHOW_WEEKEND)` — `:135` · `RESET_SCHEDULE`, red when `canResetCourse` else gray, disabled — `:143-147` · `HamCardView(配色设置)` + `NavigationLink { 前往配色设置 }` — `:150-153` · watermark `questionmark` `180`, `.gray`, `offsetY 30` — `HelpSection.swift:18`; titles `.padding(.bottom, 1)` — `:21,:29`; `Divider` — `:26` | `HamSwitch` `ham_green`/`ham_lightGray` — `Switch.kt:34`; label `body` 16 — `:113-117` · two-tap `course_setting_reset` → `_confirm` — `:155-169`; toast `_done` — `:160,:66` · `HamCardView(…)` + `HamButton` — `:141-150` · watermark `QuestionMark` `128.dp`, `offsetY 4.dp`, pad top 48 — `:174-179`; header `headlineBold` 14 — `:183-186` | `#34C759`/`#EDEEEF` · two-tap confirm · card + blue link · watermark 128, pad top 48 |
**Strings:** `课程设置` · `基础信息` · `开学日期` · `决定获取哪一学期的课表` · `第一学期`/`第二学期`/ `第三学期` · `%lld-%lld年度 %@` · `按照开学日期更新课程表` · `从教务系统获取课程表` · `自定义配置` · `背景设置` · `设置课程表的背景` · `背景透明度` · `课程透明度` · `移除图片` · `选择图片` · `高级设置` · `显示周末等` · `显示周末` · `重置课表` · `配色设置` · `前往配色设置` · `帮助` · `如何添加、编辑课程` ·
`在课表空白处长按可添加课程，在课程出长按可编辑该课程。` · `如何快速查看某一周的课程表` · `长按顶部的"第几周"，可快速选择周数。` · `重置课表时遇到了错误`. Android: `course_setting_title` · `course_setting_table_title` · `course_setting_term_start` · `course_setting_term_format` · `course_semester_numbers` · `course_setting_show_weekend` · `course_setting_fetch_timetable` ·
`course_background_personalize` · `course_background_set_image` · `course_background_opacity` · `course_course_opacity` · `course_background_remove_image` · `course_color_setting_title` · `course_color_setting_go` · `course_setting_other` · `course_setting_reset` · `course_setting_reset_confirm`
· `course_setting_reset_done` · `course_setting_help_title` · `course_setting_help_header` · `course_setting_help_desc` · `not_set`/`picker_confirm`/`picker_cancel`.
**States:** No background image — choose-image button only. Background set — preview + two sliders + remove,
cross-faded by `AnimatedContent` (`BackgroundCard.kt:77`). Reset — iOS greys out permanently after a success (`CourseSettingViewModel.swift:27,:85-87`); Android re-arms and fires the success toast unconditionally (`CourseSettingView.kt:155-162`). Reset failure — iOS error toast (`:79-81`); Android has no
error path (`CourseSettingViewModel.kt:37-40`). Image picking — iOS `PhotosPicker` → `MantisImageCropView`, ratio `screenW/screenH` (`UISection.swift:157-173`); Android `PickVisualMedia` → `ImageCropActivity`, `ratioX/Y = view.width/height` (`BackgroundCard.kt:143-172`). No loading or error state beyond toasts.
**Divergence:** iOS nav title `课程设置` with three section headers; Android `课程表设置` with none plus an extra 其它 card. Card order and slider ranges differ (Android's inversion yields an effective `0.05…1`). Reset is one-tap-then-disabled on iOS, a re-arming two-tap on Android.

---

### 4. Theme selection (配色) — platforms: both
**Purpose:** Pick the built-in palette or a user-defined one; import/export palettes by QR code.
**Entry:** `Route.courseSettingTheme` (`CourseSettingViewUISection.swift:151`) / `CourseRoutePath.THEME` (`CourseSettingView.kt:143`).
**Layout:** nav `配色`, bg `ham_bg_b1`, pad `16`, cards gap `16`. Card 1 `默认配色` — swatch grid + `选取` (iOS offsets the button `-16,+16`). Card 2 `自定义(配色)` — swatches or `无自定义配色`, then `编辑颜色`. Card 3 — `扫码导入自定义配色`, the whole card is the button. Card 4 — `分享我的自定义配色` with a QR image `200` revealed on tap.
**Blocks:** 1 **Default palette** — read-only swatches + `选取`. 2 **Custom palette** — swatches (or empty label) + `编辑颜色` + `选取`. 3 **Scan-to-import row.** 4 **Share card** — QR on tap.
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| title · bg · padding · card gap · card | `配色` — `CourseSettingThemeView.swift:129` · `ham_bg_b1Color` — `:128` · `16` — `:124` · default `VStack` spacing · `HamCardView(title:)` 16/16, `b2`, title `.semibold` — `CardView.swift:60` | `course_theme_title` — `:66` · `ham_bg_b1` · `16.dp` — `:68` · `spacedBy(16.dp)` — `:67` · `HamCardView { }` 16/16 — `Card.kt:49,:53`; title `bodyBold` 16 — `:114-119` | 配色 · `#F9F9F9` · 16 · 16 · 16/16, 16 bold |
| select button · swatch · grid | `选取`, `.disabled(selectType == .default)`, `.offset(-16, 16)` — `:37-46`; custom `:74-85` · `24×24`, r `4` — `:28-31,:57-60` · `LazyVGrid(adaptive 24, gap 4)` — `:24-26,:53-55` | `course_theme_select` `body` 16, `ham_blue`/`ham_gray`, `enabled = !selected` — `:120-131,:166-177` · `20.dp`, r `4.dp` — `:138-143,:184-189` · `FlowRow` gap `4.dp` — `:133-136,:179-182` | blue 16, disabled when active · 24, r 4, gap 4 |
| empty label · edit link · import · share · QR | `无自定义配色` plain `Text` — `:51` · `编辑颜色`, `.padding(.top, 4)` — `:66-71` · `HamCardView` in `NavigationLink(scanCode, 扫描配色分享二维码)` — `:86-90` · `分享我的自定义配色` — `:109` · `CIFilter.qrCodeGenerator`, `.interpolation(.none)`, h `200` — `:113-118,:133-144` | none (empty `FlowRow`) · `course_theme_edit_color` `body` 16 `ham_blue` — `:192-200` · `HamButton → QrCodeRoute.Scan(COURSE_THEME, autoPopBack = true)` — `:207-222` · `course_theme_share` `body` `ham_blue` — `:234-239` · `QrCodeView(200.dp)` — `:241-243` | `无自定义配色` · blue 16 · card-wide button · blue 16 · QR 200 |
| payload · validation · result · share visibility | hex-array JSON — `:113` · ≤30 valid 6-digit hex — `CourseSettingThemeViewModel.swift:50-66` · toast `成功从二维码导入配色` + pop — `:95-98` · `if !customColorList.isEmpty` — `:101` | same format — `:94` · shared QR route · `autoPopBack` — `:211` · always — `:91-99` | hex-array JSON, ≤30 · only with a custom palette |
**Strings:** `配色` · `默认配色` · `选取` · `自定义` · `无自定义配色` · `编辑颜色` · `扫描配色分享二维码` · `扫码导入自定义配色` · `成功从二维码导入配色` · `分享我的自定义配色`. Android: `course_theme_title` · `course_theme_default` · `course_theme_custom` · `course_theme_select` · `course_theme_edit_color` · `course_theme_import_qr` · `course_theme_share`.
**States:** No custom palette — iOS shows the empty label and hides both the custom `选取` and the share card (`:51,:75,:101`); Android shows everything. Selected theme — `选取` disabled/greyed (`:43,:81`, `:124,:170`). QR expanded — local `@State` toggled with animation (`:104-107`, `:63,:96`). No loading or error state.
**Divergence:** Swatches 24 vs 20; iOS hides the empty-palette affordances, Android always shows them; share-card visibility differs.

---

### 5. Custom theme (自定义配色) — platforms: both
**Purpose:** Add, list and delete the colours in the user's custom palette.
**Entry:** `Route.courseSettingCustomTheme` (`CourseSettingThemeView.swift:67`) / `CourseRoutePath.CUSTOM_THEME_SETTING` (`CourseThemeSelectView.kt:193`).
**Layout:** nav `自定义配色` + trailing `清空所有配色` (red; iOS also grey + disabled when empty) → pad `16`; iOS one card containing all rows separated by `Divider`, Android one `HamCardView` per colour (gap `8`). Each row: swatch + hex text + `删除`. Add row: iOS inline `ColorPicker` width `16`, Android a `ColorButton`
(`24` box r6, `ColorLens` `20`, preview bar `32×10` r4) opening a bottom-sheet `HamColorPicker(ColorPickerType.Ring(showAlphaBar = false))` (`:153-160`).
**Blocks:** 1 **Colour list** (`CourseSettingCustomThemeView.swift:18-44`, `:88-115`). 2 **Row** — swatch, hex, delete. 3 **Add row.** 4 **Clear-all.**
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| title · bg · padding · spacing | `自定义配色` — `:74` · `ham_bg_b1Color` — `:73` · `16` — `:59` · n/a (one card) | `course_theme_custom` — `:71` · `ham_bg_b1` · `16.dp` — `:83` · `spacedBy(8.dp)` — `:84` | 自定义配色 · `#F9F9F9` · 16 · 8 |
| row · swatch · hex · delete | `HStack(spacing: 4)` — `:22` · `24×24`, r `6` — `:24-26` · default body 17 — `:27` · `删除` red — `:33-35` | `Row` gap `8.dp`, centre-vertical — `:89-92` · `20.dp`, r `4.dp` — `:93-98` · `body` 16, `weight(1f)` — `:99-104` · `course_action_delete` `body` 16 `ham_red` — `:105-113` | gap 8 · 24, r 6 · 16 · red 16 |
| row divider · add control | `if color != last { Divider() }` — `:38-40` · `ColorPicker(supportsOpacity: false)`, width `16` — `:48-49` | n/a · `ColorButton` box `24.dp` r6 `ham_gray@0.3f`, icon `ColorLens` `20.dp`, bar `32×10` — `:164-194` | dividers between rows · swatch button 24, r 6 |
| add button · clear-all | `添加` — `:54` · toolbar, red or gray, `.disabled(isEmpty)` — `:62-70` | `course_theme_add_color` `body` `ham_blue` — `:134-147` · `rightToolBar` `body` `ham_red`, always enabled — `:73-81` | `添加颜色`, blue 16 · red, disabled when empty |
**Strings:** `自定义配色` · `删除` · `添加` · `清空所有配色` · `最多选择30种颜色` · `存在相同的颜色` · `已清空所有配色`. Android: `course_theme_custom` · `course_action_delete` · `course_theme_add_color` ·
`course_theme_clear_all` · `course_theme_duplicate_color`.
**States:** Empty list — iOS hides the list card and disables clear-all (`:18,:67-69`); Android shows no colour cards but keeps clear-all red and enabled (`:73-81`). Max colours — iOS caps at 30 with an info toast
(`CourseSettingCustomThemeViewModel.swift:38-41`); Android has no cap (`CourseCustomThemeSettingViewModel.kt:31-35`). Duplicate — iOS info toast and reject (`:43-46`); Android error toast via `ToastManager.showError` with an exact `Color` comparison (`:135-138`). Delete-all — iOS
success toast (`:59-62`), Android silent (`…ViewModel.kt:27-29`). Picker default `Color.gray` on both (`…ViewModel.swift:17`, `CourseCustomThemeSettingView.kt:67,:140`).
**Divergence:** Swatches 24/r6 vs 20/r4; one card with dividers vs one card per colour; iOS disables clear-all when empty and caps at 30, Android does neither.

---

### 6. Course edit / add (编辑课程 · 添加课程) — platforms: both
**Purpose:** Create or modify a course — colour, name/instructor/location, week/period/weekday slots.
**Entry:** iOS `courseInsert(...)` → `CourseAddView` (`…CourseItemView.swift:148`) and `courseEdit(courseGridID:)` → `CourseEditView` (`:71`, `CourseViewDetailFunctionView.swift:21`); Android one route `course/insert-edit?id=&classNum=&week=&weekday=` (`CourseGraph.kt:23-45`).
**Layout:** nav `添加课程` (iOS add) / `编辑课程` (iOS edit **and** Android both modes) → pad `16`, bg `ham_bg_b1`, gap `8` → card `课程背景色`/`背景色` (swatch grid + colour well; iOS `Divider`, Android a `16`-high vertical divider) → card `基础信息`/`基本信息` (three fields; iOS labels are `64` wide, Android
uses `Bookmark`/`Person`/`Map` icons) → card `时间设置` (three button grids: 周数, 节数, 星期) → commit button. iOS edit instead shows floating `保存` (basic info) and `重置时间` (red) at `offset(-16, 16)`.
**Blocks:** 1 **Colour** (`CourseAddColorView.swift:20-49`, `CourseEditViewColorCell.kt:54-115`). 2 **Basic info** (`CourseAddBasicInfoView.swift:20-35`, `CourseEditViewInfoCell.kt:31-60`). 3 **Time setting** (`CourseAddTimeSettingView.swift:21-65`, `CourseEditViewTimeCell.kt:50-176`).
4 **Commit** — iOS: one explicit save in add mode, per-section floating saves in edit mode; Android: a single button whose label varies.
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| title · bg · padding · gap · card | add `添加课程` — `CourseAddView.swift:53`; edit `编辑课程` — `CourseEditView.swift:65` · `ham_bg_b1Color` — `:51`,`…EditView.swift:63` · `16` — `:47` · default · `HamCardView` 16/16, `b2`, title `.semibold` | `course_title_edit` 编辑课程 for both — `CourseEditView.kt:46` · `ham_bg_b1` · `16.dp` — `:48` · `8.dp` — `:47` · `HamCardView` 16/16, `b2`, title `bodyBold` | add `添加课程`, edit `编辑课程` · `#F9F9F9` · 16 · 8 · 16/16 |
| swatches · colour well · divider | `24×24` r `4`, pad `2`, selected `stroke(.gray@0.3, lw 2)`; `GridItem(adaptive 24, gap 8)`, row gap 8 — `CourseAddColorView.swift:20-42` · `ColorPicker(supportsOpacity: false).labelsHidden()` — `:45-48` · `Divider()` — `:44` | `20.dp` r `4.dp`, selected `border 1.dp ham_gray`; `FlowRow` gap `4.dp` — `CourseEditViewColorCell.kt:54-81` · box `24.dp` r6 `ham_gray@0.3f` + `ColorLens` `20.dp` + bar `32×10` — `:84-115` · `HamDividerVertical(16.dp)` — `:83` | 24, r 4, gap 8 · swatch button 24, r 6 · vertical divider 16 |
| labels/icons · text field | `.frame(width: 64, alignment: .leading)` — `CourseAddBasicInfoView.swift:21,:27,:33`; no icons · `TextField` + `RoundedBorderTextFieldStyle` — `:22-23,:28-29,:34-35` | none; `Bookmark`/`Person`/`Map` tint `ham_gray`, gap 8 — `CourseEditViewInfoCell.kt:32-54` · `HamTextField` r `8.dp`, `b2`, `border(1.dp, ham_lightGray)`, pad 8 — `TextField.kt:42-52,:74-79` | icons, gap 8 · r 8, 1 dp border, pad 8 |
| slot buttons · labels · dividers | `36×36`, selected `blue@0.2` / unselected `gray@0.1`, `CustomUnevenRoundedRectangle` (end cells r `8`); `GridItem(adaptive 36, gap 0)`, row gap `8` — `CourseAddTimeSettingView.swift:80-84,:22-24,:29,:46` · `周数`/`节数`/`星期` plain `Text` — `:21,:38,:55` · `Divider` — `:36,:53` | `Box 32.dp`, r `8.dp`, `border(1.dp, ham_lightGray)`, background `vm.color` when selected with `animateColorAsState`; `FlowRow` gap `8.dp`; text colour by `isHSLLightColor(bg)` — `CourseEditViewTimeCell.kt:69-82,:132-143,:165-176` · `bodyBold` 16 — `:50-54,:88-92,:150-154` · `HamDivider` — `:86,:148` | 32, r 8, theme-colour fill · 16 bold · dividers |
| weeks · periods | `timetableWeekTotal`, default **17** — `CourseConfigCenter.swift:22-27` · `max = 13` — `CourseAddTimeSettingView.swift:42` | `weekMax = 25` — `CourseConfig.kt:60` · `classNumTotal = 13` — `CourseConfig.kt:72` | 17 · 13 |
| commit · floating saves · colour auto-save | `保存这门课`, `.padding()`, `.blue`, max width, `blue@0.1`, r `8` — `CourseAddView.swift:37-44` · `保存` — `CourseEditView.swift:31-40`; `重置时间` red — `:46-56` · `.onChange(of: vm.backgroundColor) { saveColor() }` — `:23-25` | pad h 4, fill width, `48.dp`, `blue@0.15f`, r `12.dp`, `headlineBold` 14 centred — `CourseEditViewButtonCell.kt:53-74` · none · none — saved on commit | 48 high, r 12, blue @0.15 · per-section saves in edit mode · auto-save colour |
**Strings:** `添加课程` · `编辑课程` · `保存这门课` · `保存` · `重置时间` · `课程背景色` · `基础信息` · `课程名` · `输入课程名` · `讲师` · `输入讲师` · `地址` · `输入课程地址` · `时间设置` · `周数` · `节数` · `星期` · `WEEKDAY_SHORT_*`. Toasts: `课程名不能为空` · `存在冲突的课程` · `第%lld周-%@` · `保存失败` · `保存颜色失败` · `已保存背景颜色` · `保存课程基础信息失败` · `已保存基础信息` · `已重置课程时间` — **nine
have no `zh-Hans.lproj` entry and render as raw keys** (`course.md:1093`). Android: `course_title_edit` · `course_edit_background_color` · `course_info_title` · `course_name_hint` · `course_instructor_hint` · `course_location_hint` · `course_edit_week_label` · `course_edit_class_label` · `course_edit_weekday_label` ·
`course_weekdays_short` · `course_edit_action_add` · `course_edit_action_add_with_name` · `course_edit_action_reset` · `course_edit_action_modify_basic` · `course_toast_incomplete_info` · `course_toast_name_empty` · `course_toast_week_empty` · `course_toast_class_empty` ·
`course_toast_weekday_empty` · `course_conflict_title` · `course_conflict_subtitle_format` · `course_period_single` · `course_period_range`.
**States:** Initial (add) — week/period/weekday from the tapped cell; colour from `getCurrentThemeRandomColor`
(`CourseAddViewModel.swift:31-42`) / `getRandomColorInCurrentTheme` (`CourseEditViewModel.kt:64-68`); custom id `customize <uuid>` vs `cust <uuid>` (`CourseEntity.kt:9`). Initial (edit) — iOS shows a placeholder `CourseGrid()` then loads async (`CourseEditViewModel.swift:34-56`); Android fields stay empty until the
flow resolves (`:269-306`). Empty name — iOS info toast, save aborts (`CourseAddViewModel.swift:79-82`); Android error toast titled `信息不完整` (`CourseEditViewModel.kt:96-104`). Empty weeks/periods/weekday — iOS silently returns `false` with **no feedback** (`CourseAddViewModel.swift:72-78`); Android error toasts with a
specific subtitle (`:105-131`). Conflict — iOS `存在冲突的课程` + `第N周-<name>` (`:49-57`); Android + `<name> - 第N周 第X[-Y]节` (`:181-207`). Save failure — iOS toast `保存失败` (`:64`); Android returns false so the screen does not pop (`CourseEditViewButtonCell.kt:47-49`). Success — iOS add pops and posts
`ham_courseTimetableNeedUpdate` (`:68`); iOS edit **stays open**; Android pops in both modes. Period range — iOS tapping outside a ≥2 range resets to that one period (`CourseAddTimeSettingView.swift:109-114`);
Android 0 → add, 1 → range-fill, else extend/shrink or reset (`CourseEditViewTimeCell.kt:100-130`). Weeks are free-form toggles on both; weekday is single-select.
**Divergence:** Two iOS routes/titles vs one Android route titled 编辑课程; week grid 17 vs 25; slot buttons
36 vs 32 with blue/gray vs theme-colour fill; iOS uses 64-wide text labels, Android icons; iOS edit keeps the screen open, Android pops; iOS silently ignores empty slot selections.

---

### 7. Fetch courses (获取课程) — platforms: both
**Purpose:** Authenticate against the university portal (CAS) and pull the term's courses.
**Entry:** iOS sheet from the settings 更新课程表 row (`CourseSettingViewBasicSection.swift:69-73`); Android bottom sheet from 获取课程表 (`CourseSettingView.kt:123-127,:196`).
**Layout:** dismiss `取消` (iOS toolbar) / `关闭` (Android `TextButton` 14 blue) → cluster `[appicon 56 r12] [link] [tablecells 48 @0.25]`, gap `16`/`12` → `Spacer 32` → `获取课程` `.title.bold()` 28 / `title` 24 + `Bold` → subtitle `将按照设定的开学日期获取课程` → card (r `8` `gray@0.15` rows gap `24` /
r `12` `ham_lightGray` gap `8`) holding the single CAS row → terminal states `正在更新` / `更新成功` / `更新失败` (Android `获取成功` + `愉快使用吧` / `获取失败`). Module colour `.ham_brand_course` `#1B5E20` (`CourseUpdateView.swift:20`, `CourseSettingViewUpdateCourseSheet.kt:35,:89`).
**Blocks:** 1 **Close** (`IntroView.swift:84-91`, `IntroView.kt:183-185`). 2 **Header cluster** — logo `56×56` r `12` / `48.dp` r `8`, `link` glyph, module glyph `48` @0.25 (`IntroView.swift:38-47`, `IntroView.kt:198-215`). 3 **Title + subtitle** (`:56-60`, `:218-233`). 4 **Method list** — one CAS row; iOS
uses `builder.buildCasNavLink()` when `CasConfig.shared.enabled()` and swaps the row in with animation on `ham_casLoginSuccess` (`CourseUpdateView.swift:23-38`). 5 **Captcha / login web view** — iOS bundled
`education-captcha-page.html` in a `WKWebView` (`EducationCaptchaView.swift:18-33`); Android `HamNavigationView(title: 验证码验证)` (`:130-143`). 6 **Terminal states.**
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| presentation · dismiss · logo · gap · glyph | `.sheet` — `CourseSettingViewBasicSection.swift:69` · `取消` — `IntroView.swift:84-91` · `56×56` r `12` — `:38-41` · `16` — `:37` · `tablecells` `.system(48)` `@0.25` — `:45-47` | `HamSheet` — `CourseSettingView.kt:63,:196` · `关闭` `TextButton` `headline` 14 `ham_blue` — `IntroView.kt:183-185` · `48.dp` r `8` — `:198-204` · `12.dp` — `:196` · `size(48.dp)` `@0.25f` — `:210-215` | sheet · `关闭` · 56 r 12 · 16 · 48 @0.25 |
| title · subtitle · list container | `.title.bold()` 28 — `:56-57` · body 17 — `:60` · `VStack(spacing: 24)`, `.padding()`, r `8` `gray@0.15` — `:66-74` | `title` 24 + `Bold`, pad top 32/bottom 4 — `:218-224` · `body` 16, pad bottom 16 — `:226-233` · pad h 16, r `12.dp`, `ham_lightGray`, pad 16, gap `8.dp` — `:235-245` | 28 · 17 · r 8, `#EDEEEF`, gap 8 |
| row icon · title · subtitle · chevron | `network` `.blue` — `:104-105` · `通过信息门户登录教务系统` `.bold` — `:108-109` · `通过信息门户登录教务系统，然后访问教务系统API获取课程` `.caption` — `:111-114` · `chevron.right` `.gray` — `:119-120` | `Public` `24.dp` `ham_blue` — `IntroView.kt:82-87` · `通过信息门户` `bodyBold` — `:89` · `从信息门户登录教务系统获取课程` `caption` — `:90` · `ChevronRight` — `:93-97` | 24 blue · 16 bold · 12 · chevron |
| captcha · loading · success · error · RN gate | bundled HTML in a `WKWebView` (`EducationCaptchaView.swift:18-33`) · `VStack { ProgressView; 正在更新 }` — `CourseUpdateByCasView.swift:32-38` · `SuccessView(更新成功)` — `:39-40` · `ErrorView(更新失败, vm.errorMessage)` — `:41-43` · `rnConfig["enable"]["RNFetchCourseView"]` — `:11,:21` | `HamNavigationView(course_fetch_captcha_title)` — `:130-143` · `HamLoadingProgressBar` or the RN view — `:170-178` · `SuccessView(获取成功, 愉快使用吧)` + clear back-stack — `:156-158,:187-194` · `ErrorView(获取失败, …)`, retry pops to intro — `:198-210` · `rnComponentConfig["enable"]` contains `RNFetchCourseView` — `:72-75` | in-app web view with a title · centred spinner · success + stack cleanup · error + retry · same CCKV flag |

Android `SuccessView` / `ErrorView` (`AND/core/…/intro/`): Lottie `lottie/lottie_congrats.json` full-screen (`SuccessView.kt:65-70`); detail after 500 ms (200 ms for error) with `slideInVertically{it/2}+fadeIn` (`:59-63,:72-75`); icon `64.dp` on a `ham_blue`/`ham_red` circle, pad 8
(`SuccessView.kt:83-91`, `ErrorView.kt:76-84`); title `title` 24sp, message `body` 16sp; button `48.dp`, `ham_blue@0.15f`, r `12.dp`, `headlineBold`, default text `完成` (`SuccessView.kt:96-108`, `ErrorView.kt:85-107`); vibrates on appear (`SuccessView.kt:62`).
**Strings:** `获取课程` · `将按照设定的开学日期获取课程` · `通过信息门户登录教务系统` · `通过信息门户登录教务系统，然后访问教务系统API获取课程` · `从信息门户验证` · `将进入武汉大学信息门户网页验证你的身份` · `取消` · `正在更新` · `更新成功` · `更新失败` · `信息门户登录失败`/`信息门户的登录状态`. Android: `course_fetch_title` · `course_fetch_subtitle` ·
`course_fetch_from_portal_title` · `course_fetch_from_portal_desc` · `course_fetch_captcha_title` · `course_fetch_success_title` · `course_fetch_success_message` · `course_fetch_failed_title` ·
`course_fetch_postgraduate_title` (route live, entry commented out at `:234-241`) · `course_fetch_from_postgraduate_title|_desc` (unused) · `common_close`/`common_done` · `common_intro_cas_title`/`_subtitle`.
**States:** Portal not linked — iOS `builder.buildCasNavLink()` → `从信息门户验证`
(`CourseUpdateView.swift:31`); Android `navigate(PATH_CAS)` (`:223-231`). Portal linked — iOS swaps the row in with animation on `ham_casLoginSuccess` (`:23-29,:34-38`); Android uses `vm.useCas` (`:66`). Captcha — bundled HTML page (iOS) or a titled `HamNavigationView` web view (Android). Loading / success / error —
spinner, then the terminal screen (Android also clears the back-stack and vibrates). Post-graduate route — absent on iOS; on Android the route exists but the entry `NavLink` is commented out (`:234-241`).
**Divergence:** Terminal copy (`更新成功/更新失败` vs `获取成功/获取失败` + `愉快使用吧`); Android adds
Lottie, haptics and back-stack clearing; row titles are full sentences on iOS and short forms on Android; the dismiss button is `取消` vs `关闭`.


## 3. Schedule (日程 Schedule)

### 11. Home (日程) — platforms: both — **primary screen**
**Purpose:** A to-do list of schedules with countdowns, filterable by group.
**Entry:** the function-grid tile, or the 状态 schedule card. Module colour `#01579B` is identity only —
every control uses `accent` (`DS:641`).
**Layout:**
```
┌────────────────────────────────────────────────┐
│ nav 日程 (inline)                        [ + ] │  → §12
├────────────────────────────────────────────────┤
│ A hero card — nearest item     cond · pad 16   │  SCROLLING (hides on scroll)
│   name · begin · location · course   #01579B   │  panel flips #FF9500 once passed
│   countdown 50 / Bold, white, on the panel     │
├────────────────────────────────────────────────┤
│ B group tab bar — PINNED · h 60                │
│   📁 全部 [n] │ 📁 group [n] …       ▾ fade 80 │
│   50 × 6 rounded underline, accent, slides     │
├────────────────────────────────────────────────┤
│ D group expand panel — cond · h 350 · scrim    │  slides down
├────────────────────────────────────────────────┤
│ C list · SCROLLING · h-pad 16 · rows gap 16    │
│   future ascending ── divider ── past desc.    │
└────────────────────────────────────────────────┘
overlay: E detail popover, 300 wide (§11e)
```
**Blocks:** 1 **A Hero card** — the nearest item, computed with **`end`**: the first where
`(end == nil && begin >= now) || (end != nil && end >= now)` (`iOS/sched/main/ScheduleView.swift:228-230`).
COND visible only when such an item exists **and** (`scrollOffset.y < 50` **or** the filtered list has
≤ 5 items) (`:236`). Left: name `title3` 20/Bold 2 lines; begin 12 `text.secondary`; COND location;
COND related-course chip (6-wide bar r3 + name 12/Bold). Right: countdown **50 / Bold** white + unit
12/Bold offset y 12 on a **`#01579B`** panel that flips **`#FF9500`** once the item has passed, h-pad 16.
Card r16 pad 16 `surface.secondary`. TAP → §11e (`:237-241`).
2 **B Group tab bar — PINNED** — h 60, `surface.secondary`, leading pad 16, chips 24 apart.
3 **C List** — future items ascending (filtered by `end > now`), a divider, then past items descending;
both filtered by the selected group. Row: card r16 `surface.secondary` h-pad 16; name 17/Bold; begin 12;
COND related-course chip; right countdown **28 / Bold** + unit 12 offset y 4 — `brand.schedule`, or
`brand.score` once passed, with `%@剩余` while in progress and `%@前` once expired.
4 **D Group expand panel** — COND the chevron is open; height 350; header `+` (new group) and `编辑位置`;
one row per group: icon, name, count badge, the group's first **future** item, chevron. Scrim black
@ 0.5, tap to dismiss. 5 **E Detail popover** — §11e.
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Nav title · add | `日程` hardcoded `:81-82` · `plus` `NavigationLink` `:85-89` | `schedule_title` `ScheduleHomeView.kt:59` · `Add` `accent` `:61-71` | `日程` localised · `icon.md` 24 `accent`, 44 target |
| Hero gate · pad / radius / top spacer | `nearest != nil && (offset<50 \|\| size≤5)` `:236` · 16 `:309` / 16 `:305` / **95** `:29` | `scheduleModel != null` `TopView.kt:41` · 16dp `:47` / 16dp `:49` / none | **iOS's gate** — hide past 50 pt when > 5 items · 16 / 16 / **0** |
| Hero name · date · location · chip | `title3` 20 bold 2 ln `:247-251` · 12 `:252-254` · present `:255-260` · present `:263-278` | `title2` 20 bold `:54-58` · 12 `:59-61` · **absent** · **absent** | 20/Bold · 12 · **present** · **present** |
| Countdown number · unit · panel | `system(50)` `:288-289` · offset y 12 `:290-292` · `ham_darkBlue` always `:295,:299-301` | `52.sp` `:71` · `body` 16 offset y 16 `:75` · `ham_orange` when past `:67` | **50 / Bold** · 12/Bold offset y 12 · **`#01579B` → `#FF9500` once passed** |
| Tab bar h · bg · lead pad · chip gap | 60 `:186` · `b2Color` `:189` · 16 `:187` · 24 `:98` | implicit · `b2` `:58` · `edgePadding 8` `:65` · default | **60** · `surface.secondary` · 16 · 24 |
| Chip icon · label · maxW | `group.icon` `:144` · 17 regular/bold `:104-107` · 80 `:150` | `Folder` hardcoded `:94,:125` · `bodyBold` 16sp `:133` · none | **render `group.icon`** · 17/Bold when selected · 80 |
| **Count badge** | 12/Bold gray, v4 h8, r12 `ham_lightGray`, minW 25 `:108-117,:151-160` | **absent** | **present** — r6, h6 v4, 12/Bold, `surface.tertiary` (`DS:437-442`) |
| Selected indicator · tint | 50 × 6 r3 offset y 12, `matchedGeometryEffect` `:122-133` · `text.primary`/`gray` `:120,:163` | h6, tab width − 20, r3, after a 40 spacer `:66-78` · `text.primary`/`text.secondary` `:96,:100` | **50 × 6 r3 `accent`, slides** (`DS:663`) · `text.primary` / `text.secondary` |
| Trailing fade · chevron | 80 pt 3-stop `[clear,b2,b2]` `:191-192` · `chevron.right` on an `ultraThinMaterial` disc, −90° `:194-205` | 45 dp hand-drawn 4-stop `:151-166` · `ChevronRight` on a `ham_lightGray` disc, +90° `:168-186` | **80**, 3-stop · `icon.md` 24 on a 32 disc, rotate 90° |
| Panel h · header · row gap · scrim | 350 `:483` · pad top 16 + h16 `:431-432` · **24** `:443` · black @0.5 `:35-36,:397-398` | 300dp `:52` · h46 h-pad 16 `:61-62` · 8 `:98` · black @0.75 `ItemListView.kt:180` | **350** · h46 · **8** · black @0.5, tap to dismiss |
| Panel row | icon 20 from `group.icon`, name, badge r10, first **future** item, chevron `:447-473` | `Folder` hardcoded, name, **no badge**, `scheduleList.first()` `:100-118` | icon + name + badge + **first future** item + chevron |
| List top / bottom / row gap | 16 `:384` · spacer 100 `:381` · **12** `:365` | spacer 24 `:88` · spacer 116 `:168` · **16** `:164` | 16 · `navBar + 24` · **16** |
| Row card · name · date | r**10** `:550` / `b2Color` `:549` · bold 17 `:502-504` · 12 `:505-507` | r16dp `Card.kt:53` / `b2` `:42` · `bodyBold` 16sp `:100-104` · 12 `:105-109` | **16** / `surface.secondary` · 17/Bold · 12 |
| Row countdown · suffix · colour | `.title` 28 bold `:530-532` · `%@剩余`/`%@前` `:537,:539` · `ham_darkBlue`/`.orange` `:545` | `title` 24sp bold `:149-152` · none · `ham_orange`/`ham_darkBlue` `:151,:158` | **28 / Bold** (`DS:645`) · **`%@剩余` / `%@前`** · `#01579B` → `#FF9500` |
| Divider · course chip · clock | `Divider().padding(.top,12)` `:371` · 6-wide bar r3 + 12/Bold `:510-520` · `Date()` per render `:227,:329` | none · same, pad top 8 `:127-141` · once, then after 2000 ms, then frozen `ScheduleHomeView.kt:42-56` | **present** · 6-wide bar r3, 12/Bold · **re-evaluated continuously** |
**Strings:** `日程` · `全部` · `无日程` · `无群组` · `编辑位置` · `删除日程` · `%@剩余` · `%@前` ·
`分钟` / `小时` / `天`.
**Countdown units** (normative): ≤ 1 min → `1 分钟`; < 2 h → `分钟`; < 2 d → `小时`; < 10000 d → `天`;
else `10000 天` (`iOS/sched/utils/ScheduleUIUtils.swift:19-31`). Three states only —
notStarted / progressing / expired — derived from `end` when it is set (`ScheduleView.swift:581-585`).
**States:** empty items — `无日程`, 12 `text.secondary`, centred, 20 pt above (`ScheduleView.swift:354-360`)
· empty groups — `无群组` in the expand panel (`:435-441`) · loading — none, Realm results are live
(`:13,:16`); Android's Flow starts empty (`ScheduleListCard.kt:26-27`) · error — none · **panel open** —
350, scrim @0.5, chevron rotated 90° · **popover open** — 300-wide card, tap scrim or close to dismiss.
**Divergence:** Android ignores the `end` field everywhere — countdown, future/past split, nearest item
and group preview all use `begin` only (`ScheduleMainViewTopView.kt:65`,
`ScheduleMainViewItemListView.kt:71-83,146`, `SchedulePopoverCard.kt:93`, `ScheduleListCard.kt:30-32`) —
so an in-progress event reads as past and there are two states instead of three. It has **no count
badges** and **no empty states** (`无日程`/`无群组` absent from `SDSTR:`), and hardcodes a `Folder` icon
because `GroupEditViewModel` never persists `icon` (`GroupEditViewModel.kt:53-59,76-81`). iOS's hero panel
never flips to orange (`ScheduleView.swift:300`), it adds an unexplained 95 pt spacer above the hero
(`:29`), and every string on this screen is a hardcoded literal.

---

### 11e. Detail popover (日程详情) — platforms: both
**Purpose:** Show one schedule's full detail; offer edit and delete.
**Entry:** TAP a hero card or a list row (`iOS/sched/main/ScheduleView.swift:237-241`).
**Layout:**
```
┌──────────────────────────────────────────┐
│ scrim black @ 0.5 — TAP dismisses        │
│ ┌────────────────────────────────────┐  │  300 wide, r12, surface.secondary
│ │ countdown 72 / Bold   header 125  │  │  #01579B → #FF9500 once passed
│ │ unit 12, offset (8,20)         [×]│  │
│ ├────────────────────────────────────┤  │
│ │ name                    title2 20 │  │  pad 12
│ │ begin – end                    12 │  │
│ │ location | note (max 200)      12 │  │  cond
│ │ ▌ related course               12 │  │  cond
│ │ [删除日程]                  [⚙]  │  │  two-stage delete
│ └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```
**Blocks:** 1 Header — h 125, `brand.schedule` (or `brand.score` once passed); countdown **72 / Bold**
white (`DS:646`) + unit 12 offset (8,20); close top-trailing, 32 circle `text.secondary`@0.75.
2 Body — name `title2` 20/Bold; the time range (one timestamp when `end == nil`, else `begin – end`);
COND location; COND note in a scroll view capped at 200; COND related-course chip.
3 Actions — a two-stage delete (first tap arms, second deletes) and an edit button → §12.
4 Presentation — slides up from the bottom; COND scrim tap dismisses.
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Card width / radius · header h | 300 `:175` / **10** `:177` · content-driven + v-pad `:60` | 300dp `:90` / **12** `:89` · **125dp** `:96` | **300** / **12** / **125** |
| Header colour | orange / `ham_darkBlue` `:62` | `ham_orange` / `ham_darkBlue` `:97` | `#01579B` → `#FF9500` once passed |
| Countdown · unit | **80** `:41` · offset y 20 `:54` | **72** `:112` · offset (8,20) `:120` | **72 / Bold** · offset (8,20) |
| Body pad · name | h/b 16 `:169` · `title2` 22 bold `:68` | 12 all `:146` · `title2` 20 `:148` | 12 · **20 / Bold** |
| End time · location · note | shown, but prints `begin` twice (bug `:76`) · present `:83-90` · scroll max 200 `:92-99` | absent · absent · absent | **`begin – end`** · **present** · **present** |
| Delete · edit · close | two-stage text `删除日程` `:119-127` · `gearshape.fill` 12, r6 stroke `:152-164` · `xmark` disc, offset (−8,8) `:184-191` | single tap, immediate `:187-202` · `Settings` 28, r8 `accent.subtle` `:204-219` · 32 circle, 8 margins `:124-142` | **two-stage** · `icon.md` 24, 44 target · 32 circle, 8 margins |
| Scrim | `ultraThinMaterial`, dismisses `:198` | black @0.75, **not** clickable `:83` | black @0.5, **tap to dismiss** |
**Strings:** `删除日程` · `分钟` / `小时` / `天` · `%@剩余` · `%@前`.
**States:** empty — n/a · loading / error — none.
**Divergence:** Android's popover is not dismissible from outside (`PopoverCard.kt:83`), deletes on a
single tap, and omits the end time, location and note. iOS's date line renders `begin` in the end slot —
a bug to fix, not a spec.

---

### 12. Insert / edit schedule (新建日程 / 编辑日程) — platforms: both
**Purpose:** Create or edit a schedule: name, note, target time, group, reminder, related course.
**Entry:** `+` on the home nav bar, the popover edit gear, or a course's 关联日程 row →
`scheduleInsert` / `schedule/insert-edit` (`And/routes:13`).
**Layout:**
```
┌────────────────────────────────────────────────┐
│ nav 新建日程 / 编辑日程 · pad 16 · gap 16      │
│  ┌──────────────────────────────────────────┐  │
│  │ 🚩 [名称]                                │  │
│  │ ☰ 更多数据                            ›  │  │  rows 16 apart
│  │ 📅 目标时间     <begin>[–<end>]        ›  │  │
│  │ 📁 群组        <不设置 | name>         ›  │  │
│  │ ⏰ 提醒时间     <choice>               ›  │  │
│  └──────────────────────────────────────────┘  │
│  关联  ▌<course>   [◯] / <location>     cond   │
│  ⋯ 更多操作 → 删除所有关联日程 (red)  editing  │
│  ┌──────────────────────────────────────────┐  │
│  │              完成     48 · r8 · accent   │  │
│  └──────────────────────────────────────────┘  │
└────────────────────────────────────────────────┘  surface.primary
```
**Blocks:** 1 **Name** — text field, `flag` icon 20, placeholder `名称`. 2 **更多数据** → 12a; shows the
current location as a 12 subtitle when set (`ScheduleInsertView.swift:57-61`). 3 **目标时间** → 12b;
shows `begin` and, when set, `end` on a second line (`:101-110`). 4 **群组** → 12c; shows `不设置` when
none. 5 **提醒时间** → 12d. 6 **关联** — COND a related course resolved; a switch enables the association
and greys the 6-wide colour bar when off (`:185-189`,`:196`). 7 **更多操作** — COND editing; expands to
`删除所有关联日程` in `feedback.error`. 8 **完成** — validate then commit. **Validation**: name non-empty;
`end` may not precede `begin` nor be more than 24 h after it; the repeat end may not precede `begin` —
each failure is a toast (`:282-311`). On success: toast `保存成功` / `已添加新的日程`, re-register system
calendar events and notifications, dismiss (`:389-392`,`:427`,`:430-436`). 9 **Repeat expansion** —
occurrences step by `repeatInfo.repeatNum` units, honouring 每隔 N (`:411-423`).
**12a More data (更多数据 / 更多信息)** — a card with a location field (`地点(可选)`) and a note editor
(`备注(可选)`; iOS `minHeight 50 / maxHeight 300` `:464`, Android `maxLines = 5` `:107`); the 48-tall
commit button returns the values to the parent. **12b Target time (目标时间)** — card 1: `目标时间`
picker, a divider, `结束时间` + switch with the end picker disabled while off; card 2: `重复` + switch,
COND on → `每隔` + two wheel pickers (1…999, 天/周/月) and `重复结束于` + picker; commit merges date +
time into one value per field. **12c Group (日程群组)** — single-select rows: `不设置` first, then one row
per group with its icon, name and a checkmark; dividers between rows. **12d Alarm (选择提醒时间)** — ten
options in a fixed order: `不设置` 0 · `1分钟` 1 · `2分钟` 2 · `5分钟` 5 · `10分钟` 10 · `30分钟` 30 ·
`1小时` 60 · `2小时` 120 · `4小时` 240 · `8小时` 480
(`iOS/sched/insert/ScheduleInsertViewModel.swift:25-45`, `InsertEditViewModel.kt:58-69`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Form pad · card gap · row gap | 16 `:272` · **12** `:32` · **24** `:34` | h16 + top 8 `:123-128` · **16** `:124` · **16** `:130` | 16 · 16 · 16 |
| Row icon · gap · chevron | SF symbol, 20-wide frame `:36-38` · 8 · `chevron.right` secondary `:66-67` | Material `ham_gray`, pad end 4 `:136` · 8 · `ChevronRight` `ham_gray` `:175-178` | `icon.sm` 20 `text.secondary` · 8 · `icon.xs` 12 |
| Label / value · no-group | bold 17 `:55,:96` / 17 `:108,:140` · `无` `:140` (picker: `不设置`) | `bodyBold` 16sp `:222` / `body` 16sp `:234` · `不设置` `SDSTR:35` (picker: `未选择`) | 17/Bold / 17 · **`不设置` everywhere** |
| Commit label · button | `确定` `:261` · blue, r10, blue shadow `:258-270` | `完成` `SDSTR:11` · 48, r12, `accent`@0.85 `:300-316` | **`完成`** · 48, h16, **r8**, `accent`, 17/Bold white (`DS:420`) |
| Checkmark · sub-screen pad | `checkmark` `title2` bold, slot omitted `:36,:58` · 16 outside the card `:71,:53` | `Check` alpha 1f/0f, slot reserved `:82-86` · `Column(padding 16.dp)` | `icon.md` 24, **slot reserved** · 16 |
| Date/time input · number picker | one `DatePicker` per field `:62,:78,:125` · wheel, width 100, `1..<1000` `:103-118` | separate date + time buttons `:107-112` · `HamPicker`, `1..999` `:189-211` | platform-native picker per field · range **1…999** |
**Strings:** `新建日程` / `编辑日程` · `名称` · `更多数据` · `目标时间` · `群组` · `提醒时间` · `不设置` ·
`关联` · `更多操作` · `删除所有关联日程` · `完成` · `未完成所有信息` · `请输入名称后继续` ·
`输入信息有误` · `开始时间与结束时间相隔不能超过24小时` · `重复结束时间不能早于开始时间` ·
`日程名不能为空` · `保存成功` · `已添加新的日程` · `地点(可选)` · `备注(可选)` · `更多信息` · `结束时间` ·
`重复` · `每隔` · `天`/`周`/`月` · `重复结束于` · `日程群组` · `选择提醒时间` · the ten alarm labels.
**States:** empty — the group picker hides its divider and shows only `不设置` · loading — none ·
error — validation toasts (three on iOS, one on Android).
**Divergence:** Android ignores `repeatNum` when expanding occurrences — it always adds exactly one unit
(`InsertEditViewModel.kt:238,245,252`) — and validates only the blank name (`:179-185`). iOS drops the old
group membership and re-registers system calendar events (`ScheduleInsertView.swift:371-392`); Android
deletes and re-adds with a `// TODO Add system schedule` (`:269`). Android shows no success toast and has
no 更多操作 / `删除所有关联日程`. Four different "no group" labels exist across the two clients.

---

### 13. Group edit (日程群组) — platforms: both
**Purpose:** Create or rename a schedule group; delete it when editing.
**Entry:** a row in the home expand panel, or the panel `+` → `scheduleGroupEdit(...)` /
`schedule/group?groupId=` (`And/routes:12`).
**Layout:**
```
┌────────────────────────────────────────────────┐
│ nav 新增群组 / 编辑群组 · pad 16 · gap 16      │
│  ┌──────────────────────────────────────────┐  │
│  │        ┌────────┐                        │  │  icon 72, r12, tertiary@0.25
│  │        │  icon  │       spacer 16        │  │
│  │        └────────┘                        │  │
│  │        [名称]                            │  │
│  └──────────────────────────────────────────┘  │
│  完成   48 · r8 · accent · white               │
│  删除   48 · r8 · feedback.error  (editing only)│
└────────────────────────────────────────────────┘
```
**Blocks:** 1 Icon — 72 in a r12 `surface.tertiary`@0.25 plate, rendering the group's persisted `icon`.
2 Name field — 17, placeholder `名称`, caret `accent`. 3 Delete — COND editing; destructive button;
removes the group and exits. 4 `完成` — validate, commit, exit. 5 Validation — the name must be
non-empty; on failure the screen **stays** and toasts (`ScheduleGroupEditViewModel.swift:33-36`).
6 Success — toast `添加成功` / `修改成功` and post the home refresh event. 7 Both `name` **and** `icon`
are written on commit (`:39-41`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Screen pad · card gap · icon plate | 16 `:108` · **24** `:21` · 48 glyph, pad 8, r10 `ham_lightGray` `:28-35` | 16dp `:68` · **16** `:68` · 64dp, pad 16, r12 `ham_gray@0.25f` `:74-79` | 16 · 16 · **72**, r12, `surface.tertiary`@0.25 |
| Icon↔field gap · name field | 8 `:24` · `TextField` `.fixedSize` `:40-41` | 16dp `:84` · `BasicTextField` 17, minW 32dp `:85-98` | 16 · 17, placeholder 12 `text.tertiary` |
| Commit · delete | `确定`, blue, r10, shadow `:91-105` · nested behind `更多操作`, red text `:75-87` | `完成`, 48, r12, `accent`@0.85 `:119-137` · always-visible 48 button `ham_red@0.85f` `:140-159` | **完成** — 48, h16, **r8**, `accent`, 17/Bold white · **always-visible second button**, 48, r8, `feedback.error`, 17/Bold white |
| Screen title | `日程群组` hardcoded `:114` | 新增群组 / 编辑群组 `SDSTR:8-9` | 新增群组 / 编辑群组 |
**Strings:** `新增群组` · `编辑群组` · `日程群组` · `名称` · `完成` · `删除` · `群组名称不能为空` ·
`添加成功` · `修改成功` · `更多操作` · `删除群组`.
**States:** empty — n/a · loading — none · error — a toast; the screen stays.
**Divergence:** Android pops the screen even when validation fails (`GroupEditView.kt:120-121`) and never
persists `icon` on insert or edit (`GroupEditViewModel.kt:53-59,76-81`), which is why its UI hardcodes a
folder everywhere; iOS nests delete behind a disclosure row and shows no success toast.

---

### 14. Group order / position edit (编辑位置) — platforms: both
**Purpose:** Reorder schedule groups by dragging; persist the order back to `position`.
**Entry:** `编辑位置` in the home expand panel header → `scheduleGroupOrderEdit` /
`schedule/group/position` (`And/routes:18`).
**Layout:**
```
┌────────────────────────────────────────────────┐
│ nav 编辑位置                          (完成)   │
│ DragListView · pad 16 · rows gap 8             │
│  ┌──────────────────────────────────────────┐  │
│  │ <icon> <name>                       ⋮⋮   │  │  card, pad 16, r16
│  └──────────────────────────────────────────┘  │
└────────────────────────────────────────────────┘
```
**Blocks:** 1 Reorderable list — one row per group with the group's icon and name; the drag handle is
`icon.md` 24 `text.secondary` (Android `:58-62`; iOS the system edit-mode control `:40`).
2 Drag → move and **persist immediately** (`ScheduleEditGroupPositionViewModel.kt:40-45`); there is no
separate confirm step. 3 Storage — canonical ordering is **ascending by `position`**, written as
`position = index` (`:54`). 4 The home screen reads the same ascending order.
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Container · row gap/pad/radius | system `List` in `.editMode = .active` `:30,:40` · system separators | `DragListView`, pad 16, bottom `navBar+16` `:40-50` · 8dp `:49` · 16dp `Card.kt:48` · 16dp `:53` | drag list, pad 16, bottom `navBar + 16` · 8 · 16 · 16 |
| Group icon · name · handle | `group.icon`, gray `:33-34` · 17 `:35` · system reorder control | **absent** `:57` · `bodyBold` 16sp `:57` · `DragHandle` 24 `ham_gray` `:58-62` | **present** · 17/Bold · `icon.md` 24, 44 target |
| Confirm · stored value | trailing `完成` `:44,:56` · `total - i` (descending) `:50` | none — saves per drag · `i + 20000` (ascending) `:54` | **none — save on every drag** · **ascending, `position = index`** |
**Strings:** `编辑位置` (home panel) · `编辑群组位置` (screen title).
**States:** empty — an empty list, no placeholder · loading — iOS loads on `DispatchQueue.main.async`
inside `init` (`:17-21`) so the first frame is empty; Android loads from a Flow (`:31-38`) · error — none.
**Divergence:** the two clients store incompatible `position` values — iOS descending `total - i` read
with a descending sort, Android ascending `i + 20000` read with an ascending sort — so a synced group
order flips on the other platform. iOS requires an explicit 完成 and shows group icons; Android saves on
every drag and shows none.

---


## 4. Library (图书馆 Library)

### 1. Home (图书馆) — platforms: both
**Purpose:** Module hub — announcements, statistics consent + availability chart, in-flight bookings, quick-book launcher, function grid.
**Entry:** iOS tab route `library`; Android `library/home`.
**Layout:**
```
┌──────────────────────────────────────────────┐
│ nav (iOS large title 图书馆) / status         │
│ Scroll, page padding 16                      │
│  ┌ A banner carousel                  200 ─┐ │
│  ┌ B retry-login        (conditional)     ─┐ │
│  ┌ C analytics          (conditional)     ─┐ │
│  ┌ D current booking ×n (0..n)            ─┐ │
│  ── divider + 16         (only if D) ─────  │
│  ┌ E quick book          (conditional)     ─┐│
│  ┌ F function grid                   150 ─┐ ││
│  │  ┌──────────┬─────────────────────┐     │││
│  │  │ 查看房间  │ 历史预约      (67)  │     │││
│  │  │  (full)  ├─────────────────────┤     │││
│  │  │          │ 设置          (67)   │     │││
│  │  └──────────┴─────────────────────┘     │││
│  ┌ G print card (both)                      ─┐│
│  bottom spacer 128                          │
└──────────────────────────────────────────────┘
```
**Blocks:** 1 **Banner carousel** — announcement cell + N remote pages, always; page `-1` → Board (`LibraryMainView.swift:17`, `LibraryMainView.kt:54-58`). 2 **Retry-login** — `if needRetryLogin` (`:19`, `:61-65`), tap → `fastLogin()`. 3 **Analytics card** — always mounted (`:30`, `:60`), self-hides unless
`enableLibraryAnalytics && permitted != false` (`AnalyticsCardView.swift:18`). 4 **Booking cards** — one per booking in `RESERVE/CHECK_IN/AWAY` (`:32-35`, `:46-52`); expansion = 4 rows of `icon + Text`, icon frame `20`/`24.dp`: `刷新` · `在地图打开` · `添加到系统日历` · `变更或取消预约`, footer `展开`/`收起` +
chevron rotated `-90°` (`CurrentBookingCard.swift:37-136`, `ReservedCard.kt:78-140`). 5 **Divider + 16** — only when 4 is non-empty (`:36`, `:75-77`). 6 **Quick book** — iOS when a starred or last-booking seat exists (`:39`); Android when a seat is selected (`LibraryMainViewQuickBookCard.kt:63`). 7 **Function
grid** — always (`:43`, `:81`). 8 **Print card** — always on both; iOS `LibraryMainViewPrintFunctionCard()` at `LibraryMainView.swift:45`, Android `spacedBy 8` (`LibraryMainView.kt:80-82`). 9 **Spacer** `128` (`:47`).

**Status.** Colours: `reserve` blue, `checkIn`/`stop` green, `away` orange, `miss`/`leaveEarly` red, else gray (`LibraryModel.swift:57-70`); Android home `RESERVE→ham_blue`, `AWAY→ham_orange`, `CHECK_IN→ham_green`,
else secondary (`ReservedCard.kt:49-56`), history adds `MISS→ham_red` (`LibraryHistoryView.kt:62-67`). Names, 8, identical on both: `预约 / 履约中 / 暂离 / 已结束 / 已取消 / 失约 / 早退 / 未签退` (`LibraryModel.swift:47-56`, `BookingVO.kt:26-35`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| page bg · padding | `ham_bg_b1Color` — `:53` · `16` — `:49` | `ham_bg_b1` — `HomeContainer.kt:64` · `16.dp` — `LibraryMainView.kt:41` | `#F9F9F9` · 16 |
| banner h · r · icon · title · tint | `200` — `BannerCard.swift:191` · `16` — `:189` · `72` — `:90` · `.title.bold()` 28 — `:94` · `.blue` — `:105` | `180.dp` — `BannerCard.kt:49` · `16.dp` — `Card.kt:53` · `72.dp` — `:79` · `title` 24 — `:86` · `ham_brand_library` — `:82` | 200 · 16 · 72 · 28 · `#007AFF` |
| tip · watermark · carousel | `.red` `lineLimit(3)` — `:98-99` · `book.fill` 36 @0.25, 6×9 — `:108-117` · `PageTabViewStyle` 5 s — `:81,:186` | none · `Book` 72 @0.2, 5×10 — `:55-71` · single page — `:44` | red, 3 lines · 36 @0.25 tiled · pager, 5 s |
| strip pad · strip text · seat no. · loc/time | h16 v8 — `CurrentBookingCard.swift:24-25` · `.bold`, fg b2 — `:21-23` · `.title.bold()` 28 — `:30` · `.caption` 12 — `:35` | h16 v12 — `ReservedCard.kt:70` · `bodyBold` white — `:65-66` · `title` 24 — `:74` · `caption` 12 — `:75-76` | h 16 v 8 · bold 16, white · 28 · 12 |
| grid h · gap · r · room icon · room fill | `150` — `FunctionButtonView.swift:132` · `8` — `:14` · `16` — `:43` · `chair.lounge.fill` 64 — `:21` · `blue@0.13` — `:44-46` | `142.dp` — `FunctionCard.kt:54` · `8.dp` · `16.dp` — `:79` · `Chair` 64 — `:87` · `blue@0.15f` — `:80` | 150 · 8 · 16 · 64 · blue @0.15 |
| right fill · right icon · right pad · print cell | `gray@0.1` — `:77` · 32 @0.75 — `:81-82` · `16` — `:74` · `ham_brand_library_tint`, r 16, pad 12, icon `printer` 48 @0.75, title `.body.bold` + `arrowtriangle.right.fill` **12** — `LibraryMainViewPrintFunctionCard.swift:14-45` | `ham_gray@0.1f` — `:129` · 32 @0.75f — `:137-138` · `8.dp` — `:130` · `blue@0.15f`, r 16, pad 12, icon 48 @0.75f, arrow `ArrowRight` **20.dp** — `PrintFunctionCard.kt:50-51,:62-64,:76-78` | `gray@0.1` · 32 @0.75 · 8 · as Android, arrow `icon.xs` 12 |
| qb seat no. · badge · reserve button | `.title.bold()` 28 — `QuickBookCard.swift:23` · `收藏座位` orange / `上次预约` green, `.caption`, pad 4, r 5 — `:26-31,:35-40` · r 10, `blue@0.1`, `.padding()` — `:116-117` | `32.sp` — `LibraryMainViewQuickBookCard.kt:74` · none · 48, r 12, `blue@0.25f` — `:119-123` | 28 · caption, white, pad 4, r 5 · 48, r 12, blue @0.25 |
| analytics bar · % · retry banner | 2×30, gap 1, r 1 — `AnalyticsCardView.swift:94-97` · default — `:137` · red, r 16, icon 128 @0.25 — `LibraryRetryLoginView.swift:33-37` | 2×20 — `…AnalyticsCard.kt:196-201` · `title2` 20 — `:167-169` · `ham_red`, r 16, icon 60 @0.15f — `…kt:75-77` | 2×30 · 20sp · `#FF3B30`, r 16 |
**Strings:** `图书馆` · `图书馆公告` · `查看房间` · `查看空余的座位预约` · `历史预约` · `你的历史预约记录` · `设置` · `图书馆预约选项` · `刷新` · `在地图打开` · `添加到系统日历` · `变更或取消预约` · `展开`/`收起` · `收藏座位` · `上次预约` · `预约` · `今天`/`明天` · `数据统计` · `不同意`/`同意` · `请求成功率` · `昨天%@` · `登录信息过期` · `点击重新登录` · `打印` · `在图书馆公共打印机打印`.
**States:**

| State | Behaviour |
|---|---|
| Loading / empty | No spinner on either platform — iOS `currentBookingLoadState` is write-only (`LibraryMainViewModel.swift:59,66,77`); Android's retry banner has its own spinner (`LibraryRetryLoginView.kt:81`). Empty omits blocks 4–6 with no copy. |
| Error / token expired | Non-token errors toast only (`LibraryMainViewModel.swift:71`, `…kt:137-141`). Expired tokens set `needRetryLogin` (`:68`, `…kt:143`) → red banner; tap re-runs `fastLogin()`. |
| Not logged in / consent | iOS sheets the intro and pops 0.3 s after dismissal (`LibraryViewModel.swift:28-34`); Android sheets after 500 ms and composes home only when `userId.isNotEmpty()` (`LibraryView.kt:42-53`). iOS shows the consent card; on Android the branch is unreachable — `canUse` is a non-null `Boolean` (`LibraryMainViewAnalyticsCard.kt:55,61`). |
**Divergence:** iOS banner is a 5 s pager with remote banners, Android a single static page; grid 150 vs 142 and the print card's arrow is 12 vs `20.dp`; quick-book keys off seat *existence* vs *selection*; Android's `在地图打开`
is a dead `{}` (`ReservedCard.kt:91`), iOS has no `添加到系统日历`; consent works only on iOS; retry copy is fail-class-specific only on Android.

---

### 2. Login / connect intro (连接图书馆) — platforms: both
**Purpose:** Obtain a CAS session, exchange it for a library session, report success/failure.
**Entry:** Sheet from `LibraryView.swift:23` / `LibraryView.kt:42-53`.
**Layout:** `[取消 / 关闭]` → cluster `[logo 56 r12] ⛓ [books.vertical 48 @0.25]`, gap `16` → `Spacer 32` → `连接图书馆` `.title.bold()` → `Spacer 16` → one card (r `8`/`12`, `gray@0.15`/`ham_lightGray`, rows
spaced `24`/`8`): `🎓 从信息门户验证` + caption subtitle + chevron. Success = `✓64 on blue`, `验证成功` (28/24), `你可以开始使用图书馆了` (12/16), `Spacer 32`, button `返回`/`完成`.
**Blocks:** 1 **Close** (`IntroView.swift:85-91`, `IntroView.kt:183-185`). 2 **Cluster** (`IntroView.swift:38-47`). 3 **Title + subtitle** — subtitle only when non-empty (`:57-61`). 4 **Method row** → CAS view (`:156-164`,
`LibraryIntroView.kt:63-67`); Android skips it when already authenticated (`:113-117`). 5 **Loading** — Android only, full-screen spinner (`:90-110`). 6 **Success** → dismiss (`SuccessView.swift:67-74`, `LibraryIntroView.kt:68-77`). 7 **Error** — Android only (`:78-89`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| presentation · logo · gap · glyph | `.sheet` — `LibraryView.swift:23` · `56×56` r `12` — `IntroView.swift:40-41` · `16` — `:37` · `48` @0.25 — `:45-47` | `HamSheet` — `IntroView.kt:117` · `48.dp` r `8` — `:201-203` · `12.dp` — `:196` · `48.dp` @0.25f — `:211-215` | sheet · 56 r 12 · 16 · 48 @0.25 |
| title · list container · row spacing | `.title.bold()` 28 — `:57` · r `8`, `gray@0.15`, `.padding()` — `:71-74` · `24` — `:66` | `title` 24 + `Bold`, pad top 32/bottom 4 — `:218-224` · r `12.dp`, `ham_lightGray`, pad h16+16 — `:237-241` · `8.dp` — `:242` | 28 · r 8, `#EDEEEF` · 8 |
| row icon · title/sub · chevron | system, `.blue` — `:104-105` · `.bold` 17 — `:109` · `.caption` 12 — `:113` · `chevron.right` `.gray` — `:119-120` | `24.dp` `ham_blue` — `:82-87` · `bodyBold` 16 — `:89` · `caption` 12 — `:90` · `ChevronRight` — `:93-97` | 24 blue · 17/12 · chevron |
| success icon · title/msg · stack · pad | `checkmark.circle.fill` 64, white on green — `SuccessView.swift:44-45` · `.title` 28 — `:47` · `.caption` 12 — `:51` · spacing `4` — `:42` · `.padding(32)` — `:81` | `Done` 64, `ham_blue` circle, pad 8 — `SuccessView.kt:83-91` · `title` 24 — `:92` · `body` 16 — `:93` · `8.dp` — `:81` · `16.dp` — `:78` | 64, white on `#007AFF` · 28 · 12 · 8 · 16 |
| entrance · button · error icon | opacity + offset y 100→0, spring, 0.8 s — `:86-93` · `返回`, `.padding()`, maxW 350, r 8, `blue@0.1` — `:67-74` · none | `slideInVertically{it/2}+fadeIn`, 500 ms, vibrate — `:59-75` · 48, r 12, `blue@0.15f`, `headlineBold` — `:96-108` · `Close` 64 on `ham_red`, 200 ms — `ErrorView.kt:58,:76-84` | slide+fade, 500 ms, vibrate · 48, r 12, blue @0.15 · 64, white on `#FF3B30` |

**CAS view:** `https://cas.whu.edu.cn/authserver/login?service=…` (`EducationCASWebView.swift:13`, `CasMobileLoginView.kt:63`); cookies cleared first (`EducationCASWebView.swift:27-40`, `CasMobileLoginView.kt:78`); iOS success = URL contains
`https://jwgl.whu.edu.cn/xtgl/index_initMenu.html` and cookies `route`, `JSESSIONID`, `iPlanetDirectoryPro` all present (`:169-176`). Both gate native-vs-RN on `CCKV rnComponentConfig.enable` containing
`RNCasMobileLogin` (`CasMobileLoginView.swift:16-31`, `CasMobileLoginView.kt:48-81`).
**Strings:** `连接图书馆` · `从信息门户验证` · `将进入武汉大学信息门户网页验证你的身份` · `验证成功` · `你可以开始使用图书馆了` · `验证失败` · `取消`/`关闭` · `完成`/`返回` · `信息门户`.
**States:** Not logged in → sheet, dismiss pops. Already authenticated → iOS always opens CAS, Android
jumps to the spinner + `loginCas()`. Loading → iOS waits for `.ham_libraryLogin`, Android has a dedicated route. Success → dismiss + `fastLogin()`. Error → iOS toast only, Android `ErrorView`.
**Divergence:** iOS has no loading or error route and hardcodes `返回`; Android short-circuits CAS when `casContext.isLogin` and defaults the button to `完成`.

---

### 3. Book a room (查看房间) — platforms: both
**Purpose:** Pick building → room → time range → seat, then submit.
**Entry:** Home grid's 查看房间 cell (`FunctionButtonView.swift:18`, `FunctionCard.kt:60`).
**Layout:**
```
┌──────────────────────────────────────────────┐
│ HEADER  bg gray@0.1 · pad 16 · inner gap 8   │
│  ▸ 请选择图书馆 / <building>          bold   │
│  [building chips]  h-scroll r8 (if nil/open) │
│  [room cards w80]  h-scroll r8 (if building) │
│  🕐 [08:00] → [22:30]      [今天|明天]       │  row h 50
├──────────────────────────────────────────────┤
│ BODY  [⚡][☀] 36×32 r8, top-leading          │
│  top inset 48/56 · grid · bottom 128/96+nav  │
├──────────────────────────────────────────────┤
│ FOOTER (bottom overlay, if a seat is chosen) │
│  已选择 / <seat> | <loc> / <day time> [预约] │
└──────────────────────────────────────────────┘
```
**Blocks:** 1 **Title button** toggles the chip row (`Header.swift:37-54`, `LibraryBookView.kt:174-201`). 2 **Building chips** — iOS when `selectedBuilding == nil` or expanded (`Header.swift:25`); Android force-opens via `LaunchedEffect` (`:110-112`), else `AnimatedVisibility` (`:203`). 3 **Room cards** —
floor, `|`, `free/total`, name (`:91-147`, `:245-312`). 4 **Time row** — two 30-min pickers + day selector (`LibraryTimePicker2.swift:12`, `HamFixedTimePickerButton.kt:62`). 5 **Filters** (`Body.swift:75-111`,
`:471,:574-610`). 6 **Seat grid** — attributes as 4-high strips. 7 **Confirm footer** slides up (`Footer.swift:52`, `:473-479`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| header bg · pad · gap · arrow | `ham_bg_b1Color` — `Header.swift:32` · `16` — `:31` · none · `arrowtriangle.right.fill` 8, rotate `-90°` — `:47-51` | `ham_gray@0.1f` — `:166` · top status+42, h12, b12 — `:168-170` · `8.dp` — `:171` · `ArrowRight` rotate `270f` — `:190-198` | `gray@0.1` · 16 · 8 · 8, rotate 90° |
| chip row bg · ends · icon · text | `gray@0.2` r8 pad v8 — `:80-87` · `16` — `:60` · per-building glyphs — `:15-20` · `.caption` — `:83` | `ham_gray@0.15f` r8 pad v6 — `:206-208` · 12 + gap 8 — `:212-213` · `LocationCity` 16 — `:223` · `caption` — `:231` | `gray@0.15` r 8 · 12 · 16 · 12 |
| room row bg · width · floor · free/total · name | `gray@0.15` r8 — `:142-145` · `80` — `:122` · `.caption` semibold — `:107-108` · `free==0?red:green` — `:113` · `.caption` 3 lines — `:117-120` | `ham_gray@0.15f` r8 — `:250` · `75.dp` — `:269` · `captionBold` — `:279` · `<=5 red, <=10 orange, else green` — `:285-288` · `caption` — `:306-311` | `gray@0.15` r 8 · 80 · 12 semibold · 3-step · 12, 3 lines |
| time row · chip · day | h `50` — `:190` · w `85` — `:164` · `Picker` segmented, maxW 100 — `:183-184` | `HamTab` 32 — `:350` · pad v6 h8 r8 — `HamFixedTimePickerButton.kt:32-35` · `HamTab` 32, r 6, bg gray@0.3, item 60 — `:349-364` | 50 · 85 · segmented 32 |
| grid · cell · no. · icon · pad · strips | `LazyVGrid(adaptive(min 70), gap 10)` — `LibrarySelectSeatView.swift:321-323` · r8, selected `blue@0.2` — `:408-409` · `.body.bold()` 17 — `:388` · `chair.lounge.fill` — `:386` · v4 — `:390` · `HStack` of `Color`, h `4` — `:396-403` | `LazyVerticalGrid(Fixed(4))`, gaps 8/8 — `:389-397` · r8, `color@0.15f` — `:416-417` · `title3` 16 — `:436` · `Chair` 20 — `:432` · v4 h6 — `:425-426` · `Spacer(fillMaxWidth, 4.dp)` — `:442-456` | adaptive min 70, gap 8 · r 8, blue @0.2 · 17 · 20 · v4 h6 · 4 high |
| insets · filters | top 48, bottom 128, h 16 — `Body.swift:48,:64,:66` · 36×32, `color@0.25` + blur, r 8 — `:85-110` | top 56, bottom 96+nav, h 16 — `:390-395` · icon 32, `color@0.25f`, pad 6, r 8, bg `b1@0.95f`, offset (16,16) — `:574-609,:471` | 48/128/16 · 36×32, r 8 |
| footer · CTA · time bounds | blur, r 8, pad 8, bottom safe — `Footer.swift:45-51` · `预约` white, pad v12 h16, r8, blue — `:31-38` · begin 08:00–23:00, end 08:30–23:30 — `LibraryDetailBookViewModel.swift:45-54` | `b1@0.95f`+`gray@0.2f`, r8, pad 8, h/v 8/16 — `:617-625` · `bodyBold` white, pad v4 h16, r8, blue — `:659-673` · end default 22:30, 08:30/23:30 — `LibraryBookViewModel.kt:67,:85-89` | translucent r 8 · r 8, blue, white · 08:00–23:00 / 08:30–23:30 |
**Strings:** `请选择图书馆` · `请选择房间` · `今天`/`明天` · `已选择` · `预约` · `没有多余的座位了` · `预约成功` · `预约失败` · `完成`/`返回` · `预约信息` · `"%1$s%2$d楼%3$s"` · `重试` · `遇到了错误` · `未设置`; building names are raw server strings on iOS, mapped to `library_info_building` /
`library_medical_building` / `library_main_building` / `library_engineering_building` on Android (`LibraryBookView.kt:150-159`, `strings.xml:180-184`).
**States:** No building → chip row open, body shows `请选择房间` with `questionmark.app` 96 @0.45
(`Body.swift:19-23`) / `QuestionMark` 64 (`:512-516`). Seats loading → iOS 48-cell shimmer skeleton (`Body.swift:55`, `LibrarySelectSeatView.swift:345-372`), Android centred spinner (`:383`). Seat error → iOS icon + message with no retry; Android `ErrorOutline` 64 + message (fallback `遇到了错误`) + `重试`
(`:528-571`). Filter/room empty → no-seat placeholder, iOS disables scrolling (`Body.swift:68`). Booking → iOS replaces header+body with a full-screen `ProgressView` (`LibraryDetailBookView.swift:21-23`), Android swaps only the banner button (`:676`). Success → whole screen replaced (`:30-39`, `:120-128`); error → iOS
full-screen error with retry (`:24-29`), Android toast only (`LibraryBookViewModel.kt:300-305`).
**Divergence:** iOS grid is adaptive-70 with a shimmer skeleton, Android fixed-4 with a spinner and the only retry affordance; only iOS replaces the screen on booking failure.

---

### 4. Select seat (选择座位) — platforms: both
**Purpose:** Standalone seat picker returning a seat to the caller (quick-book card, starred-seat insert).
**Entry:** iOS `librarySelectSeat` (`QuickBookCard.swift:19`, `LibraryPreferredSeatSettingView.swift:26`); Android `library/select/seat`.
**Layout:**
```
┌──────────────────────────────────────────────┐
│ A LastBookingCard (Android, if any)          │
│ B StarredSeatCard (Android, if any)          │
│ ┌ C SelectSeatCard, HamCardView pad16 r16 ─┐ │
│ │ 房间          [building chips] gap 8     │ │
│ │ [room cards w60] r8 gray@0.15            │ │
│ │ ── HamDivider ──                         │ │
│ │ 座位                                     │ │
│ │ [⚡40×32][☀40×32]        …   [↻40×32]    │ │
│ │ grid: adaptive 70 / FlowRow, maxH 400    │ │
│ └──────────────────────────────────────────┘ │
│ 已选择 / <seat> | <loc>  [选择]  (iOS bar)  │
│ 选择该座位  (Android inline, body, blue)     │
└──────────────────────────────────────────────┘
```
**Blocks:** 1 **Last-booking card** — Android, when a last booking differs from the filter seat (`SelectSeatView.kt:52-62`). 2 **Starred-seat card** — Android, when a starred seat is configured (`:64-75`). 3 **Building chips** — iOS preselects `buildingList.first`
(`LibrarySelectSeatViewModel.swift:35-36`); Android sorts by id (`SelectSeatCard.kt:67-93`). 4 **Room cards** — iOS auto-scrolls to the current room (`:133-137`). 5 **Filters** — Android adds `↻` (`:211-222`). 6 **Seat grid.** 7 **Confirm** (`LibrarySelectSeatView.swift:417-454`, `SelectSeatCard.kt:363-381`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| container · nav title · icon · tint | `VStack`, pad `[.h,.top]` 16 — `:60-61` · none · per-building glyph — `:132` · `Color.blue` vs `t2` — `:135` | scroll page, pad 16, gap 16 — `SelectSeatView.kt:47-51` · `library_select_seat` — `:48` · `LocationCity` 16 — `SelectSeatCard.kt:81` · `ham_blue` vs secondary — `:76` | scroll page, pad 16 · `选择座位` · 16 · `#007AFF` |
| room w · row bg · floor · free/total · name | `80` — `:185` · `gray@0.15` r8, ends 8 — `:198-203` · `.caption` semibold — `:169` · `free==0?red:green`, `|` — `:171-175` · `.caption` 3 lines — `:180-183` | `60.dp` — `:120` · `Gray@0.15f` r8, pad v12, ends 4 — `:98-104` · `captionBold` — `:128-131` · 3-step, no `|` — `:138-141` · `caption` — `:150-154` | 80 · `gray@0.15` r 8 · 12 semibold · 3-step, no `|` · 12 |
| filters · refresh | 36×32, `@0.25` + blur — `:283-289,:298-305` · none | 40×32 r8, `@0.25f` — `:180-206` · 40×32 gray — `:211-222` | 40×32, r 8, @0.25 · refresh present |
| grid · seat no. · cell bg · attributes | `LazyVGrid(adaptive(min 70), gap 10)` — `:321-323` · `.body.bold()` 17 — `:388` · selected `blue@0.2` else `gray@0.2`, r8 — `:409` · 4-high strips — `:396-403` | `FlowRow(gaps 12/12, maxH 400)` — `:262-274` · `24.sp` — `:286-289` · container `gray@0.15f` r8, pill r6 — `:264-266,:296-300` · 14 dp icons on a pill — `:309-325` | adaptive min 70, gap 12 · 17 · r 8, blue @0.2 · 14 icons on a pill |
| weekday chip · last-booking badge | none | yellow, bg `@0.2f`, r6, pad 4, 14sp — `StarredSeatCard.kt:53-58` · seatNum 36 bold + white-on-blue badge r6 pad4 — `LastBookingCard.kt:38-43` | yellow r 6 · 36 bold + badge |
**Strings:** `选择座位` · `选择该座位` · `房间` · `座位` · `选择` · `已选择` · `首选座位` · `上次预约` · `"%1$s预约"` · `每天` · `周日…周六` · `"%1$dF"` · `该房间暂无座位` · `筛选条件下无座位，共剩余%1$d个座位` · `选择房间继续` · `没有多余的座位` · `遇到了错误`.
**States:** Initial — iOS first building, no room → `请选择房间`; Android `Unload` → `选择房间继续`
(`SelectSeatCard.kt:349-357`). Prefilled — iOS from the passed seat (`LibrarySelectSeatView.swift:22-30`), Android from nav args (`SelectSeatViewModel.kt:88-93`). Loading/error — iOS 48-cell skeleton vs Android spinner; iOS error is silent with no retry, Android toasts and offers `↻`. Room-empty vs filter-empty —
iOS renders the same placeholder twice, Android uses two strings. Room switch — iOS cancels the in-flight task (`:70-71`), Android does not. Confirm — iOS posts `.ham_libraryOnSelectSeat` (`:65-71`), Android writes `savedStateHandle["selectedSeat"]` and pops (`:78-84`).
**Divergence:** iOS sends no time range, so it lists the room's full layout with no availability filtering (`LibrarySelectSeatViewModel.swift:77-80`); Android adds starred/last-booking shortcut cards and a refresh button; iOS uses a bottom blur footer, Android an inline text button.

---

### 5. Select time (选择预约时间) — platforms: both
**Purpose:** Edit begin / end / tomorrow for the quick-book card.
**Entry:** `libraryQuickBookTemporaryModifyTime` (`QuickBookCard.swift:64`) / `library/select/time` (`LibraryMainViewQuickBookCard.kt:80-88`).
**Layout:** nav `选择预约时间` → pad `16` → `开始时间` + picker (iOS stacked and centred, separated by `Divider`; Android puts both chips on one row as `[chip] - [chip] ␣ 预约明天 [switch]`) → `预约明天` switch → spacer → full-width `确定` bar, `48` high, r `12`, `blue@0.85f`.
**Blocks:** 1 **Begin picker**, 30-min steps (`LibraryTimePicker2.swift:12`). 2 **End picker**, min `begin + 30 min` (`:26`, `SelectTimeViewModel.kt:58-62`). 3 **Tomorrow switch** (`:85-87`, `:64-65`). 4 **Confirm** — writes the result and pops.
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| container · labels · separators | `ScrollView` + `VStack(leading,0)`, `.padding()` 16 — `:55-56,:113` · `开始时间`/`结束时间` — `:58,:73` · `Divider`×2 — `:72,:84` | `Column(pad 16, gap 16)` — `:44` · none — `:54` · none | pad 16, gap 16 · labelled rows · none |
| pickers · switch | `UIDatePicker` `.wheels`, 30-min, `en_GB`, stacked, centred — `:61,:76` · system `Toggle` after a divider — `:85-87` | chip → `TimePickerDialog` 1/30/60, one row with `-` — `:45-62`, `HamFixedTimePickerButton.kt:32-35,:62` · `HamSwitch` on the picker row, track `ham_green`/`ham_lightGray` — `:64-65`, `Switch.kt:47` | one row, 30-min · `#34C759`/`#EDEEEF` |
| spacer · CTA · result | `30` — `:89` · `确认` white, `.padding()`, r `10`, blue, full width — `:100-110` · `.ham_libraryOnSetQuickBookTemporaryModifyTime` — `:92-96` | `16.dp` — `:44` · `确定` `headlineBold`, 48, r 12, `blue@0.85f`, h-pad 12 — `:78-91` · `savedStateHandle["selectedTime"]` — `:69-76` | 16 · `确定`, 48, r 12, blue @0.85 · return-on-pop |
| bounds | begin 08:00–23:00, end 08:30–23:30 — `:20,:23,:26,:29` | same, default end 22:30 — `SelectTimeViewModel.kt:44,:52-62` | 08:00–23:00 / 08:30–23:30 |
**Strings:** `选择预约时间` · `开始时间` · `结束时间` · `预约明天` · `确定` · `未设置`.
**States:** Initial values come from the caller, `minEndTime = begin + 30 min` (`:42-52`, `SelectTimeViewModel.kt:81-117`); moving begin past end pushes end to `begin + 30 min` (`:63-66`); iOS
reverts an end below `begin+30` (`:77-79`), Android relies on the dialog bound (`:72-74`); toggling tomorrow raises begin on Android (`SelectTimeViewModel.kt:69-79`) but not iOS; dismissing writes nothing; no loading or error state.
**Divergence:** CTA copy `确认` vs `确定` — normative `确定`; only Android adjusts begin on the tomorrow toggle.

---

### 6. Quick book (快速预约) — platforms: both
**Purpose:** Fire `smartBook` for the pre-configured seat/time and report the outcome.
**Entry:** `libraryQuickBook` (`QuickBookCard.swift:97-104`) / `library/quick-book` (`LibraryMainViewQuickBookCard.kt:107-116`). Booking starts on appear.
**Layout:** bright bg, pad `16`, gap `8` — loading: `ProgressView(value:)` (iOS) or Lottie `lottie_flying.json` at `200` (Android), then `正在快速预约` (24) + `请稍等` (16). Success: `✓64` on blue,
`预约成功`, a `预约信息` card carrying seat / location / `yyyy-MM-dd HH:mm-HH:mm`, optional `座位发生了变更`, then `完成`/`返回`. Error: `✕64` on red, `快速预约失败`, the message, `返回`.
**Blocks:** 1 **Loading indicator** (`LibraryQuickBookView2.swift:123-131`, `QuickBookLoadingCell.kt:35-40`). 2 **Loading copy** (`:30-43`). 3 **Success card** shared with Book (`BookSuccessCard.kt:99-151`, `LibraryBookSuccessView.swift:52-125`). 4 **Error card** (`QuickBookFailCell.kt:36-63`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| bg · transition · nav title | `ham_bg_b1Color` — `:136` · none · none | `ham_bg_b1` — `QuickBookFailCell.kt:29` · `fadeIn() togetherWith fadeOut()` — `LibraryQuickBookView.kt:46` · `library_quick_reservation` — `:51` | `#F9F9F9` · crossfade · `快速预约` |
| loading indicator · title · subtitle | `ProgressView(value:)`, synthetic fill — `:123,:131` · `.body` — `:125,:127` · none | Lottie `lottie_flying.json` 200, fillMaxWidth — `…LoadingCell.kt:36` · `title` 24 — `:42` · `body` 16 — `:43` | determinate bar, pad 16 · 24 · 16 |
| success icon · error icon | `checkmark.circle.fill` 64, white on green — `LibraryBookSuccessView.swift:56-58` · `xmark.circle.fill` 64, white on red — `ErrorView.swift:34-35` | `Done` 64, `ham_blue` circle, pad 8 — `BookSuccessCard.kt:99-107` · `Close` 64, `ham_red` circle — `:36-44` | 64, white on `#007AFF` · 64, white on `#FF3B30` |
| error title · detail · CTA · hint | `预约失败` `.title` — `ErrorView.swift:37` · hint `.caption` — `:40-41` · `返回` bold, pad, maxW 350, r 12, `blue@0.15` — `:50-57` · `座位发生了变更` `.caption` — `LibraryQuickBookView2.swift:113` | `title` 24 — `:46` · `body` 16 — `:47` · `headlineBold`, 48, r 12, `blue@0.85f`, h-pad 12 — `:53-63` · none | 24 · 16 · 48, r 12, blue @0.85 · caption under the title |
**Strings:** `正在快速预约` · `请稍等` · `快速预约` · `快速预约失败` · `预约成功` · `座位发生了变更` · `返回` · `完成` · `预约信息`.
**States:** In-flight — determinate bar on iOS (synthetic +0.001/0.01 every 100 ms to 0.95, **not** reset on re-entry — `:47-60`) or Lottie on Android. Success — with or without the seat-changed hint. Error —
`error.message` + dismiss. `Unload` — iOS falls through to the loading branch, Android renders **nothing** (`LibraryQuickBookView.kt:70`). Both guard re-entry on `Unload` (`LibraryQuickBookView2.swift:138-140`, `LibraryQuickBookViewModel.kt:51-53`).
**Divergence:** iOS has a synthetic bar and the seat-changed hint; Android uses Lottie, prints an absolute timestamp instead of `今天/明天`, and renders nothing in `Unload`.

---

### 7. Modify booking (变更预约) — platforms: both
**Purpose:** Change the time window of an existing booking, or cancel / end it.
**Entry:** Expanded booking card → `libraryModifyBooking` (`CurrentBookingCard.swift:99-106`) / `library/history/modify-booking` (`ReservedCard.kt:109-119`).
**Layout:** pad `16`, cards gap `16`. Card 1 `更改预约时间` — iOS two inline pickers w`80`, row `minHeight 50`, `padding(.leading, -8)`; Android two labelled rows with chips; then a full-width
`更改时间` bar, `48` high, r `12`, `blue@0.85f`. Card 2 `取消预约` — a drag-to-confirm slider, `48` high, r `12`, track `gray@0.15`, fill `ham_red`, label `滑动以取消预约` + chevron in red.
**Blocks:** 1 **Time row** (`LibraryModifyBookingView.swift:65-82`, `:55-79`). 2 **Change CTA** — iOS disabled until a time differs (`:84,:105-106`), Android always enabled; spinner in flight (`:83-85`). 3 **Cancel slider** — iOS `Unlocker` threshold `95`, h `50`, r `10`, shadow `r 12×%`, scale `1+%×0.05`
(`:144-174`, `Unlocker.swift:58,:135-168`); Android `HamLocker` h `48`, r `12`, bg `ham_gray@0.15f`, fling threshold `distance×0.85f` (`Locker.kt:159-172`). 4 **Feedback** — toast, refresh, pop.
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| container · card · title | `ScrollView` + `VStack().padding()` — `:54` · 16/16 — `CardView.swift:22-23` · `.semibold` — `:61,:119` | `Column(pad 16, gap 16)` — `:52` · 16/16 — `Card.kt:43-48` · `bodyBold` — `Card.kt:81` | pad 16, gap 16 · 16/16 · 16 bold |
| pickers · change CTA · disabled | two `.inline`, w `80`, row minH `50`, pad leading `-8` — `:65-82` · `更改` white, `.padding()`, r 10, blue + `shadow(.blue,r3,y2)` — `:93-103` · `.opacity(0.5).disabled(true)` — `:105-106` | labelled rows + chips, gap 8 — `:55-79` · `更改时间` `headlineBold`, 48, r 12, `blue@0.85f`, h-pad 12 — `:97-110` · `enabled` flag + `ham_gray` — `:95-96` | 80 wide, row 50 · 48, r 12, blue @0.85 · opacity 0.5 |
| slider h · r · track · fill | `50` — `:144-174` · `10` · `gray@0.2` · red `Rectangle` — `:146-151` | `48.dp` — `Locker.kt:159-172` · `12.dp` · `ham_gray@0.15f` · `ham_red` — `:176` | 48 · 12 · `gray@0.15` · `#FF3B30` |
| slider label · threshold · loading | `滑动以取消预约` + chevron, red — `:137-141` · `95` — `:156` · spinner inside the button — `:90-92`; spinner replaces the slider — `:122-128` | `滑动取消预约` `bodyBold` red, start pad 32, gap 8 — `Locker.kt:56-70` · `0.85f` — `:167` · spinner replaces the button — `:83-85`; replaces the slider — `:123-124` | red, start pad 32, gap 8 · 85 % · spinner replaces the control |
**Strings:** `变更预约` · `更改时间` · `更改预约时间` · `取消预约` · `滑动以取消预约` · `开始时间`/`结束时间` · `已更改预约时间` · `已取消预约` · `操作成功`.
**States:** Pristine — iOS disables the CTA until a time differs, Android does not. Change → spinner, then toast `已更改预约时间`, widget + watch refresh, pop (`LibraryModifyBookingView.swift:244-253`,
`…ViewModel.kt:151-177`); error toast with `error.message` (`:231-243`). Cancel → spinner replaces the slider and iOS keeps `disabled` true (`Unlocker.swift:138`); success toast `已取消预约`, Android also deletes the calendar event (`:100-112`); error toast, slider usable again. Racing actions — no interlock on
iOS, Android guards with `if (… == Loading) return false` (`:87-89,:152-154`). Non-actionable booking — no-op unless `reserve`/`checkIn`/`away` (`LibraryModifyBookingView.swift:270-278`).
**Divergence:** CTA copy `更改` vs `更改时间`; iOS has a dirty check and a scale-up slider animation; Android has an explicit loading interlock.

---

### 8. History (历史预约) — platforms: both
**Purpose:** Show past bookings, clustered by date with a per-day study total.
**Entry:** Grid's 历史预约 cell (`FunctionButtonView.swift:54`) / `library/history` (`FunctionCard.kt:64`).
**Layout:** nav `历史预约` → optional red retry banner → per date cluster: header `今天` or `yyyy-MM-dd`, `-`, `共计%lld分钟` (caption 12) → card (`b2`, r `16`, pad bottom `16`) of rows: `12`-wide status rail · seat column `48` · location · begin-over-end stack with trailing pad `16`.
**Blocks:** 1 **Retry-login banner** (`LibraryHistoryView2.swift:88-93`, `LibraryHistoryView.kt:55-59`). 2 **Date header** (`:29-37`). 3 **Cluster card** (`:40-76`). 4 **Row** — rail, seat column, location, times.
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| container · bg · empty text/container | `VStack`, `ScrollView` only when loaded — `:15,:22` · `ham_bg_b1Color` — `:97` · `没有历史记录`, default, full width — `:18-20` · bare text | `HamNavigationView2` — `:54` · `ham_bg_b1` — `NavigationView.kt:385` · `暂无历史记录` `caption` 12 — `:123-127` · inside `HamCardView(pad 16)` — `:61` | scroll page · `#F9F9F9` · `暂无历史记录` 12 · inside a card |
| card r · cluster gap · date header · rail · seat column | `16` — `:74-75` · `padding(.bottom,16)` — `:77` · `.caption` 12 — `:37` · `12` wide, stat colour, `@0.15` when gray — `:43-45` · `48` — `:55` | `16.dp` — `Card.kt:53` · `spacedBy 8.dp` — `:70` · none · none · `fillToConstraints` — `:83` | 16 · 16 · 12, shown · 12 wide · 48 |
| seat no. · status · location · times | `.body.bold()` 17 — `:49` · `.caption` 12, stat colour — `:51-53` · `.caption` multiline — `:56-58` · `begin` over `end`, `.caption`, trailing pad 16 — `:63-68` | `bodyBold` 16 — `:88` · `body` 16 — `:104-108` · `caption` — `:93` · one line `date begin-end` — `:96-100` | 17 · 12 · 12 · stacked, pad 16 |
| row pad · divider · sort · totals | v `4` — `:60` · none · clusters by date desc — `LibraryHistoryViewModel2.swift:95` · `away`/`checkIn` elapsed + `stop` end−begin — `:76-87` | none · `HamDivider` between rows, none after last — `:117-119` · rows by id desc — `:71` · unused strings | v 4 · between rows · date desc · per-day total |
**Strings:** `历史预约` · `暂无历史记录` · `今天` · `共计%lld分钟` · `未知` · the 8 status names (§1).
**States:** Loading — iOS centred `ProgressView` filling the screen (`:86-87`); Android exposes no loading flag, so the empty card renders until data arrives and is indistinguishable from "no history". Empty — no
clusters, empty copy. Error (generic) — iOS shows the red retry banner **even for non-token errors** (`:88-92`) plus a toast (`LibraryHistoryViewModel2.swift:51`); Android toasts only (`LibraryHistoryViewModel.kt:49-56`). Token expired — red banner; `doRelogin()` re-runs `fastLogin()` then
refetches (`:23-39`). Totals unavailable — the `共计…分钟` suffix is hidden (`:32`).
**Divergence:** iOS groups by date with totals and a coloured status rail; Android is a flat id-descending list with a coloured status word. Nav titles differ — iOS `历史预约` (hardcoded) vs Android `历史记录`.

---

### 9. Settings (设置) — platforms: both
**Purpose:** Account, captcha notice, starred seat, local data, statistics, diagnostics.
**Entry:** Grid's 设置 cell (`FunctionButtonView.swift:90`) / `library/setting` (`FunctionCard.kt:66`).
**Layout:** nav `设置` / `图书馆设置` → pad `16`, cards gap `8`. iOS order: account → captcha → starred seat → analytics → local data. Android: account → captcha → starred seat → local data → statistics (only when the remote flag is on) → 其他. Each card is a `HamCardView` with a bottom-leading watermark; the account
card ends in a `48`-high, r `12`, `blue@0.85f` login bar on Android and a plain text `重新登录` on iOS.
**Blocks:** 1 **Account** — id row + login button; iOS shows `ProgressView` + `正在登录` and disables the button (`AccountCard.swift:69-81`), Android has no per-card state (`:99-117`). 2 **Captcha** — title, subtitle, blue accent line (`CaptchaCard.swift:13-22`, `:127-140`). 3 **Starred seat** — iOS lists one
every-day entry with a `每天` chip (`PreferredSeatCard.swift:54-70`); Android shows `设置了%1$d项` + `设置`, or `未设置` + `去设置` (`:150-184`). 4 **Statistics** — Android renders the card only when `analyticsToggle`
is true (`:255`). 5 **Local data** — date line + update button; Android adds `复制当前数据` (`:208-247`). 6 **Other (Android)** — masked serial + two-step reset (`:288-330`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| container · card · title/sub | `ScrollView` + `VStack.padding(.horizontal)` 16 — `LibrarySettingView2.swift:15,:25` · `16`/`.padding()` — `AccountCard.swift:95,:86` · `.bold` — `:53` · `.caption` 12 — `:55` | scroll page, pad 16, gap 8 — `:79-83` · `16.dp`/`16.dp` — `Card.kt:48,:53` · `bodyBold` — `Card.kt:81` · `caption` 12 — `:83` | pad 16, gap 8 · 16/16 · 16 bold · 12 |
| watermark | `156` account, `142` captcha, `128` seat + local data, `gray@0.15` bottom-leading — `AccountCard.swift:89-92`, `CaptchaCard.swift:27-31`, `PreferredSeatCard.swift:90-93`, `UpdateBasicInfoCard.swift:87-91` | `200.dp` captcha, `250.dp` seat + other, `offsetX 48.dp`, `ham_gray@0.25f` — `:131-132,:145-148,:282-285` | per-card sizes, `@0.15` bottom-leading |
| label width · primary button | none · plain text `重新登录` — `:79` | `65.dp` — `:94,:294` · 48, r 12, `blue@0.85f`, `headlineBold` white, h-pad 12 — `:100-116` | 65 · 48, r 12, blue @0.85 |
| accent note · weekday chip · link · switch | `该版本已默认开启验证码识别功能` `.blue` — `CaptchaCard.swift:22` · `每天` white on blue, r 6, pad v2/h6 — `PreferredSeatCard.swift:63-70` · `去设置` `NavigationLink` — `:84` · system `Toggle` — `AnalyticsCard.swift:16` | `已默认开启验证码识别` `body` — `:134-138` · none · `设置`/`去设置` `body` blue — `:163-167,:178-182` · `HamSwitch` `ham_green`/`ham_lightGray` — `:271-273` | blue 16 · blue, r 6, pad 2/6 · blue 16 · `#34C759` |
| update icon · loading · destructive · masking | `building.columns.fill` 16×16 — `UpdateBasicInfoCard.swift:73-76` · `HStack(spacing:4)` + `正在更新` — `:58-62` · none · none | `Book` 16, gap 8 — `:215-220` · `Row(gap 4)` + `正在更新基本信息` — `:195-204` · two-step, `body` `ham_red` — `:307-330` · first 6 chars → `******`, `caption` — `:290-302` | 16, gap 8 · spinner + label · two-step, red · masked serial |
**Strings:** `设置` · `账号信息` · `你的图书馆系统认证信息` · `学号` · `正在登录` · `重新登录`/`登录` · `登录成功` · `验证码设置` · `设置验证码账号信息、是否使用验证码` · `该版本已默认开启验证码识别功能` · `收藏座位` · `使用快速预约或自动预约时将自动选用收藏座位` · `未设置` · `每天` · `去设置` · `设置了%1$d项` · `统计服务` · `查看图书馆可用程度` · `开启统计服务` · `本地数据` · `本地存储的图书馆数据可以让你更快查看图书馆房间` · `正在更新` · `数据日期: %@` · `暂无` ·
`从图书馆更新数据` · `复制当前数据` · `其他` · `更改基础数据、诊断账号状态` · `登录序列号` · `重置账号状态`/`确认重置账号状态` · `操作成功`.
**States:** Login in flight/success/failure — iOS spinner then toast (`:40`), Android toast only (`LibrarySettingViewModel.kt:52-68`). Starred seat set/unset — row list vs `未设置`. Local-data refresh —
spinner replaces the date line + button (iOS) or both buttons (Android). Local-data date — iOS always reads `暂无` because `lastUpdateDate` is never assigned (`…UpdateBasicInfoCardViewModel:16`). Statistics gate —
iOS always shows the card, Android only when the remote flag is true (`:255`). Reset account — Android only. Empty/error — toasts.
**Divergence:** Android adds the 其他 card and `复制当前数据`; iOS adds the data-date line and per-card watermarks. Three cards use different copy (`账号信息` vs `图书馆账号信息`, `验证码设置` vs
`验证码识别设置`, and the captcha note).

---

### 10. Preferred / starred seat (首选座位设置) — platforms: both (+ Android insert)
**Purpose:** Configure the seat, time window, weekday and attribute filters used by quick/auto booking.
**Entry:** `libraryPreferredSeatSetting` (`PreferredSeatCard.swift:83-85`) / `library/setting/starred-seat` + `.../insert` (`LibrarySettingView.kt:163`).
**Layout:** pad `16`, gap `16`. iOS: card `座位` (seat + location, or `设置座位`), card `时间` (two inline pickers `80×50`), card `附加条件` (two `仅` + icon switches), then `已自动保存`. Android: explainer card `首选座位是什么`, then a `座位列表` card whose header carries a pencil and `＋`, rows of
`seatNum / location / HH:mm-HH:mm` with a yellow weekday chip and `删除`, or `未添加座位`. The insert screen adds `座位`, `持续时间` (begin/end chips), `星期` (7 chips `32` + `每天`), and `确定`.
**Blocks:** 1 **Explainer (Android)** (`StarredSeatSettingView.kt:56-69`). 2 **Seat** — iOS card, empty → `设置座位` (`LibraryPreferredSeatSettingView.swift:23-52`); Android list card with an edit toggle and `＋` (`:71-119`). 3 **Time** — iOS inline pickers, row pad leading `-8` (`:66-85`); Android edits the window per
entry (`:96-134`). 4 **Attributes** (`:102-128`, `:220-256`). 5 **Weekday (insert)** — 7 chips plus `每天`, which sets `weekday = 8` (`InsertStarredSeatView.kt:139-185`). 6 **Commit** — iOS auto-saves with `已自动保存` (`:134-138`); Android `确定`, enabled only when a seat exists and `endTime > beginTime` (`:196`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| container · card · title/sub | `ScrollView` + `VStack(leading).padding()` 16 — `:21-22,:140` · 16/16 — `CardView.swift:22-23` · `.semibold`/`.caption` — `:60,:64-65` | scroll page, pad 16, gap 16 — `StarredSeatSettingView.kt:51-55` · 16/16 — `Card.kt:48,:53` · `bodyBold`/`caption` — `:81,:83` | pad 16, gap 16 · 16/16 · 16 bold / 12 |
| seat no. · location · empty CTA · pickers | `.body.bold()` 17 — `:29-31` · `.body` 17 multiline — `:34-36` · `设置座位` — `:39` · inline `UIDatePicker` `80×50`, row pad leading `-8` — `:66-85` | `bodyBold` 16 list, `title` 24 insert — `:138`, `InsertStarredSeatView.kt:71` · `caption` 12, 1 line, ellipsis — `:143-147` · `去选择`/`更改` — `:87-89` · chips + dialog, both 08:00–23:00 — `:106-130` | 17 / 24 · 12, 1 line · `去选择` · 80×50 inline |
| attribute icons · weekday chip · selector | `battery.100.bolt` green / `sun.max.fill` orange, `.title3` — `:102-128` · none · none | `BatteryChargingFull` `ham_green` / `WbSunny` `ham_orange` — `:220-256` · yellow, bg `@0.2f`, r 6, pad v4/h6 — `:166-172` · `32.dp`, r 8, `color@0.2f`, FlowRow gaps 8/4 — `InsertStarredSeatView.kt:145-160` | 20, green / orange · yellow r 6 · 32, r 8 |
| edit affordances · empty state · save | none · `设置座位` · none — auto-save + `已自动保存` caption, centred, pad top 16 — `:134-138` | pencil 24 / add 28 / `取消编辑` / `删除` red — `:87-119,:186-190` · `未添加座位` `caption` secondary — `:197-203` · `确定` 48, r 12, `blue@0.85f` (disabled `gray@0.5f`) — `:196-216` | as Android · `未添加座位` · auto-save, no button |
| list ordering | n/a | `weekday*1000 + (beginTime − 08:00)/60000` — `:127-129` | weekday then start |
**Strings:** `首选座位设置` · `添加首选座位` · `座位` · `当目标座位被占用时，Ham将会预约最近的座位` · `设置座位` · `时间` · `Ham会按照你的预定时间准确进行预约` · `附加条件` · `当首选座位被占用时，Ham将会按照附加条件预约附近的座位` · `仅` · `已自动保存` · `首选座位是什么` · `座位列表` · `未添加座位` · `偏好` · `电源`/`靠窗` · `取消编辑` · `删除` · `更改`/`去选择` · `持续时间` ·
`开始时间`/`结束时间` · `星期` · `每天` · `确定` · `存在时间冲突的座位` · `首选座位`.
**States:** No seat — iOS `设置座位`, Android `未添加座位` with the add button still shown and insert `确定` disabled. Seat configured — tap → Select seat; Android exposes `删除` in edit mode. Seat picked — iOS
notification → immediate write (`:43-49`); Android `savedStateHandle["selectedSeat"]` (`InsertStarredSeatView.kt:52-59`). End ≤ begin — iOS pushes end to `begin + 30 min` (`:69-71,:78-80`),
Android disables `确定` (`:196,:203-206`). Overlapping entry — Android only, toast `遇到了错误 / 存在时间冲突的座位` and reject (`InsertStarredSeatViewModel.kt:56-67`). Empty list in edit mode — Android hides the pencil (`:109`). Persistence — write-through on every mutation. No loading or error state.
**Divergence:** iOS stores exactly one every-day entry with inline pickers and auto-save; Android stores a per-weekday multi-entry list with a separate insert screen, a weekday grid, an explicit `确定`, and a conflict check.

---

### 11. Board / announcements (图书馆公告) — platforms: both
**Purpose:** Render the library notice-board HTML.
**Entry:** Banner page `-1` → `libraryBanner` (`LibraryMainViewBannerCard.swift:87`); Android the whole banner card → `library/board` (`LibraryMainView.kt:54-58`).
**Layout:** nav `图书馆公告` (Android only) → `ScrollView`, pad `16`, top-aligned rendered HTML.
**Blocks:** 1 **Nav title** `library_announcements` (`LibraryBoardView.kt:26`). 2 **Body** — iOS `Text(AttributedString)` from `NSAttributedString` HTML (`LibraryBannerView.swift:16-19`, `LibraryBannerViewModel.swift:27-35`); Android `android.widget.TextView` +
`Html.fromHtml(…, FROM_HTML_OPTION_USE_CSS_COLORS)` (`LibraryBoardView.kt:28-37`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| container · padding · alignment | `ScrollView` + `Text` — `LibraryBannerView.swift:16-19` · `.padding()` 16 — `:19` · `.leading`, pinned `.top` — `:18,:21` | `HamNavigationView` + `Column` — `:26-27` · `padding(horizontal 16.dp)` — `:27` · top-start | nav + scroll body · 16 · leading, top |
| nav title · back · font | none · inherited from the stack · from the HTML payload | `library_announcements` — `:26` · chevron, `ham_blue`, start margin 16 — `NavigationView.kt:130-148` · from the payload (raw `TextView` defaults) | `图书馆公告` · chevron, blue, 16 · from the payload |
**Strings:** `图书馆公告` · `遇到了错误` (Android toast).
**States:** Loading — none, blank until the HTML arrives. Loaded — rendered HTML. Empty payload — blank. Error — iOS silent (`catch { return }`, `LibraryBannerViewModel.swift:36-38`); Android toasts `遇到了错误` (`LibraryBoardViewModel.kt:42-47`).
**Divergence:** iOS has no nav title and fails silently; Android has a title and toasts.

---

### 12. Print (打印) — platforms: both
**Purpose:** Pick a file to send to a library printer, and list available print stations.
**Entry:** Home print card → iOS `Route.libraryPrint` (`Route.swift:33`, composed at `:231`); Android `LibraryRoutePath.PRINT` (`PrintFunctionCard.kt:46`).
**Layout:** nav `打印` → h pad `16` → spacer `16` → drop zone `168` high, r `16`, tint, centred `＋` `72` + `添加文件` → share hint (`caption`, pad v `8`) → spacer `36` → `打印机位置` `bodyBold` → spacer `8` →
state switch: spinner + `加载中`, `加载异常`, or a list (gap `8`, pad bottom `16`) of rows r `16`, `gray@0.15`, pad `16`: `🖨48` + name (2 lines) + status (3 lines); empty shows `没有找到打印机`.
**Blocks:** 1 **Drop zone** → iOS `.printFileSourcePicker` (`:49-51`), an action sheet `添加文件` offering `照片图库` · `文件` · `取消` (`PrintFileSourcePicker.swift:59-68`) then `.fileImporter` (`:70`); then `router.push(.printPrepare(url:name:))` (`LibraryPrintView.swift:50`). Android
`ActivityResultContracts.OpenDocument()` MIME `*/*` (`LibraryPrintView.kt:64,:83`); the URI is bundled into `savedStateHandle` and the app navigates to `PrintRoutePath.MAIN` (`:66-73`). 2 **Share hint** (`LibraryPrintView.swift:27-30`; `LibraryPrintView.kt:101-104`). 3 **Section header** (`LibraryPrintView.swift:34-37`; `LibraryPrintView.kt:107`).
4 **Printer list** (`LibraryPrintView.swift:78-137`; `LibraryPrintView.kt:134-152`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| page bg · padding · top spacer | `ham_bg_b1Color` — `LibraryPrintView.swift:47` · `16` — `:45` · `16` — `:23` | `ham_bg_b1` — `LibraryPrintView.kt` · `16.dp` — `:80` · `16.dp` — `:81` | `#F9F9F9` · 16 · 16 |
| nav title | `打印` — `:48` | `打印` — `:107` | `打印` |
| drop zone h · r · tint | `168` — `:69` · `16` — `:71` · `ham_brand_library_tint` — `:72` | `168.dp` · `16.dp` · `ham_blue @0.15f` — `:85-90` | 168 · 16 · `#007AFF` @0.15 |
| add icon · stack gap · label | `plus` `72` — `:62-63` · `VStack(spacing: 8)` — `:61` · `.body` — `:64` | `Add` `72.dp` — `:92-95` · centred — `:85-90` · `body` 16sp — `:96` | 72 · 8 · 16 |
| share hint · hint pad | `.caption` `.secondary` — `:27-29` · v `8` — `:30` | `caption` 12sp, secondary — `:101-104` · v `8.dp` | 12 · v 8 |
| header spacers · header | `36` before — `:32`, `8` after — `:39` · `.body.bold` — `:34-36` | `36.dp` before, `8.dp` after — `:106-108` · `bodyBold` 16sp | 36 · 8 · 16 bold |
| loading row | `HStack(spacing: 2)` = `ProgressView` + `.body` — `:83-89` | `Row(spacedBy 2.dp)` = spinner + `body` — `:111-118` | shared labelled spinner: 20 / 2, `body` 17, gap `space.3` **8** ([§11](#11-shared-components-共享组件)) — both ship gap 2 |
| printer row · icon | `padding(16)`, r `16`, `Color.gray @0.15` — `:131,:134-136` · `printer` `48`, `Color.gray` — `:113-115` | `padding(16.dp)`, r `16.dp`, `ham_gray @0.15f` — `:128-133` · `Print` `48.dp`, `ham_gray` — `:134-137` | pad 16 · r 16 · gray @0.15 · 48 |
| name / status · row gap | `.body.bold` `lineLimit(2)` / `.caption` `lineLimit(3)` — `:118-126` · `spacing: 8` — `:112` | `bodyBold` 2 lines / `caption` 3 lines, ellipsis — `:139-152` · gap `8.dp` | 16 bold · 12 · 2 / 3 lines · gap 8 |
| list container · bottom | `VStack(spacing: 8)` — `:102` · spacer `16` — `:43` | `LazyColumn`, gap `8.dp`, pad bottom `16.dp` — `:125-126` | gap 8 · bottom 16 |
| load trigger | `.task { await vm.updatePrinterList() }` — once — `:52-54` | `SideEffect { vm.updatePrinterList() }` — re-fires on **every recomposition** — `:60-62` | once per appearance |
**Strings:** iOS `Localizable.strings:811-822` — `打印` · `在图书馆公共打印机打印` · `添加文件` · `或者通过其他应用分享文件到Ham中打印` · `打印机位置` · `加载异常` · `没有找到打印机` · `正在上传` · `上传失败` · `上传成功` · `请到支持的打印机执行作业` · `读取文件失败`. Android `feature/library/…/strings.xml:98-106` —
`library_print` · `library_print_in_library` · `library_print_tasks` · `library_printer_location` · `library_add_file` · `library_print_share_hint` · `library_no_printer_found`. Both are fully localised; **no string is missing on either side**.
**States:** `unload`/`Unload` renders nothing below the header (`LibraryPrintView.swift:80-81`; `LibraryPrintView.kt:168`). Loading → spinner + `加载中`. Loaded → list. Empty → iOS replaces the list with `没有找到打印机` (`:97-100`); Android appends it as a trailing list
item (`:157-165`). Load error → `加载异常` only; the captured `printerListErrorMessage` (`LibraryPrintViewModel.kt:28`) is never surfaced. File picked → iOS pushes `.printPrepare` (`LibraryPrintView.swift:50`), Android navigates to `PrintRoutePath.MAIN` (`LibraryPrintView.kt:64-74`); cancelled → unchanged.
**Divergence:** the share hint is two different strings — iOS `或者通过其他应用分享文件到Ham中打印`, Android `或者，通过其他APP（例如QQ/微信）分享文件到Ham中打印`. iOS picks the file behind an action sheet (`照片图库` / `文件`), Android opens the document
picker directly. Empty state: iOS replaces the list, Android appends. Android re-fetches the printer list on every recomposition. Android ships `library_print_tasks` (`打印任务`) for `PrintStatusCard.kt`, which has **zero call sites** — dead code, and no iOS counterpart exists or is needed.

> **Correction.** Earlier revisions of this document, `design-system.md` §8.1, `ui-parity.md` and `logic-parity.md` all stated that print is Android-only and that iOS ships only an unreferenced data layer. That was wrong. iOS has shipped the whole flow since
> `811c03f6` (2026-09-21): `Route.libraryPrint` and `Route.printPrepare` (`Route.swift:33-34`, composed at `:231,:234`), `LibraryPrintView.swift` (138 lines), `PrintPrepareView.swift` (330 lines), `PrintFileSourcePicker.swift`, `PrintPrepareRouteView.swift`,
> the `PrintActionExtension` share target, four unit-test files and one E2E suite. The iOS data layer is <ins>not</ins> dead. The two clients are near-identical; what remains is the divergence list above.

---


## 5. Sport (运动 Sport)

Normative ("build it this way"), condensed from `/tmp/audit/sport-a.md` + `/tmp/audit/sport-b.md`, plus
direct source measurement for §6, §7, §13 (not covered by the audit). Every number carries one
`file:line`. Platform columns are **evidence**; `normative` is the requirement. `Divergence:` lines are
migration tasks, not spec.

`IOS/` `repos/ham-ios/Ham/iOS/ui/sport/` · `IOSSH/` `repos/ham-ios/Ham/iOS/ui/common/` ·
`AND/` `repos/ham-android/android/feature/sport/…/ui/` · `ANDCORE/` `android/core/ui/…/common/ui/` ·
`IS:` `Ham/zh-Hans.lproj/Localizable.strings` · `AS:` feature `res/values/strings.xml` ·
`ASCOMMON:` core-ui `res/values/strings.xml`. Citations use the **basename**: `.swift` = iOS,
`.kt` = Android. pt and dp are 1:1.

**Selection rule.** Where the platforms drift and neither value is functionally required, normative is the
**Android** value — Android routes colour/type through design-system tokens, iOS writes raw SwiftUI
literals. Where Android *omits* a treatment iOS has (padding, state, guard, affordance), the **iOS** value
is normative.
**Terminology.** The app standard is **预约**; this module ships **预定** in 8 zh-Hans strings. The normative
column uses 预约; every 预定 occurrence sits on the relevant `Divergence:` line. **场馆预约** is the WeChat
mini-program's proper noun and is never altered.
**Type.** Android `HamFontStyle` (`ANDCORE/config/Font.kt`): `title` 24 `:46` · `title2` 20 `:47` · `body`
16 `:49` · `headline` 14 `:50` · `caption` 12 `:51` · `caption2` 11 `:52`; `bodyBold` `:28` ·
`captionBold` `:41` · `headlineBold` `:32`; `body`/`caption` default to `ham_text_primary`. iOS semantic:
`.title` 28 · `.title2` 22 · `.body` 17 · `.caption` 12 · `.caption2` 11.
**Colour.** `ham_brand_sport` = `ham_green` #34C759 (`ANDCORE/config/Color.kt:78-79`; iOS `Color.green`
`Color+Ham.swift:37`) · `ham_blue` #007AFF `colors.xml:79` · `ham_red` #FF3B30 `:72` · `ham_gray` #8E8E93
`:84` · `ham_lightGray` #EDEEEF `:68` · `ham_text_primary` black `:63` · `ham_text_secondary` gray `:64` ·
`ham_bg_b1` #F9F9F9 `:65` · `ham_bg_b2` white `:66` (iOS `ham_bg_b1Color`/`b2Color` `:22-23`).
**Card** — padding 16, radius 16, surface `ham_bg_b2`, header→body gap 8 (`Card.kt:48-85`); iOS writes
`ham_bg_b2Color` + `cornerRadius(16)` + `.padding()` inline. **Court-number badge** — 36 box, radius 6,
`ham_text_primary` fill, `title` 24sp in `ham_bg_b1` (`StadiumAreaIcon.kt:28-34`).

### 1. Sport home (运动) — platforms: both
**Purpose:** Module landing tab — announcement banner, one-tap 预约 of a remembered court, in-flight orders
with payment deadlines, three function entries.
**Entry:** Tab 运动. iOS `IOS/main/SportMainView.swift`; Android `AND/main/SportMainView.kt`.

**Layout:**
```
▓▓ title 运动 · pinned
░░ SCROLL · VStack · pad 16 · gap 8
  1 Banner carousel          h 180 · r16 · paged · dots bottom-pad 16
  2 Quick 预约 card  dynamic · IF starred · badge 36 · 收藏 chip · 今天|明天 · CTA
  3 Current-order card ×0..n · gap 8 · status strip + body pad 12
  4a 查看场馆 h160 │ 4b1 订单中心 h76 · gap 8 · 4b2 设置 h76
```
**Blocks:** 1 **Banner** — pager; page 0 is always a decorative 公告 entry (sport icon 72 + title over a
tiled 5×10 icon grid), pages 1…n are fetched bulletins capped at **5**; *tap* page 0 → §9, page n → §10.
2 **Quick 预约 card** — conditional on a starred court: badge, 收藏 chip, type + venue + time, 今天/明天
segmented picker, 预约 CTA → §6. 3 **Current-order card** — one per live order: status strip, badge, venue +
time, payment block when unpaid; 去支付 → §5. 4 **Function card** — 查看场馆 → §2 · 订单中心 → §8 · 设置 → §11.

**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| page pad · block gap | 16 h only `SportMainView.swift:33` · divider padV 8 `:24` | 16 all `:31` · `spacedBy(8)` `:32` | 16 all · 8 |
| banner h · radius · surface | 200 · 16 · `ham_bg_b2Color` `SportMainViewBannerCard.swift:141,139,138` | 180dp · card default `SportMainViewBannerCard.kt:73` | 180 · 16 · `ham_bg_b2` |
| page-0 icon · tint · title | `sportscourt.fill` 72 `:46-47` · `.green` `:55` · `.title.bold()` 28 `:51` | `Stadium` 72dp `:106` · brand `:107` · `title` 24 Bold `:108` | 72 · `ham_brand_sport` · 24/Bold |
| grid icon · tint | 36 · 6×9 · spacing 8 · gray@0.25 `:66-71` | 56dp · 5×10 · no spacing · `ham_gray`@0.2 `:87-94` | 56 · 5×10 · `ham_gray`@0.2 |
| bulletin title · date · body · 查看详情 | bold 1-line `:95-97` · `.caption` t2 `:98-101` · 5-line limit `:103-105` · link + arrow blue offset(−16,−4) `:111-118` | `bodyBold` `:122` · `caption` `:123` · no limit `:125` · absent | `bodyBold` · `caption` secondary · 5 lines · link present |
| dots · auto-advance | native always-bg `:137` · 5 s timer `:38`,`:127-135` | 8dp circle pad 2 DarkGray/LightGray bottom-pad 16 `:141-148` · none | 8 · bottom-pad 16 · 5 s loop |
| order card radius · surface · status strip | 16 · `ham_bg_b2Color` `SportMainViewCurrentOrderCard.swift:99,98` · white bold pad v8 on `Color.blue` `:21-26` | card default · `bodyBold` white on `ham_blue` pad v8/h12 `SportMainViewCurrentOrderCard.kt:65-73` | 16 · `ham_bg_b2` · `bodyBold` white on `ham_blue` pad v8/h12 |
| court badge · venue line · time line | 32 r6 white `title2` rounded bold `:29-35` · `{type} \| {stadium.title}` `:37-38` · `yyyymmddhhmm-HHmm` `:40-41` | 36 r6 `StadiumAreaIcon` `:28-34` · `{type.title} \| {address}` `:82-84` · `yyyyMMdd HHmm-HHmm` `:87-90` | 36 r6 · `{type} \| {stadium.title}` · `yyyyMMdd HHmm-HHmm` |
| watermark · body pad · 去支付 | SF Symbol 128 gray@0.15 `:83-96` · h+bottom 16 top 8 `:76-77` · white pad 8/8 blue r6 `:63-68` | absent · 12 all `:77` · `ham_blue` r8 pad 8/8 `headline` white `:139-155` | 128 gray@0.15 · 12 all · r8 pad 8/8 `headline` white |
| pay divider · deadline · out-of-window | `Divider()` `:46` · `.caption` `.red` `:50-51` · bold + `.caption` `:53-56` | `HamDivider` pad v8 `:92` · `caption` `ham_red` `:110-117` · `captionBold`+`caption` `:119-134` | pad v8 · `caption` `ham_red` · `captionBold`+`caption` |
| quick card pad · radius · 收藏 chip | 16 · 16 `SportMainViewQuickOrderCard.swift:104-107` · `caption` orange pad 4 orange@0.15 r4 `:37-47` | card default · orange@0.25 r4 pad h8/v2 star 18dp `:93-116` | 16 · 16 · orange@0.25 r4 pad h8/v2 star 18 |
| chevrons · picker · 预约 CTA | `arrowtriangle.right.fill` 8 green `:51-53` · native segmented `:83-90` · pad v16 maxWidth green@0.25 r8 `:94-99` | `ArrowRight` 24 green offset −8 `:119-126` · `HamHorizontalPicker` `:142-152` · green@0.20 r8 pad v12 `bodyBold` `:171-185` | 8 brand · segmented · brand@0.20 r8 pad v12 `bodyBold` brand |
| function card h · gap · left tile · right tile | 150 · 8 · icon 64 green@0.13 r16 bold+triangle 8 `.caption2` 11 `SportMainViewFunctionCard.swift:132,13,19-46` | 160dp · 8 · icon 72dp brand@0.15 r16 `body` Bold `caption` 12 `SportMainViewFunctionButtonCard.kt:53,54,67-94` | 160 · 8 · icon 72 · brand@0.15 r16 · `body` Bold · `caption` 12 |

**Strings:** `运动` `AS:3` · `场馆预约公告` `AS:26` · `查看详情` · `查看场馆` `AS:27` · `查看空余的运动场馆预约`
`AS:28` · `订单中心` `AS:29` · `你的历史预约记录` `AS:30` (correct twin already at `IS:539`) · `设置` `AS:31` ·
`场馆预约选项` `AS:32` · `收藏` `AS:34` · `今天` `AS:35` · `明天` `AS:36` · `预约` `AS:37` · `去支付` `AS:38` ·
`请于%1$s前完成支付` `AS:40` · `当前未处于可支付时间` `AS:41` · `请于%1$s-%2$s完成支付` `AS:42`. Status names are
data-driven (iOS `status.name.localized` `:20`; Android `status.displayNameResID` `:66`).

**States:** unload — page 0 only, dots hidden, no cards. loading — no indicator on either platform
(`withAnimation` `:19-25`; `LaunchedEffect` `:64-68`). error — banner failure is silent
(`Log.e("获取Sport Banner")` `:27-29`). success — pages 1…n + cards. empty orders — section omitted; unpaid
in window — red deadline + 去支付; out of window — `当前未处于可支付时间` + range, no button.

**Divergence:** Block order — iOS Banner → CurrentOrder → Quick → Function (`SportMainView.swift:19-27`);
normative takes Android's Banner → Quick → CurrentOrder → Function (`:80`,`:61`). Auto-advance, 查看详情, the
5-bulletin cap, the watermark and the segmented picker are iOS-only and become normative; Android's page 0
is untappable (`SportMainViewBannerCard.kt:76-110`) and must open §9. Android's venue line reads
`stadium.address`, iOS `stadium.title`. **预定 → 预约**: the CTA (`SportMainViewQuickOrderCard.swift:94`,
`AS:37`), `场馆预定公告` (`SportMainViewBannerCard.swift:50`, `AS:26`), `你的历史预定记录` (`IS:538`, `AS:30`),
`场馆预定选项` (`AS:32`).

### 2. Select area & order (选择场地并预约) — platforms: both
**Purpose:** The main booking screen — pick a sport type and a date in a pinned header, browse venue cards,
drill into a court, select contiguous slots, submit from a pinned footer. The same body is reused by §3 with
a different footer CTA.
**Entry:** 查看场馆 on the home function card → iOS `Route.sportOrder`, Android `SportRoutes.Order`.

**Layout:**
```
▓▓ PINNED HEADER — outside the scroll container · pad top = status bar + toolbar · h 12 · bottom 12 · gap 8
  A sport-type button          bodyBold + caret 8, rotates
  B chip strip IF expanded || no type selected · h-scroll · pad v6 · lead 16 · gap 20 · icon 32 + caption
  C date row   [◀ 36 r8] [yyyy年M月d日] [▶ 36 r8] · gap 8
░░ SCROLLING BODY — ham_bg_b1 · pad v16 · cards pad h16 · gap 8
  ┌─ venue card · ham_bg_b2 · r16 · pad 16 ───────────────────────────────┐
  │ img 104 r12 │ title bodyBold · address caption · 已闭馆 red IF status==0 · ¥N起
  │ ── collapsed: 预约时间段 + time chips h40 r8 ───────────────────────── │
  │ ── expanded:  court rows · gap 8 · divider between ─────────────────── │
  │   badge 28 r6 │ brief chips + 详情 ▸   (collapsed row)                 │
  │               │ slot grid  tile 120w × 50h r8 gap 8  (expanded row)   │
  └───────────────────────────────────────────────────────────────────────┘
▓▓ PINNED FOOTER — visible IF selection non-empty
   r8 · ham_bg_b1@0.95 over ham_gray@0.2 · pad 8 · safe-area
   已选择 / {venue}-{N}号场 / {date} {start}-{end}  ‖  [ 预约 ]
```
**Blocks:** 1 **Sport-type button** — `bodyBold` title, placeholder `请选择运动类别`/`请选择场所` when unset;
trailing caret in `ham_brand_sport`. *Tap:* toggles the strip with animation. 2 **Chip strip (conditional)**
— visible when expanded **or** when no type is selected. H-scroll: icon 32 + `caption`; selected
`ham_brand_sport`, unselected `ham_gray`; fallback icon for unknown types. *Tap:* set type, collapse,
refetch venues, clear the held selection. 3 **Date row** — prev/next day buttons (36, r8, brand@0.15) around
the date label. *Tap arrow:* ±1 day, refetch, clear the selection. Default date rolls to tomorrow when now is
past 18:00 (`SportSelectAreaViewModel.swift:29-30`). 4 **Venue card (scrolling)** — image, title, address,
`已闭馆` when `status == 0`, `¥N起`. *Tap anywhere:* toggles expand; expanding fetches that venue's courts
first. 5 **Collapsed summary strip** — `预约时间段` + time chips (40 box, r8, brand@0.2 when bookable else
`ham_gray`@0.2). 6 **Expanded court rows** — gap 8 with a divider between rows. Empty → `无可用时间段`;
loading → spinner; error → `加载时遇到了错误`. 7 **Court row** — leading court-number badge 28 r6
(`ham_text_primary` fill, `title2` 20sp in `ham_bg_b2`). Collapsed: brief chips (each carrying its own
`canAppointment`) + `详情`. Expanded: the slot grid. 8 **Slot grid** — flow layout, tile 120 wide, 8 gap on
both axes.
9. **Slot tile — unselected** — radius 8, height 50, surface `ham_brand_sport`@0.15 when bookable else
   `ham_gray`@0.15. Body row: start/end times (`caption2` 11) → vertical divider 16 @`ham_gray`0.2 → `学专`
   badge when student-exclusive → price → divider → `余{n}`. Bottom strip 15 tall, filled with the same colour
   at full alpha, holding `选择` + a right arrow, both white.
10. **Slot tile — selected** — the whole tile becomes a solid `ham_brand_sport` fill (alpha 1), same radius 8
    and height 50. Content is replaced by a centred white row: start/end times (`caption2`) → white vertical
    divider 12 → `已选择` (`caption`, white). The bottom CTA strip disappears.
11. **Slot tap** — no-op when the slot is not bookable. Selecting on a *different* court clears the held
    selection first. A slot that is not contiguous with the held run, or that would exceed
    `maxAppointTimerPeriod`, clears the run before being added. When the venue allows one-tap ordering
    (`maxAppointTimerPeriod == 1`) the tile label is `预约` and tapping an unselected tile inserts it **and
    submits immediately**, bypassing the footer.
12. **Pinned footer** — `已选择`; `{venue}-{areaNo}号场`; `{date} {min start}-{max end}`, all `caption`.
    Trailing `预约`. *Tap:* submit.

**Values — header:**

| Element | iOS | Android | normative |
|---|---|---|---|
| header surface · pad · gap | none `SportSelectAreaViewHeader.swift:16-118` · h+top 16 bottom 8 `:116-117` · 8 `:78` | `ham_gray`@0.10f `SportSelectHeaderView.kt:88` · top status+toolbar, h 12, bottom 12 `:90-92` · `spacedBy(8)` `:93` | `ham_gray`@0.10 · h 12 · bottom 12 · 8 |
| type title · caret · rotation | bold 17 `:26` · `arrowtriangle.right.fill` 8 `.green` `:28-30` · −90° collapsed / 0° expanded `:31-32` | `bodyBold` `:106` · `ArrowRight` no size brand `:114-117` · 270° expanded / 0° collapsed `:110` | `bodyBold` · 8 brand · 0° collapsed → 270° expanded |
| strip surface · radius · padding · inner | gray@0.15 · 8 · 16 `:74-75`,`:72` · spacer 8 `:40`,`:70` | `ham_gray`@0.15 · 8 · v6 `:125-127` · lead 16 `:131`, gap 20 `:133`, item v8 `:134` | `ham_gray`@0.15 · 8 · v6 · lead 16 · gap 20 · item v8 |
| chip icon · label · gap · colour | 24 in a 36 frame `:60-61` · `.caption` `:63` · 4 `:52` · `.green`/`.gray` `:65` | 32 `:158` · `caption` `:164-167` · 8 `:163` · brand/`ham_gray` `:148-149` | 32 · `caption` · 8 · brand/`ham_gray` |
| date arrow · row gap · label | 32×32 green@0.25 r8 `:85-89` · none `:79` · native `DatePicker` `:92-93` | 36 brand@0.15 r8 `SportSelectDateView.kt:40-49` · 8 `:36` · `HamDatePickerButton` `yyyy年M月d日` `:51-53`, `AS:84` | 36 · brand@0.15 · r8 · 8 · `yyyy年M月d日` |

**Values — venue card, court row, slot:**

| Element | iOS | Android | normative |
|---|---|---|---|
| card radius · pad · surface · image | 16 · 16 · `ham_bg_b2Color` `SportSelectAreaViewStadiumAreaCard.swift:107,104,106` · 85×85 r8 `:29-34` | 16 · 16 · `ham_bg_b2` `SportSelectItemView.kt:45,48,47` · 104dp r12 `SportSelectItemTitleView.kt:45-58` | 16 · 16 · `ham_bg_b2` · 104 · r12 |
| title · address · 已闭馆 · price | bold `:37-38` · `.caption` `:39-40` · `.caption` `.red` `:42-46` · `"¥\(%g)起"` concatenated `:49` | `bodyBold` `:66` · `caption` `:67` · `caption` `ham_red` `:68-70` · `¥%1$s起` `AS:68`, `:84-88` | `bodyBold` · `caption` · `caption` `ham_red` · `¥%1$s起` |
| expand caret · title→summary gap | `chevron.down.circle` rotate 180 `:53-54` · 16 `:60` | `ExpandCircleDown` 24 rotate 180 `:72-82` · 8 `SportSelectItemView.kt:58` | 24 · rotate 180 · 8 |
| summary label · time chip | `预约时间段` `.caption` `:64-65` · `.caption` pad 6 (green\|gray)@0.25 r8 `SportVerticalTimePeriodCell.swift:27-30` | `sport_select_time_slot` `:66` · `caption` 40dp (brand\|gray)@0.2 r8 `SportSelectItemTimeView.kt:26-33` | `预约时间段` `caption` · 40 · @0.2 · r8 |
| row gap · leading gap · badge · 详情 | 8 `:79` · 16 `SportOrderViewAppointmentAreaCell.swift:21` · 24 r6 `ham_bg_b1Color` on `ham_text_t1Color` `:22-29` · `.caption` + arrow 8 green `:53-60` | 8 `SportSelectItemStadiumAreaDetailView.kt:35` · 8 `SportSelectItemStadiumAreaAppointmentAreaView.kt:49` · 28 r6 `ham_text_primary` `title2` `:50-59` · `caption` + arrow 20 brand offset −4 `:81-91` | 8 · 8 · 28 r6 · `caption` + arrow 20 offset x −4 |
| grid · tile h · radius | `LazyVGrid` adaptive 110–150 `:68-70` · 56 `:213` · 6 `:151`,`:210` | `FlowRow` gap 8/8, tile 120 `SportSelectItemAppointmentDetailView.kt:44-54` · 50dp `SportSelectItemCourtListDetailItemView.kt:57` · 8 `:55` | flow · tile 120 · gap 8 · 50 · 8 |
| tile surface unselected / selected | (green\|gray)@0.25 `:206-208` · `Color.green` `:150` | `color`@0.15f / @1f `:56` | brand or `ham_gray` @0.15 · brand @1 |
| tile text · padding · bottom strip | `.system(.caption, design:.rounded)` 12 `:146`,`:184` · top 4 / h 8 `:185-186` · implicit, (green\|gray) `:200`, `.caption` white `:193-197` | `caption2` 11 `:72`,`:89` · none · `Dimension.value(15.dp)` `:104` on `color` `:106`, `caption2` `ham_white` `:109-111` | `caption2` 11 · top 4 / h 8 · 15 · `color` · `caption2` white |
| 学专 badge | `.caption2` bold · `ham_bg_b2Color` on `ham_text_t1Color` · r4 · pad h4/v2 `:163-171` | `captionBold` · `ham_white` on black · r4 · **no padding** `:79-87` | `captionBold` white on `ham_text_primary` · r4 · pad h4/v2 |
| 余 · slot price · 已选择 | `余%lld` `.caption` `:181` · `¥%g` of `lightingPrice + price` `:174-176` · `.caption` rounded white `:143-146` | `余%1$d` `caption2` `AS:81`, `:95` · `¥${price}` `:89` · `caption` `ham_white` `:136` | `余%1$d` `caption2` · `¥{lightingPrice + price}` · `caption` white |

**Values — footer:**

| Element | iOS | Android | normative |
|---|---|---|---|
| surface · radius · padding · bottom inset | `VisualEffectBlurView(.systemThinMaterial)` `SportOrderViewFooter.swift:44-45` · 8 · 8 · `UIScreen.bottomSafeArea` `:46`,`:42`,`:47-48` | `ham_bg_b1`@0.95 then `ham_gray`@0.2 `SportSelectFooterView.kt:53-54` · 8 · 8 · system+navigation bars + 8 `:52`,`:47-55` | `ham_bg_b1`@0.95 + `ham_gray`@0.2 · 8 · 8 · system bars + 8 |
| text · line 2 · range | `.caption` `:25` · `已选择` + `%@-%lld号场` `:18-19` · first.start–last.end `:17-25` | `caption` `:64-67` · `已选择` + `{title} \| {address}` `:65` · min–max `:58-68` | `caption` · `已选择` + `%1$s-%2$d号场` · min(start)–max(end) |
| gap · CTA label | `Spacer()` `:27` · `预约` `:31` | `end.linkTo(btnRef.start, 8.dp)` `:61` · `sport_reserve` = 预定 `:87`, `AS:37` | 8 · **`预约`** |
| CTA padding · radius · fill · label | v12 · h16 `:33-34` · 8 · `Color.green` `:36-37` · `.white` `:32` | v4 · h16 `:83-84` · 8 · `ham_brand_sport` `:80-82` · `bodyBold` `ham_white` `:88-90` | v12 · h16 · 8 · `ham_brand_sport` · `bodyBold` white |
| transition | `.move(edge:.bottom)` `:49` | `slideInVertically{it/2}+fadeIn` `SportSelectOrderView.kt:138-139` | slide + fade |

**Strings:** `请选择运动类别` `IS:808` / `请选择场所` `AS:64` · `预约时间段` `IS:858`, `AS:65` · `已闭馆` `IS:647`,
`AS:66` · `¥%1$s起` `AS:68` · `加载时遇到了错误` `IS:203`, `AS:71` · `无可用时间段` `IS:684` · `详情` `IS:800`,
`AS:70` · `学专` `IS:629`, `AS:80` · `余%1$d` `AS:81` · `选择` `IS:839`, `AS:82` · `已选择` `IS:646`, `AS:69` ·
`%1$s-%2$d号场` `IS:468` · `预约` `IS:132`. Sport-type names are data (`sport_type_*_keyword` `AS:86-88`).

**States:** unload — header + venue list, footer hidden. loading (submitting) — full-screen spinner, header
and footer both removed (`SportOrderView.swift:25-27`; `SportSelectOrderView.kt:81-85`). error — §13 error
card with a retry returning to unload (`:28-33`, `:96-100`); venue-list and slot-detail failures are surfaced
**inline** as `加载时遇到了错误`. success — §13 success card. empty — `无可用时间段` inside an expanded court row;
footer hidden whenever the selection is empty.

**Divergence:** Caret rotation is inverted — iOS points the caret up when collapsed (`:31-32`), Android when
expanded (`:110`); normative is collapsed→right, expanded→up. The strip auto-opens when no type is selected
on iOS only (`SportSelectAreaViewHeader.swift:37`). iOS resets the held selection on type/date change
(`:43-49`,`:108-111`); Android does not (`:145-146`). iOS holds card-expand state in a per-card ViewModel,
Android in `remember` (`SportSelectItemView.kt:43`), so it is lost on configuration change. Tile height
56/radius 6 on iOS vs 50/8 on Android. Android shows only the base `price`; iOS adds `lightingPrice`, the
truer total, and is normative. iOS force-unwraps `stadiumAreaList.first(where:)`
(`SportOrderViewFooter.swift:15`) — it must not. **Terminology:** the footer CTA is `预约` on iOS and `预定` on
Android (`AS:37`); the slot strip reads `预约`/`选择` on iOS (`SportOrderViewAppointmentAreaCell.swift:192`) and
always `选择` on Android. `SportSelectAreaTimeHeaderCard.swift` is dead code (no call sites, blue chips) — do
not spec from it.

### 3. Select / picker (选择场地) — platforms: both
**Purpose:** The §2 body reused as a picker — the footer returns the selection to the caller instead of
placing an order.
**Entry:** iOS `Route.sportSelect(id:)` from §7; Android `SportRoutes.Select()`
(`SportStarredSeatSettingView.kt:84`).

**Layout:** identical to §2 — pinned header (blocks 1-3) + scrolling venue cards (4-7) — except the slot tile
is narrower, carries no price or remaining count, and the footer CTA is `确定`.

**Blocks:** 1-7 as §2. 8 **Slot tile (picker variant)** — width **100** (not 120), height 50, radius 8;
unselected surface is always `ham_brand_sport`@0.15, with **no grey/unavailable variant** because the picker
ignores availability; content is start/end times plus the `学专` badge only — no price, no `余`, no dividers;
bottom strip always brand-filled with `选择`. Selected tile is identical to §2 block 10. 9 **Slot tap** — every
tile toggles; no `canAppointment` guard, no one-tap order path. 10 **Footer** — as §2 block 12 but labelled
`确定`; *tap* writes the result back and dismisses.

**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| tile width · surface | adaptive 110–150 `SportSelectViewAppointmentAreaCell.swift:74` · `Color.green`@0.25, always brand `:184`,`:178` | 100dp `SportAreaSelectItemAppointmentDetailView.kt:49` · brand `@0.15f` always `:49` | 100 · brand `@0.15` |
| tile h · radius · content | 56 · 6 · times + 学专 only `:189`,`:186`,`:140-166` | 50 · 8 · times + 学专 only `:50`,`:48`,`:62-79` | 50 · 8 · times + 学专 only |
| brief chip free flag | hardcoded `true` `:39` | hardcoded `true` `SportAreaSelectItemStadiumAreaAppointmentAreaView.kt:75` | real `canAppointment` |
| footer CTA label · padding · fill · radius | `确定` · v12/h16 `SportSelectViewFooter.swift:33,35-36` · `Color.green` · 8 `:37-40` | `预定` `AS:37` · v4/h16 `SportSelectFooterView.kt:87,83-84` · `ham_brand_sport` · 8 `:80-82` | **`确定`** · v12/h16 · `ham_brand_sport` · 8 |
| CTA text · inset | `.caption` `:26` · `UIScreen.bottomSafeArea` `:50` | `caption` `:64-67` · `navigationBarsPadding()` `:50` | `caption` · system bars |

**Strings:** as §2 plus `确定` (`IS:243`; **absent on Android**, which reuses `预定`).

**States:** no screen-level state machine. unload — header + venue cards. loading — iOS renders a centred
`ProgressView` (`SportSelectViewBody.swift:23-26`); Android has none, only per-court/per-slot spinners
(`SportAreaSelectItemAppointmentDetailView.kt:85`). error — iOS falls through to an empty view (`:15`);
Android inherits the inline `加载时遇到了错误` (`:88`). success — result written back, screen pops.

**Divergence:** iOS dismisses unconditionally after `confirm()`, even on an empty selection
(`SportSelectViewFooter.swift:30-31`); Android silently returns when any result field is null
(`SportSelectAreaView.kt:56-58`). Both are wrong — the CTA must be disabled while the selection is
incomplete. Both hardcode the brief-chip availability flag to true, so every collapsed chip renders
brand-coloured; it must carry the real `canAppointment` as §2 does. **Terminology:** the Android picker CTA
reads `预定` from the shared `sport_reserve`; it must read `确定`.

### 4. Captcha (输入验证码) — platforms: both
**Purpose:** A full-screen bundled HTML challenge; on success it hands a token back to the caller (§2, §6)
and pops.
**Entry:** pushed when the backend demands a challenge — iOS `Route.sportCaptcha(id:)`
(`SportQuickOrderViewModel.swift:88-90`); Android navigates or renders `SportCaptchaCard` inline inside §6
(`SportQuickOrderView.kt:34-37`).

**Layout:**
```
▓▓ title 输入验证码 · pinned · trailing 刷新
░░ WebView — bundled sport-captcha-page.html · fills · bridge: token → caller, then pop
```
**Blocks:** 1 **Title bar** `输入验证码` + trailing `刷新`. 2 **Refresh** — reloads the web view.
3 **Web challenge** — the bundled asset, filling the remaining space. 4 **Token handoff** — the page posts the
token over the platform bridge; the screen pops and the caller resumes.

**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| container · title · refresh | `NavigationStack` child `SportCaptchaView.swift` · `输入验证码` `:34` · `ToolbarItem(.topBarTrailing)` `刷新` `:35-43` | `HamNavigationView(isBounceScroll = false)` `SportCaptchaView.kt:23` · none · absent | full-screen container · `输入验证码` · trailing `刷新` |
| asset | `sport-captcha-page.html` from bundle `:24` | `file:///android_asset/web/sport-captcha-page.html` `:33` | bundled page, one asset per platform |
| bridge · payload | `captchaValidateToken` `:31` · `[String:String]["token"]` `:70-73` | `android.postCaptchaValidateToken` `:34`,`:42` · `JSONObject["token"]` `:44` | platform-native bridge · `{ "token": … }` |
| dismissal | `dismiss()` when `autoDismiss` `:51-53` | `popBackStack()` unconditionally `:26` | pop after handoff |

**Strings:** `输入验证码` (`SportCaptchaView.swift:34` — no `IS:` key, renders the literal; must be added) ·
`刷新` `IS:417`, `AS:14`. Page copy lives in the HTML asset.

**States:** unload/loading — none modelled, the page owns its own. error — none; a failed challenge never
fires the bridge. success — token posted, screen popped.

**Divergence:** iOS alone has a title, a `刷新` button and an `autoDismiss = false` embedding mode
(`:11`,`:17-20`); Android splits the embeddable case into `SportCaptchaCard` (`:32-36`). All three become
normative.

### 5. Pay (支付) — platforms: both
**Purpose:** Two ways to settle a placed order — method 1 instructs the user to continue in the WeChat
mini-program; method 2 renders a WHU unified-payment URL. Method 2 opens as a "check payment info" JSON panel
and expands to a copyable URL after 确认.
**Entry:** 去支付 on a current-order card (§1) or the success screen (§13) → iOS `Route.sportPay(orderNo:)`,
Android `SportRoutes.Pay(orderNo)`.

**Layout:**
```
▓▓ title 支付 · pinned
░░ SCROLL · pad 16
  METHOD 1 (always) · [方法1 chip] 前往微信小程序“场馆预约”继续支付
  ── divider pad v8 ── METHOD 2 IF paymentInfo != null
    [方法2 chip] 从武汉大学统一支付网页支付 · 您在武汉大学统一支付的交易与Ham无关。
    gap 16 · 核对支付信息 · JSON panel h 256 r12 ham_gray@0.15 pad 8
    gap 32 · [ 确认 ] full-width · blue · r12
  ── expanded ── 长按下方文本复制到浏览器打开 · selectable URL caption r12 ham_gray@0.15
```
**Blocks:** 1 **Title bar** `支付`. 2 **Method 1** — always visible: `方法1` chip (`body` rounded bold, white,
r8) then the description in bold. 3 **Method 2** — conditional on the payment-info fetch: chip, description,
disclaimer. 4 **JSON panel** — `核对支付信息` above pretty-printed JSON in a scrollable panel. 5 **确认** —
expands block 4 into the URL state. 6 **Expanded URL** — the hint above the selectable URL
`https://gym.whu.edu.cn/hsdsqhafive/transit/formSubmit.html?url=…&json=…&signature=…`.

**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| title · root padding | none `SportPayView.swift` · h 16 `:111` | `sport_pay_title` `AS:16`, `SportPayView.kt:62` · 16 all `:63` | `支付` · 16 all |
| method chip · description · disclaimer | `body` rounded bold white r8 pad v4/h8, no explicit fill `:28-35` · bold 17 `:37` · unstyled `:60` | plain `bodyBold` text `:65`,`:75` · `body` `:66` · `body` `:77` | chip as left · `bodyBold` · `body` |
| divider · pre-panel gap · panel h | `Divider()` pad v8 `:44` · 16 `:61` · 256 `:72` | `HamDivider` pad v8 `:72` · 16 `:79` · unbounded `:86-93` | pad v8 · 16 · 256 with inner scroll |
| panel surface · radius · pad · label | gray@0.25 `:74` · 12 `:75` · h16+v4 `:69-70` · unstyled `:65` | `ham_gray`@0.15 `:91` · 12 `:90` · 8 `:92` · `bodyBold` `:84` | `ham_gray`@0.15 · 12 · 8 · `bodyBold` |
| pre-button gap · 确认 fill · radius · pad · label | 32 `:77` · `Color.blue` `:87` · 12 `:88` · v16 `:85` · `.white` unstyled `:83` | `spacedBy(4)` `:83` · `ham_blue` `:98` · 12 `:97` · v12 `:99` · `body` `ham_white` `:101` | 32 · `ham_blue` · 12 · v12 · `body` white |
| URL hint · text · surface · encoding | unstyled `:94` · `.caption` pad 8 `textSelection(.enabled)` `:96-98` · gray@0.25 r12 `:100-101` · `.urlHostAllowed` `:91` | `body` `:111` · read-only `HamTextField` `caption` `:112-115` · none · `URLEncoder.encode(…, UTF-8)` `:109` | `body` · `caption` pad 8 selectable · `ham_gray`@0.15 r12 · UTF-8 form encoding |

**Strings:** `支付` `AS:16` · `方法1` `IS:682`, `AS:17` · `前往微信小程序“场馆预约”继续支付` `IS:584`, `AS:18` ·
`方法2` `IS:683`, `AS:19` · `从武汉大学统一支付网页支付` `AS:20` · `您在武汉大学统一支付的交易与Ham无关。` `AS:21` ·
`核对支付信息` `IS:732`, `AS:22` · `确认` `IS:209`, `AS:23` · `长按下方文本复制到浏览器打开` `IS:853`, `AS:24`.

**States:** unload — method 1 only. loading — centred `HamLoadingProgressBar` (`SportPayView.kt:70`); iOS has
no loading state, method 2 simply does not render until the payload arrives (`SportPayView.swift:42`).
error — **neither platform renders one**; `获取付费信息失败` (`AS:90`) is declared but unused and must be shown
with a retry. success — the JSON panel / expanded URL.

**Divergence:** The expand default is **inverted** — iOS starts collapsed showing the JSON panel
(`SportPayView.swift:17`,`:63`), Android starts expanded showing the URL (`SportPayView.kt:60`,`:82`);
collapsed-first is normative. iOS has no title and no loading state; Android has both. iOS renders the method
number as a chip, Android as plain text — the chip is normative. iOS percent-encodes with `.urlHostAllowed`,
the wrong character set for a query value — use UTF-8 form encoding. iOS's hint says "long-press" but enables
the standard selection menu. **场馆预约 is a proper noun — never rewritten.**

### 6. Quick order (快速预约) — platforms: both
**Purpose:** Submit a remembered (starred) court in one tap from the home screen, with no intermediate
booking UI; shows progress, then reuses the §13 success or error card.
**Entry:** the 预约 CTA on the home quick-order card — iOS `Route.sportQuickOrder(orderForm:tomorrow:)`
(`Route.swift:141-142`, pushed from `SportMainViewQuickOrderCard.swift:93`); Android
`SportQuickOrderView(navController, orderForm)` (`SportNavGraph.kt:64`).

**Layout:**
```
▓▓ full-screen · ham_bg_b1 · no title
  submitting: ◜◝ spinner (iOS: 正在预约 → 很快就好) · on challenge: inline captcha card (§4)
  then: §13 success | error card
```
**Blocks:** 1 **Progress** — iOS uses a determinate bar with a rotating label (`正在预约` below 1, `很快就好` at
1) driven by a 10 ms timer adding 0.01 per tick, slowing to 0.001 past 0 and stopping at 0.95
(`SportQuickOrderViewModel.swift:59-83`) — decorative, never completes. 2 **Captcha (conditional)** — presented
when the backend demands a challenge; submission resumes on token handoff. 3 **Result** — the §13 card,
unchanged.

**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| container · indicator · label | `VStack` maxW/maxH `ham_bg_b1Color` `SportQuickOrderView.swift:47-48` · `ProgressView(value:)` + label, `.padding()` `:35-43` · `正在预约` `:37` → `很快就好` `:39` | `HamNavigationView(isBounceScroll = false)` `SportQuickOrderView.kt:33` · `HamLoadingProgressBar` centred `:41-43` · none | full-screen `ham_bg_b1`, no title · indeterminate spinner · no label |
| captcha · wait | pushes `Route.sportCaptcha` `SportQuickOrderViewModel.swift:88-90` · `withCheckedContinuation` by UUID `:92-100` | `SportCaptchaCard` inline `SportQuickOrderView.kt:34-37` · 100 ms poll on `captchaToken` `SportQuickOrderViewModel.kt` | inline card · callback |
| result · dismissal · error text | success `:27-29` / error `:31-33`, then `dismiss()` · `error.message ?? error.code.description` `:52` | success `:47` / error `:51-53`, then `popBackStack()` · `e.message.orEmpty()` | show result, then pop · server message, else generic fallback |

**Strings:** `正在预约` `IS:736` · `很快就好` `IS:657`. Result copy is §13's.

**States:** unload — unreachable, submission starts in the initialiser
(`SportQuickOrderViewModel.swift:21`) or on first composition (`SportQuickOrderView.kt:28-32`).
loading — progress indicator. error — §13 error card with the server message; the action pops. success — §13
success card. captcha — challenge presented, then submission resumes.

**Divergence:** iOS shows a labelled determinate bar driven by a synthetic timer that never completes; Android
shows a plain spinner. Neither is a real progress signal — normative is an indeterminate spinner with no label.
iOS navigates to the captcha screen and blocks on a continuation; Android renders the card inline, which is
normative. iOS falls back to the numeric error code when the server sends no message; Android shows an empty
string, rendering a blank error — it must fall back like iOS.

### 7. Starred order setting (收藏预约设置) — platforms: both
**Purpose:** Configure the court remembered by the home quick-order card, and choose whether quick 预约
targets today or tomorrow.
**Entry:** 去设置 on the settings favourite card (§11) → iOS `Route.sportStarredOrderSetting`
(`Route.swift:147-148`); Android `SportRoutes.StarredConfigSetting` (`SportNavGraph.kt:68`).

**Layout:**
```
▓▓ title 收藏预约设置 · pinned
░░ SCROLL · pad 16 · gap 8
  1 card ham_bg_b2 r16 pad 16 · badge 36 r6 · gap 8 · type · venue（address）· HHmm-HHmm (or 未设置)
    gap 12 · [ 去设置 ] full-width brand@0.15 r12 pad v16
  2 card IF configured · 预约明天 [switch]
```
**Blocks:** 1 **Summary card** — court-number badge, then sport type, venue and time range, all `caption`; when
unconfigured the three lines collapse to `未设置`. 2 **去设置 CTA** — full-width; *tap* pushes the picker (§3) and
the result replaces the whole persisted item list. 3 **预约明天 row (conditional)** — only when a court is
configured; a switch bound to the stored `tomorrow` flag that persists immediately.

**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| title · page pad · gap | none (no `navigationTitle` in the module) · 16 · none `SportStarredOrderSettingView.swift:95` | `sport_starred_setting_title` `AS:61`, `SportStarredSeatSettingView.kt:62` · 16 · `spacedBy(8)` `:63` | `收藏预约设置` · 16 · 8 |
| card surface · radius · pad · badge | `ham_bg_b2Color` · 16 · 16 `:71-74` · 32 r6 white `title2` rounded bold, **no explicit fill** `:26-32` | `HamCardView` defaults · `StadiumAreaIcon` 36 r6 `ham_text_primary`, text `ham_bg_b1` `StadiumAreaIcon.kt:28-34` | `ham_bg_b2` · 16 · 16 · 36 r6 |
| badge→text gap · detail lines · venue line | none · `.caption` `:33-38` · `stadiumArea.title` only `:35-36` | 8 `:68` · `caption` `ham_text_primary` `:69-77` · `{title}（{address}）` `:72-74` | 8 · `caption` · `{title}（{address}）` |
| empty text · time line | `未设置`, inherits body 17 `:40` · `HHmm-HHmm` `:37-38` | `sport_setting_not_set` `caption` `:79`, `AS:55` · `HHmm-HHmm` `:75-77` | `未设置`, `caption` · `HHmm-HHmm` |
| CTA gap · label · fill · radius · pad · label style | 16 `:43-46` · `设置` `:44` · `Color.green`@0.25 · 8 · v16 `:48-49`,`:46` · `ham_text_t1Color` `:45` | 12 `:82` · 去设置 `AS:56` `:94` · brand@0.15 · 12 · v16 `:90-91` · `bodyBold` brand `:94` | 12 · **`去设置`** · brand@0.15 · 12 · v16 · `bodyBold` brand |
| CTA alignment · switch row | leading `:72` · `Toggle` + `预约明天` `:79-81` | centred `:92` · `HamSwitch` + `sport_reserve_tomorrow` `:103-107` | centred · labelled switch |

**Strings:** `收藏预约设置` (`AS:61` value is 收藏预定设置) · `未设置` `IS:716`, `AS:55` · `去设置` `AS:56` /
`设置` `IS:134` · `预约明天` `IS:208`, `AS:62`.

**States:** unload — configured card, or `未设置` with only the CTA. loading — none, the config is read
synchronously (`SportStarredOrderSettingView.swift:9`; `SportStarredSeatSettingView.kt:52`). error — none
modelled; a failed picker round-trip silently leaves the config unchanged. success — config replaced, switch
row appears.

**Divergence:** iOS has no screen title and labels the CTA `设置` (`:44`) where Android uses `去设置` (`AS:56`),
which is normative and matches the §11 card that links here. The iOS badge writes
`.background(RoundedRectangle(cornerRadius: 6))` with no fill (`:30-32`), inheriting the default foreground,
while the select screen's badge uses an explicit `ham_text_t1Color` fill
(`SportOrderViewAppointmentAreaCell.swift:22-29`) — the explicit fill is normative. iOS filters the config to
the weekday-agnostic entry (`itemList.first { $0.weekday == -1 }`, `:22`); Android takes `firstOrNull()`
(`:53`), so the two can display different courts when per-weekday entries exist. iOS rewrites the whole
`itemList` on every change (`:66-67`); Android toggles a single flag. **Terminology:** `AS:61` `收藏预定设置` →
**`收藏预约设置`**.

### 8. Order center (订单中心) — platforms: both
**Purpose:** Order history — a thin web-view wrapper around the university's own order-list page. The native
layer's only jobs are to inject the session token into the page's `localStorage` and to provide a title bar.
**Entry:** 订单中心 on the home function card → iOS `Route.sportMyCenter`, Android `SportRoutes.OrderCenter`.

**Layout:**
```
▓▓ status-bar spacer ham_bg_b1 · header h36 · ◀ 订单中心 · [刷新] · pinned
░░ WKWebView / WebViewCompose · fills · …/hsdsqhafive/pages/order/orderList · wxtoken injected before load
```
**Blocks:** 1 **Title bar** — fixed `订单中心`, back button, trailing `刷新`. 2 **Web view** — the whole content
area; everything inside is owned by the remote page and is outside this design system. 3 **Refresh** — reloads
the web view.

**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| URL · title · title behaviour | `…/hsdsqhafive/pages/order/orderList` `SportMyCenterView.swift:20` · `订单中心` `:21` · fixed (`fixTitle = true`, `加载中` swap suppressed) | same `SportOrderCenterView.kt:31` · `sport_my_orders_title` `AS:13`, `:60` · fixed resource | as left · `订单中心` · fixed on both |
| header height · surface · back · trailing | system default · system · system + swipe back/forward gestures · none (commented out in `InnerWebView`) | 36dp `NavigationView.kt:197`,`:287` · `ham_bg_b1` `:288-291` · `ChevronLeft` brand, start margin 16 · `刷新` `body` `ham_blue`, end margin 8 `:61-63` | 36 · `ham_bg_b1` · both · `刷新` |
| token source · injection | `LocalStorageHelper …sport_auth` `:27` · `localStorage.clear()` then `Object.assign(localStorage, {"wxtoken":…})`, at document start, main frame only `:31-51` | `sportContext.auth` minus `Bearer ` `SportOrderCenterViewModel.kt:16` · `localStorage.setItem('wxtoken', …)` on every `onPageCommitVisible` and `onPageFinished` `:35-48` | bearer-stripped · inject on commit and on finish |
| injection guard · encoding · CAS bounce | none · `JSONSerialization.isValidJSONObject` check `:42` · not handled | URL contains `gym.whu.edu.cn` `:36` · raw interpolation into a JS literal `:38` · URLs containing `cas` redirect back `:49-51` | host-guarded · JSON-encode the value · redirect back |

**Strings:** `订单中心` `AS:13` · `刷新` `AS:14` (absent on iOS) · `加载中` (hardcoded in `InnerWebView.swift`,
unreachable because `fixTitle` is true). No other native copy.

**States:** unload/loading — no native indicator on either platform. error — neither overrides the platform
failure callback, so there is no native error UI and no retry. success — remote page renders. empty — owned by
the remote page. retry — `刷新` → `webView.reload()`.

**Divergence:** iOS has no refresh control and no CAS redirect guard; both are Android behaviours and are
normative. Both inject an empty `wxtoken` without validating it, and Android interpolates it unescaped into a
JS string literal — the value must be JSON-encoded, and an empty or absent token must route to
re-authentication instead of loading the page.

### 9. Bulletin list (公告) — platforms: iOS only
**Purpose:** The full announcement list, reached from the home banner's first page.
**Entry:** Banner page 0 → `Route.sportBulletin` (`SportMainViewBannerCard.swift:44`).

**Layout:**
```
▓▓ title 公告 · pinned
░░ SCROLL · ham_bg_b1 · pad 16 · gap 8
  card ham_bg_b2 r16 pad 16 · title bodyBold · createTime caption secondary · gap 8 · content body
  … one card per bulletin, data order
```
**Blocks:** 1 **Bulletin card (repeating)** — title, timestamp, 8 gap, full body; data order, no sorting,
grouping or section headers. 2 No other block — no header, refresh control or empty-state view.

**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| screen exists | yes `SportBulletinView.swift` | **no** — no route in `SportNavGraph.kt`; page 0 untappable `SportMainViewBannerCard.kt:76-110` | yes, on both |
| title · page pad · surface | none set · 16 `:35` · `ham_bg_b1Color` `:39` | n/a | `公告` · 16 · `ham_bg_b1` |
| card pad · radius · surface · gap | 16 · 16 · `ham_bg_b2Color` · `VStack` default `:16`,`:28-31` | n/a | 16 · 16 · `ham_bg_b2` · 8 |
| title · timestamp · content | `TSText` bold 17 `:19-20` · `.caption` t2 `ham_toyyyymmddhhmm` `:21-23` · `TSText` body 17 `:26` | n/a | `bodyBold` · `caption` secondary `yyyyMMdd HHmm` · `body`, no limit |
| gap · translation · fetch | 8 `:25` · `TSText` `.translationTask(zh → preferred)` on iOS 26+ `TSText.swift:12-41` · `.onAppear` `:40-42` | n/a | 8 · translatable · fetch on appear |

**Strings:** none — title, timestamp and content are all server data.

**States:** unload — blank scroll view (`bulletinList` starts empty, `:12`). loading — **none**. error —
`ToastUtils.showError` only, no inline view and no retry (`:50-52`). empty — none. success — cards render.

**Divergence:** Android has no list screen and its banner page 0 is untappable; the screen is normative on both
platforms. Loading, empty and error states are entirely missing and must be added — a spinner, a placeholder,
and an inline error with retry.

### 10. Bulletin detail (公告详情) — platforms: both
**Purpose:** Read a single announcement.
**Entry:** Banner page n → iOS `Route.sportBulletinDetail(bulletin:)` (`SportMainViewBannerCard.swift:93`),
Android `SportRoutes.Bulletin(json)` (`SportMainViewBannerCard.kt:114`); on iOS also from §9.

**Layout:**
```
▓▓ title 公告 · pinned
░░ SCROLL · ham_bg_b1 · pad 16
  card ham_bg_b2 r16 pad 16 · title bodyBold · createTime caption secondary · gap 8 · content body
```
**Blocks:** 1 **Title** · 2 **Timestamp** · 3 **Body**. All data-driven from `SportBulletin`; no static blocks,
no actions, no share/copy affordance.

**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| title · padding · surface | none set `SportBulletinDetailView.swift` · 16 `:27` · **not set** | `sport_bulletin_title` `AS:4`, `SportBulletinDetailView.kt:30` · 16 `:31` · `ham_bg_b1` | `公告` · 16 · `ham_bg_b1` |
| card container | yes — `ham_bg_b2Color` r16, leading-aligned `:28-30` | **none** — plain `Column` | card · `ham_bg_b2` · r16 · pad 16 |
| title · timestamp | `TSText` bold 17 `:18-19` · `.caption` t2 `ham_toyyyymmddhhmm` `:20-22` | `bodyBold` `:32` · `caption` `ham_text_secondary` `toYYYYMMDDHHMM()` `:33-37` | `bodyBold` · `caption` secondary `yyyyMMdd HHmm` |
| gap · content · translation | 8 `:24` · `TSText` body 17, no limit `:25` · `TSText` on iOS 26+ `TSText.swift:24-40` | 4 `:38` · `body` `ham_text_primary` `:39` · none | 8 · `body`, no limit · translatable |

**Strings:** `公告` `AS:4` (absent on iOS). `公告内容` `AS:5` and `666` (`SportBulletinDetailView.kt:50`) are
**preview-only** literals and must not ship.

**States:** unload/loading — none, the bulletin is passed in already loaded. error — none; Android deserialises
with `.orEmpty().toObject<SportBulletin>()` (`SportNavGraph.kt:77`) with no error UI. empty — none. success —
the card.

**Divergence:** iOS sets no title and no explicit background; Android has no card container — take Android's
title and surface with iOS's card. The gap is 8 on iOS, 4 on Android. Android leaks `公告内容` and `666` into the
`@Preview` surface (`:49-50`) — remove them.

### 11. Settings (运动设置) — platforms: both
**Purpose:** Manage the sport module's session (login, token validity) and the starred court used by the home
quick-order card; Android additionally exposes a destructive reset.
**Entry:** 设置 on the home function card → iOS `Route.sportSetting`, Android `SportRoutes.Setting`.

**Layout:**
```
▓▓ title 运动设置 · pinned
░░ SCROLL · ham_bg_b1 · pad 16 · gap 8
  1 account card r16 pad 16 · 账号信息 bodyBold · subtitle caption secondary
    gap 8 · 登录时间：… · 过期时间：… (or 未登录) · gap 8 · [ 登录 ] / spinner
  2 favourite card r16 pad 16 · 收藏预约 · gap 8 · badge 36 r6 · type · area · 明天 … · gap 8 · [ 去设置 ]
  3 other card r16 pad 16 · 其它设置 · 停止使用、诊断信息等 · [ 重置 ] red
```
**Blocks:** 1 **Account card** — title + subtitle header; body shows the decoded issue/expiry times or `未登录`;
trailing `登录`, replaced by a spinner while in flight; the expiry line turns red and gains `已过期` once past
due. 2 **Favourite card** — title + subtitle header; body shows the configured court (badge, sport type, area,
day + time range) or `未设置`; trailing action pushes §7. 3 **Other card** — a single destructive `重置`.

**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| title · pad · gap · surface | **none set** — no `navigationTitle` in `IOS/setting/` · 16 · none `SportSettingView.swift:18` · `ham_bg_b1Color` `:23` | `sport_setting_title` `AS:44`, `SportSettingView.kt:44` · 16 · `spacedBy(8)` `:46-47` · `ham_bg_b1` | `运动设置` · 16 · 8 · `ham_bg_b1` |
| card radius · surface · pad | 16 · `ham_bg_b2Color` · 16 `SportSettingViewAuthCard.swift:129,128,126` | 16 · `ham_bg_b2` · 16 `Card.kt:53,42,48` | 16 · `ham_bg_b2` · 16 |
| card title · subtitle | bold 17 `:84-85` · `.caption` 12, **primary** `:86-87` | `bodyBold` `ham_text_primary` `Card.kt:81` · `caption` `ham_text_secondary` `:83`, `colors.xml:64` | `bodyBold` · `caption` `text.secondary` |
| title→body gap · body style · body→action gap | 16 `SportSettingViewAuthCard.swift:90` · `.caption` 12 `:109` · 8 `:114` | 8 `Card.kt:85` · `caption` `ham_text_primary` `:61-76` · 4 (auth `:80`) / 8 (favourite `:130`) | 8 · `caption` · 8 on both cards |
| action label · spinner · badge · badge gap | `Button` default blue 17 `:119-123` · `ProgressView()` `:117` · 32 r6 white `title2` rounded bold `SportSettingViewStarredOrderCard.swift:27-33` · none | `body` `ham_blue` `:84-90` · `HamLoadingProgressBar()` `:82` · `StadiumAreaIcon` 36 r6 `:101` · 4 `:102` | `body` `ham_blue` · spinner · 36 r6 · 4 |
| empty text · reset | `未设置`, inherits body 17 `:41` · unreachable (component commented out) | `caption` `ham_text_primary` `:123-127` · `body` `ham_red` `:146-152` | `未设置`, `caption` · `body` `ham_red` |

**Strings:** `运动设置` `AS:44` (**absent on iOS**) · `账号信息` `AS:45` · `你的体育场所系统认证信息` `AS:46` ·
`登录时间：%1$s` `AS:47` (iOS uses a full-width colon, `:99`) · `过期时间：%1$s` `AS:48` · `过期时间：%1$s 已过期`
(iOS only, `:102`) · `未登录` `AS:49` (**absent on iOS**) · `登录` `AS:50` · `登录成功` `AS:91` · `取消` (iOS
web-login sheet only, `:147`) · `收藏预约` `AS:51` · subtitle `AS:52` (iOS copy differs, `:19`) · `明天` `AS:53` ·
`今天` `AS:54` · `未设置` `AS:55` · `去设置` `AS:56` · `其它设置` `AS:57` · `停止使用、诊断信息等` `AS:58` ·
`重置` `AS:59`.

**States:** logged in — 登录时间 + 过期时间, the expiry line red with `已过期` when past due
(`SportSettingViewAuthCard.swift:101-106`). not logged in — `未登录` (`:73-77`); iOS skips the block entirely
(`:91`). undecodable token — iOS skips the block and logs; Android shows `未登录` even though a token exists
(`SportSettingViewModel.kt:47`) — both need a distinct state. login loading — spinner replaces the button.
login success — `登录成功` toast. login error — iOS toasts **and** opens a CAS web-login sheet (`:65-70`);
Android only toasts (`:62-64`). favourite set / not set — as §7 block 1. reset — `sportContext.reset()` then
back (`:71-74`).

**Divergence:** iOS sets no screen title and lacks `未登录`, `已过期` and the reset affordance; all become
normative. Android's two cards use different body→action gaps (4 vs 8) — use 8 on both. The favourite subtitle
copy differs: iOS `在设置中设置收藏的运动场地和时间，然后在首页快速预约`
(`SportSettingViewStarredOrderCard.swift:19`) vs Android `在收藏中设置收藏的运动场地和运动时间，然后在首页快速预约，
或通过Siri捷径完成自动化预约` (`AS:52`) — take Android's minus the Siri clause, which is iOS-only. Day labels
differ: iOS `隔天`/`当天` (`:38`) vs Android `明天`/`今天` (`AS:53-54`) — the latter is normative and matches §7.
iOS renders `未设置` at body 17 rather than `caption`. **Terminology:** `收藏预定` → **`收藏预约`** (`AS:51`,
`AS:61`); `在首页快速预定` → **`在首页快速预约`** (`AS:52`).

### 12. Intro / connect (连接体育场所预约) — platforms: both
**Purpose:** First-run gate — explains that Ham must verify enrolment, offers a CAS (university portal)
verification path, runs the sport login, and celebrates success.
**Entry:** shown when the module has no valid token — iOS `SportIntoView.swift` (file name is a typo; the struct
is `SportIntroView`), Android `SportIntroView.kt` as a bottom sheet.

**Layout:**
```
▓▓ bar [关闭] · pinned
░░ pad 16 · logo 48 r8 ⟶ link ⟶ module icon 48 @0.25
  gap 32 · 连接体育场所预约 title 24/Bold · 使用前，Ham需要验证你的在校信息 body
  gap 16 · panel r12 #EDEEEF pad 16 gap 8 · 🌐 从信息门户验证 › · 将进入武汉大学信息门户网页验证你的身份
  sub-routes: CAS web login → loading → success | error
```
**Blocks:** 1 **Header** — app logo 48 r8, a link glyph, the module icon 48 tinted at 25% of `ham_brand_sport`;
communicates "Ham ⟷ this module". 2 **Title** `连接体育场所预约`. 3 **Subtitle**
`使用前，Ham需要验证你的在校信息`. 4 **Choice list** — one CAS row: icon 24 brand, bold title, `caption` subtitle,
trailing chevron; *tap* goes straight to login if already CAS-logged-in, otherwise opens the CAS web view.
5 **Post-login flow** — a dedicated loading step, then success or error.

**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| container · dismiss · back | `NavigationView` + `VStack` top-aligned `IntroView.swift:35,82` · toolbar `取消` `:86-90` · n/a | `HamSheet` → `Box` → `HamNavHost` `IntroView.kt:116-132` · `TextButton` `关闭` `headline` `ham_blue` `:183-185` · `BackHandler` → hide `SportIntroView.kt:52-56` | push/sheet host with a nav graph · `关闭` · dismiss on back |
| padding · logo · link glyph · module icon · icon gap | 16 `:78` · 56×56 r12 `:38-41` · `link`, no size/tint `:43` · `sportscourt.fill` 48 brand@0.25 `SportIntoView.swift:21`,`:24` · 16 `:37` | header top 12 `:189` · 48dp r8 `:198-204` · `Link` 24 `ham_text_primary` `:205-209` · `SportsVolleyball` 48dp brand@0.25 `SportIntroView.kt:62-63` · 12 `:196` | 16 · 48 r8 · 24 `ham_text_primary` · 48 brand@0.25 · 12 |
| header→title · title · title→subtitle · subtitle | 32 `:51-52` · `.title.bold()` 28 `:56-57` · 8 `:55` · present, body 17, no secondary colour `SportIntoView.swift:23` | 32 `:222` · `title` 24 Bold `:220-221` · 4 `:222` · **absent** — parameter omitted `SportIntroView.kt:61-63` | 32 · 24/Bold · 8 · present, `body` `text.secondary` |
| panel pad · radius · surface · item gap | 16 `:69` · 8 `:72` · gray@0.15 `:73` · 24 `:66` | h16 outer + 16 inner `:237,241` · 12 `:239` · `ham_lightGray` #EDEEEF `:240` · 8 `:242` | 16 · 12 · #EDEEEF · 8 |
| row icon · gap · title · subtitle · chevron · glyph | brand blue, no size `:104-105` · 8 `:106` · bold `:108-110` · `.caption` 12 `:111-114` · `chevron.right` `.gray` `:119-120` · `graduationcap.fill` `IntroView.swift:158` | 24 `ham_blue` `IntroView.kt:82-87` · 8 `:80` · `bodyBold` `:89` · `caption` `:90` · `ChevronRight` `ham_text_primary` `:93-97` · `Icons.Filled.Public` `:68` | 24 · `ham_blue` · 8 · `bodyBold` · `caption` · `text.primary` · `Public` |
| loading | none | `HamLoadingProgressBar` centred `SportIntroView.kt:111-113` | centred spinner |
| success animation · Lottie · haptics | opacity+offset y100 `.spring()` after 0.8 s `LoginSuccessView.swift:80-94` · `lottie-congrats` once, speed 1 `:11-32` · none | `slideInVertically{it/2}+fadeIn` after `delay(500)` `SuccessView.kt:59-75` · `lottie_congrats.json` speed 1f `:52-70` · `doVibrate` `:62` | slide+fade 0.5 s · play once speed 1 · haptic |
| success mark · title · message | `checkmark.circle.fill` 64 plain `.green` `:55-57` · `登录成功` `.title` 28 `:58-59` · **absent** | `Done` 64 on `ham_blue` circle pad 8 white `:83-91` · `title` 24 `:92` · `body` 16 `:93` | 64 on `ham_blue` circle pad 8 white · `title` 24 · present, `body` |
| success gap · button · label · surface | 32 `:61` · pad 16 maxW 350 blue outline r8 over `ham_bg_b2Color` `:63-78` · `去使用` `:66` · `ham_bg_b1Color` `:84` | 16 `:95` · pad h4 fillMaxWidth h48 `ham_blue`@0.15 r12 `headlineBold` `:96-108` · `完成` `SuccessView.kt:57`, `strings.xml:7` · Lottie fills, no explicit bg | 16 · fillMaxWidth h48 `ham_blue`@0.15 r12 `headlineBold` · `完成` · `ham_bg_b1` |
| error screen | **absent** — toast only `SportIntroViewModel.swift:20-23` | `Close` 64 on `ham_red` circle pad 8 white; title `title`; message `body`; gap 16; brand button `ErrorView.kt:50-101` | full error step with retry |

**Strings:** `连接体育场所预约` `AS:6` (value is 连接体育场所**预定**) · `使用前，Ham需要验证你的在校信息` (**absent
on Android**) · `从信息门户验证` `ASCOMMON:9` · `将进入武汉大学信息门户网页验证你的身份` `ASCOMMON:10` ·
`关闭` `ASCOMMON:8` / `取消` · `验证成功` `AS:7` (iOS hardcodes `登录成功` `LoginSuccessView.swift:58`) ·
`你可以开始使用体育场所预约了` `AS:8` (**absent on iOS**; value is …**预定**了) · `验证失败` `AS:9` ·
`登录场馆预约失败` `AS:10` (proper noun, unchanged) · `重新登录` `AS:11` · `完成` `ASCOMMON:7` / `去使用` (iOS only).

**States:** idle — intro with the CAS row. already CAS-logged-in — skip CAS, go straight to login (Android
`SportIntroView.kt:117-118`; iOS has no such branch). CAS web login — the CAS view, back returns to the intro.
loading — centred full-screen spinner. success — Lottie, haptic, mark, title, message, button. error — full
error step with `重新登录` returning to the intro. A second verification option exists but is commented out on
iOS (`SportIntoView.swift:28-41`) and absent on Android.

**Divergence:** Android omits the subtitle (`SportIntroView.kt:61-63`); iOS has no dedicated loading or error
step — the Android sub-route model is normative. iOS keeps the CAS branch unconditional
(`SportIntoView.swift:26`). Success copy differs: iOS `登录成功`/`去使用` vs Android `验证成功`/`完成`, and only
Android shows a message line — take Android's three-line success with `完成`. iOS styles the success mark plain
green where Android puts it on a blue circle. **Terminology:** `连接体育场所预定` → **`连接体育场所预约`** (`AS:6`);
`你可以开始使用体育场所预定了` → **`你可以开始使用体育场所预约了`** (`AS:8`).

### 13. Order success + order error (预约成功 / 预约失败) — platforms: both
**Purpose:** Terminal states of the booking flow. Shared components rendered inline by §2 and §6 — not
separately routable.
**Entry:** §2 on success/error (`SportOrderView.swift:34-38`, `:96-100`); §6 on load-state completion
(`SportQuickOrderView.swift:27-33`, `SportQuickOrderView.kt:45-54`).

**Layout:**
```
SUCCESS                            ERROR
▓▓ Lottie congrats, full-bleed     ▓▓ centred column · pad 16
░░ centred column · pad 16              ✕ 64 ham_red
     ✓ 64 on brand circle               预约失败 title 24
     预约成功 title 24                   {hint} body secondary
     {hint} caption                     gap 16 · [ 返回 ] 128w · r8
     gap 16
   ┌ detail card · r8 ───────────┐
   │▌16 brand rail               │
   │ badge 36 · venue · time     │
   │ ── divider ──               │
   │ 请于…前完成支付    [去支付] │
   └─────────────────────────────┘
     gap 32 · [ 返回 ] r12
```
**Blocks (success):** 1 **Celebration** — full-bleed `lottie_congrats` played once at speed 1, plus a haptic.
2 **Result mark + title** — a 64 mark on a brand circle, then `预约成功` in `title`, then an optional `hint` in
`caption`. 3 **Detail card** — radius 8 with a 16-wide brand rail on the leading edge: court badge, venue title,
`yyyyMMdd HHmm-HHmm`, divider, then the payment block — identical to §1 block 3 (red deadline + `去支付` inside
the window; `当前未处于可支付时间` + range outside it, no button). 4 **返回** — full-width; *tap* runs the caller's
`backAction`.
**Blocks (error):** 1 **Mark + title** — a 64 mark, then `预约失败`. 2 **Hint** — the caller's message, or a
generic fallback when empty. 3 **返回** — compact; *tap* runs the caller's `backAction`.

**Values (success):**

| Element | iOS | Android | normative |
|---|---|---|---|
| container · entry animation · haptic | `ZStack` maxW/maxH `ham_bg_b1Color` `SportOrderSuccessView.swift:130-131` · opacity+offset y100 `.spring()` after 0.8 s `:125-141` · `ImpactManager…heavy` `:138` | `Box` fillMaxSize `ham_bg_b1` `SportOrderSuccessCard.kt:81-83` · `slideInVertically{it/2}+fadeIn` after `delay(500)` `:72-88` · `DeviceManager.doVibrate` `:76` | fillMaxSize `ham_bg_b1` · slide + fade 0.5 s · haptic |
| mark · title · hint | `checkmark.circle.fill` 64 plain `.green` `:34-36` · `预约成功` `.title` 28 `:37-38` · `.caption` when non-empty `:40-43` | `Done` 64 on `ham_blue` circle pad 8 white `:98-106` · `sport_select_order_success` `title` 24 `:107`, `AS:72` · none | 64 on `ham_blue` circle pad 8 white · `title` 24 · `caption` when supplied |
| title→card gap · card radius · brand rail | 16 `:45` · 8 `:99-103` · `Color.blue` 16 wide, leading `overlay` `:102` | 8 `:108` · `HamCardView` 16 `:109` · none | 8 · 8 · 16-wide brand rail |
| card pad · surface · title | trailing 8, top 8, leading 28, vertical 8 `:93-96` · `ham_bg_b2Color` `:98-101` · none | card 16 · `ham_bg_b2` · `预约信息` `AS:73`, `:109` | leading 28, else 8 · `ham_bg_b2` · `预约信息` |
| badge · badge gap · venue · time | 32 r6 white `title2` rounded bold, no explicit fill `:47-53` · none · `.caption` `:55-59` · `yyyymmddhhmm-HHmm` `:58` | `StadiumAreaIcon` 36 r6 `:111` · 4 `:112` · `caption` `:113-116` · `yyyyMMddHHmm-HHmm` `:114` | 36 r6 · 4 · `caption` · `yyyyMMdd HHmm-HHmm` |
| pay divider · deadline · out of window | `Divider()` `:63` · `请于%@前完成支付` `.red` `.caption` `:67-68` · bold + range `:70-72` | `HamDivider` pad v8 `:119` · `sport_select_payment_deadline_format` `caption` `ham_red` `:123-126`, `AS:74` · `captionBold`+`caption` `:128-135` | pad v8 · `caption` `ham_red` · `captionBold`+`caption` |
| 去支付 · button gap · 返回 | white pad 8/8 `Color.blue` r6 `:80-86` · 32 `:105` · pad 16 maxW 350 blue outline r8 over `ham_bg_b2Color` `:110-121` | `ham_blue` r8 pad 8/8 `headline` white `:143-150`, `AS:76` · 16 `:157` · pad h12 fillMaxWidth h48 `ham_blue`@0.85 r12 `headlineBold` white `:161-172`, `AS:77` | r8 · pad 8/8 · `headline` white · 16 · fillMaxWidth h48 `ham_blue`@0.85 r12 `headlineBold` white |

**Values (error):**

| Element | iOS | Android | normative |
|---|---|---|---|
| container · gap · mark | `VStack` maxW/maxH `ham_bg_b1Color` `SportOrderErrorView.swift:49-50` · 4 `:22` · `xmark.circle.fill` 64 `.white`/`.red` `:23-25` | `Box` fillMaxSize pad 16 centred `SportOrderErrorCard.kt:39` · 4 `:40` · `ErrorOutline` 64 `ham_gray` `:41-46` | fillMaxSize pad 16 centred · 4 · 64 `ham_red` |
| title · hint | `预约失败` `.title` 28 `:26-27` · `.caption` when non-empty `:29-32` | none, raw message only `:47` · `body` `ham_text_secondary`, falls back to `遇到了错误` `:47`, `AS:79` | `预约失败` `title` 24 · `body` `text.secondary` with fallback |
| button gap · label · button | 32 `:34` · `返回` bold `:39-40` · pad 16 maxW 350 `Color.blue`@0.15 r12 blue label `:41-45` | 2 `:49` · `sport_select_back` `AS:78`, `:58` · w128 `ham_gray`@0.2 r8 pad v16 `body` `ham_gray` `:52-61` | 16 · `返回` · w128 `ham_gray`@0.2 r8 pad v16 |

**Strings:** `预约成功` `IS:857`, `AS:72` · `预约失败` `IS:856` · `预约信息` `AS:73` · `请于%1$s前完成支付`
`IS:802`, `AS:74` · `当前未处于可支付时间` `IS:655` · `请于%1$s-%2$s完成支付` `IS:801` · `去支付` `IS:594`, `AS:76` ·
`完成` `IS:630`, `AS:77` / `返回` `IS:830` · `遇到了错误` `IS:384`, `AS:79`.

**States:** unload — unreachable, the card replaces the booking screen when the order resolves. loading — none,
the caller owns the spinner. error — error card with the caller's message; `返回` pops. success — success card;
`返回` pops; inside the payment window the card also offers `去支付` → §5. error with an empty message — Android
falls back to `遇到了错误` (`:47`); iOS renders no hint at all (`SportOrderErrorView.swift:29`) — the fallback is
normative.

**Divergence:** iOS puts the success mark plain green where Android uses a blue circle; iOS adds a 16-wide blue
rail to the detail card that Android lacks; iOS's card is radius 8 where Android uses the standard 16. The
success button reads `返回` on iOS (`:110`) and `完成` on Android (`AS:77`) — `完成` is normative and matches §12.
iOS has no error title and no empty-message fallback; Android has the fallback but no heading, showing only the
raw server message — the spec requires a `预约失败` heading plus the message. iOS's success card and §1's
current-order card render the payment block with slightly different padding; they must share one component.
**Terminology:** `预定成功` (`AS:72`, `IS:857`) → **`预约成功`**; `预定失败` (`IS:856`) → **`预约失败`**.


## 6. Score (成绩 Score)

### 1. Root / gate (成绩) — platforms: both
**Purpose:** Decide whether the module shows content, an auth error, or the first-run connect sheet;
re-lock whenever the app backgrounds.
**Entry:** iOS tab route `score`; Android `ScoreRoutes.Home` (`AND/score/ui/ScoreView.kt:37`).
**Layout:**
```
┌──────────────────────────────────────────┐
│ nav 成绩 (inline)       iOS/score/ScoreView.swift:32-33
├──────────────────────────────────────────┤
│ gate — exactly one of  iOS :22–:29 · AND/score/ui/ScoreView.kt:76–:97
│  A ScoreMainView     fetched + unlocked  │
│  B auth error view   unlock failed  (§8) │
│  C nothing           loading/unlocking   │
├──────────────────────────────────────────┤
│ sheet: connect / update (§3, §6)         │  iOS :35–:37 · AND :100
└──────────────────────────────────────────┘
overlay (on Main): full-screen blur when backgrounded — §2e
```
**Blocks:** 1 **Gate** — COND `canUseScore && unlockState == loaded` → Main; COND `loadedWithError` →
auth error; else nothing (`iOS/score/ScoreView.swift:22-29`, `ScoreView.kt:76-97`).
2 **Auto-connect** — COND never fetched → show the connect sheet **300 ms** after entry
(`iOS/score/ScoreViewModel.swift:24`; Android 500 ms `AND/score/ui/ScoreView.kt:47`).
3 **Dismiss without fetching** — pop the back stack (`AND/score/ui/ScoreView.kt:40-44`); iOS dismisses
after 300 ms (`iOS/score/ScoreView.swift:48-54`). 4 **Re-lock** — background → `unlockState = unload`;
foreground → re-authenticate (`:55-65`; `ON_STOP`/`ON_START` `AND/score/ui/ScoreView.kt:52-74`).
5 **Events** — `.ham_scoreUpdated` forces `canUseScore = true` (`:38-42`); `.ham_scoreReset` re-runs
`prepare()` (`:43-47`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Nav title · page bg | `成绩` inline `:32-33` · `ham_bg_b1Color` `ScoreMainView.swift:87` | `score_title` on Main `ScoreMainView.kt:102` · `ham_bg_b1` `:199` | `成绩` inline · `surface.primary` |
| Gate transition · auto-sheet delay · re-lock | `withAnimation` `:39-41` · 300 ms `ScoreViewModel.swift:24` · `scenePhase == .background` `:62-64` | `fadeIn()+fadeOut()` `:80` · 500 ms `:47` · `ON_STOP` `:56-58` | cross-fade · **300 ms** · platform background signal |
**Strings:** `成绩` · `连接成绩` · `保护你的成绩数据` (biometric reason) · `请重新验证` ·
`你已开启成绩保护，请开启生物认证权限` · `验证失败`.
**States:** loading — renders nothing, no spinner (`iOS/score/ScoreView.swift:22`, `ScoreView.kt:95`) ·
error — §8 · empty/never-fetched — block 2.
**Divergence:** the six iOS auth strings are `String` literals, so `String(localized:)` never runs even
though every key exists in `LS:` (`ScoreViewModel.swift:51,59,72`, `ScoreFaceIdErrorView.swift:31`);
Android's are resources (`SCSTR:24,26,27`).

---

### 2. Main — score overview (成绩) — platforms: both — **primary screen**
**Purpose:** Aggregate GPA / F2 statistics, three entry actions, per-semester score list, and a
multi-select mode that recomputes statistics from the chosen subset.
**Entry:** the function-grid tile; rendered by the root gate (§1).
**Layout:**
```
┌────────────────────────────────────────────────┐
│ nav 成绩 (inline)                              │
├────────────────────────────────────────────────┤
│▓ PINNED select header — select mode only  ≈96 ▓│  §2d · opaque surface.primary
├────────────────────────────────────────────────┤
│ ScrollView · h-pad 16 · SCROLLING              │  iOS/score/main/ScoreMainView.swift:45
│  ┌ spacer 96 — only in select mode ─────────┐  │  :27
│  ┌ A stat card 成绩概览 ─────────────────┐  │  :22 · hidden in select mode
│  ┌ B function card            h 56 ──────┐  │  :23 · hidden in select mode
│  ┌ C semester card ×n (newest first) ────┐  │  :40–:43 · gap 8
│  │  header 2024-2025 第二学期     ▸ / 全选│  │
│  │         平均成绩 x (GPA y)              │  │
│  │  ── 12 ── rows, gap 8                  │  │
│  └────────────────────────────────────────┘  │
│  spacer 24                                    │
└────────────────────────────────────────────────┘
overlay: full-screen blur when backgrounded (PINNED) — §2e
```
**Blocks:** 1 **A Stat card** (`ScoreMainViewMyScoreDataCard.swift`) — `成绩概览` 17/Bold; `GPA %.2f`
hero 28/Bold; `平均成绩: %.2f` 12 after a 6×6 dot; COND `scoreDataList.count > 0` → divider (v-pad 12,
maxWidth 200) + `年度成绩` 17/Bold + one row per year (gap 4): year label, dot, `综测成绩: %.6f`,
`平均成绩: %.2f (GPA%.2f)`. Watermark: crown **128 @ 0.15** bottom-trailing (16,16) clipped
(`DS:767`; today iOS 220 @0.1 offset(60,20) `:76-82`, Android `School` 240 @0.15 `:38-39`).
2 **B Function card** (`ScoreMainViewFunctionCard.swift`) — three equal tiles, gap 8, h 56, r16,
`surface.tertiary`, pad 8: **获取成绩**/`更新成绩信息` → update sheet · **选择**/`自选成绩然后统计` →
toggles select mode · **设置**/`编辑成绩选项` → Settings. Trailing arrow `icon.xs` 12 `brand.score`.
COND hidden in select mode (`:22-24`).
3 **C Semester card** (`ScoreMainViewSingleSemesterCard.swift`) — header `{year}-{year+1} 第N学期`
17/Bold + `平均成绩: %.2f (GPA%.2f)` 12 with a 6×6 dot; trailing = collapse chevron normally,
**全选/全不选** in select mode when the semester has a selectable row (`:42-43,:62-72`). COND body when
`selectMode || expanded`; `expanded` defaults **true** (`:14`).
4 **Score row** (`ScoreMainViewSingleScoreCell.swift`) — 6 × 28 colour bar r3 (`DS:773`); line 1 name
17 + instructor 12; line 2 `courseType · %.1f学分` 12 (2 lines max, `跨学院专业课` substituted); score
22/Bold right-aligned, `--` when absent, strikethrough when excluded from F2.
5 **Comment button** — COND `enableCourseComment && !selectMode` (`:56`); TAP → course detail when
logged in, else the login sheet (`:59,:64-68`). 6 **Checkbox** — COND select mode; `icon.md` 24 r4
`accent` checked (`:81-84`). 7 **D Pinned header** (§2d) · 8 **E Privacy blur** (§2e).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| List h-pad · top · bottom | 16 `:45` · spacer 24 `:38` · none | 16dp `:110-111` · `16+header+statusBar` `:109` · 16dp `:112`+`navBar+24` `:186` | 16 · 16 · `navBar + 24` |
| Card gap · pad · radius · bg | 24 then 0 `:38` · 16 · 16 · `b2Color` `CardView.swift:22,23,128` | 8dp `:114` · 16dp · 16dp · `b2` `Card.kt:48,53,42` | **8** · 16 · 16 · `surface.secondary` |
| 成绩概览 · GPA hero | 17/Bold `:24` · `.title` 28 `:30` | `bodyBold` 16sp `:41` · `title` 24sp forced Normal `:42-45` | 17/Bold · **28 / Bold** |
| 年度成绩 · divider · year gap | semibold `:48-50` · v12 maxW 200 `:47` · 5 `:52` | `bodyBold` `:51` · v4 `:49` · 4dp `:50` | 17/Bold · v **12** · 4 |
| Year row · year labels | dot + `综测成绩: %.6f` + `平均成绩: %.2f (GPA%.2f)` `:60-66` · 6 labels `:12-19` | `综测成绩：%.6f  平均成绩：%.6f` `:71-74` · 5 labels `SCSTR:61-67` | dot + `综测成绩：%.6f` + `平均成绩：%.2f (GPA %.2f)` · **6** — blank + 大一…大五 |
| Function card h · radius · bg · gap · pad | 56 `:59` · 16 `:63` · `gray@0.1` `:64` · 0 · lead 8 `:61` | intrinsic `:51` · 12dp `:91` · `ham_lightGray` `:92` · 8dp `:52` · 8dp `:93` | **56** · **16** · `surface.tertiary` · 8 · 8 |
| Tile title / subtitle / arrow | bold 17 `:44` / `caption2` 11 `:54` / `arrowtriangle` 8pt `:47-49` | `bodyBold` 16sp `:99` / `caption` 12 `:110` / `ArrowRight` unscaled `:102-106` | 17/Bold / **11** / `icon.xs` 12 `brand.score` |
| Semester title · chevron · 全选 | semibold `:48` · `chevron.up/down` `:79-80` · system blue `:71` | `bodyBold` `:86` · `ChevronRight` ±90° `:108-110` · `body` `ham_blue` `:133-136` | 17/Bold · rotate ±90° · 17 `accent` |
| Header→rows · row gap | 10 `:86` · **15** `:87` | 4dp `:77` · **8** `:142` | 12 · **8** |
| Colour bar · score · name / meta | 6 × 35 r6 `:22-24` · `.title2` 22 `:52` · 17 `:26` / `caption` 12 `:29-32` | 6 × 28dp r3 `:248-263` · `title` 24sp Normal `:190-192` · `body` 16sp `:266` / 12 `:273-285` | **6 × 28 r3** · **22 / Bold** · 17 / 12 |
| Absent score · disabled | `--` `:44` · `gray@0.65` `:39` | `--` `:184` · `ham_gray` `:193` | `--` · `text.tertiary` |
| Strikethrough | `isEnable && !selectMode && !inF2` `:54` | `isEnable && !containF2` any mode `:191` | **always when excluded** |
| Comment glyph | `bubble.left…fill` 14pt, 32 plate r8 `:100-108` | `Forum` 20dp, plate r8 `:207-215` | **20** in a 32 plate r8 pad 4, 44 target |
| Checkbox · unselectable | `checkmark.square.fill`/`square` `.title2` `:81-84` · `opacity(0)` `:87` | `CheckBox`/`…OutlineBlank` `:225-234` · visible `ham_gray` `:229` | 24 r4 `accent` · **not rendered** |
**Strings:** `成绩` · `成绩概览` · `GPA %.2f` · `平均成绩` · `年度成绩` · `综测成绩` · 大一…大五 ·
`获取成绩` · `更新成绩信息` · `选择` · `自选成绩然后统计` · `设置` · `编辑成绩选项` · 第一/二/三学期 ·
`平均成绩: %.2f (GPA%.2f)` · `全选` · `全不选` · `学分` · `跨学院专业课` · `退出`.
**States:** empty — no dedicated state; the semester list is empty and 年度成绩 self-hides
(`ScoreMainViewMyScoreDataCard.swift:46`, `ScoreMainInfoCard.kt:48`) · loading — none
(`ScoreMainViewModel.swift:68-73`, `ScoreMainViewModel.kt:208`) · error — not representable ·
**select mode on** — stat + function card removed, 96 spacer, pinned header, per-semester 全选,
checkboxes · **select mode off** — `selectMode = false` **and the selection cleared**
(`ScoreMainViewStatHeader.swift:65-66`) · **backgrounded** — full-screen blur (§2e).
**Divergence:** Android keeps the stat card in select mode (`ScoreMainView.kt:125-127`) and retains
`selectScoreList` on exit (`:236`); it also gates the comment button on an extra `permittedUploadScore`
(`:155`). iOS's `跨学院专业课` is a literal (`ScoreMainViewSingleScoreCell.swift:28`). iOS prints the
yearly 平均成绩 at 2 dp, Android at 6 dp (`SCSTR:7`); iOS keys the year-dot colour by year, Android by
loop index (`ScoreMainInfoCard.kt:57`).

---

### 2d. Pinned select-mode header (选择统计) — platforms: both
**Purpose:** Live statistics for the current selection, plus exit.
**Layout:**
```
┌────────────────────────────────────────────────┐
│▓ PINNED · offset = navBar + statusBar        ▓│
│▓ ● 综测成绩: x  平均成绩: y (GPA z)    12   ▓│  rows 4 apart · pad h16 v8
│▓ ● 总学分: a  必修学分: b  选修学分: c       ▓│
│▓ ● 专业必修: d  专业选修: e  跨专业课程: f   ▓│
│▓                                    退出   ▓│  right-aligned
└────────────────────────────────────────────────┘
```
**Blocks:** 1 COND `selectMode`, pinned to the top below nav + status bar (iOS `.overlay(alignment:.top)`
`ScoreMainView.swift:48-55`; Android `AnimatedVisibility` `:191`). 2 Three dot + text rows: 综测/平均/GPA
(`ScoreMainViewStatHeader.swift:28-31`) · 总/必修/选修学分 (`:39-41`) · 专业必修/专业选修/跨专业课程
(`:52-57`); credit bucketing: cross-college-major → 跨专业课程, else 专业必修 / 专业选修 (`:83-92`).
3 **退出** — exits select mode **and clears the selection** (`:61-71`,`:65-66`). 4 Recompute on every
selection change (`:76-98`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Material | `systemThinMaterial` blur `ScoreMainViewStatHeader.swift:75` | `ham_gray@0.25` then `ham_bg_b1` `:198-199` | **opaque `surface.primary`** |
| Clearance · pad · row gap · dot | 96 `ScoreMainView.swift:27` · 16 `:73` · 4 `:22` · 6×6 r6 `:24-26` | 92 `:121` · h16 v8 `:200` · 4dp `:202` · 6dp, gap 6 `ScoreDotView.kt:28-33` | **96** · h16 v8 · 4 · 6 r3 gap 6 |
| Row 1 precision | 综测 `%.6f`, 平均 `%.2f`, GPA `%.2f` `:28-31` | `SCSTR:8` — GPA `%.6f` | 综测 6 dp · 平均 **2 dp** · GPA **2 dp** |
**Strings:** `综测成绩` · `平均成绩` · `GPA` · `总学分` · `必修学分` · `选修学分` · `专业必修` ·
`专业选修` · `跨专业课程` · `退出`.
**States:** n/a — the header exists only in select mode.
**Divergence:** Android prints GPA with `%.6f` (`SCSTR:8`) — a formatting bug against iOS's `%.2f`.

---

### 2e. Privacy blur overlay — platforms: both
**Purpose:** Hide grades in the app switcher / recents.
**Blocks:** 1 COND `background == true`, driven by the platform background signal
(`iOS/score/main/ScoreMainView.swift:56-64`, state at `:77-85`). 2 Full-screen
`VisualEffectBlurView(blurStyle: .systemThinMaterial)`, `ignoresSafeArea()` — covers nav bar, list and
the pinned header. 3 Cleared on foreground, which re-runs authentication (§1).
**Divergence:** Android has no overlay — it sets `authState = Unload` on `ON_STOP`
(`AND/score/ui/ScoreView.kt:56-58`) so content is blanked entirely (`:95`) and relies on the system
recents screenshot. The blur is normative; Android must add it.

---

### 3. Update / fetch score (获取成绩) — platforms: both
**Purpose:** Authenticate against the education portal (CAS), solve the captcha, pull the score list.
The same composable is reused as the first-run connect flow (§6), retitled.
**Entry:** 获取成绩 tile (`ScoreMainViewFunctionCard.swift:14-20`, `…FunctionCard.kt:54-61`) → sheet
(`ScoreMainView.swift:66-70`, `ScoreMainView.kt:251`); or auto-presented by the root (§1).
**Layout:**
```
iOS IntroView shell · Android HamSheet + nested NavHost    (ScoreMainViewUpdateScoreSheet.kt:57–:61)
┌──────────────────────────────────────────┐  intro/home → cas → cas-captcha
│ [取消]                              :89  │            → load{spinner|成功|失败} → face-id
│  (logo 56 r12) ─ link ─ (doc 48 @.25)    │  :37–:48
│         获取成绩   title 28 Bold   :56   │
│  ┌────────────────────────────────────┐  │  r8 gray@.15, pad 16
│  │ 🌐 通过信息门户登录教务系统     ›  │  │  :103–:121
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
iOS states: captcha → ProgressView + 正在更新 → SuccessView(更新成功) | ErrorView(更新失败, hint)
            iOS/score/update-score/cas/ScoreUpdateByCasView.swift:22–:44
```
**Blocks:** 1 **Entry row** — one `NavLink`, icon `Public` 24 `accent`, title `通过信息门户`, subtitle
`从信息门户登录教务系统获取成绩`, chevron (`ScoreMainViewUpdateScoreSheet.kt:226-240`). COND iOS branches at
build time on `CasConfig.shared.enabled()` (`ScoreUpdateView.swift:23-31`); Android always renders one row
and branches at tap time (`:231-239`). TAP `useCas` → captcha, else → CAS login.
2 **CAS login** — `CasMobileLoginView` (`IntroView.swift:162`; `PATH_CAS` `:108-118`).
3 **Captcha** — bundled `education-captcha-page.html` in a web view, token via the JS bridge
(`EducationCaptchaView.swift:18,31`; `ScoreMainViewUpdateScoreCasCaptchaWebView.kt:20-29`); chrome is a
titled page `验证码验证`, no bounce scroll. 4 **In progress** — centred spinner + `正在更新`.
5 **Success** — 64 icon, `获取成功`, `愉快使用吧`, one `完成` button. 6 **Failure** — 64 icon,
`获取失败`, `message，hint`, one `完成` button. 7 **RN path** — COND the flag `RNFetchScoreView` is on →
the RN module replaces steps 3–4 (`:75-78,:154-159`). 8 **Back-stack** — success pops captcha/home/CAS
(`:140-146`); iOS dismisses. 9 **Face ID upsell** — COND `enableFaceId` unset → `PATH_FACE_ID` (`:169-174`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Sheet title | `获取成绩` `ScoreUpdateView.swift:20` | `score_fetch_scores` `:66` / `score_connect_scores` `ScoreView.kt:100` | `获取成绩` · `连接成绩` (first run) |
| Logo · feature icon · row gap · title pad | 56 r12 `:39-41` · `doc.plaintext` 48 @0.25 `:45-47` · 16 `:37` · 16 `:64` | 48 r8 `IntroView.kt:201-204` · `Article` 48 @0.25 `:210-215` · 12dp `:196` · top 32/bottom 4 `:222` | 48 r8 · 48 @0.25 `brand.score` · 12 · top 32 / bottom 4 |
| Choice container · row | r8 `gray@0.15` pad 16, rows 24 `:66-74` · icon 24, gap 8, bold title, `caption` sub `:103-121` | r12 `ham_lightGray`, h16+16, rows 8 `IntroView.kt:237-244` · same `:77-99` | r12 `surface.tertiary` pad 16 rows 8 · as Android |
| Success icon · type · entrance | `checkmark.circle.fill` 64 `:43-45` · 28/12 `:46-51` · y 100→0 spring 0.8 s `:78-93` | `Done` 64 in `ham_blue` `:83-91` · 24/16 `:92-93` · slide+fade 500 ms `:59-75` | 64 · **28** / 16 · slide-in half-height + fade **500 ms**, one haptic |
| Error icon · entrance | `xmark.circle.fill` 64 `:31-33` · static `:29` | `Close` 64 in `ham_red` `:76-84` · slide+fade 200 ms `:56-67` | 64 · slide-in + fade 200 ms |
| Button | `返回` maxW 350 r8/r12 `SuccessView.swift:67-75`, `ErrorView.swift:47-53` | 48 tall, full width, r12 `SuccessView.kt:96-108`, `ErrorView.kt:89-107` | **完成** — 48, h16 v12, r12, `accent.subtle`, 17/Bold `accent` |
**Strings:** `获取成绩` · `连接成绩` · `取消`/`关闭` · `通过信息门户` · `从信息门户登录教务系统获取成绩` ·
`从信息门户验证` · `将进入武汉大学信息门户网页验证你的身份` · `验证码验证` · `正在更新` · `获取成功` ·
`愉快使用吧` · `获取失败` · `完成`.
**States:** empty — n/a · loading — block 4 · error — block 6, message `message，hint` joined
(`ScoreMainViewUpdateScoreSheetViewModel.kt:69-70`).
**Divergence:** iOS says 更新成功/更新失败 (`ScoreUpdateByCasView.swift:41,43`) against Android's
获取成功/获取失败 (`SCSTR:29,31`) while both entry rows say 获取成绩 — Android's verb is normative, and
both iOS titles are un-localised. iOS's RN kill-switch reads `RNFetchCourseView`
(`ScoreUpdateByCasView.swift:22`), Android reads `RNFetchScoreView` (`:77`). iOS's captcha is a bare
`WKWebView` with no title and no back affordance.

---

### 4. Settings (成绩设置) — platforms: both
**Purpose:** Biometric protection, F2 method, per-course enablement, zero-score filtering, reset.
**Entry:** 设置 tile → `scoreSetting` / `ScoreRoutes.Setting` (`ScoreGraph.kt:27`).
**Layout:**
```
┌────────────────────────────────────────────────┐
│ nav 成绩设置 · ScrollView pad 16 · cards gap 8 │  ScoreSettingView.swift:11,17
│ 1 保护成绩数据  使用生物识别保护成绩     [ ◯ ] │
│                 <red hint 12 when blocked>     │
│ 2 F2计算方式    ◯ 使用计算机学院F2计算方式     │
│                   B1+B2×0.002（B2最多选8门）   │
│                 ◯ 使用其它F2计算方式           │
│                   B1×0.98+B2×0.02              │
│                 自定义计算方式 [编辑代码]   ◯  │  disabled while code empty
│                 使用其它计算方式(基于JavaScript)│
│                               [选择计算方式] ◯ │
│ 3 偏好         忽略0分成绩               [ ◯ ] │
│ 4 成绩启用     编辑成绩启用状态             › │
│ 5 重置成绩     重置所有成绩数据            red │
└────────────────────────────────────────────────┘
```
**Blocks:** 1 **保护成绩数据** — one switch row. Biometric availability is a **three-state** model —
permitted / banned / hardware-unsupported (`FaceIdManager.swift:8-12,19-31`) — with two distinct red
hints; the switch is disabled whenever the state forbids a change
(`ScoreSettingViewProtectionCard.swift:70-81`). Turning protection **off** requires re-auth and
**reverts the toggle if auth fails** (`:41-49,:58-60`).
2 **F2计算方式** — four mutually exclusive rows: CS · Other · 自定义计算方式 (own JS) · 使用其它计算方式
(基于JavaScript) (marketplace). COND the custom toggle is disabled while `f2JsCode == ""` (`:119`);
TAP `编辑代码` → editor (§10); TAP `选择计算方式` → picker (§9), and with no item chosen the tap
navigates instead of selecting (`:101-112`). Every change posts `.ham_scoreUpdated` (`:83,90,97,110`).
3 **偏好** — `忽略0分成绩` → `scoreContext.ignoreZeroScore`. 4 **成绩启用** — one navigation row → §5.
5 **重置成绩** — destructive row; immediate, no confirmation; pops to the score home afterwards.
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Screen pad · card gap · card pad/r/bg | 16 `:17` · 8 `:11` · 16/16/`b2Color` `CardView.swift:22,23,128` | 16dp/8dp `ScoreSettingView.kt:29` · 16dp/16dp/`b2` `Card.kt:48,53,42` | 16 · 8 · 16/16/`surface.secondary` |
| Card title · title→content | semibold `CardView.swift:59-61` · `padding(.bottom, 8)` `:68` | `bodyBold` `Card.kt:81` · `Spacer(8.dp)` `:85` | **17 / Bold** · 8 |
| Row label / subtitle / switch gap | 17 / `caption` 12 / system | `body` 16sp / `caption` 12 / 4dp `PickerRow.kt:42,46,38` | **17** / 12 / 4 |
| Divider · hint · action link | `Divider()` `:35,:43,:63` · `.red` `:29` · `NavigationLink` `:53-55,:68-71` | `HamDivider(pad top 4)` `:44,:53` · `ham_red` `:38` · `Text` `caption` `ham_blue` `ScoreSettingF2TypeCard.kt:57-59` | 1px v4 · `feedback.error` 12 · 12 `accent` |
| Destructive · navigation row | `.red` `ScoreSettingViewResetCard.swift:16` · — | `ham_red` `:28` · `ham_blue` `body` `ScoreSettingEnableCard.kt:29` | `feedback.error` 17 · 17 `accent` |
**Strings:** `成绩设置` · `保护成绩数据` · `使用生物识别保护成绩` · `Face ID权限已被禁用，请前往设置开启` ·
`该设备不支持使用Face ID保护成绩数据` · `验证你的身份` · `F2计算方式` · `使用计算机学院F2计算方式` ·
`B1+B2×0.002（B2最多选8门）` · `使用其它F2计算方式` · `B1×0.98+B2×0.02` · `自定义计算方式` ·
`使用自定义的JavaScript计算F2` · `编辑代码` · `使用其它计算方式(基于JavaScript)` · `选择计算方式` ·
`偏好` · `忽略0分成绩` · `成绩启用` · `编辑成绩启用状态` · `重置成绩` · `重置所有成绩数据`.
**States:** empty — the enable list renders zero rows, no placeholder · loading — none · error — a toast.
**Divergence:** Android's protection switch stays enabled while protection is off even when biometrics
are unusable (`ScoreSettingProtectCard.kt:40`) and never reverts on failed re-auth
(`ScoreSettingViewModel.kt:50-56`); it collapses "banned" and "unsupported" into one hint. Android's F2
card has **three** rows — no custom-JS mode (`ScoreSettingF2TypeCard.kt:34-71`; `ScoreCalcType` is
CS/NORMAL/JS only, `ScoreCalculator.kt:14-18`) — so an iOS `f2Type == .js` decodes to `CS` on Android
(`ScoreCalculator.kt:20-22`). iOS has no zero-score filter at all (`ScoreMainViewModel.swift:17-66` vs
`ScoreMainViewModel.kt:209-211`). Android's card order (Protection · F2 · **Preference** · Enable ·
Reset) is normative. iOS's `B1×0.98+B2×0.02` has no key in any `LS:` file (`:39`).

---

### 5. Enable (成绩启用) — platforms: both
**Purpose:** Toggle `isEnable` per course so disabled courses drop out of GPA / F2 and strike through.
**Entry:** Settings card 4 → `ScoreRoutes.Enable` (`ScoreGraph.kt:41`).
**Layout:**
```
┌──────────────────────────────────────────┐
│ nav 成绩启用                             │
│ LazyColumn · pad 16 · rows gap 8         │  ScoreSettingEnableView.kt:22–:26
│  course name                      [ ◯ ]  │  :29–:37
│  instructor                       12     │
│  ── divider (between rows only) ──       │  :39–:43
└──────────────────────────────────────────┘
```
**Blocks:** 1 One switch row per score: name 17 `text.primary` + instructor 12 `text.secondary`.
2 Divider **between** rows only. 3 TAP writes `isEnable` and persists immediately
(`ScoreSettingEnableViewModel.kt:52-62`). 4 Sorted **disabled-first, newest-first within each group**
(`:37-40`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Container · row gap · dividers · truncation | inline in the Settings card (r16, pad 16) · none · none · `lineLimit(1)` `:18,:19-21` | dedicated scrolling screen, pad 16dp `:24` · 8dp `:25` · `HamDivider` `:41` · none | **dedicated screen** · 8 · between rows · none |
| Sort | disabled-first `…ViewModel.swift:14` | disabled-first, newest-first `:37-40` | disabled-first, newest-first |
**Strings:** `成绩启用` · `编辑成绩启用状态`.
**States:** empty — an empty list, no placeholder · loading / error — none.
**Divergence:** iOS inlines the whole toggle list into a non-lazy `ScrollView` (`ScoreSettingView.swift:10`),
so a long score history materialises every row on the first frame. Android's split is normative.

---

### 6. Intro / connect (连接成绩) — platforms: both
**Purpose:** First-run flow when the user has never fetched scores.
**Entry:** auto-presented by the root gate (§1).
**Layout:** the update sheet (§3) retitled `连接成绩` (`AND/score/ui/ScoreView.kt:100`, `SCSTR:25`). iOS
hosts a three-branch switch: `showFetchOption` → §3 · `showLoginCasOption` → an intro page offering only
`从信息门户验证` · `showFaceIdProtectionOption` → §7 (`iOS/score/intro/ScoreIntroView.swift:18-32`).
**Blocks:** 1 COND no CAS cookie → the CAS-only branch, else the fetch branch
(`ScoreIntroViewModel.swift:13-19`). 2 `.ham_casLoginSuccess` → fetch branch (`ScoreIntroView.swift:34-40`).
3 `.ham_scoreUpdated` **and** biometrics available → Face ID branch (`:41-49`).
4 Dismissing without fetching exits the module (§1).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Title · feature icon | `连接成绩` `.title.bold()` `ScoreIntroView.swift:23` · `doc.plaintext` 48 **orange** @0.25 `:23` | `score_connect_scores` `ScoreView.kt:100` · `Article` 48 `brand.score` @0.25 `ScoreMainViewUpdateScoreSheet.kt:90-91` | `连接成绩`, 28 / Bold · **48 @ 0.25 `brand.score`** |
| Face ID step | branch 3 `ScoreIntroView.swift:28-32` | `PATH_FACE_ID` after success `:169-174` | after the first successful fetch, COND `enableFaceId` unset |
**Strings:** `连接成绩` · `从信息门户验证` · `将进入武汉大学信息门户网页验证你的身份` · `取消`/`关闭`.
**States:** loading / error / empty — n/a.
**Divergence:** iOS's connect icon is orange while the fetch icon is `brand.score` — two colours for one
shell. iOS renders a brief blank frame before `init()` resolves (`ScoreIntroViewModel.swift:15-18`).

---

### 7. Face ID enable (开启生物识别保护) — platforms: both
**Purpose:** Offer biometric protection right after the first successful fetch.
**Entry:** intro branch 3 (§6) / `PATH_FACE_ID` (`ScoreMainViewUpdateScoreSheet.kt:210-223`).
**Layout:**
```
┌──────────────────────────────────────────┐
│         🔒 lock 72          pad 16 · gap 8│
│  使用生物识别保护成绩             [ ◯ ]  │
│  <无法访问… red 12 when unavailable>     │
│              spacer 16                   │
│  ┌────────────────────────────────────┐  │
│  │ 确定    48 · r12 · accent.subtle   │  │  fadeIn after 200 ms
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```
**Blocks:** 1 Lock icon `icon.hero` 72 `text.secondary` (`ScoreMainViewUseFaceIdCard.kt:70-77`; iOS 48pt
`ScoreIntroFaceIdEnableView.swift:22-24`). 2 Switch row seeded from availability (`:50`), disabled when
unavailable. 3 COND unavailable → red 12 hint beneath the label (`:84-88`). 4 `确定` — commits
`enableFaceId`, guarded by the permission state (`ScoreIntroFaceIdEnableView.swift:35-37`).
5 Entrance `fadeIn()` after 200 ms (`ScoreMainViewUseFaceIdCard.kt:52-62`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Lock icon · label | `lock.fill` 48 `:22-24` · `开启Face ID保护你的成绩数据` `:26` | `Lock` 72 `ham_gray` `:70-77` · `使用生物识别保护成绩` `:81-83` | **72** · "开启{platform biometric}保护你的成绩数据" |
| Unavailable hint · seed | none — auto-dismisses `:57-61` · hard `false` `:10` | `无法访问…` `caption` `ham_red` `:85-87` · `canUseFaceId` `:50` | red 12 hint, switch disabled · **availability** |
| Commit guard · button | only when `.permitted` `:35-37` · maxW 350, r8, blue stroke `:40-52` | always `:102` · 48, r12, `accent.subtle` `:101-115` | **only when permitted** · 48, h16 v12, r12, `accent.subtle`, 17/Bold `accent` |
**Strings:** `开启Face ID保护你的成绩数据` · `使用生物识别保护成绩` · `无法访问你的生物识别模块` · `确定`.
**States:** loading / empty — n/a · error — the unavailable-hint branch.
**Divergence:** iOS self-dismisses on devices that cannot authenticate
(`ScoreIntroFaceIdEnableView.swift:57-61`); Android stays with a disabled switch and a hint.

---

### 8. Face ID / auth error (验证失败) — platforms: both
**Purpose:** Block the module when biometric unlock fails and offer a retry.
**Entry:** the root gate when unlock fails — `iOS/score/ScoreView.swift:25-29`,
`AND/score/ui/ScoreView.kt:88-93`.
**Layout:**
```
┌──────────────────────────────────────────┐
│              🔒 lock 72 (TAP = retry)    │  surface.primary, centred, gap 8
│              验证失败      title 28 Bold │
│           <error message>     body 17    │
└──────────────────────────────────────────┘
```
**Blocks:** 1 The lock icon **is** the retry button, 72 `text.secondary`
(`ScoreFaceIdErrorView.swift:21-27`, `ScoreAuthView.kt:37-44`). 2 Title `验证失败` 28/Bold.
3 Message, body 17. 4 Stack gap 8. 5 TAP → reset to unload and re-authenticate
(`ScoreViewModel.swift:34-36`, `AND/score/ui/ScoreView.kt:90-91`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Lock / retry · title · message | `lock.fill` 48 `:21-27` · `验证失败` `.title.bold()` `:31-32` · default body `:34` | `Lock` 72 `ham_gray` `:37-44` · `score_auth_failed_title` `title` 24sp `:45` · `body` 16sp `:46` | **72**, 44 min target · **28 / Bold** · **17** |
**Strings:** `验证失败` · `请重新验证` (evaluation failed) · `你已开启成绩保护，请开启生物认证权限`
(protection on, no permission) · `保护你的成绩数据` (system prompt reason).
**States:** this whole screen is the error state · loading / empty — n/a.
**Divergence:** iOS's three runtime strings are un-localised (`ScoreViewModel.swift:51,59,72`); Android's
are resources (`SCSTR:24,26,27`). iOS supplies a localizedReason to the system prompt, Android does not.

---

### 9. JS calc picker (选择计算方式) + detail — platforms: both
**Purpose:** Browse a React Native marketplace of community F2 calculators and inspect one.
**Entry:** `选择计算方式` on the Settings F2 card → `scoreJsCalc` / `ScoreRoutes.JsF2Edit`
(`ScoreGraph.kt:31`); detail via the RN event `ScoreJsCalcViewOpenDetail`.
**Layout — picker:** the RN module `RNScoreCalcView` fills the screen under a native nav title
`选择计算方式` (`iOS/score/js-f2/ScoreJsCalcView.swift:18-20`; `AND/score/ui/scorecalc/ScoreJsCalcView.kt:36-41`).
No layout values — the body is 100 % RN and its strings live in the RN bundle
(`RN/src/i18n/zh/translation.json`, namespace `scorecalc`), not in the native string files.
**Layout — detail:**
```
┌──────────────────────────────────────────┐
│ Column pad 16                            │
│  ┌────────┐  title      title2 20 1 ln  │  gap 16
│  │  icon  │  brief      caption 12 2 ln │  72, r12, tertiary@0.2
│  │  72 r12│  [ 选择 / ✓ 已选择 ]        │
│  └────────┘                              │
│  👤 author      📅 %1$s 更新     12 bold │  row gap 8, v-pad 16
│  ── divider (bottom pad 16) ──           │
│  【更新日志】                    12 bold │
│  updateBrief              12, ≤8 lines   │  spacer 8
│  desc  or  暂无简介        12            │
│  ── divider (v-pad 16) ──                │
│  代码详情                          ›     │  17 accent — hidden for APP items
└──────────────────────────────────────────┘
```
**Blocks:** 1 Icon 72 r12 `surface.tertiary`@0.2 (iOS a 64 box with a 24 glyph `:25-30`; Android flat 72
`:69-77`). 2 Title `title2` 20/Bold 1 line; brief 12, 2 lines (`:32-39`, `:79-93`).
3 Select button — COND selected shows a check + `已选择`, else `选择` (`:40-61`, `:96-133`).
4 Meta row — COND author non-blank; COND date non-blank (`:68-91`, `:138-184`).
5 Update log — COND `updateBrief` non-blank; heading `【更新日志】` 12/Bold then body 12, **≤ 8 lines**
(iOS prints one combined string, unclamped `:102`). 6 Description — `desc`, falling back to `暂无简介`
when blank (`ScoreJsCalcDetailView.kt:206`). 7 `代码详情` — COND a URL exists **and** the item is not an
APP item; TAP opens it externally.
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Outer pad · icon · gap | spacer 16 + h16 `:23,:129` · 64 box/24 glyph `:25-30` · 8 `:63` | `Column(padding 16.dp)` `:64` · 72dp `:69-77` · 16dp `:66` | 16 all sides · **72** · 16 |
| Title · brief · select pad | `title3` 20 1 ln `:32-35` · `caption` 2 ln `:36-39` · h8 **v8** `:57-60` | `title2` 20 Bold `:79-85` · `caption` 2 ln `:87-93` · h8 **v4** `:100-106` | 20/Bold · 12 · h8 v4, r8, 17/Bold `accent` |
| Selected · author/date | `checkmark` + `已选择` gap 4 `:45-50` · 12 2 ln maxW 150, always shown `:73-78` | `Check` 24 + `已选择` gap 2 `:114-129` · 12/Bold 1 ln maxW 200, hidden when blank `:153-160` | check 20 + `已选择` gap 4 · 12, 1 line, maxW 200, **hidden when blank** |
| Meta pad · log · desc | `padding(.top, 8)` `:93` · one string `【更新日志】\n%@`, unclamped `:102` · blank when absent `:106` | `padding(vertical = 16.dp)` `:139` · heading + body `maxLines 8` `:190-201` · `暂无简介` `:206` | v-pad 16 · heading 12/Bold + body 12 **≤ 8 lines** · **fallback `暂无简介`** |
| Code row guard · nav title | `!url.isEmpty` `:112` · none set | `url.isNotEmpty() && type != APP` `:213` · untitled `:63` | **also hidden for APP items** · none |
**Strings:** `选择计算方式` · `选择` · `已选择` · `%1$s 更新` · `【更新日志】` · `暂无简介` · `代码详情`.
**States:** empty — the description falls back to `暂无简介` · loading / error — none.
**Divergence:** iOS ignores the top and bottom safe areas on the picker (`ScoreJsCalcView.swift:19`);
Android keeps them. iOS passes the item **object** to the detail route; Android passes only a URL and
resolves it through a process-wide `ScoreJsCalcItemHolder.urlItemMap` (`ScoreGraph.kt:36-39`), which
silently renders nothing after process death — pass the object. iOS always renders the author row.

---

### 10. JS F2 editor (JS代码编辑) — platforms: **iOS only**
**Purpose:** Author the custom JavaScript that computes F2.
**Entry:** `编辑代码` on the Settings F2 card → `Route.scoreJsF2`.
**Layout:**
```
┌──────────────────────────────────────────┐
│ nav JS代码编辑 (inline)    [重置] [保存] │  ScoreJsF2View.swift:34–:52
│ VStack · pad 16                          │
│  ┌────────────────────────────────────┐  │
│  │ inputJsonStr的示例数据为            │  │  :19
│  │ [ read-only JSON sample ]   h 128  │  │  :20–:21
│  │ 请完成calc方法，返回相应的数据      │  │  :22
│  └────────────────────────────────────┘  │
│  ┌────────────────────────────────────┐  │
│  │ CodeEditor(javascript, .ocean) r16 │  │  :25–:26
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘  surface.primary
```
**Blocks:** 1 Example card — label, read-only JSON sample at height **128**, instruction (`:19-22`).
2 Editable editor bound to the code, language `.javascript`, theme `.ocean`, radius 16 (`:25-26`).
3 `重置` → restores the seed template (`ScoreJsF2ViewModel.swift:76-80`). 4 `保存` → evaluate the script
against a one-item fixture (高等数学 / 王老师 / 5.0 学分 / 96 分) and require `[Double, [String]]`
(`:43-65`); success persists `f2JsCode`, toasts `保存成功`, posts `.ham_scoreUpdated` (`:66-68`); failure
toasts `存在语法错误` / `请修复语法错误，然后重试` (`:70-73`). 5 Seed template — a `calc` JSDoc block
returning `[100, []]` (`:26-37`).
**Values:** screen pad 16 (`:29`) · card pad/radius/bg 16/16/`b2Color` (`CardView.swift:22,23,128`) ·
sample editor height **128** (`:21`) · editor radius 16, theme `.ocean` (`:26`,`:25`) · nav title
`JS代码编辑` inline (`:34-35`) · two trailing toolbar items (`:37`,`:45`).
**Strings:** `JS代码编辑` · `inputJsonStr的示例数据为` · `请完成calc方法，返回相应的数据` · `重置` ·
`保存` · `保存成功` · `存在语法错误` · `请修复语法错误，然后重试`.
**States:** empty — `code` falls back to the seed template when `f2JsCode == ""`
(`ScoreJsF2ViewModel.swift:113-117`), so the editor is never empty · loading — none · error — a toast.
**Divergence:** Android has no editor — `ScoreRoutes.JsF2Edit` is wired to the RN **picker**
(`ScoreGraph.kt:31-33`), `ScoreCalcType` has no hand-written-JS member (`ScoreCalculator.kt:14-18`), and
no editor strings exist in `SCSTR:`. The only JS-authoring surface on Android is the RN dev card
`开发调试`, which verifies pasted code but never writes `f2JsCode`.

---


## 7. CourseScore (课程评分 CourseScore)

Module colour `brand.coursescore` #283593 — identity only; every control uses `accent`.

### 2.1 Gate / intro (权限请求)

```
┌──────────────────────────────────────────────┐
│ 取消                                          │ 17 / Regular · accent
│                 🔒                            │ lock 48 · accent
│              权限请求                          │ title2 22 / Bold
│        "给分"需要使用你的成绩数据                │ 17 / Regular · text.secondary
│┌────────────────────────────────────────────┐│ panel h 350 · r 8 · surface.tertiary
││ 使用"给分"前Ham会自动…                       ││ pad 16 · gap 16 · body 17 / Regular
││ 你的以下信息将会被上传到服务器                 ││
││ - **学号不可逆特征值** 用于下次更新数据         ││
││ - **脱敏成绩信息** 用以提供数据                ││
││ - **设备信息** 辨别你是否正常使用Ham           ││
││ 数据库里的成绩信息不能逆向定位…                ││
││ 同意后，每次使用"给分"前，Ham会…              ││
│└────────────────────────────────────────────┘│
│ ⊘ 你尚未获取成绩信息，因此无法使用"给分"         │ ← when no scores yet
│   请前往"成绩"页面获取成绩后重试 · <server text> │
│┌────────────────────────────────────────────┐│ ← when scores exist
││                  授权                        ││ filled primary h48 r8
│└────────────────────────────────────────────┘│
└──────────────────────────────────────────────┘
```

**Blocks** — 1. **`取消`** (always; borderless; **tappable** → dismiss; dismissing without consent
pops the module — `CSIOS/intro/CourseScoreIntroView.swift:99-105`,
`CSAOS/ui/intro/CourseScoreIntroView.kt:81-91`) · 2. **lock icon** (always; `:28-30`, `:103-106`) ·
3. **title `权限请求`** (always; `:32-33`, `:110`) · 4. **subtitle** (always; `:39`) ·
5. **consent panel** (always; scrollable, fixed height — `:41-63`, `:115-211`) ·
6. **warning block XOR `授权`** (`hasFetchScore`; the button **tappable** → grant consent, upload
scores, hide — `:65`, `:213-232`, `CSAOS/ui/CourseScoreViewModel.kt:60-63`).

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Sheet / row padding | `.sheet` after 0.3 s `CourseScoreView.swift:29,37-45` / `.padding()` `:97` | `HamSheet` after `delay(500)` `CSAOS/ui/CourseScoreView.kt:46-48,60-67` / `16.dp` `:77` | sheet after **0.3 s** / padding 16 |
| Lock size / tint | `56` / `.blue` `:28-30` | `56.dp` / `ham_gray` `:103,106` | **48 / `accent`** |
| Title | `.title.bold()` `:33` | `HamFontStyle.title` `:110` | `title2` **22 / Bold** |
| Panel height / radius / bg | `350` `:62` / `10` `:63` / gray@0.1 `:61` | `320.dp` `:121` / `12.dp` `:118` / `ham_gray`@0.2 `:119` | **350** / **8** / `surface.tertiary` |
| Panel padding / body / gap | h16 `:60` / `.system(16)` `:58` / `16` `:43` | `8.dp` `:120` / default / `16.dp` `:122` | 16 / `body` **17 / Regular** / 16 |
| Bullet markup | Markdown `**…**` **inside** the localized string `:49-51` | `buildAnnotatedString` + `FontWeight.Bold`, hardcoded `"- "` `:139-193` | **one localized string per bullet, term marked `**…**`, renderer parses it** |
| `授权` button | `.blue` bg, white, pad v16, r10 `:91-93` | h 48, `ham_blue`@0.15, r12, `headlineBold` `:219-228` | **filled primary: h 48, r 8, `accent`, white 17 / Bold** |
| Warning icon / text | `xmark.circle.fill` `:69` / `.red` 12 `:74,78` | `Close` 16 on circle `:239-247` / `ham_red` `:251-257` | `icon.sm` 20 `text.danger`; message 17 / Regular + subtitle 12 / Regular, both `text.danger` |
| Server description | CCKV `.courseScoreEnableDescription` `:18,80` | CCKV `CourseScoreEnableDescription` `:63` | CCKV-driven, not localized |

**Strings:** `取消` (`LS:237`/`STR:9`) · `权限请求` (`LS:723`/`STR:10`) ·
`"给分"需要使用你的成绩数据` (`LS:870`) · `使用"给分"前Ham会自动将你的成绩数据匿名发送到Ham的服务器上，作为该功能的数据来源。`
(`LS:544`/`STR:11`) · `你的以下信息将会被上传到服务器` (`LS:535`/`STR:12`) ·
`- **学号不可逆特征值** 用于下次更新数据` (`LS:484`; Android splits `STR:13`+`STR:14`) ·
`- **脱敏成绩信息** 用以提供数据` (`LS:485`; `STR:15`+`STR:16`) · `- **设备信息** 辨别你是否正常使用Ham`
(`LS:486`; `STR:17`+`STR:18`) · `数据库里的成绩信息不能逆向定位到任何一个人，Ham也不会将这些成绩作为非法用途`
(`LS:678`/`STR:19`) · `同意后，每次使用"给分"前，Ham会自动上传你的成绩信息。` (`LS:604`/`STR:20`) ·
`授权` (`LS:670`/`STR:21`) · `你尚未获取成绩信息，因此无法使用"给分"` (`LS:533`/`STR:22`) ·
`请前往"成绩"页面获取成绩后重试` (`LS:803`/`STR:23`).

**States:** `hasFetchScore ? 授权 : warning`. No loading state. Consent writes back
(`CourseScoreConfig.shared.permissionAgreed = true`, `CourseScoreView.swift:42`). Entry requires
consent **and** at least one score fetch (`CSAOS/ui/CourseScoreView.kt:36`).

`Divergence:` Android splits each bullet into two resources so translators see different
granularity; it also uploads scores on consent and uses typographic quotes where iOS uses escaped
ASCII.

### 2.2 Search home

```
┌──────────────────────────────────────────────┐
│ 给分                              👤 我的数据 │ nav title + trailing entry
│                 ▤                            │ BarChart 64 · brand · v-pad 32
│┌────────────────────────────────────────────┐│ r 8 · surface.tertiary · pad h16 v10
││ 搜索                                        ││ 20 / Bold · text.secondary
│└────────────────────────────────────────────┘│
│ ←→  [📖 高等数学] [👤 张三] [📖 线性代数]       │ 4 rows · cap 30 · newest first
│ 其他服务                                      │ 17 / Bold
│┌────────────────────────────────────────────┐│ card r16 pad16
││ (icon) Title                             › ││
││        Subtitle                            ││
│└────────────────────────────────────────────┘│
└──────────────────────────────────────────────┘
```

**Blocks** — 1. **spacer 16** (`CSIOS/home/CourseScoreHomeView.swift:19`) · 2. **nav + `我的数据`**
(gated on CCKV `enableCourseCenter`; **tappable** → course center when logged in, else the login
dialog — `:34-46`, `CSAOS/ui/main/CoursScoreMainView.kt:63-76`) · 3. **brand icon** (always; not
tappable — `home/component/CourseScoreHomeViewHeader.swift:10-13`, `CoursScoreMainView.kt:104-111`) ·
4. **search bar** (always; **tappable** → search; matched-geometry ids `search_item` /
`search_item_text`, transition `.easeIn(0.3)` — `home/component/CourseScoreHomeViewSearchBar.swift:17-36`,
`CoursScoreMainView.kt:112-134`) · 5. **history cloud** (when non-empty; chips **tappable** → run
that search — `home/component/CourseScoreHomeViewHistoryCard.swift:22-35`,
`CoursScoreMainView.kt:137-180`) · 6. **external-service card** (when the CCKV list is non-empty;
rows **tappable** → open the URL, logs `promotion_btn` —
`home/component/CourseScoreHomeViewExternalServiceCard.swift:16-55`).

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Page background | `ham_bg_b1Color` `main/CourseScoreMainView.swift:26` | navigation default | `surface.primary` |
| Brand icon / v-pad | `48` `…Header.swift:12` / `16` spacer `CourseScoreHomeView.swift:19` | `64.dp` `CoursScoreMainView.kt:107` / `32.dp` `:106` | **`icon.xl` 64** / **32** |
| Search radius / fill / padding | `16` `…SearchBar.swift:35` / gray@0.15 `:32-33` / `.padding()`+16 lead `:29,23` | `16.dp` `:122` / `ham_gray`@0.2 `:61,123` / h24 v10 `:124` | **8** (text field) / `surface.tertiary` / **h16 v10** |
| Search label | `.title.bold()` **28** `:26` | `24.sp` Bold `:129-130` | `title3` **20 / Bold**, `text.secondary` |
| History rows / cap / order | `4` `…HistoryCard.swift:23` / `.prefix(30)` `:24` / Realm `createTime` desc `:14` | `8` `:136` / none `:146` / `asReversed()` `:146` | **4** / **30** / newest first |
| History height | intrinsic `…HistoryFlowLayout.swift:57-59` | fixed `240.dp` `:139` | intrinsic |
| Chip radius / pad / fill | `8` `…HistoryItem.swift:29` / h8 v6 `:26-27` / gray@0.15 `:28` | `10.dp` `:158` / h4 `:160` / `ham_gray`@0.2 `:159` | **6** / **h6 v4** / `text.secondary`@0.10 |
| Chip label / line limit / icon | 17 @0.65 `:25` / `1` `:23` / `text.book.closed.fill`, gap 4 `:20-21` | `16.sp` secondary `:174` / none / `Book` @18, gap 2 `:161-173` | **12 / Bold `text.secondary`** / **1** / `icon.sm` **20**, gap **4** |
| `其他服务` header | bold, gap 8 `…ExternalServiceCard.swift:30-32` | **absent** | 17 / Bold, gap 8 — **present on both** |
| Service row | icon 20 on a 40 circle @0.1, title semibold, subtitle 12 gray, chevron `MyViewSettingCard.swift:211-244` | absent | as iOS; card r16 pad16 |
| `我的数据` | `person.fill` + bold, gap 4, brand `CourseScoreHomeView.swift:54-61` | `Person` + `bodyBold`, gap 4, brand `:81-95` | `icon.md` 24 + 17 / Bold, gap 4, `brand.coursescore` |

**Strings:** `给分` (`LS:51,317`/`STR:3`) · `我的数据` (`LS:877`/`STR:4`) · `搜索` (`LS:672`/`STR:5`) ·
`其他服务` (`LS:872`) · external titles/subtitles from CCKV JSON keyed by locale
(`…ExternalServiceCard.swift:20-23`) · history keywords are user data.

**States:** empty history → the cloud collapses to zero height, no placeholder · empty external
list → the whole section disappears · not logged in → `我的数据` still shows and opens the login
dialog · no loading or error state (both read synchronously from local storage).

`Divergence:` Android has no external-service card, uses an 8-row fixed 240 dp grid, has no
history cap and no line limit on chips.

### 2.3 Search and results

```
SEARCH                                RESULTS
┌──────────────────────────────┐      ┌──────────────────────────────┐
│‹ [ 输入课程名或授课人      ] ⊗│      │┌────────────────────────────┐│
├──────────────────────────────┤      ││ 🔍 高等数学                 ││
│ 🔍 高<strong>等</strong>数学↗│      │└────────────────────────────┘│
│ 🔍 高等代数 (张三)         ↗│      │[人数倒序][均分倒序][中位数倒序]│
└──────────────────────────────┘      │┌────────────────────────────┐│
                                      ││ 高等数学 (2 lines) │  均分  ││
                                      ││ 张三               │  84.5  ││
                                      ││ 128位同学的成绩    │中位:90 ││
                                      ││ 0-59 ▓▓▓▓ 12      │    ⎘   ││
                                      ││ 60-69 ▓▓▓▓▓▓▓▓ 40 │    💬   ││
                                      ││ 90-100 ▓ 11       │        ││
                                      │└────────────────────────────┘│
                                      └──────────────────────────────┘
```

**Blocks** — 1. **back affordance** (always; iOS custom `chevron.left` with the system button
hidden, Android `popBackStack()` — `CSIOS/search/cell/CourseScoreSearchViewSearchBar.swift:21-28`,
`CSAOS/ui/search/CourseScoreSearchResultView.kt:59-70`) · 2. **text field** (always; 20 / Regular,
1 line, autofocus after 0.3 s — `:30-37,62-66`, `:80-87`) · 3. **clear button** (when the keyword
is non-blank; **tappable** → clear keyword, list and load state — `:39-49`, `:89-105`) ·
4. **divider** (always — `:57-58`, `:108`) · 5. **suggestion rows** (when the keyword is non-blank;
**tappable** → commit — `search/cell/CourseScoreSearchViewSearchResultItem.swift:22`,
`CourseScoreSearchResultView.kt:117-120`) · 6. **result header** (always on results;
**tappable** → back to the field — `CSIOS/result/CourseScoreResultView.swift:19-23`,
`CSAOS/ui/result/CourseScoreResultView.kt:88-91`) · 7. **filter chips** (**always**, not CCKV-gated
— `result/component/CourseScoreResultViewSearchBar.swift:52-57`,
`CSAOS/ui/result/CourseScoreResultView.kt:125-133`) · 8. **result list** (one card per course;
pagination fires on the second-to-last card — `CourseScoreResultView.swift:26-28`,
`CourseScoreResultView.kt:223-227`).

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Placeholder | `输入关键词` `…SearchBar.swift:30`, `LS:820` | `输入课程名或授课人` `:80`, `STR:6` | **`输入课程名或授课人`** |
| Field font / row padding | `.system(20)` `:55` / none + 16 spacer `:57-58` | default `:80-87` / v `8.dp` `:56` | 20 / Regular / **v 8** |
| Clear icon | `multiply.circle.fill`, trailing 8 `:47-49` | `Clear` 18 on `ham_gray` circle, end 8 `:96-105` | `icon.sm` 20, margin 8, min 44 target |
| Row gap / h-padding | `8` `…SearchResultBody.swift:17` / ≈16 `:22` | `8.dp` `:112` / `12.dp` `CourseScoreSearchItemView.kt:42` | 8 / **16** |
| Row icon / gap / trailing icon | `magnifyingglass.circle.fill` 24 `:24-26` / default / `arrow.up.right` `:30-31` | `Search` 16 on gray@0.3 `:51-58` / `8.dp` `:59` / `NorthEast` 16 `:73-82` | `icon.sm` 20 / 8 / `icon.xs` 12, 16 from the text |
| Hit highlight | `t1Color`/`t2Color`, tag lengths 8 & 9 hardcoded `:38-59` | `buildSearchString`; spans 14.sp then overridden by `body` 16.sp `CourseScoreSearchHitUtils.kt:11-35`, `CourseScoreSearchItemView.kt:60-70` | strong `text.primary`, plain `text.secondary`, **16 / Regular**; malformed markup must not crash |
| Header pad outer/inner/radius | 16 `:48` / v16 h12 `:38-39` / `16` `:45` | 16 `:92` / 12 `:101` / `16.dp` `:94` | 16 / 12 / **16** |
| Header fill / shadow / icon | `ham_bg_b1`+gray@0.1 `:41-44` / `.gray`@0.5 r16 `:46` / `magnifyingglass` 20 `:32-33` | `ham_gray`@0.1 over `ham_bg_b1` `:95,99` / elevation 15 `:94` / `Search` 24, gap 4 `:106-112` | `surface.secondary` + `text.secondary`@0.10 / elevation 15 / `icon.sm` 20, gap 4 |
| Chip radius / pad / font | `6` `:75` / h6 v4 `:69-70` / `.caption` 12 `:68` | `6.dp` `:71` / h6 v4 `:73` / `caption` 12 `:75` | **6** / h6 v4 / **12 / Bold** |
| Chip colours | selected `.blue`, unselected `.gray` `:63` | `ham_blue` / `ham_gray` `:67` | selected `accent`@0.10 + `accent`; unselected `text.secondary`@0.10 + `text.secondary` |
| Chip gate / order / default | none / totalDesc, scoreDesc, medianDesc `:53-55` / `.totalDesc` `result/CourseScoreResultViewModel.swift:24` | CCKV `enableSearchResultFilter` `:125` / same / same | **always** / 人数倒序, 均分倒序, 中位数倒序 / **人数倒序** |
| List gap / padding / card padding | `8` `…ResultViewBody.swift:21` / ≈16 `CourseScoreResultView.swift:30` / `HamCardView` 16 `CourseScoreSingleCard.swift:36` | `8.dp` `:219-221` / 16 `:219-221` / `padding(0)` + 12 insets `:272,280-283` | 8 / 16 / **16 uniform** |
| Name / instructor / gap to stats | bold, 1 line `:40-41` / 12 `:42-43` / `16` `:46` | 16 Bold, 2 lines `:286-292` / 12 `:293` / `8.dp` `:294` | 17 / Bold **2 lines** / 12 / Regular `text.secondary` / **8** |
| Range row gap / label width / format | `4` `:49` / `45`, 11 `:132-134` / `"\(from)-\(to)"` hardcoded `:52` | `2.dp` `:297` / `48.dp`, 12 `:311-319` / `"${from}-${to}"` hardcoded `:312` | 4 / **48**, 12 / Regular tabular / **one localized `%1$d-%2$d`** |
| Bar max / height / radius / colour | `180` `:130` / `6` / `3` `:138-139` / server `:136` | `180.dp` `:328` / `4.dp` / `2.dp` `:335-337` / server → colourMap → `ham_blue` `:307-310` | 180 / **4** / **2** / server colour else **`accent`** |
| Zero-count row | zero-width bar `:140-143` | omitted + weighted spacer `:333-356` | **omitted**, count pad 4 when non-zero |
| Stats column / divider | only when `total > 2` `:61` / h `130` `:63` | always / `120.dp`, end 81 `:363-368` | **always** / **130** tall, 16 top/bottom |
| Average / median | `"%.1f"`, `title` rounded 28 `:67-68` / always `:69-70` | `"%.1f"`, `title` 24 `:439-443` / `median > 0` `:444-450` | **`%.1f`, 28 / rounded Bold** / **only when `median > 0`** |
| Share / comment buttons | `square.and.arrow.up.fill` 16 / `bubble.left.and.bubble.right.fill` 14, both on a 32×32 r8 gray@0.2 plate, offsets (−16,16) and (−16,−16) `:81-125` | `IosShare` / `Forum` 20 on r8 chips, top/end 12 and bottom/end 12 `:375-423` | `icon.sm` 20, 32 target, 12 from top/end and bottom/end |
| Bottom spacer | none | `navigationBarHeight + 24.dp` `:257` | `navigationBarHeight + 24` |

**Strings:** `人数倒序` (`LS:871`/`STR:36`) · `均分倒序` (`LS:874`/`STR:37`) · `中位数倒序`
(`LS:520`/`STR:38`) · `未知倒序` (`LS:878` — 4th default case, Android absent) ·
`%1$d位同学的成绩` (`LS:477`/`STR:45`) · `均分` (`LS:275`/`STR:46`) · `中位数: %1$s`
(`LS:519`/`STR:47`) · `%1$s-%2$s的给分数据` (`LS:467`/`STR:34`) · `分享到` (`STR:44`) ·
`来自Ham` (`STR:35`).

**Share** — system share sheet on both. URL
`{DocHost}/inner/course-score/result?name=&instructor=&data=&dataTime=&sign=&v=2`; signature
`md5("v2‖name‖instructor‖json‖yyyyMMddHHmm‖<shared salt>")` lowercased —
`CourseScoreSingleCard.swift:166-180`, `CSAOS/ui/search/CourseScoreSearchViewModel.kt:269-273`.
Host from remote config, `whu-ham.github.io` as fallback.
**States:** empty → add an empty state (icon `icon.xl` 64 `text.tertiary` + 12 / Regular
`text.tertiary`) · loading → no spinner · error → gRPC toast only · end of list → no footer ·
not logged in → the comment button opens the login dialog.

`Divergence:` iOS splits suggestions and results into two routes; Android keeps both in one with
an `AnimatedContent` switch. Android gates the chips behind CCKV, forks the comment destination
between `CourseDetail(id)` and the legacy `Comment(name, instructor)`
(`CourseScoreResultView.kt:236-249`), and shares to QQ only through a `HamSheet` at 0.4 height
(`:83,146-201`). Its clear button is always visible and does not reset the list, and its list fade
observes the suggestion list rather than the results (`:212-217`).

### 2.4 Course detail (课程详情)

Five stacked cards · gap 16 · page padding 16 · bottom 32 · `surface.primary`.

```
┌──────────────────────────────────────────────┐
│< 课程详情                                     │
│┌────────────────────────────────────────────┐│ ① stat card · pad 16
││ 高等数学 (2 lines)          │      均分     ││  28 / rounded Bold
││ 张三                        │      84.5    ││  12 / Regular · tabular
││ 128位同学的成绩              │      中位:90 ││
││ 0-59  ▓▓▓▓ 12               │              ││  bars 180 max · 4/2
││ 60-69 ▓▓▓▓▓▓▓▓ 40           │              ││
││ 90-100 ▓ 11                 │              ││
│├────────────────────────────────────────────┤│
││ [全部] │ 2024-1学期  2024-2学期 …        → ││ semester selector
│└────────────────────────────────────────────┘│
│┌────────────────────────────────────────────┐│ ② rate card · ★ watermark 128 @0.15
││ 评分                                        ││
││ ★☆☆☆☆ ▓▓▓▓▓▓▓▓░░░░░░░ 40        4.0       ││ 5 rows × 5 stars · track 144 4/2
││ ★★☆☆☆ ▓▓▓▓░░░░░░░░░░ 12        平均       ││
││ ★★★★★ ▓▓░░░░░░░░░░░░░ 4                   ││
││ 67位同学的评分                              ││
│└────────────────────────────────────────────┘│
│┌──────────────────┐┌────────────────────────┐│ ③ function card · 1f : 1f
││ ♥ 想上            ││ ✎ 写评价                ││ tinted · r12 · v-pad 8
││   36人想上这门课   ││   说点什么…             ││
│└──────────────────┘└────────────────────────┘│
│┌────────────────────────────────────────────┐│ ④ comment card
││ 评论 24                                     ││
││ (av) name  ★★★★★                           ││ avatar 32 · username 12
││      body text                              ││ 14 / Regular
││      2026-01-05 12:30                👍 8   ││ 12 / secondary
││ ────────────────────────────────────────── ││
││                  查看全部 >                  ││
│└────────────────────────────────────────────┘│
│┌────────────────────────────────────────────┐│ ⑤ self-review (conditional)
││ 你的评价                       收获8个 👍    ││
││ ★★★★★                                       ││ stars 24 · brand.score
││ ────────────────────────────────────────── ││
││ 讲课很清楚                                   ││
││ 2026-01-05 12:30                            ││
│└────────────────────────────────────────────┘│
└──────────────────────────────────────────────┘
```

**Blocks** (`CSIOS/coursedetail/CourseScoreCourseDetailView.swift:60-83`) — ① stat card (always
`:61-65`) · ② rate card (always `:66`) · ③ function card (always `:67-73`) · ④ comment card
(always `:74`) · ⑤ self-review card (**when a rating or a comment exists** `:75-78`).

**Container**

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| List | `LazyVStack(spacing: 16)` `:60` | `LazyColumn(spacing 16, contentPadding v32 h16)` `CSAOS/ui/detail/CourseScoreCourseDetailView.kt:117-120` | gap 16, padding h16 v32 |
| Background / nav title | `ham_bg_b1Color` `:44` / `课程详情` `:46`, `LS:815` | inherited / `course_score_course_detail_title` `:60` | `surface.primary` / `课程详情` |
| Loading / error | `ProgressView` with no response `:29-34` / `CourseScoreCourseDetailErrorView { vm.fetchData() }` `:25-28` | same / same, but takes **no message** `ui/detail/CourseScoreCourseDetailErrorView.kt` | centred spinner, full page / empty-state icon `icon.xl` 64 `text.tertiary`; title 17 / Bold; subtitle 12 / Regular = server message when present else `试试重新请求呢`; retry tinted button h48 r12 |
| Refresh signal | `.ham_courseDetailShouldRefresh` `…ViewModel.swift:28-37` | `NotificationChannel.CourseDetailNeedRefresh` | refresh on the notification |

**① Stat card** — `CSIOS/coursedetail/component/CourseScoreCourseDetailCourseGradeStatCard.swift`,
`CSAOS/ui/detail/component/CourseScoreCourseDetailViewStatCard.kt`

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Card padding | `0` + row h16, v12 `:27,58,61` | `0` + 12 margins `:80,93-97` | **16 uniform** |
| Name / instructor | bold, 1 line `:33` / 12 `:35` | `bodyBold`, 2 lines `:231-232` / `caption` `:236` | 17 / Bold **2 lines** / 12 / Regular `text.secondary` |
| Gaps (name→count / count→bars / rows) | 8 / 4 / 4 `:38,41,79` | 8 / — / 2 `:237,153` | 8 / 4 / 4 |
| Stats column / divider | width 64, gap −2, h 130 `:47-56` | divider 120, end 81 `:100-105` | divider **130**, gap 4, width 64 |
| Average / median | `"%.1f"` 28 rounded bold `:51-52` / always, 11 `:53-54` | `"%.1f"` `title` 24 `:255` / `median > 0` `:257-259` | **`%.1f`, 28 / rounded Bold** / **only when `median > 0`**, 12 / Regular |
| Bar max / h / r / colour | `180` `:80` / `4` / `2` `:90-91` / server `:88` | `180.dp` `:191` / `4.dp` / `2.dp` `:198-199` / server else `ham_gray` `:170-171` | 180 / 4 / 2 / server colour else **`accent`** |
| Band label | width 45, 11 `:83-85` | width 48, 12 `:175-181` | **48**, 12 / Regular tabular, localized `%1$d-%2$d` |
| Loading overlay | `ZStack` spinner over black@0.4, swallows taps `:62-73` | `matchParentSize` + `ham_black`@0.35 + spinner 48/4 `:126-143` | **scrim black@0.35**, spinner 48, swallows taps |
| Semester chip | `全部` first, pad 4, r6, selected blue@0.15, unselected clear, text blue, 12 `:109-114` | pad 4/4, r6, selected blue@0.1 `:325-339` | chip r6 h6 v4; selected `accent`@0.10 + `accent`; unselected transparent + `text.secondary` |
| Semester divider / scroller | h 24, leading pad 8 `:118-119` / h-scroll, no indicators, gap 6, pad 8, row lead 16 `:121-140` | 24.dp, start 8, `ham_gray`@0.3 `:286-290` / `LazyRow` gap 6, pad h12, row start 12 `:275,294-295` | 24 tall, gap 8, `surface.tertiary` / h-scroll, no indicators, gap 6, padding 16 |
| Semester format / error / guard | `%@-%@学期` `:127` / toast, selection and stats unchanged `…StatCardViewModel.swift:45-51` / `if loadState == .loading { return }` `:29-32` | `%1$d-%2$d学期` `:332-336` / same `:56-59` / **absent** `:44-66` | `%1$d-%2$d学期` / toast, keep previous selection and stats / **guard re-entrancy** |

**② Rate card** — `CSIOS/coursedetail/component/CourseScoreCourseDetailRateCard.swift`,
`CSAOS/ui/detail/component/CourseScoreCourseDetailViewRateCard.kt`

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Title | `评分` 17 / Bold `:17` | `course_score_rating_title` `:43` | `评分` 17 / Bold |
| Watermark | `star.fill` 128, gray@0.1, offset (24,24) `:19-23` | `Star` **172.dp**, gray@0.1, offsetY 24 `:45-47` | `icon.watermark` **128 @0.15**, bottom-trailing (16,16), clipped |
| Star rows / glyphs | `ForEach(1..<6)` always 5 `:33` | `repeat(maxStar)` from data `:66,70` | **5 rows × 5 glyphs** |
| Star size / tint / row gap | `8` `:35` / filled `.gray`, empty gray@0.2 `:36` / `2` `:27` | `10.dp` `:73` / `ham_gray`, `ham_gray`@0.2 `:75-76` / — | `icon.xs` **12** / filled `text.primary`, empty `text.secondary`@0.20 / 2 |
| Track width / h / r / fill | `144.0` `:30` / `4` / `2` / gray@0.2 `:43-45` | `144.dp` `:85` / `4.dp` / `2.dp` / `ham_gray`@0.1 `:83-86` | 144 / 4 / 2, track `surface.tertiary`, fill **`accent`** |
| Gaps (star→bar→count) | `8` `:46` / `8` `:48` | `8.dp` `:80` / `8.dp` `:95` | 8 / 8 |
| Count | `"\(total)"` 12, `t2Color` `:47-50` | `toString()` `caption2` 11, `ham_gray` `:97-99` | 12 / Regular **tabular**, `text.secondary` |
| Average / label / column | `"%.1f"` 28 rounded bold `:58-59` / `平均` 10 `:60-61` / width 84 trailing `:63` | **`"%.2f"`** 28 Bold `:112` / `course_score_average_simple` 10.sp `:113` / 84.dp end `:109-110` | **`%.1f`** 28 / rounded Bold / `平均` `caption2` **11** / width 84 trailing |
| Total | `%lld位同学的评分` 12, top pad 4 `:65-67` | `course_score_ratings_count_format` `:57` | 12 / Regular `text.secondary`, top pad 4 |

**③ Function card** — `CSIOS/coursedetail/component/CourseScoreCourseDetailFunctionCard.swift`,
`CSAOS/ui/detail/component/CourseScoreCourseDetailCreateReviewFunctionView.kt`

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Want / comment icon | `heart.fill` 24 `:33-34` / `paperplane.fill` 24 `:57-58` | `Favorite` **28.dp** `:157-161` / `Edit` **28.dp** `:203-207` | heart / **pencil (edit)**, `icon.md` 24 |
| Titles / subtitles | server `:36,60`; `%lld人想上这门课` or server hint, 12 `:38-40,62` | server `:163,209-210`; `course_score_want_count_format` `caption2` 11 `:165-169` | server titles 17 / Bold; subtitle 12 / Regular, leading aligned |
| Colour | `Color.blue` enabled / `Color.gray` not `:31,55` | `ham_blue` / `ham_gray` `:134,185` | enabled `accent`, disabled `text.secondary` (palette swap, no opacity) |
| Button radius / pad / bg / gap | r `12`, v-pad 8, `color`@0.1 `:45-47` / — | r `12.dp`, v-pad 8 `:145-149` / `8.dp` `:155` | r **12**, v-pad 8, `accent`@0.10 over `surface.secondary`; row gap 8, weights 1 : 1, icon→text 8 |
| Enabled | `!wantRecorded` / `info.enabled` `:49,72` | `!state.wanted` / `info.enabled` `:140,188` | disabled once recorded |
| Want tap | `vm.recordWant()` `:29` — optimistic `+1`, rollback `…FunctionCardViewModel.swift:33-53` | `:99-100`, rollback `:107-108` | optimistic `+1` + `recorded = true`, rollback on error; coroutine **tied to composition lifecycle** |
| Comment tap | push create-review `CourseScoreCourseDetailView.swift:67-73` | `onGoToCreateReview()` `:187` | push create-review with `star` / `comment` args |

**④ Comment card** — `CSIOS/coursedetail/component/CourseScoreCourseDetailCommentCard.swift`,
`CSAOS/ui/detail/component/CourseScoreCourseDetailCommentView.kt`

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Title / body gap / empty | `评论 %1$d` `:39` / `12` `:40` / `暂无评论` `t1Color` `:42-43` | `course_score_comments_count_format`, `0` when nil `:122-125` / `12.dp` `:127` / `course_score_no_comments` `:135` | `评论 %1$d` / 12 / `暂无评论` 12 / Regular `text.tertiary` |
| Row divider | v-pad 4 `:49` | `HamDivider()` per item `:131` | 1 px `surface.tertiary`, v-pad 4, **not after the last** |
| `查看全部` | `chevron.right` 12, centred `:53-59` | `course_score_view_all` + `ChevronRight`, `text.secondary` `:145-152` | 12 / Regular **`accent`** + `icon.xs` chevron, centred → comment thread |
| Disabled branch | **no card, no title**; `text.bubble.fill` **64**, blue@0.65, pad 12 + server `reason` bold, gap 16 `:66-82` | card titled `评论`; `Comment` **72.dp** blue@0.65, gap 12 `:81-94` | **no card, no title**: `icon.xl` 64 `accent`@0.65 + server reason 17 / Bold, gap 16 |

**Comment item** (shared with §2.5) —
`CSIOS/coursedetail/component/coursecomment/CourseScoreCourseDetailCommentItemView.swift`,
`CSAOS/ui/detail/component/CourseCommentItemView.kt`

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Avatar / fallback | `WebImage` 32×32 circle, fade 0.5 `:45-56` / gray@0.2 + `face.smiling` 20 `:27-38` | `SubcomposeAsyncImage` 32 circle `:68-97` / `ham_gray`@0.2 + `Person` `:50-64` | 32×32 circle, cross-fade / `surface.tertiary` + person `icon.sm` 20 |
| Avatar branch | — | `avatar_url?.isEmpty() == false` → **null URL takes the network branch** `:49` | **null or empty URL takes the fallback branch** |
| Row gaps / username | `8` body, `4` username→stars `:59-60` / `.caption2` 11 `:62` | `8.dp` `:48,101` / `caption` 12, 1 line, Ellipsis `:105-112` | 8 / 4 / 12 / Regular, 1 line, Ellipsis |
| Star glyph | `star.fill` **10**, `t1Color` `:66-68` | `Star` **16.dp**, **no tint** `:115-121` | `icon.xs` **12**, **`text.primary`** |
| Body / timestamp | `14`pt, `t1Color` `:73-75` / `ham_toyyyymmddhhmm` 12 `t2Color` `:76-78` | `headline` 14.sp `:125-129` / `toYYYYMMDDHHMM()` `caption` secondary `:130-134` | **14 / Regular** `text.primary` / 12 / Regular `text.secondary` |
| Like icon / colour / count | `hand.thumbsup.fill`/`hand.thumbsup` 16 `:89-90` / `.blue`/`.gray` `:94` / `"\(likeTotal)"` 12, gap 4 `:88-92` | `ThumbUp`/`ThumbUpOutlined` 16.dp `:168-176` / `ham_blue`/`ham_gray` `:163-167` / `toString()` `:178` | 16 / liked `accent` else `text.secondary` / 12 / Regular tabular, gap 4 |
| Like enabled / optimistic | `!item.selfComment` `:96`, `…CommentItemViewModel.swift:26` / `±1` + flip, rollback `:45-48,59-64` | `!item.self_comment` `:139` / same `CourseCommentItemViewState.kt:44-53,69-78` | disabled on your own comment / optimistic ±1 with rollback; guard re-entrancy both directions |

**⑤ Self-review card** — `CSIOS/coursedetail/component/CourseScoreCourseDetailSelfReviewCard.swift`,
`CSAOS/ui/detail/component/CourseScoreCourseDetailSelfReviewView.kt`

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Gate | `hasRateInfo \|\| hasCommentInfo` `CourseScoreCourseDetailView.swift:75-76` | `self_review_info != null && (comment_info != null \|\| rate_info != null)` `…DetailView.kt:160-166` | render when a rating **or** a comment exists |
| Title / body gap | `你的评价` 17 / Bold `:17` / `12` `:18` | `course_score_your_review` `:37` / `8.dp` `:38` | `你的评价` / **12** |
| Stars | `star.fill` **24**, `.orange`, gap 2 `:22-26` | `Star` **32.dp**, `ham_orange` `:72-78` | `icon.md` **24**, gap 2, **`brand.score`** |
| Not-rated | `未评分`, `.title` **28** `:29-30` | `course_score_not_rated`, `title` 24 `:83-84` | `未评分`, `title2` **22 / Bold** |
| `收获%1$d个` / divider | + `hand.thumbsup.fill` 12, gap 0 `:35-39` / only when rating **and** comment exist `:45` | + `ThumbUp` 12.dp, gap 4 `:92-106` / `comment_info != null && rate_info != null` `:40` | 12 / Regular + `icon.xs` thumb, gap 4 / only when both exist |
| Comment body / time | body + `ham_toyyyymmddhhmm` 12 secondary, gap 8 `:50-54` | same, gap 4 `:47-57,168` | gap 4 |

**Strings:** `课程详情` (`LS:815`) · `%1$d位同学的成绩` (`LS:477`/`STR:45`) · `均分`
(`LS:275`/`STR:46`) · `中位数: %1$s` (`LS:519`/`STR:47`) · `%1$d-%2$d学期` (`LS:869`/`STR:39`) ·
`全部` (`LS:563`/`STR:43`) · `评分` · `平均` (`LS:649`/`STR:49`) · `%1$d位同学的评分`
(`LS:478`/`STR:50`) · `%1$d人想上这门课` (`LS:476`/`STR:51`) · `评论 %1$d` (`LS:794`/`STR:52`) ·
`暂无评论` (`LS:701`/`STR:53`) · `查看全部` (`LS:724`/`STR:56`) · `你的评价` (`LS:541`/`STR:57`) ·
`未评分` (`LS:717`/`STR:58`) · `收获%1$d个` (`LS:673`/`STR:73`) · `试试重新请求呢`.

`Divergence:` Android renders two-decimal averages (`…ViewRateCard.kt:112`), derives the star count
from data, uses a 172 dp watermark, larger stars (32.dp self-review, 16.dp comment, untinted),
`Favorite`/`Edit` icons, and a **titled** disabled-comment card; its error view cannot show a
server message, its stat card has no re-entrancy guard, and its avatar branch is inverted. iOS
gates only the self-review card; Android gates all five.

### 2.5 Comment thread (课程评论)

```
┌──────────────────────────────────────────────┐
│< 课程评论 24                                  │ nav title 17 / Bold
│ (av) name  ★★★★★                             │ avatar 32 · username 12
│      body text                                │ 14 / Regular
│      2026-01-05 12:30                   👍 8 │ 12 / secondary
│ ─────────────────────────────────────────── │ divider, except after the last
│ (av) name                                     │
│      body text                                │
│      2026-01-04 09:12                   👍 0 │
│                    到底咯                     │ end-of-list 12 / tertiary
└──────────────────────────────────────────────┘
```

**Blocks** — 1. **nav title** `课程评论 %1$s`, empty string when the total is unknown
(`CSIOS/commentnew/CourseDetailCourseCommentView.swift:51,61-69`; Android
`course_score_comments_title_with_count` `:61-64`) · 2. **list** (rows use the shared comment item
from §2.4④) · 3. **pagination trigger** (second-to-last row `:39`; Android `:91`) ·
4. **end-of-list text** (when `loadFinish`; `到底咯` 12 / Regular `text.tertiary`, centred —
absent on both today) · 5. **first-page spinner** (loading with an empty list `:30-32`; `:76-79`) ·
6. **error view** (failure **and** an empty list `:24-27`; `:68-71`) · 7. **trailing spinner**
(while paginating; Android delays it 1 s `:106-111` — show it immediately).

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Container / padding | `LazyVStack(spacing: 12)` `:34` / `.padding()` + bottom 32 `:48-49` | `LazyColumn` gap 12, `userScrollEnabled = false` + `bouncy` `:84-86` / top `statusBar + TOOLBAR + 16`, bottom 64, h 16 `:57,87` | list gap 12 / h 16, bottom 32 |
| Divider rule | `if comment != commentList.last` `:43-45` | `item != lastOrNull()` `:99` | except after the last |
| Title total | `response.total`, set once `CourseDetailCourseCommentViewModel.swift:61-63` | recomputed `:61-64` | **set once** from the first response |
| Load finish | `response.pageInfo.courseComment.isEmpty` `:65`; Android `:73` | — | stop paging when a page comes back empty |

**Strings:** `课程评论 %1$s` · `到底咯`.
`Divergence:` Android delays its trailing spinner by 1 s, disables list scrolling in favour of a
`bouncy` modifier, and recomputes the nav title on every response.

### 2.6 Create review (创建评价)

```
┌──────────────────────────────────────────────┐
│< 创建评价                             [ 发布 ]│ borderless 17 / accent
│            点击/滑动评分                       │ 17 / Regular
│        ★    ★    ★    ★    ★                │ 32 each · tap or drag
│   你给这门课程评过分了，不如去填写评价呢         │ 12 / danger · top pad 8
│ ──────────────────────────────────────────── │ divider · top pad 12
│┌────────────────────────────────────────────┐│
││ 发布一条课程评价吧～                          ││ minH 80 · maxH 180 · 6 lines
│└────────────────────────────────────────────┘│
│   评论需满足20-100字，当前0字                  │ 12 / secondary → danger
│   你评论过该课程了，不如去评下分呢              │ 12 / danger
└──────────────────────────────────────────────┘
   on success → 发布成功 / 已成功发布课程评分与评论 → dismiss + refresh detail
```

**Blocks** (`CSIOS/createreview/CourseDetailCreateReviewView.swift`,
`CSAOS/ui/comment/create/CourseCommentCreateView.kt`) — 1. **nav title `创建评价`** (`:48`; `:78`) ·
2. **`发布`** (always, hidden once `success`; **tappable** → submit; disabled while submitting —
`:55-57`; `:81-99`) · 3. **prompt `点击/滑动评分`** (`:78`; `:200`) · 4. **star row** (always;
5 stars; **tap sets** the value; **horizontal drag sets** it by hit-testing each star's bounds,
min drag distance 5 — `:80-110`; `:225-247`) · 5. **already-rated hint** (`!canRate`; 12 / Regular
`text.danger`, top pad 8 — `:113-118`; `:214`) · 6. **divider** (top pad 12 — `:31`; `:129`) ·
7. **comment field** (placeholder from server `comment_hint` else `发布一条课程评价吧～`; newlines
stripped — `:152-167`; `:150-165`) · 8. **counter** (always — `:197-199`; `:174-184`) ·
9. **already-commented hint** (`!canComment` — `:193-195`; `:171`) · 10. **success view**
(`:39`; `:111-114`).

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Star size / row width | `32` `:82-83` / fixed `32*5 + 16*4` `:96` | **`56.dp`** `:234-239` / unconstrained `Row` `:234` | `icon.lg` **32** / fixed `32*5 + 16*4` |
| Star fill / empty | `.orange` / gray@0.2 `:84` | `ham_orange` / `ham_gray`@0.2 `:237` | **`brand.score`** / `text.secondary`@0.20 |
| Drag clearance | — | clears focus + hides the keyboard `:206-207` | clear focus and hide the keyboard on drag |
| Placeholder colour | `t2Color`@0.3 `:152` | `text.secondary` `:165` | `text.tertiary` |
| Field height / lines | `80` … `180` `:159` / `6` `:156` | `heightIn(80,180)` `:157` / IME Done `:151-154` | 80 … 180 / max 6 lines, Done clears focus |
| Counter style / colour | `.caption` 12 `:199` / `t2Color` in range, `.red` out `:197` | `caption` 12 `:184` / `text.secondary` / `ham_red` `:176` | 12 / Regular / `text.secondary` in range, `text.danger` out |
| Counter when config missing | falls back to 20/100, **always renders** `:133-134` | **hidden**, and validation **skipped** `:174`; `CourseCommentCreateViewModel.kt:77-79` | **fall back to 20/100; always render and always validate** |
| Min / max / hint / padding | 20 / 100 / `发布一条课程评价吧～` `:133-135` / — | `dynamic_config` overrides `:156-157` / `16.dp`, gap 16 `:124-125` | 20 / 100 default, server-overridable / padding 16, gap 16 |
| Initial star | `star == 0 ? 5 : star` `…CreateReviewViewModel.swift:34` | `initStar.takeIf { it > 0 } ?: 5` `CourseCommentCreateViewModel.kt:51` | **5** when unset |
| `canRate` / `canComment` | `star == 0` / `comment.isEmpty` `:37-38` | `initStar == 0` / `initComment.isEmpty()` `:52,56` | rating editable when no incoming star; comment editable when no incoming comment |
| Submit | rate then comment; guard re-entrancy `:42-51,63-81` | rate then comment, each clearing its own flag `:90-134` | rate first, then comment; guard re-entrancy; validate the comment length only when the comment is editable |
| Post-success | `.ham_courseDetailShouldRefresh` `:92` | `NotificationChannel.CourseDetailNeedRefresh` `:68` | notify the detail screen to refresh, then dismiss |

**Strings:** `创建评价` · `发布` · `点击/滑动评分` · `你给这门课程评过分了，不如去填写评价呢` ·
`你评论过该课程了，不如去评下分呢` · `评论需满足%1$d-%2$d字，当前%3$d字` · `发布一条课程评价吧～` ·
`发布成功` · `已成功发布课程评分与评论` · `评论字数未符合要求` (toast).
`Divergence:` Android's stars are 56 dp, its counter and validation both vanish when the config is
missing, and its want/like coroutine runs on an unscoped `MainScope()`.

### 2.7 Course center (我的数据)

```
┌──────────────────────────────────────────────┐
│< 我的数据                                     │
│┌────────────────────────────────────────────┐│ ① brief card · pad 16
││ (av64) Nickname                            ││ title2 22 / Bold
││        description (3 lines)               ││ 12 / secondary
│└────────────────────────────────────────────┘│
│┌────────────────────────────────────────────┐│ ② rank card
││ 成绩排行                              查看全部›││
││ 高等数学 张三                    你的分数 98 ││
││ 0-59 ▏60-69 ▏70-100               前 3.2%   ││ bar column 210
││                                     TOP1    ││
││ ────────────────────────────────────────── ││
││                   查看全部 >                 ││
│└────────────────────────────────────────────┘│
│┌────────────────────────────────────────────┐│ ③ want card
││ 想上                                        ││
││ ♥ 高等数学 张三              ›  2026-01-05   ││ heart 28 · #FF68B5
│└────────────────────────────────────────────┘│
│┌────────────────────────────────────────────┐│ ④ comment card
││ 我的评价                                    ││
││ ▌高等数学 ›                        ★★★★★    ││ bar 6×12 r3
││ body text                                   ││
││ 2026-01-05 12:30                            ││
│└────────────────────────────────────────────┘│
└──────────────────────────────────────────────┘
```

**Blocks** (`CSIOS/coursecenter/main/CourseCenterView.swift:28-54`,
`CSAOS/ui/coursecenter/CourseCenterView.kt:98-132`) — ① **brief card** (`hasBriefInfo` — `:31`;
`:110-114`; not tappable) · ② **rank card** (`hasScoreRankInfo` — `:35`; `:116-120`) ·
③ **want card** (`hasWantInfo` — `:39`; `:122-126`) · ④ **comment card** (`hasCourseCommentInfo`
— `:43`; `:128-132`) · **loading** (centred spinner — `:48-50`; `:59-63`) · **error** (title +
message + `重试` — `:51-54`; `:70-90`).

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Nav title / background / container | `我的数据` `:18`, `LS:877` / `ham_bg_b1Color` `:17` / `ScrollView` + `.padding()` `:28,47` | `course_score_my_data` `:55` / inherited / `LazyColumn` gap 16, h-pad 16, bottom 16, top `statusBar + 64` `:98-109` | `我的数据` / `surface.primary` / list gap 16, h-pad 16 |
| Brief: avatar / fallback | `WebImage` 64×64 circle, fade 0.5 `CourseCenterUserInfoView.swift:19-34` / gray@0.2 + `person.fill` 32 `:22-29` | `SubcomposeAsyncImage` 64 `CourseCenterBriefCardView.kt:65-91` / `ham_gray`@0.2 + `Person` 32 `:73-87` | 64×64 circle, cross-fade / `surface.tertiary` + person `icon.lg` 32 |
| Brief: row gap / name / desc | `16` `:18` / `.title2` 22 Bold `:37-39` / `.caption` 12 gray 3 lines `:42-46` | `16.dp` `:62` / `title2` `:93-97` / `caption` secondary 3 lines `:99-106` | 16 / `title2` 22 / Bold / 12 / Regular `text.secondary`, 3 lines |
| Brief: tap / background | none `:17-52` / none | `UserCenterPath.MAIN` `:59` / server `background_color` else `ham_blue`@0.1 `:53-56` | **not tappable** / `surface.secondary` |
| Card title / body gap | server `title` 17 / Bold `CourseCenterScoreRankView.swift:17` / `16` `:18` | server title `CourseCenterRankCardView.kt:55` / `16.dp` `:56` | server title / **16** |
| Empty state | none | `没有已上传的成绩记录` `…RankCardView.kt:58-62`; `没有想上课程记录` `…WantCardView.kt:47-49`; `没有课程评论历史记录` `…ReviewCardView.kt:64-66` | **explicit empty text on all three**: 12 / Regular `text.tertiary` |
| `查看全部` | gated on `showMore`; 12; blue on rank, inherited elsewhere `:26-34` | `course_score_view_all` + `ChevronRight`, `ham_blue` `:71-89` | 12 / Regular **`accent`** + `icon.xs` chevron; only when `show_more` |
| Rank: name / instructor | 17 Bold / 12 `:36-40` | `bodyBold` / `caption`, 1 line, Ellipsis `:112-123` | 17 / Bold 1 line / 12 / Regular 1 line |
| Rank: bar row height / band gap / label | `16.0` per band `:49` / `8.0` `:50` / `"\(from)-\(to)"` 11 tabular `:55-57` | `16.dp × progress_item.size` `:143` / `4.dp` `:147-148` / `"${from}-${to}"` `caption2` `:154` | **16 per band** / **4** / localized `%1$d-%2$d`, 12 / Regular tabular |
| Rank: column / radius / colour / track | width `210` `:93` / `RoundedCorner(itemHeight/2)` = 8 trailing `:87` / server else `.blue` `:67` / `color`@0.15 `:86` | `210.dp` `:169` / `topEnd/bottomEnd 6` `:188` / server else `ham_blue` `:185` / `color`@0.15 `:201-237` | 210 / **`radius.4` 8 trailing** / server colour else `accent` / fill @0.15 |
| Rank: fill rules | above band `1.0`; within `selfProgress`; below `0.0` `:70-74` | above solid; within `selfProgress/progress` + remainder @0.15; below solid @0.15 `:193-237` | above → solid; within → `selfProgress/progress` solid + remainder @0.15; **below → track only** |
| Rank: clamp | `max(progress/maxProgress, 0.01)` `:82` | `coerceIn(0.01f, 1f)` `:190` | `[0.01, 1]` |
| Rank: `你的分数` / score / `TOP1` / percentile | 11 `:104-105` / `"\(data.score)"` `title` rounded 28 `:106-107` / hardcoded 11 Bold at `progress >= 1.0` `:109` / `前` + `"%.1f%%"` `:111-114` | `course_score_your_score` `:257` / `toString()` `title` 24 `:258-262` / hardcoded at `>= 1f` `:264` / `前%1$.1f%%` `:265`, `STR:74` | `caption2` 11 / `title` **28 / rounded Bold** / **localize**, 11 / Bold at `progress >= 1` / `前%1$.1f%%` 12 / Regular |
| Want: heart / name / instructor | `heart.fill`, inline RGB (1, 0.41, 0.71) `CourseCenterSelfWantItemView.swift:18-19` / `body` bold / `caption`, 3 lines `:22-33` | `Favorite` 28.dp, same RGB `…WantCardView.kt:91-96` / `bodyBold` / `caption`, 3 lines `:106-119` | **promote to a token**; `icon.lg` 28 / 17 / Bold 3 lines / 12 / Regular 3 lines |
| Want: timestamp / chevron | `ham_toyyyymmddhhmm` 12 `t2Color` `:37-39` / `chevron.right` `t2Color` `:42-43` | **absent** `:85-129` / `ChevronRight` secondary `:121-125` | **timestamp present** 12 / Regular `text.secondary` / `icon.xs` 12 `text.secondary` |
| Comment: identifier bar | r3, 6×12, course colour else `.blue` `CourseCenterReviewCardItemView.swift:22-24` | r3, 6×12, `identifier_color` else `ham_blue` `…ReviewCardView.kt:117-127` | 6×12, **r 3**, course colour else `accent` |
| Comment: name / chevron / row gap | `.headline` 17, 1 line `:25-30` / 8×16, lead pad 2 `:31-35` / `3` `:21` | `headline` 14.sp `:128-135` / h16, offset −6 `:136-143` / `3.dp` `:114` | **17 / Bold** 1 line / `icon.xs` 12, gap 4 / **4** |
| Comment: stars / column / body / time | `star.fill` 16×16 `.orange`, gap 4 `:45-49` / width 80 trailing, lead pad 12 `:51-52` / `body` / 12 `t2Color`, gap 4 `:56-62` | `Star` 16.dp `ham_orange` `:156-163` / width 80, start 12, `Arrangement.End` `:150-155` / `body` + top pad 8 / 12 `:169-179` | `icon.sm` 16 **`brand.score`**, gap 4 / width 80 trailing, gap 12 / body 17 / Regular / 12 / Regular `text.secondary`, gap 4 |
| Error message | — | raw gRPC `e.message` `CourseCenterViewModel.kt:50` | a localized generic message, not gRPC text |

**Strings:** `我的数据` (`LS:877`/`STR:4`) · `成绩排行` (`LS:646`/`STR:80`) · `想上` · `我的评价` ·
`查看全部` (`LS:724`/`STR:56`) · `你的分数` (`STR:73`) · `前%1$.1f%%` (`STR:74`) ·
`没有已上传的成绩记录` (`STR:72`) · `没有想上课程记录` (`STR:76`) · `没有课程评论历史记录`
(`STR:75`) · `请求失败` (`STR:70`) · `重试` (`STR:71`).

### 2.8 Course-center sub-pages (成绩排行 / 想上历史 / 评论历史)

```
┌──────────────────────────────────────────────┐
│< 成绩排行                                     │ flat title — no count
│┌────────────────────────────────────────────┐│
││ 高等数学 张三                    你的分数 98 ││ one card per item · r16 · pad 16
││ 0-59 ▏60-69 ▏70-100               前 3.2%   ││
│└────────────────────────────────────────────┘│
│┌────────────────────────────────────────────┐│ gap 16
││ 线性代数 李四                    你的分数 87 ││
│└────────────────────────────────────────────┘│
│                    ⟳                          │ trailing spinner + divider
└──────────────────────────────────────────────┘
```

**Blocks** — 1. **nav title** (always) · 2. **item list** (one card per item, using the row layouts
from §2.7) · 3. **pagination trigger** (second-to-last item — `i == itemList.count - 2`
`CSIOS/coursecenter/rank/CourseCenterScoreRankPageView.swift:44`;
`it === list.getOrNull(size - 2)` `CSAOS/rank/CourseCenterRankView.kt:98`) · 4. **first-page
spinner** (loading with an empty list) · 5. **trailing spinner + divider** (while paginating).

| Element | iOS | Android | Normative |
| --- | --- | --- | --- |
| Container / background | `LazyVStack(spacing: 16)` + `.padding()` `:21,26` / `.ham_bg_b2Color` `:21` | `LazyColumn` gap 16, `bouncy`, `userScrollEnabled = false`, top `statusBar + 16 + headerHeight` `:87,94` / inherited | list gap 16, h-pad 16 / `surface.primary` |
| Titles | `成绩排行` / `想上历史` / `评论历史` `:46` each | `成绩排行(%1$d)` / `想上历史(%1$d)` when a total exists `…RankView.kt:45-47`, `…WantView.kt:39-41`; `评论历史` has no count variant `…CommentHistoryView.kt:51` | **flat titles**: `成绩排行`, `想上历史`, `评论历史` |
| Item wrapping | bare rows | rank `:102` and want `:96` wrap in `HamCardView`; comment history `:103` does **not** | **every item wrapped in a card**: r16, pad 16 |
| Gap / pagination | `16` / `count - 2` | `16.dp` / `size - 2` | 16 / second-to-last item |

**Strings:** `成绩排行` (`LS:646`/`STR:80`) · `想上历史` (`STR:77`) · `评论历史` (`STR:78`).

---


| # | Defect | Cite |
| --- | --- | --- |
| 1 | CAS card white-on-white — `.white` applied to the outer `HStack`, which contains the white@0.7 pill | `IOS/card/cas/StatusCasAlertCard.swift:16,22` |
| 2 | Weather sub-line drops the city name — `??` binds looser than `+` | `IOS/card/weather/StatusWeatherCardViewModel.swift:138` |
| 3 | Schedule summary always reads `待完成0项日程` — wrong list in the format argument | `IOS/card/schedule/StatusScheduleCard.swift:125` |
| 4 | Sport chip has no explicit fill → white-on-white in dark mode | `IOS/card/sport/StatusSportCardView.swift:26-28` |
| 5 | Card ordering unstable on ties — no secondary sort key on either platform | `IOS/StatusContentViewModel.swift:155-158`; `AOS/utils/StatusViewCardScoreManager.kt:88-91` |
| 6 | Android course card never publishes a score, so it sinks | `AOS/component/course/CourseCardViewModel.kt:76` |
| 7 | Android weather card publishes nothing on failure, so a failing card sinks | `AOS/component/weather/WeatherCardViewModel.kt:84` |
| 8 | Android computes `weekProgress` and never renders it | `AOS/component/course/CourseCardViewModel.kt:135` |
| 9 | Course view-toggle labels inverted between platforms | `IOS/card/course/StatusCourseCard.swift:81`; `AOS/component/course/CourseCard.kt:378-380` |
| 10 | iOS bus card's 60 s refresh loop defined but never started | `IOS/card/bus/StatusBusCardViewModel.swift:53-64` |
| 11 | Library countdown cadence and rounding differ (1 s / ceil vs 10 s / floor; 1 h vs 2 h) | `StatusLibraryCardReserveInfoView.swift:57-78`; `AOS/component/library/LibraryCard.kt:207-250` |
| 12 | Detail rate card shows `4.0` on iOS, `4.00` on Android; star count and watermark size also differ | `CSIOS/coursedetail/component/CourseScoreCourseDetailRateCard.swift:58`; `CSAOS/ui/detail/component/CourseScoreCourseDetailViewRateCard.kt:112` |
| 13 | Android avatar branch inverted — a null URL takes the network branch | `CSAOS/ui/detail/component/CourseCommentItemView.kt:49` |
| 14 | Android match flow returns before assigning the comment config holder; its error view cannot show a server message | `CSAOS/ui/detail/CourseScoreCourseDetailMatchViewModel.kt:65-67`; `…/CourseScoreCourseDetailErrorView.kt` |
| 15 | Android create-review hides the counter **and** skips validation when config is null | `CSAOS/ui/comment/create/CourseCommentCreateView.kt:174`; `…CreateViewModel.kt:77-79` |
| 16 | Android status page sets no background, falling through to `0xFFFFFBFE` | `AOS/StatusContainerView.kt:72` |
| 17 | Dead code: `ScheduleCard.kt` (empty stub, zero call sites), `WeatherCard.kt` (second weather card), `StatusScheduleCard.swift:213-242`, `StatusLibraryCardModifyBookingTimeButton.swift`, `CourseCommentMainView.kt` (mock-only) | `AOS/component/schedule/ScheduleCard.kt:21`; `AOS/component/weather/WeatherCard.kt:69` |
| 18 | Search-hit parsing: iOS force-unwraps (`range(of:)!`), Android swallows exceptions silently | `CSIOS/search/cell/CourseScoreSearchViewSearchResultItem.swift:43,48`; `CSAOS/ui/search/CourseScoreSearchHitUtils.kt:33` |


## 8. My tab (我的 My)

Normative: "build it this way". Values measured from source; **iOS is the baseline** where clients
disagree. Token names follow `docs/design-system.md` §2. `pt` and `dp` are 1:1.

## Global context

**Inventory** (§0.1) — route → view. iOS `Route.swift:12,108-252`; Android `MyViewRoute.kt:8-18`, `UserCenterRoute.kt:8-16`, `CasRoute.kt:10-14`, `QrCodeViewRoute.kt:10-13`.

| # | Screen | iOS | Android |
|---|---|---|---|
| 1 | My tab main | tab 3 — `my/MyView.swift` | `my-view/home` — `MyMainView.kt` |
| 2 | Settings hub · About | hub **none**; `about` (`Route.swift:100`) — `AboutView.swift` | `my-view/setting` — `SettingView.kt`; `my-view/about` — `AboutView.kt` |
| 3 | Widget · Language · Automatic | widget/language **none**; `automatic` (`:99`) — `MySiriView.swift` | `my-view/widget`, `/setting/language-setting`, `/automatic` |
| 4 | User center main | `userCenter` (`:87`) | `user-center/main` — `UserCenterMainView.kt` |
| 5 | Info · devices · social · passkey | `userCenterInfo` (`:88`), `userCenterLoginDevice` (`:89`), `userCenterSocialAccount` (`:90`), `userCenterPasskeyConfig` (`:91`) | `user-center/edit-info`, `/device`, `/social-account`, `/passkey` |
| 6 | Logout / deactivate | `userCenterLogout` (`:92`) — `SyncLogoutView.swift` | **none** (`SyncLogoutView.kt` unreachable) |
| 7 | Authorized apps · Scan · QR login · SSO · login · CAS · Debug · Bus | `userCenterAuthorizedApps` (`:93`), `scanCode` (`:96`), `qrCodeLogin` (`:97`), `casSetting` (`:71`), `debug` (`:103`), `bus` (`:72`) | `user-center/authorized-apps`, `qrcode/scan`, `qrcode-login`, `login`, `CasRoutes.Setting`, `my-view/debug`, `my-view/bus` |

**Tokens** (§0.2): `surface.primary` `ham_bg_b1` #F9F9F9 / #000000 (`Color+Ham.swift:22`, `Color.kt:31-33`) · `surface.secondary` `ham_bg_b2` #FFFFFF / #0F1010 (`Color+Ham.swift:23`, `Card.kt:42`) · divider `ham_lightGray` #EDEEEF / #0F0E0F (`Divider.kt:17-24`) · `text.primary` system label / #000000 / #FFFFFF (`Color+Ham.swift:16`, `Color.kt:24-26`) · `text.secondary` #8E8E93 / **#FF888888 hardcoded** (`Color+Ham.swift:17`, `Color.kt:28-29`) · `accent` `ham_blue` #007AFF / #0A84FF (`Color.kt:21-22`) · 8 `brand.*` (`Color+Ham.swift:35-42`, `Color.kt:75-94`).
**Type:** Android title2 20 · title3 16 · body 16 · headline 14 · caption 12 · caption2 11 (`Font.kt:45-52`); **iOS has no type-token layer** — raw SwiftUI: body 17, callout 16, caption 12.

**Gaps** (§0.3): settings hub, widget and language are Android-only (`MyViewRoute.kt:15,11,16`; widget and language are correct per `design-system.md:882-883`) · logout is iOS-only (`Route.swift:92`; Android's `SyncLogoutView.kt` has zero call sites) · iOS `debug` is `#if DEBUG` (`Route.swift:102-104,243-249`), Android unconditional · iOS `automatic` is a 31-line `MySiriView` (`Route.swift:219-220`), Android's is 132 lines.

**Localisation** (§0.4): Android locale coverage complete for `my` (82/79/79 — 3 `translatable="false"` language names at `values/string.xml:88-90`), `user-center`, `auth`, `automatic`, `cas`, `qrcode`. Hardcoded sites **iOS 38 / Android 1**: iOS's 34 `LocalizedStringKey` literals do localise but duplicate source copy; **4 genuinely do not** — `MyUserCenterCard.swift:48`, `QrCodeLoginView.swift:28,30,54`. Android: `MyViewLinkCard.kt:159` 调试入口 (debug-only). Defect: `feature/my`'s default locale file is `string.xml` (singular) vs `strings.xml` for en/ja.

**Defects** (§0.5):

| # | Sev | Defect | Evidence |
|---|---|---|---|
| 1 | High | Android logout is a one-tap unconfirmed destructive action | `UserCenterMainView.kt:171-187`, VM `:21-24` |
| 2 | High | Android has no deactivation UI; 注销该账号/确定/成功/已注销账号 defined but unread | `values/strings.xml:25-29` |
| 3 | High | Android tiles render **blank titles** on cold install — `@SerialName("title-content")` vs bundled `"title"` | `CCMyViewButtonModel.kt:21-22`, `CCKVDefaultValue.kt:11-72` |
| 4 | High | `HamTheme` not applied on the Android user-center path — dialogs render stock M3 (r28, elev 6) | `MainActivity:41-43`, `MyMainView.kt:37` |
| 5 | Med | Android authorized-apps / device render a blank page on first-load error (`else -> {}`) | `AuthorizedAppsView.kt:84-112,111` |
| 6 | Med | Promotion card cannot be dismissed on either platform | `MyViewPromotionCard.swift:14-53`, `MyPromotionView.kt` |
| 7 | Med | Two reds on Android destructive actions: #FF0000 vs `ham_red` #F44336; iOS #FF3B30 unused; iOS `automatic` is not Android's automation screen; Android's device screen has no empty state, no kick confirmation, unstyled 本机 marker | `SyncLogoutView.kt:49`, `UserCenterMainView.kt:183`, `colors.xml:72`, §0.5/8-9 |
| 8 | Low | `feature/my` default-locale filename singular · Android uses `android.R.string.cancel` (`AuthorizedAppsView.kt:144`) · authorized-apps has no nav-bar spacer | §0.4, §0.5/11-12 |


### My hub (我的) — platforms: both

**Purpose:** the account and navigation hub — collapsing remote-branded header, identity/CAS card, module grid, board, promotion list, settings/link list.

```text
┌ status bar (system) ─────────────────────────────────────────────────┐
│ [L0] remote header image h 250, fill/crop; +max(0,-y) on pull-down   │
│ [L1] ScrollView · column iPad maxWidth 400, h-pad 16 · top inset     │
│      statusBarHeight+100 (iOS) | 80 spacer + 32 pad (Android)        │
│   ①user-center ↕15/10 ②grid ③board ④promotion ⑤settings/link        │
│   bottom 100 (iOS) | navBars + 80 · [L2] collapsed bar h = SBH + 56  │
└ tab bar (system inset) ───────────────────────────────────────────────┘
```

**Blocks:** `header image` remote `myViewConfig["imageUrl"]` h 250 (`MyViewNavContainer.swift:135-153`, `MyMainViewContainer.kt:71-88`) · `content column` 5 cards (`MyView.swift:16-40`, `MyMainView.kt:39-58`) · `collapsed bar overlay` (`MyViewNavContainer.swift:99-133`, `MyMainViewContainer.kt:100-134`) · `bottom blur scrim` iOS <26 only, `VisualEffectBlurView(.systemThinMaterial)` h 85 (`MyViewNavContainer.swift:90-96`) · `bottom spacer` nav inset + 80 (`:59`, `Spacer.kt:19-31`).

| Element | iOS | Android | normative |
|---|---|---|---|
| Screen background · card | `ham_bg_b1Color` · r16 pad16 `ham_bg_b2Color` | `ham_bg_b1` · r16 pad16 `ham_bg_b2` | `surface.primary` · **Card §3.1** (`MyViewNavContainer.swift:75`, `MyMainViewContainer.kt:66-70`, `CardView.swift:22-23,49`, `Card.kt:48,53`) |
| Card gap · h-padding | 15 · 16 | 10 · 16 | **15 · `space.5` 16** (`MyView.swift:22,25`, `MyMainView.kt:40,52`) |
| Content top inset | statusBarHeight + 100 | 80 + 32 + statusBars | **statusBarHeight + 100** (`:58`, `MyMainView.kt:44-46`) |
| Content bottom · iPad column · overscroll | 100 · 400 centred (`:57`) · — | navBars + 80 · none · factor 2.5 | **nav inset + 80** (§1.3 tab-root) (`:59`, `Spacer.kt:19-31`) · **400 centred** · **2.5** (`BounceScrollView.kt:25-28`) |

**Strings:** 我的 tab label — `Localizable.strings:10` `MY`, `values/strings.xml:5` `tab_my`. All block copy is remote CCKV (per block below).

**States:** `loading` — **none**, no skeleton or spinner; remote images fade in 0.5 s (`MyUserCenterCard.swift:77`, `MyViewNavHeader.swift:33`) · `error` — **none** screen-level; image failures fall back to a glyph (`MyUserCenterCard.swift:69-73`, `MyViewUserCenterCard.kt:91-93`), a malformed config yields an empty card · `empty` — only the settings/link card renders; grid height 0 (`MyViewFunctionCard.swift:56`, `MyViewFunctionComponentView.kt:199`) · `debug build` — extra 调试入口 / Debug row (`MyViewSettingCard.swift:113-124`, `MyViewLinkCard.kt:153-171`).

**Divergence:** block gap 15 vs 10 — adopt 15 and amend §1.3 (which says 8). iOS ignores the safe area (`:81`) and adds a bottom blur scrim; Android re-adds `statusBarsPadding()` and double-counts the nav-bar inset (`MyMainView.kt:46` + `Spacer.kt:20`).

### Function grid / module tiles (功能网格) — platforms: both

**Purpose:** the app's real navigation hub — remote-configured shortcuts into every campus module.

```text
┌ h-pad 16 ───────────────────────────────────────────────┐
│ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐    row 1 (scrolls →)   │
│ │图书馆│ │运动 │ │成绩 │ │给分 │                        │
│ │E卡  │ │校巴 │ │课程表│ │日程 │    row 2               │
│ └─────┘ └─────┘ └─────┘ └─────┘                        │
└ h-pad 16 ───────────────────────────────────────────────┘
  grid h = rows × 64 · tile h 52 · r 12 · brand @ 0.15 over surface.primary
```

All 8 module entries, in code order (`MyViewFunctionCard.swift:27-34`; Android's sorted emission order is identical, `MyViewFunctionComponentView.kt:62-70`); telemetry keys are `<key>_btn` on both (`MyViewFunctionComponentView.kt:87-189`).

| # | Config key | Label (bundled zh) | Brand colour | iOS destination | Android destination |
|---|---|---|---|---|---|
| 1 | `library` | 图书馆 | `brand.library` **#007AFF** | `Route.library` → `LibraryView()` (`Route.swift:173`) | `library/home` (`LibraryRoute.kt:21`) |
| 2 | `sport` | 运动 | `brand.sport` **#34C759** | `Route.sport` → `SportView()` (`:125`) | `sport/main` (`SportRoute.kt:10`) |
| 3 | `score` | 成绩 | `brand.score` **#FF9500** | `Route.score` → `ScoreView()` (`:153`) | `ScoreRoutes.Home` (`ScoreRoute.kt:13`) |
| 4 | `course_score` | 给分 | `brand.coursescore` **#283593** | `Route.courseScore` (`:111`) | `CourseScoreRoutes.Home` (`CourseScoreRoute.kt:16`) |
| 5 | `pay` | E卡 | `brand.pay` **#BF360C** | **no route** — posts `Notification.ham_showPay` (`MyViewFunctionCard.swift:88-90`) | **no navigation** — `FloatViewManager.showFloatView { PaySheetView }` (`:136-153`) |
| 6 | `bus` | 校巴 | `brand.bus` **#A2845E** | `Route.bus` (`:217`) | `my-view/bus` (`MyViewRoute.kt:14`) |
| 7 | `course` | 课程表 | `brand.course` **#1B5E20** | `Route.courseSetting` (`:163`) | `course/setting` (`CourseRoute.kt:14`) |
| 8 | `schedule` | 日程 | `brand.schedule` **#01579B** | `Route.schedule` (`:223`) | `schedule/home` (`ScheduleRoute.kt:11`) |

| Element | iOS | Android | normative |
|---|---|---|---|
| Container · height · padding · spacing | `ScrollView(.horizontal)` of per-row `HStack`s (`:48-62`) · implicit · 16 gutter each side (`:53,60`) · HStack default | `LazyHorizontalStaggeredGrid` `rows=Fixed(row)` (`:201-205`) · `row × 64` = 128 (`:199`) · 16 h / 8 v (`:206`) · 8 h / 8 v (`:207-208`) | staggered h-grid, rows = max `row`, **height rows × 64** · **16 h / 8 v** · **`space.3` 8** |
| Tile height · radius · padding | 52 (`:120`) · 12 (`:124`) · h 16 (`:119`) | 48 · 10 · h 12 v 0 (`MyViewFunctionButtonView.kt:45-48`) | **52 · `radius.6` 12 · h 16** |
| Icon · gap · title · subtitle | SF Symbol unsized → 17 (`:109`) · default · 17 Bold, no line limit (`:111`) · 12 if non-empty (`:113-116`) | Material icon unsized → 24 (`:52`) · 8 (`:50`) · 16 Bold (`:54`) · 12 if not blank (`:55-57`) | **`icon.sm` 20 · `space.3` 8 · 17 / Bold 1 line · `caption` 12 1 line** |
| Fill · foreground | brand @ 0.15 over `ham_bg_b1Color`; brand text (`:121-123`) | `color.copy(alpha=0.15f)`; brand text (`:47,54`) | **`tint.brand`** over `surface.primary`; tinted text matches its tint (§2.2 rule 2) |

**Strings / states:** remote only — `myViewButtonConfig.buttonConfig.<key>["title-content"|"subtitle-content"][locale]` (`MyViewFunctionCardVM.swift:36-37`, `MyViewFunctionComponentView.kt:83-84`). **No bundled iOS labels**; Android's 8 bundled labels above never reach the UI (defect 3). `tile hidden` (config absent or `visible == false`; Android force-shows when `App.DEBUG`, `:222`) → invisible expanding spacer (`MyViewFunctionCard.swift:128`) · `empty` → grid height 0, Android requests `StaggeredGridCells.Fixed(0)` (`:199,203`) · `unknown type` → silently dropped (`:56`).

**Divergence:** iOS emits each `row` group as a side-by-side `HStack` inside a horizontal scroller, so multi-row configs lay out horizontally (quirk, `MyViewFunctionCard.swift:48-62`); specify the staggered grid. Android's sport/score resolve to legacy Material #4CAF50 / #FF9800 (`Color.kt:78-82`). Android's `pay` opens a floating sheet, iOS posts a notification. Label drift: §1.2 says 课程评分 / 校车, Android ships 给分 / 校巴.

### Collapsing header (折叠导航栏) — platforms: both

**Purpose:** a full-bleed remote header image that reveals a condensed title bar once content scrolls.

| Element | iOS | Android | normative |
|---|---|---|---|
| **Collapse threshold** | offset **> 7** (`-value > 56/8`, `MyViewNavContainer.swift:64,68`) | **> 50 dp** (`y > 50`, `MyMainViewContainer.kt:59`) | **> 7** |
| Image height · offset · mode | `250 + max(0,-y)` (`:151`) · `-max(y,0)` (`:152`) · `.scaledToFill()` (`:141,148`) | `(250 - y).dp` (`:80`) · `-y.dp` (`:79`) · `ContentScale.Crop` (`:86`) | **250 + max(0,-y)** · **1:1 with scroll** · fill / crop |
| Bar height · fill · radius/elevation/divider | statusBarHeight + 56 (`:129`) · `ham_bg_b2Color` masked black→clear top→bottom (`:101-111`) · none | statusBarHeight + 56 (`:64`) · `ham_bg_b1 @ 0.85` (`:107`) · none | **statusBarHeight + 56** · `surface.secondary`, opaque top → transparent bottom · **none** (§2.7) |
| Collapsed title | remote title 17 Bold #555555, else 30⌀ avatar + nickname Bold (`MyViewNavHeader.swift:18-47`) | remote title only, 20 Bold, `text.primary` (`MyMainViewContainer.kt:120-131`) | remote title if set, else 30⌀ avatar + nickname |
| Title position · transition · dead state | leading, h-pad 16, centred (`:116-127`) · `.easeInOut(0.3)` (`:113,120`) · `hasPull` (offset < -10) computed, never read (`:20,65,69`) | `CenterStart`, top pad statusBarHeight + 12 (`:123-128`) · `fade + slide{it/2}`, no duration (`:103,116-117`) · `animatedAlpha` computed, never read (`:60-63`) | **leading, h-pad 16, centred · 0.3 s easeInOut** · remove the dead state |

**Strings:** title — `myViewConfig["title-content"][locale]` (`MyViewNavHeader.swift:18-19`, `MyMainViewContainer.kt:120-121`); Android has **no bundled default**, so it renders `""` (`CCKVContext.kt:95`). Fallback 未登录 (`MyViewNavHeader.swift:40`, `Localizable.strings:714`).

**States:** `expanded` (≤ threshold) — image only, no bar, no scrim (`:77-79`, `MyMainViewContainer.kt:71-98`) · `collapsed` (> threshold) — bar and title fade/slide in; on iOS 26+ the header moves into `.toolbar(.title)` and the in-screen overlay is suppressed (`MyView.swift:82-87`, `MainTabView.swift:42-56`) · `remote title set` — avatar and nickname suppressed (`MyViewNavHeader.swift:18-22`).

**Divergence:** threshold 7 vs 50 — adopt 7; iOS 26+ uses a different placement mechanism, Android a flat 85 % fill; neither has an over-pull UI.

### User-center card (用户中心卡片) — platforms: both

**Purpose:** show Ham-account identity (or the sign-in prompt) and the CAS 信息门户 binding state.

```text
┌ card r16 pad16 surface.secondary ──────────────────────────┐
│ (48⌀ avatar) 点击登录 / {nickname}   17/Bold primary       │
│              不登录也可以使用校内功能哦  12 secondary        │
│ ── divider 1px surface.tertiary, v-pad 4 ──────────────────│
│ (40⌀ plate) 登录信息门户 / 管理信息门户设置  17/Bold    ▸  │
│             使用校内服务的前提         12 secondary     ▸  │
└────────────────────────────────────────────────────────────┘
```

| Element | iOS | Android | normative |
|---|---|---|---|
| Card · outer h-padding | `HamCardView(16, r16)` (`:15`) · 16 (`MyView.swift:25`) | default 16 / r16 (`:53-57`) · 16 + top 16 (`:59-61`) | **Card §3.1** · **16** |
| Avatar · gap · placeholder · loading | 48 × 48 circle (`:81`) · default · `face.smiling` 28, gray, bg gray @ 0.15 (`:69-73`) · none | 48.dp circle (`:80-81`) · 12 (`:73`) · `SentimentVerySatisfied` 48, `ham_gray@0.15`, pad 8 (`:175-186`) · `HamLoadingProgressBar` 20, stroke 2 (`:86-90`) | **48⌀ · `space.4` 12 · `icon.md` 24 on gray @ 0.10 · 20 / 2 stroke** |
| Name · prompt · sub-line | 17 Bold `text.primary`, 2 lines (`:96-99`) · 17 Bold gray (`:86-88`) · 12 gray (`:91-92`) | 16 Bold (`:113-117`) · 16 Bold secondary (`:101-105`) · 12 secondary (`:106-110`) | **17 / Bold, 2 lines · 17 / Bold `text.secondary` · `caption` 12 `text.secondary`** |
| CAS plate · glyph | ⌀32 (16 glyph + 8 pad), `blue@0.15` (`:42-45`) | 48⌀ (36 glyph + 6 pad), `ham_blue@0.15` (`:131-140`) | **40 × 40 @ `accent` 0.10 · `icon.sm` 20** (§3.3) |
| CAS title · subtitle · colour · chevron | 16 Bold (`callout`) (`:49-50`) · 12 (`:52`) · `text.primary` if bound else `.blue` (`:54`) · `chevron.right` 12 (`:56-57`) | 14 Bold (`:151`) · 11 (`caption2`) (`:156`) · `ham_text_primary` if bound else `ham_blue` (`:152,157`) · 24, unsized (`:162-166`) | **17 / Bold** (§2.4) · **`caption` 12** · **`text.primary` if bound, `accent` if not** · **`icon.xs` 12** |

**Strings:** 点击登录 (`Localizable.strings:45` `TAP_TO_LOGIN`; `string.xml:4` `user_center_login_prompt`) · 不登录也可以使用校内功能哦 (**hardcoded** `MyUserCenterCard.swift:90`) · 登录信息门户 / 管理信息门户设置 (**hardcoded ternary — does not localise** `MyUserCenterCard.swift:48`; `string.xml:6-7`) · 使用校内服务的前提 (**hardcoded** `MyUserCenterCard.swift:51`; `string.xml:8`) · nickname and avatar URL are data (`login.pb.swift:182,184`) — no student-ID line exists.

**States:** `logged out` (token empty, `AccountContext.swift:90-93`) — placeholder avatar, 点击登录 + hint; tap posts `ham_loginShow` → app-level `LoginView` (`:23`, `ContentView.swift:38`) / `NotificationChannel.ShowLoginDialog` (`MyViewUserCenterCard.kt:64-69`) · `logged in` — remote avatar + nickname, row navigates to `Route.userCenter` (`:18`, `Route.swift:199`) / `user-center/main` (`:68`) · `CAS unbound` — 登录信息门户 in `accent`; `CAS bound` — 管理信息门户设置 in `text.primary` (`:48,54,58`) · `error` — avatar failure falls back to the placeholder glyph.

**Divergence:** iOS gates the card on CCKV `ham_sync_useUserCenter`, **default false** (`CCKVContext.swift:34-35`); Android always shows it (`MyMainView.kt:49`) — show it unconditionally. Android's card emits no telemetry (`:64,124`).

### Board card (公告卡片) — platforms: both

**Purpose:** render a remote-configured announcement as a markdown card.

| Element | iOS | Android | normative |
|---|---|---|---|
| Guard · show/hide · card · title · gap · body | CCKV `ham_ui_showBoard`, default **false** (`MyView.swift:31`, `CCKVContext.swift:28-29`) · none · `HamCardView(padding 10, r16)` + inner pad 8 (`:13,21`) · 17 **Semibold** `text.primary`, spacing 0 (`:15-17`) · 0 · `Text(.init(...))` AttributedString markdown (`:19`) | same (`MyMainView.kt:34`, `CCKVContext.kt:42-43`) · `AnimatedVisibility` (`:51`) · default 16 / r16 (`MyViewBoardCard.kt:28`) · `bodyBold` 16 (`:30`) · 8 (`Card.kt:85`) · `MarkdownText` 0.3.1, `text.primary` (`:32-35`) | **flag-gated, default false · `AnimatedVisibility` · Card §3.1 — pad 16, r 16 · 17 / Bold** (§2.4), inner spacing 0 · **`space.3` 8** · markdown, `body` 17 / `text.primary` |

**Strings / states:** remote — `boardData[locale]["title"|"content"]` (`MyViewBoardCard.swift:16,19`; `MyViewBoardCard.kt:30,33`). `hidden` (flag false) · `empty data` — blank title and blank body · no error state.

### Promotion card (推广卡片) — platforms: both

**Purpose:** a remote-driven list of promotional entries, each opening an external URL.

| Element | iOS | Android | normative |
|---|---|---|---|
| Guard · card · row spacing | `promotion["itemList"]` non-empty (`:20`) · `HamCardView(10, r16)` + inner pad 8 (`:21,49`) · 15 (`:22`) | non-empty, else `return` (`MyPromotionView.kt:34-36`) · `PaddingValues(8.dp)` (`:37`) · row pad 8, container pad 8 (`:204-209`) | **non-empty itemList · Card §3.1 — pad 16, r 16 · `space.3` 8** |
| Icon | remote image, **no fit modifier (stretches)**, clip circle, outer 40⌀ blue@0.1 (`:32-34`, `MyViewSettingCard.swift:221-225`) | `SubcomposeAsyncImage` 30⌀ `Crop`, clip circle (`:74-84`) | **40 × 40 plate @ `accent` 0.10, `icon.sm` 20** (§3.3) |
| Title · subtitle | 17 **Semibold** `text.primary` (`MyViewSettingCard.swift:230-233`) · 12 gray (`:234-237`) | 16 Bold (`:223-228`) · 12 secondary (`:229-233`) | **17 / Bold `text.primary` · `caption` 12 `text.secondary`** |
| Chevron · divider | `chevron.right` gray, **17 pt unsized** (`:241-242`) · `Divider()` compared by **value**, duplicates suppress it (`:44-46`) | 24 `Color.Gray` (`MyViewLinkCard.kt:236`) · `HamDivider` v-pad 4, omitted after the last (`:40-45`) | **`icon.xs` 12 `text.secondary` · 1px `surface.tertiary`, inset 0, v-pad 4** |
| Tap · analytics | `UIApplication.shared.open(url)` (`:25-28`) · `promotion_btn{title,subtitle,url}` (`:38-42`) | `ACTION_VIEW` in `try/catch {}` (`:55-63`) · same (`:67-72`) | external browser; surface a failure · same |

**Strings / states:** remote — `promotion.itemList[i]["title-content"|"subtitle-content"][locale]`, `iconUrl`, `url` (`MyViewPromotionCard.swift:31-32`; `MyPromotionViewModel.kt:40-57`). `hidden` (empty itemList) · `dismissed` — **impossible on both platforms** (defect 6): no close affordance, no persisted flag, no CCKV key — add both.

### Settings card (设置卡片) — platforms: both

**Purpose:** the always-present list of in-app destinations and external links.

| Element | iOS | Android | normative |
|---|---|---|---|
| Card · row spacing · divider | `HamCardView(10, r16)` + inner pad 8 (`MyViewSettingCard.swift:130,151`) · 15 (`:131`) · `Divider()` no insets (`CollectionView.swift:36`) | `PaddingValues(8.dp)` (`MyViewLinkCard.kt:84`) · row pad 8 (`:204-209`) · `HamDivider` v-pad 4 (`:109,135,154`) | **Card §3.1 — pad 16, r 16 · `space.3` 8 · 1px `surface.tertiary`, inset 0, v-pad 4** |
| Icon plate · glyph · title · subtitle · chevron | 40⌀ `blue@0.10`, pad 8 (`:186-193`) · 20 Semibold (`:185`) · 17 Semibold `text.primary` (`:196-198`) · 12 gray (`:199-201`) · `chevron.right` gray 17 pt (`:205-206`) | 40⌀ `ham_blue@0.10` (`:211-219`) · 25 guide/feedback/debug, **30 settings** (`:103,129,147,165`) · 16 Bold (`:223-228`) · 12 secondary (`:229-233`) · 24 `Color.Gray` (`:236`) | **40 × 40 @ `accent` 0.10 · `icon.sm` 20 for every row · 17 / Bold `text.primary` · `caption` 12 `text.secondary` · `icon.xs` 12 `text.secondary`** |

Rows, in order (`MyViewSettingCard.swift:44-124`, `MyViewLinkCard.kt:78-171`):

| # | Title | Subtitle | Guard | Destination |
|---|---|---|---|---|
| 1 | 自动化 | 添加Siri捷径 | always (**iOS only**) | `MySiriView()` (`:44-50`) |
| 2 | 使用指南 | 查看使用文档 | `guide.visible` else **true**; default `https://docs.ham.nowcent.cn/` | external URL (`:67-86`, `MyViewLinkCard.kt:78-110`) |
| 3 | 反馈 | 加入 Discord 社区 | `feedback.visible` **and** valid URL; default `https://discord.gg/GwwGksTDVE` | external URL (`:88-100`) |
| 4 | 关于 | 版本信息与隐私协议 | always | `AboutView()` (`:102-111`) |
| 5 | 设置 | 自动化、小组件、关于等相关设置 | always (**Android only**) | `my-view/setting` (`MyViewLinkCard.kt:137-152`) |
| 6 | Debug / 调试入口 | Debug settings (dev only) | `#if DEBUG` / `App.DEBUG` | debug screen (`:113-124`, `MyViewLinkCard.kt:153-171`) |

**Strings:** 自动化 (`Localizable.strings:35`) · 添加Siri捷径 (`:36`) · 使用指南 (`:43`, `string.xml:45`) · 查看使用文档 (`:44`, `string.xml:46`) · 关于 (`:41`) · 版本信息与隐私协议 (`:42`) · 设置 (`string.xml:47`) · 自动化、小组件、关于等相关设置 (`string.xml:48`) · 调试入口 / `Debug settings` — **hardcoded, debug-only** (`MyViewSettingCard.swift:115-116`, `MyViewLinkCard.kt:159-160`) · hardcoded fallback URL (`MyViewLinkCard.kt:79`) · rows 2–3 titles are remote-overridable (`:72-73,92-93`).

**States:** `remote override` — rows 2–3 fall back to bundled defaults when a remote field is empty (`CCKVContext.swift:115-150`) · `link hidden` — with rows 2 and 3 hidden, only 设置 and Debug remain · dead flag `canShowCasSetting` read at `:40`, never used.

**Divergence:** iOS chips settings into this card with no 设置 row; Android's 设置 row opens a hub and is mislabelled `about_btn` for telemetry (`MyViewLinkCard.kt:143`) — ship rows 1 and 5 on both, rename to `setting_btn`.

### Settings hub (设置) — platforms: Android

**Purpose:** a pushed hub grouping the sub-settings that the My card only links to. Rows, in order: 小组件 · 自动化操作 · 语言 · 关于 (`SettingView.kt:51,57,67,87`).

| Element | iOS | Android | normative |
|---|---|---|---|
| Route · entry · row metrics · sub-destinations | **none** | `my-view/setting` (`MyViewRoute.kt:15`, `MyView.kt:47-49`) · 设置 row on the My card (`MyViewLinkCard.kt:137-152`) · see Settings card · `WIDGET_SETTING` / `AUTOMATIC_SETTING` / `LANGUAGE_SETTING` / `ABOUT` | **`my-view/setting` — iOS builds this route too** · always-present 设置 row · list row §3.3 · 4 rows, in this order |

**Strings / states:** 设置 (`string.xml:47`) · 自动化、小组件、关于等相关设置 (`string.xml:48`) · 小组件 / 自动化操作 / 关于 row titles are unreferenced leftovers at `string.xml:37-44` (`my_link_widget`, `my_link_automation`, `my_link_about`) — wire them up rather than re-adding. `debug build` — Android registers `my-view/debug` only when `App.DEBUG` (`MyView.kt:53-57`).

**Divergence:** Android-only today (§0.3); iOS folds 自动化 / 关于 into the My card with no 设置 row — iOS adds the hub route and the 设置 row; Android keeps the hub.

### About (关于) — platforms: both

**Purpose:** version, changelog, and the outward-facing actions.

| Element | iOS | Android | normative |
|---|---|---|---|
| Route · entry · content | `about` (`Route.swift:100`) · settings card row 4 (`MyViewSettingCard.swift:102-111`) · logo, version, changelog, actions (§4.8) | `my-view/about` (`MyViewRoute.kt:16`, `MyView.kt:41-43`) · settings hub row 4 (`SettingView.kt:87`) · logo, version, changelog, actions | both · settings hub row 4, once iOS ships the hub · logo · version · changelog · action rows |

**Strings / states:** 关于 (`Localizable.strings:41`) · 版本信息与隐私协议 (`:42`) · actions per §4.8: 前往 App Store, 复制课程信息, 来 Github 找我, 分享日志 · no loading, empty, or error state on either platform. Defects: iOS `AboutPrivacyView` renders a markdown link as unstyled, untappable text (`design-system.md:968`); Android duplicates the markdown-link parser in `LoginView.kt` and `AboutView.kt` (`design-system.md:969`). Metrics were not measured in the audit range.

### Platform-only surfaces — one platform ships each; the other adds it, unless the row says the gap is correct as-is.

| Surface | Platform | Purpose | Values / evidence | Strings | Action |
|---|---|---|---|---|---|
| Share sheet | iOS | Export via the system activity view | **not measured in audit range 1-1185** — no file:line | unknown | Android: add a system share surface |
| Full-text reader | iOS | Render a remote string as an in-app article (`design-system.md:867`) | **not measured in audit range** | remote | Android: webview / external browser only today — add the reader |
| Automatic / Siri shortcuts | both routes | Expose shortcuts to platform automation | iOS 31-line `MySiriView`, caption + `SiriButtonView` `frame(height: 60)` for `.libraryQuickBookIntent` (`Route.swift:219-220`); Android 132-line `AutomaticSettingView.kt` | 自动化, 添加Siri捷径 (`Localizable.strings:35-36`) | One automation screen listing every shortcut; each platform renders its own system affordance |
| Authorized apps | both routes | List and revoke third-party SSO grants | iOS `Route.swift:93`; Android `AuthorizedAppsView.kt` (revoke dialog `:117`, `android.R.string.cancel` `:144`) | not measured | Android: fix the blank page on error (`:84-112`) and the missing nav-bar spacer |
| Passkey config | both routes | Register and manage passkeys | iOS `Route.swift:91` (5 hardcoded strings `:19,27,40,44,66`); Android `user-center/passkey` | not measured | iOS: localise the 5 literals |
| Social binding | both routes | Link and unlink social accounts | iOS `Route.swift:90` (7 hardcoded strings `:139,174,206,211,248,280,285`); Android `user-center/social-account` | 微信, 自强 — hardcoded, keys exist (§0.4) | iOS: localise all 7 literals |
| Settings hub | Android | Group the sub-settings | `my-view/setting` (`MyViewRoute.kt:15`) | 设置, 自动化、小组件、关于等相关设置 | iOS: add the route and the 设置 row |
| Language | Android | In-app locale override | `my-view/setting/language-setting` (`MyViewRoute.kt:16`) | 3 `translatable="false"` language names (`values/string.xml:88-90`) | None — iOS follows the system locale (`design-system.md:883`) |
| Widget settings | Android | Configure the widget update interval | `my-view/widget` (`MyViewRoute.kt:11`, `MyView.kt:35-37`) | `my_link_widget` / `my_link_widget_subtitle` (`string.xml:39-40`), unreferenced | None — WidgetKit owns refresh (`design-system.md:882`) |


## 9. User center (用户中心)

Android cells read `= iOS`: the Android build mirrors these values; the parallel Android section supplies them. Tokens are from `docs/design-system.md`.

### User center main (用户中心) — platforms: iOS
**Purpose:** Signed-in account hub: avatar + nickname header, six account-management rows, a 更多 row, and a destructive 退出登录 button.
**Layout:**
```text
┌ ScrollView bg b1 · VStack pad 16 · no nav title ───┐ :81,79,16-87
│ sp 36 · avatar 120×120 Circle (gray@15% + face.smiling 64, fade 0.5 s) :19-36
│  sp 8 · nickname .title2 22 · sp 32 · CARD 1 HamCardView(pad 4) r16 :38-42
│  6 rows [32×32 r8 gray, glyph 20 white]–16–title 17–›16 ≈218 :94-105
│  sp16 · CARD2 [32]更多 › ≈56 · sp32 · logout 退出登录 17 red ≈38 :58-74
```
**Blocks:** header (avatar + nickname, no ID/学号 label) `:20-41` · card 1 → `UserCenterInfoView`, `SyncLoginDeviceView`, `SyncSocialAccountView`, `UserCenterPasskeyConfigView`, `ScanCodeView`, `AuthorizedAppsView` `:44-54` · card 2 更多 → `SyncLogoutView` `:59-64` · logout `vm.logout()` + `dismiss()` `:66-77` · auto-dismiss on `isLogin == false` `:82-86`.
**Values:**
| Element | iOS | Android | normative | source |
|---|---|---|---|---|
| Colour | `ham_bg_b1Color` #F9F9F9 / #000000 · card `ham_bg_b2Color` #FFFFFF / #0F1010 · row plate `Color.gray` ≈#808080 · logout `.red` | = iOS | `surface.primary` / `surface.secondary` / `text.secondary` / `text.danger` | `:81,42,97,71`; `Color+Ham.swift:22,23` |
| Type + geometry | nickname `.title2` 22 pt (no `lineLimit`) · row title 17 pt (no `.font` → `.body`) · chevron 16 pt · placeholder glyph 64 · root pad 16, card pad 4 + inner 8, spacers 36/8/32/16/32 · plate 32×32 r8, glyph 20, gap 16 · row 32 · card 1 ≈218, card 2 ≈56, logout ≈38 · hairline `Divider()`, no inset, no tint | = iOS | `title2`, `body`, `icon.xs` (12) chevron, `icon.xl` (64) · `space.5` margin, `space.6` separation, `radius.card` · §3.3 plate 40×40, `space.3` gap · `surface.tertiary` divider | `:40,100,103-105,25-28,79,42,56,19,38,41,58,65,94-105,45-53` |
| Chrome + states | no navigation title · no loading or error state · nickname `""` until `userInfoFlow` emits; a bad avatar URL keeps the placeholder | = iOS | title on every pushed screen · spinner, error copy, retry | `:16-87`; `UserCenterViewModel.swift:15`; `:20-31` |
**Strings:** 个人信息 (`:518`), 登录设备 (`:223`), 社交账号 (`:239`), Passkey管理 (`:502`), 扫码登录 (`:668`), 扫描登录二维码 (`:665`), 授权应用 (key `user_center_authorized_apps`, `:918`), 更多 (`:703`), 退出登录 (`:227`) — all `String(localized:)` at `UserCenterView.swift:44-71`; zero hardcoded.
**States:** `logged out` — unreachable (entry card gates on `isLogin` `MyUserCenterCard.swift:17-19`; observer dismisses); the sibling copy 点击登录 / 不登录也可以使用校内功能哦 lives in the entry card `MyUserCenterCard.swift:85,90`, the second HARDCODED · `loading`/`error` — none · `post-logout` — `logout()` + immediate `dismiss()` `:67-68`.

### Personal info / edit info (个人信息) — platforms: iOS
**Purpose:** Profile editor: one editable field (昵称) plus a tappable avatar opening the system photo picker, committed by a nav-bar 保存 button.
**Layout:**
```text
┌ NavBar 个人信息 (bare literal) · trailing [保存] :73-82
├ VStack(spacing:32) pad .vertical 16 · bg b1 ignoreSA :18,69-72
│ sp 16 · avatar 120×120 Circle (gray@15% + 64 pt) · badge 36×36 pad 8 :19-47
│ VStack(spacing:0) full width · Divider · TextField 昵称 pad h8 v6 bg b2 :51-62
│  ≈34 pt · Divider · Spacer() — no ScrollView :53-56,66
```
**Blocks:** top spacer `:19` · avatar editor `PhotosPicker(matching:.images)` + placeholder + fade 0.5 s + pencil badge `:21-49` · nickname field block (divider / field / divider), seeded on appear only when empty `:51-64,57-61` · trailing `Spacer()`, no `ScrollView` `:66` · save toolbar item `vm.confirmEditUserInfo()`, always enabled, no confirmation, no pop on success `:74-82`.
**Values:**
| Element | iOS | Android | normative | source |
|---|---|---|---|---|
| Colour | bg `ham_bg_b1Color` · field and card `ham_bg_b2Color` #FFFFFF / #0F1010 · placeholder `ham_text_t2Color`@0.15 · badge `Color.gray` (literal) · error toast red/white, success `.normal` (`secondarySystemBackground`) | = iOS | `surface.primary` / `surface.secondary` / `text.tertiary`; §3.8 toast | `:70-72,56,24-31,45`; `Color+Ham.swift:22,23`; `Toast.swift:31-34` |
| Type + geometry | root pad 16, `VStack(spacing:32)` · avatar 120×120 Circle scaledToFill, bg b1, fade 0.5 s, glyph 64 · badge 36×36, pad 8, `pencil` white · `TextField("昵称")` `.body` 17 pt, pad h8/v6, ≈34 pt, full width · nav title default `.automatic`, save `Text("保存")` 17 pt accent | = iOS | `space.5`, `space.6` · 120 pt circle + `icon.xl` · badge `icon.md` on `surface.tertiary` · §3.6 field: r8, `surface.secondary`, 1 px `surface.tertiary`, pad 8 · inline `title3` + borderless `text.link` | `:69,18,35-38,39-47,53-56,64,73,74-82` |
| Limits + feedback | nickname ≤ 20 chars · avatar `jpegData(0.5)` else png, ≤ 4 MB, 2 MB gRPC chunks, name `UUID().uuidString + ".jpg"` · toast: top banner, HStack 5, icon 36, title 17 semibold, content 12 (2 lines), pad 16 · `.cancelled` swallowed | = iOS | same caps · §3.8 toast · surface cancellation | `UserCenterInfoViewModel.swift:34,64-66,81`; `ToastView.swift:85-117`; `ToastUtils.swift:89-91` |
**Strings:** HARDCODED bare literals — 个人信息 (`:518` @ `:73`), 保存 (`:553` @ `:79`), 昵称 (`:697` @ `:53`) · localised — 用户名输入有误 (`:451`), 用户名不能大于20个字符 (`:452`), 图片大小不能大于4MB (`:453`), 保存成功 (`:423`), 遇到了错误 (`:384`).
**States:** `logged out` — no guard, no auto-dismiss; placeholder avatar, empty field, save fails `用户名输入有误` `:22-31,53` · `loading` — none; save and avatar stream run with zero feedback, button stays tappable so taps duplicate (`UserCenterInfoViewModel.swift:39-41,71-101`) · `error` — top toast only, no inline field error · `success` — 保存成功, screen does not pop; avatar upload emits no toast (`:46-47`).

### Login devices (登录设备) — platforms: iOS
**Purpose:** Lists every device holding a session for the account so the user can audit sessions and remotely kick a device; the device matching this app's push token shows 本机.
**Layout:**
```text
┌ ← 登录设备 (inline nav bar) · VStack padding(16) :38-44
├ ScrollView bg b1 · ignoreSafeArea(.bottom) :21,41,42
│ [.loading] ProgressView().padding() · [else] HamCardView(pad 8) r16 :23-26
│  row [32 r4 gray, glyph 24]–16–desc 17 / id 12 / 最近上线 12 ≈70 :93-108
│   –Spacer+4– [下线 r8 pad 4/8] | 本机 12 | ◌ · Divider between :114-133
```
**Blocks:** inline nav bar `:43-44` · loading `ProgressView().padding()` on `.loading` `:23-24` · device card `HamCardView(padding:8)` + `ForEach` `:25-36` · row = 32×32 `iphone` plate (r4, `ham_text_t2Color`, glyph 24 white) → 16 → desc / `deviceID[3...]` / 最近上线 → `Spacer` + 4 → 下线 button or 本机 `:93-135` · kick `deleteDevice()`, no confirmation, `deleteCallback()` runs even on failure → refetch `:137-162`.
**Values:**
| Element | iOS | Android | normative | source |
|---|---|---|---|---|
| Colour | bg `ham_bg_b1Color` · card `ham_bg_b2Color` · plate `ham_text_t2Color` ≈#808080 · kick fill `ham_btn_bColor` #EFF2F2 / #444444 with `ham_text_t1Color` text · card overlay `Image(systemName:"")` 200 @0.2 `Color.lightGray` #F0EFEF | = iOS | `surface.primary` / `surface.secondary` / `text.secondary` · §3.4 destructive pill: 1 px `text.danger`@0.3 | `:41,26,99,123,126-127`; `Color+Ham.swift:22,23,27`; `CardView.swift:80-86,128` |
| Type + geometry | desc 17 pt `.primary` · id and last-seen `.caption` 12 pt · kick `.caption` 12 pt, pad v4/h8, r8 · plate 32×32 r4, glyph 24 · gap 16 · row ≈70 pt · root pad 16, card pad 8 · plain `Divider()` between rows only | = iOS | `body` title + `caption` meta in `text.secondary` · §3.3 plate 40×40 + `icon.md` · `space.3` gap · `surface.tertiary` divider | `:103-108,121-127,95-101,39,26,30-32` |
| Data + behaviour | `最近上线：%@` from `update_time`/1000 → `yyyy-MM-dd HH:mm` · id shown as `deviceID[3...]`, `""` when ≤4 chars · no IP (`user.proto:14-18`) · no confirmation, row never `.disabled`, in-flight button → `ProgressView()` | = iOS | `ham_toyyyymmddhhmm` · confirm before kick, row disabled in flight, per-row spinner | `:104-108`; `Date+Format.swift:54-56`; `:137-162,115-116` |
**Strings:** 登录设备 (`:223` @ `:44`) · 最近上线：%@ (`:233` @ `:107`) · 下线 (`:232` @ `:121`) · 本机 (`:231` @ `:131`) · unused: 管理你的登录设备 (`:224`), 已登录设备 (`:229`), 设备已删除 (`:230`) · hardcoded: `iphone` `:95`, tag `SyncLoginDeviceView` `:45`.
**States:** `loading` — card replaced by centred spinner `:23-24` · `loaded` — row per device, own device shows 本机 · `error`/`unload` — both fall through to the card branch, so an error looks like an empty list; toast only, no retry, no `.refreshable` `:16,23,67` · `empty` — ~16 pt blank white rounded card; 无记录 (`:246`) unused · `kick in flight` — button → spinner, row not disabled `:115-116`.

### Social accounts (社交账号) — platforms: iOS
**Purpose:** Shows which OAuth providers are linked to the account and starts the bind flow for unlinked ones.
**Layout:**
```text
┌ ← 社交账号 (inline nav bar) · VStack padding(16) :60-61,110-111
├ ScrollView bg b1 · ignoreSafeArea(.bottom) :24,108,109
│ [.loading] ProgressView().padding() · [error/unload] ∅ :56-58
│ [.loaded] HamCardView(pad 8) r16 · [28×28 r8]–16–name 17 :124-147
│  已登录 17 | › 16 · QQ·微信·Apple·Github·自强 · ≈160 pt :51-53
Overlays: A GitHub WebContainer + Github登录 + 取消 [DEAD] :65-79
          B WeChat NavigationView + UIKWebView + 微信登录 + 取消 :80-107
```
**Blocks:** nav bar `:110-111` · GitHub `fullScreenCover` dead (`isShowingLoginWithGithub` never true) `:65-79` · WeChat cover live, handoff via `UIPasteboard` → `decryptWeixinRawData` → clear `:80-107` · provider card over CCKV `ham_validLoginType`, dispatch qq/wechat/apple/github/ziqiang, **no else branch** so a `passkey` entry emits no row `:26-55` · row tap guarded `if !login { onClick() }`, so a bound row does nothing `:114-294` · bind flows never refresh `openList` (`UserCenterSocialAccountViewModel.swift:48-169`).
**Values:**
| Element | iOS | Android | normative | source |
|---|---|---|---|---|
| Colour | bg `ham_bg_b1Color` · card `ham_bg_b2Color` r16 · boxes QQ #12B7F5, WeChat #58BE6A, Apple `text.primary`, GitHub black/white, 自强 #01579B@0.1 · trailing `ham_text_t2Color` ≈#808080 | = iOS | `surface.primary` / `surface.secondary` · provider brand @0.1 plate · `text.secondary` trailing | `:108,27,131,166,203,240,277,146` |
| Type + geometry | name and 已登录 17 pt `.body` `text.primary` · unbound `chevron.right` 16 pt · box 28×28 r8, glyph 24×24 raster or 20 pt SF Symbol · gap 16 · row 28 pt, no vertical padding · root pad 16, card pad 8 · `Divider()` between rows only, tested against `.last` | = iOS | `body` title · `icon.xs` chevron · §3.3 plate 40×40 · `space.3` gap · 52 pt row · `surface.tertiary` divider | `:134-146,126-132,163,60-61,27,51-53` |
| Overlays | `.fullScreenCover`, full height, no detents · WeChat URL force-unwrapped from CCKV `ham_sync_wechatLoginUrl` · webview min font 10, bounces off · GitHub/自强 fall back to hardcoded URLs that exit to Safari | = iOS | §3.7 sheet: 85 % detent, drag handle, scrim 0.5; in-app webview | `:65,80-83,318,337`; `UserCenterSocialAccountViewModel.swift:18-22` |
**Strings:** 社交账号 (`:239` @ `:111`) · Github登录 (`:236`) · 微信登录 (`:238`) · 取消 (`:237`) · HARDCODED `Text(_:)` (never localised): 已登录 (`:139,174,211,248,285`), 微信 (`:280`), 自强 (`:206`), QQ (`:134`), Apple (`:169`), Github (`:243`) — unused entries exist at `:643`, `:658`, `:782` · dead: 解绑 (`:234`), 未绑定 (`:235`).
**States:** `loading` — `ProgressView().padding()` `:57` · `loaded` — provider card · `error`/`unload` — blank #F9F9F9 canvas, no retry, no text `:26,56` · `empty` — ~16 pt blank white card (CCKV default `[]`) · `bind in flight` — no visual state, no per-row spinner.

### Passkey config (Passkey管理) — platforms: iOS
**Purpose:** Manage WebAuthn passkeys: an explainer paragraph, a full-width 添加Passkey button driving `ASAuthorizationController`, and registered passkeys each with an inline 删除 button.
**Layout:**
```text
┌ NavBar "Passkey管理" (bare literal) :66
├ ScrollView bg b1 · VStack pad 16 · maxWidth/.top :65,62,64
│ ① paragraph 12 pt gray ≈80 · ② ＋ 添加Passkey pad 16 bg b2 r12 ≈49 :19-33
│ ③ pad .vertical 16 · .loading ProgressView · ∅ on error · heading 17 bold :36-41
│  row name 14/2 · yyyy-MM-dd HH:mm创建 12 · [删除] red +4 · pad 16 r12 ≈65–86 :43-53
```
**Blocks:** explainer paragraph, always shown `:19-21` · add button `vm.registerPasskey()`, no loading or disabled state `:22-33` · section gated on `passkeyListLoadState == .loaded` `:35-59` · row `UserCenterPasskeyConfigItemView` + delete closure, no confirmation, no undo, no per-row spinner `:19-37` · refresh from `init()`, after delete, after register; no pull-to-refresh (`UserCenterPasskeyConfigViewModel.swift:24,41,82-85`).
**Values:**
| Element | iOS | Android | normative | source |
|---|---|---|---|---|
| Colour | bg `ham_bg_b1Color` · button and rows `ham_bg_b2Color` #FFFFFF / #0F1010 · paragraph literal `.gray` · empty + date `ham_text_t2Color` · 删除 `.red` · success toast `.success` green/white | = iOS | `surface.primary` / `surface.secondary` / `text.secondary` / `text.danger`; §3.8 toast | `:65,31,36,21,46,30`; `Color+Ham.swift:22,23`; `Toast.swift:27-28` |
| Type + geometry | content pad 16, VStack spacing unspecified · paragraph `.caption` 12 pt, ≈80 pt · add button full width, pad 16, r12, 17 pt accent, ≈49 pt · heading 17 pt `.bold` · 暂无数据 12 pt +8 · row pad 16, r12, name 14 pt `lineLimit(2)`, date `.caption` 12 pt, ≈65 pt (≈82 pt wrapped), no divider between rows · delete 17 pt `.red`, `.padding(.leading,4)` | = iOS | `space.5` + explicit `space.3` · `caption` in `text.secondary` · §3.4 tinted: 48 tall, r12, `tint.subtle`, `accent` text · `bodyBold` heading · §3.1 card body r16 · §3.9 empty `icon.xl` + `caption` · §3.4 destructive pill | `:62,18-33,40-47`; `UserCenterPasskeyConfigItemView.swift:19-37`; `Date+Format.swift:54-56` |
| Feedback | `withAnimation` default on every mutation · success toast 已删除Passkey + refresh · errors `showGRPCError` · `ASAuthorizationError` logged and dropped with no user-visible message · register refreshes silently | = iOS | §3.8 toast; surface registration failure with a toast | `UserCenterPasskeyConfigViewModel.swift:40,55,65,73,82-85,87-92` |
**Strings:** HARDCODED bare literals — Passkey 是一种安全、便捷的无密码身份验证技术… (`:501` @ `:19`), 添加Passkey (`:746` @ `:27`), 已注册的Passkey (`:641` @ `:40`), 暂无数据 (`:700` @ `:44`), Passkey管理 (`:502` @ `:66`) · localised — 删除 (`:577`), %@创建 (`:469`), 已删除Passkey (`:424`), 遇到了错误 (`:384`).
**States:** `loading` — spinner replaces the section; no spinner for register or delete `:36-37` · `error` — `.loadedWithError` renders nothing, no retry `:36-38` · `empty` — 暂无数据 under the still-visible heading `:43-47` · `success` — delete toast + refresh; register refreshes silently `:39-42,82-85`.

### Logout / deactivate account (退出登录) — platforms: iOS
**Purpose:** Nominal home for signing out and permanently deactivating the account; it renders one control, 注销该账号, whose 确定 performs an ordinary logout.
**Layout:**
```text
┌ ← 退出登录 (inline nav bar) :97-98
├ ScrollView bg b1 · VStack(.leading) pad h16 v8 · bgFill b2 r=0 :21,36,78-84
│  8 · 注销该账号 17 pt red · Spacer · › rot 0°/90° :38-49
│  [expanded] 12 pt gray warning · HStack Spacer → [确定] | ◌ :53-74
│  collapsed ≈38 · expanded ≈135 · 确定 ≈33 (pad v8/h16 r8 red@0.1) :65-70
```
**Blocks:** nav bar `:97-98` · [disabled] logout row, commented out `:23-34` (logout lives on the user-center screen) · deactivate row `openDeActiveWarning()` — pure disclosure toggle, chevron rotates 90° `:38-49,101-105` · warning + confirm block, right-aligned, **no cancel button** `:51-76` · `deActive()` → `HamAccountManager.shared.logout()`; the real deactivation RPC is commented out; `isLogin == false` drives `dismiss()` `:107-140,89-93`.
**Values:**
| Element | iOS | Android | normative | source |
|---|---|---|---|---|
| Colour | screen `ham_bg_b1Color` · block `ham_bg_b2Color` #FFFFFF / #0F1010 · label `ham_btn_tdColor` red #FF0000 · chevron `.black.opacity(0.2)` (not dark-mode adaptive) · confirm fill `ham_btn_bdColor` red @0.1 | = iOS | `surface.primary` / `surface.secondary` / `text.danger` · `text.secondary` chevron · §3.4 destructive pill 1 px `text.danger`@0.3 | `:95,84,43,47,69-70`; `Color+Ham.swift:22,23,25,29` |
| Type + geometry | row label 17 pt (no `.font`) · chevron 17 pt (no `.font`), rotation 0° collapsed / 90° expanded · warning `.type(.caption)` → 12 pt in `ham_text_t2Color` (`.ham_text_t1Color` on `:54` is dead) · 确定 17 pt, pad v8/h16, r8, ≈33 pt · block pad h16/v8, **no corner radius** · collapsed ≈38 pt, expanded ≈135 pt | = iOS | `body` in `text.danger` · `icon.xs` chevron in `text.secondary` · `caption` in `text.secondary` · §3.4 destructive pill: 28 tall, h12/v6, r8 · §3.1 card r16, pad 16 | `:42,45-47,53-55,65-70,78-79,36`; `SyncLoginDeviceView.swift:172-184` |
| Flow + feedback | no alert, no typed confirmation, no cancel · `withAnimation` default · `logout()` swallows the RPC with `try?` · `isDeActiveLoading` never reset · no success toast (`已注销账号` is commented out); the screen pops via `dismiss()` | = iOS | typed confirmation before permanent deletion · surface failures · success toast | `:51-76,102-110`; `HamAccountManager.swift:150`; `:89-93,133` |
**Strings:** 退出登录 (`:227` @ `:98`, also the commented-out row `:27`) · 注销该账号 (`:241` @ `:42`) · 注销该账号后，您将无法使用同步功能，您的一切个人信息将被清除，是否继续？ (`:242` @ `:53`) · 确定 (`:243` @ `:65`) · 更多 (`:703`, entry row) · entry subtitle 从该设备退出登录或永久注销该账号 (`:228`) · dead 已注销账号 (`:133`).
**States:** `collapsed` — red row, 0° chevron · `expanded` — 90° chevron, warning, right-aligned 确定 · `loading` — 确定 → spinner, row not disabled `:58-60,108-110` · `error` — none · `success` — no toast, screen pops via `dismiss()` `:89-93`.

### Authorized apps (授权应用) — platforms: iOS
**Purpose:** Lists third-party apps granted SSO authorization and revokes them, with pagination and a native confirmation alert.
**Layout:**
```text
┌ ← 授权应用 (inline nav, key user_center_authorized_apps) :28-29
├ Group bg ham_bg_b1Color :18,27
│ [empty&&.loaded] ✔ 48 pt gray + 12 gap + 暂无授权应用 17 · [empty&&.loading] ◌ :50-69
│ [else] LazyVStack pad top 8 · row [44 r10]–12–name 17 / 授权于 %@ 12 / scope 11 :72-174
│  [取消授权] outline red | ◌ 24 · Divider inset 16 every row · sentinel h1 :83-103
ALERT 取消授权 / 确定要取消对该应用的授权吗？ / 取消 / 取消授权(destructive) :35-44
```
**Blocks:** nav bar `:28-29` · three-branch switch (empty / first-load spinner / list) `:18-26` · empty view 48 pt symbol + 12 pt gap + 17 pt text, no retry `:47-60` · list `LazyVStack(spacing:0)`, divider after **every** row incl. the last, load-more sentinel, page spinner `:71-107` · row `AuthorizedAppRow`: 44×44 r10 icon (fade 0.3 s, placeholder `app.fill` 22 pt on gray@15 %), info stack spacing 4, 取消授权 outlined pill or 24×24 spinner `:112-190` · revoke: alert → `revokeAuthorization(appId:)`, row removed, success toast (`:78-81`; `AuthorizedAppsViewModel.swift:56-82`).
**Values:**
| Element | iOS | Android | normative | source |
|---|---|---|---|---|
| Colour | `Group` on `ham_bg_b1Color` · empty icon and text `ham_text_t2Color` ≈#808080 · placeholder `RoundedRectangle r10` fill gray@0.15 · name `ham_text_t1Color` · meta `ham_text_t2Color` · 取消授权 `.red` on a `red@0.3` 1 px outline, no fill | = iOS | `surface.primary` · §3.9 `text.tertiary` · `text.primary` / `text.secondary` · §3.4 destructive pill | `:27,52,55-56,182-183,138,145,151,166-172` |
| Type + geometry | list `LazyVStack(spacing:0)`, top pad 8 · empty icon 48 pt, 12 pt gap, text `.body` 17 pt · row `HStack(spacing:12)`, pad h16/v12, ≈84/66/46 pt · icon 44×44 r10, fade 0.3 s · info `VStack(spacing:4)` · name 17 pt 1 line, 授权于 %@ 12 pt, scopes 11 pt `caption2` 1 line · pill 12 pt, pad h12/v6, r8 · in-flight `ProgressView()` 24×24 · footer `ProgressView().padding()` 16 | = iOS | §1.3 16 margin / 8 gap · §3.9 `icon.xl` (64) + `caption` · §3.3 row: 40×40 plate, `space.3` gap, `body` title, `caption` meta · §3.4 destructive pill | `:72-73,105,50-56,118,121-129,135-153,159-174,176-177,96-103` |
| Data + alerts | `PAGE_SIZE 10`, first page 1, sentinel `Color.clear` h1 while `loadState != .loading` and `count < total` · date `formatted(date:.abbreviated, time:.omitted)` (system locale, not `ham_toyyyymmddhhmm`) · native `.alert(_:isPresented:)`, title and destructive button share the string 取消授权 · revoke guarded one at a time | = iOS | page spinner · app-wide `yyyy-MM-dd HH:mm` · distinct alert title and action label | `AuthorizedAppsViewModel.swift:14,23,30-54,57`; `AuthorizedAppsView.swift:88-93,142,35-44`; `Date+Format.swift:54-56` |
**Strings:** 授权应用 (`user_center_authorized_apps`, zh-Hans `:918`) · 暂无授权应用 (`:919`) · 取消授权 (`:920`) · 确定要取消对该应用的授权吗？ (`:921`) · 已取消授权 (`:922`) · 授权于 %@ (`:923`) · 取消 (`sso_auth_cancel`, `:905`) · hardcoded: `app.badge.checkmark`, `app.fill` (`:50,185`), `sso_authPreferences` (`SSOAuthPreferenceManager.swift:25`).
**States:** `first load` — centred spinner `:62-69` · `loaded with apps` — rows + dividers · `loaded, zero apps` — empty view `:47-60` · `loading next page` — footer spinner, rows stay, sentinel suppressed `:88,96-103` · `error` — `.loadedWithError` matches no branch → blank canvas when empty, frozen rows otherwise; toast only `:19,21` · `revoking` — pill → spinner, second revoke blocked at the VM (`AuthorizedAppsViewModel.swift:57`) · `revoked` — row removed, `total` decremented, prefs cleared, success toast `:70-76`.

### Scan code (扫一扫) — platforms: iOS
**Purpose:** Full-screen ScanKit camera scanner for QR-login tickets (`ham://qrcode-login?ticket=…`) and course palette sharing; the app owns only a hint caption and a decorative sweep — mask, window and reticle are SDK-drawn.
**Layout:**
```text
┌ status bar 47 · nav bar ~44 · layer 1 HmsScanCodeView full-bleed :23-29,120
│  SDK-drawn dim mask, scan window, corner reticle · continuouslyScan = true :27
│  layer 2 caption 17 pt white · offset y −250 → top 172 pt (390×844) :121-123
│  layer 3 lidar: 20 pt band, inset 16 (358 pt), sweeps y 262→732 :126-149
└  background Color.black · ignoresSafeArea(.top,.bottom) :151-153
```
**Blocks:** camera layer `HmsCustomScanViewController` in a `UINavigationController`, `cutArea` never set → SDK default `:23-29` · hint caption `Text(title).foregroundStyle(.white).offset(y:-250)` `:121-123` · lidar band `Spacer` h20 `.clipped()` + `Circle` radial gradient, `showLidar` gate, `startAnimation()` from `.onAppear` `:126-179` · permission toast from `checkPermission()`, auto-dismiss 3.3 s (`ScanCodeViewModel.swift:37-57`; `ToastView.swift:37`) · no torch and no album button exist under `Ham/`.
**Values:**
| Element | iOS | Android | normative | source |
|---|---|---|---|---|
| Root + camera | `ZStack` full screen, `Color.black` @1, `.ignoresSafeArea([.top,.bottom])` · `continuouslyScan = true`, `backButtonHidden = true`, `cutArea` unset → SDK default | = iOS | black canvas edge-to-edge; explicit scan-window rect + dim @0.5 if the SDK allows theming | `:119,151-153,23-29` |
| Caption + lidar | caption `.body` 17 pt `Color.white`, `.offset(y:-250)` → 172 pt on 390×844, 83.5 pt on 375×667 · band 20 pt visible, inset 16 each side (358 pt) · base `Circle()` 200×200 scaled x1.79 / y0.2 (→40 pt ellipse) · `RadialGradient` `Color.blue` 0.9→0.7→0.5→0, `startRadius 0`, `endRadius 100` | = iOS | `body` white, positioned relative to the scan window · sweep band inset `space.5`, `accent` gradient, 20 pt tall | `:121-143` |
| Animation + permission | fade-in `.linear(1 s)` ∥ sweep `.easeInOut(3 s, y −150→300 = 450 pt)`; fade-out `.linear(1 s)` after a 2 s sleep; idle 1 s; loop ≈4 s; task never cancelled on disappear · `AVCaptureDevice.requestAccess(for:.video)`; denied/restricted → toast, no dedicated screen, no 去设置 link · toast `.error` red bg / white fg, icon `Image(systemName:"")` (never renders), HStack 5, icon 36, title 17 semibold, content 12 (2 lines), pad 16, top `safeAreaTop()`, 3.3 s | = iOS | 4 s loop cancelled on disappear · permission-denied view with a settings deep link · §3.8 toast with a visible icon | `:156-179`; `ScanCodeViewModel.swift:37-57`; `ToastType+UI.swift:18,28`; `ToastView.swift:37,85-117`; `ToastUtils.swift:76,88` |
**Strings:** 扫描登录二维码 (`Localizable.strings:665`) · 扫描配色分享二维码 (`:666`) · 相机权限未开启 (`:418`) · 请前往“设置”开启 (`:419`, full-width quotes U+201C/201D) · 扫码登录 (`:668`, entry row) · 是否允许Ham访问你的相机，以启用扫一扫功能 (`Ham/iOS/Info.plist:108`) · zero CJK literals in `ScanCodeView.swift`.
**States:** `authorized` — preview + caption + looping sweep · `denied/restricted` — toast then return; camera stays black, sweep keeps animating, no retry (`ScanCodeViewModel.swift:46-54`) · `scan success (ham://)` — deep link → `DeepLinkManager` → `.qrCodeLogin(ticket:)`; duplicate pushes suppressed by `router.path.last != route` (`MainDeeplinkView.swift:18`) · `scan success (other)` — `.ham_onReceiveQrCode` with `userInfo["id"]`/`["text"]`; only `CourseSettingThemeView.swift:91-99` listens · `result dropped` — payloads without a `ResultPoint` array discarded silently (`ScanCodeView.swift:66-68`).

### QR code login (二维码登录) — platforms: iOS
**Purpose:** Confirmation surface for a scanned desktop login QR: fetch the server message for the ticket, show it under a computer icon, and offer 确认登录 when the server state is `requestConfirm`.
**Layout:**
```text
┌ status bar 47 · nav bar 二维码登录 44 (default .automatic) :39
├ VStack(spacing:8) pad h16 · maxWidth/.top :20-21,37-38
│  spacer 64 · 🖥 desktopcomputer 72 pt · message 17 pt ≈22 :25-28
│  40 gap (8 + pad 32) · 确认登录 maxWidth 350 pad 16 r8 ≈54 :30,63-80
│  success: sp 64 · checkmark.circle.fill 64 · 登录成功 .title 28 · 返回 :43-61
```
**Blocks:** top spacer 64 `:25` · device icon `desktopcomputer` 72 pt, no colour modifier `:26-27` · message `loginInfo.message.ifEmpty { "确定在电脑上登录Ham吗" }` `:28` · confirm button `ConfirmButton(title:)` + `.padding(.top,32)`, gated on `state == .requestConfirm` `:29-33` · success block 64 pt spacer, 64 pt two-tone checkmark, 登录成功 `.title`, 返回 → `dismiss()` pops to the live scanner `:43-61` · no cancel button, no timer, no expiry handling.
**Values:**
| Element | iOS | Android | normative | source |
|---|---|---|---|---|
| Chrome | `VStack(spacing:8)`, pad h16, `maxWidth/.top`; **no background declared** → inherits the stack page background | = iOS | `surface.primary`, `space.5` margin, `space.3` gap | `:20-21,37-38` |
| Content | `Spacer().height(64)` · `desktopcomputer` `.system(size:72)`, no colour → `.primary` · message `.body` 17 pt `.primary`; fallback only when the server sends `""`; server string rendered verbatim, not localised · success: spacer 64, `checkmark.circle.fill` 64 pt `.foregroundStyle(.white,.green)`, 登录成功 `.title` 28 pt, 返回 button | = iOS | `space.7`×2 · `icon.hero` (72) in `text.primary` · `body`; always localise the fallback · `feedback.success` icon, `title` label | `:25-28,43-61`; `String+Utils.swift:32-34` |
| Button + data | 17 pt, pad 16, `maxWidth 350` (358 offered, capped 350, 20 pt from each edge), `Color.blue` text on `Color.blue.opacity(0.1)`, `.cornerRadius(8)` after `.background`, ≈54 pt; no `.buttonStyle`, no disabled or pressed state · `GetQrCodeTicketInfo` from `init()` (not `.task`); `ConfirmQrCodeLogin`; `CheckQrCodeLogin` never called → expiry unobservable; `state`: none 0 / success 1 / requestConfirm 2 / fail 3 | = iOS | §3.4 tinted: 48 tall, r8, `tint.subtle` + `accent` text, full content width, pressed state · fetch in `.task`; poll `CheckQrCodeLogin` and render expiry | `:63-80`; `QrCodeLoginViewModel.swift:25-35,50-77`; `QrCodeLoginRequestModel.swift:12,18`; `qr_login.pb.swift:73-78` |
**Strings:** HARDCODED bare literals — 二维码登录 (`:525` @ `:39`, resolves at runtime) · 确定在电脑上登录Ham吗 (`:765` @ `:28`, `String` intermediate → never localised) · 确认登录 (`:766` @ `:30`) · 登录成功 (`:196` and `:421`, duplicate key, @ `:52`) · 返回 (`:830` @ `:54`) · 遇到了错误 (`:384`, via `String(localized:)`).
**States:** `initial / fetch error` — content area entirely blank, no spinner, no retry (`QrCodeLoginView.swift:22,24`) · `requestConfirm` — icon + message + button · `none/success/fail` — icon + message, no button; `fail` is a dead end `:29` · `confirm in flight` — no visual change, re-tap is a silent no-op (`QrCodeLoginViewModel.swift:53-55`) · `success` — checkmark screen, 返回 pops to the still-live scanner `:55` · `error` — `.loadedWithError` written but unread by the view; `.cancelled` fails silently (`:66-68`; `ToastUtils.swift:96-98`) · `expired` — does not exist.

### SyncLoginView — platforms: iOS
**Not a surface.** `Ham/iOS/ui/sync/SyncLoginView.swift` (47 lines) declares no `View` — no `struct SyncLoginView`, no `body`, no reference outside its own header (`:2`, still reading `SyncSettingView.swift`) and `project.pbxproj`. Its only content is two `WKNavigationDelegate` helpers: `LoginWithGithubWebviewDelegate` (`:14-33`, posts `.ham_receivedDeepLink` for `ham://login-server/open/github/redirect`) and `LoginWithWechatWebviewDelegate` (`:35-47`, hands `weixin://` to `UIApplication.shared.open`), live only via `LoginView.swift:226,276` and `SyncSocialAccountView.swift:83`.
**Values / Strings / States:** none declared. Hardcoded URL fragments `ham`, `login-server`, `/open/github/redirect`, `weixin` (`:26-28,42`). **Normative:** treat the file as dead and misnamed; the cross-client login screen maps to `Ham/iOS/ui/common/login/LoginView.swift`.

### Shared tokens (iOS user-center / account stack)
| Token | Resolved value | file:line |
|---|---|---|
| `ham_bg_b1Color` = `surface.primary` | #F9F9F9 (0.977) light / #000000 dark | `Color+Ham.swift:22`; `bg_b1Color.colorset` |
| `ham_bg_b2Color` = `surface.secondary` | #FFFFFF light / #0F1010 (0.060,0.061,0.061) dark | `Color+Ham.swift:23`; `bg_b2Color.colorset` |
| `ham_text_t1Color` / `ham_text_t2Color` | `text.primary` = `Color.primary` (black light / white dark) · `text.secondary` = `Color.gray` ≈ #808080 — **fixed, no dark variant** | `Color+Ham.swift:16,17` |
| `ham_btn_tdColor` / `ham_btn_bdColor` / `ham_btn_bColor` | `Color.red` #FF0000 / red @0.1 / #EFF2F2 light · #444444 dark | `Color+Ham.swift:25,29,27` |
| `ham_lightGray` / `ham_lightBlue` | #F0EFEF / #DEEBFE | `Color+Ham.swift:45,44` |
| `HamCardView` | `padding` param · r16 `clipShape` · bg `ham_bg_b2Color` · overlay `Image(systemName:"")` 200 @0.2 `Color.lightGray`, offset (0,0) | `CardView.swift:23,45,49,80-86,122-129` |
| `Spacer().height(_:)` / `.width(_:)` | SwiftUIX `frame(height/width:)` shim | `Component/SwiftUIX/…/View.frame+.swift:172-179` |
| `Date.ham_toyyyymmddhhmm` · `LoadStatus` | `yyyy-MM-dd HH:mm`, `locale .current`, `Date.ham_timezone` · `.unload / .loading / .loaded / .loadedWithError` | `Date+Format.swift:54-56,64-70`; `LoadState.swift:10-15` |
| Toast | error red/white · success green/white · normal `secondarySystemBackground`; top banner pinned to window width, HStack 5, icon 36, title 17 semibold, content 12 (2 lines), pad 16 + `safeAreaTop()`, asymmetric opacity + `move(edge:.top)`, 3.3 s | `Toast.swift:23-45`; `ToastUtils.swift:65-100`; `ToastView.swift:37,85-133`; `ToastType+UI.swift:18,28` |
| Type scale used | `.caption2` 11 · `.caption` 12 · `.body` 17 · `.title2` 22 · `.title` 28 | system defaults; call sites cited per screen |
| `Device.tpnsToken` | `"IOS" + xgTokenString`, else `""` | `Device_iOS.swift:8-16` |

#### Android measurements

`iOS` reads `—` where §2a supplies the value; `normative` is what both clients build to. Android measures `body` at 16sp and `headline` at 14sp against spec 17 (`design-system.md` §2.3, open item §7 #1). Scroll container, 42dp nav chrome and safe areas follow §1.3 / §6.
### User center main (个人中心) — platforms: Android
**Purpose:** the authenticated account hub — identity header, one grouped card of six navigation rows, one destructive logout card.
**Layout:**
```text
◀ toolbar 42dp + status bar · 个人中心 16sp Bold · bg ham_bg_b1 @ 0.95f   NavigationView.kt:112,158-164
 ⬤ avatar 108dp + 8 + nickname 24sp (weight forced Normal) → ~132dp   :91,110-114
 ↕32 · pad v32/h16 · bg #F9F9F9 / #000000 · bottom spacer = nav-bar inset + 80dp   :78-79; Spacer.kt:19-31
 ╭ 6 rows (pad v4, spacedBy 12): chip 28 = 24dp icon + 2dp, r6, bg #888888 · 16sp label · chevron 24 #888888 ╮ → ~36dp/row, ~245dp   :117-216
  1dp #EDEEEF between rows only — 5 dividers, none after the last   :128,135,142,149,156
 ╰ ↕32 · 退出登录 16sp #F44336, centred, pad v8 → ~37-40dp ╯   :171-187
```
**Blocks / anatomy:**
- `account header` — 108dp avatar (`SubcomposeAsyncImage`, `crossfade(true)`, `Crop`, `contentDescription = null`) or grey `Person` placeholder (bg #EDEEEF, #888888 glyph, pad 12); 20dp/2dp spinner while loading; nickname below; **not tappable** — `:81-115,254-259`
- `nav card` — one `HamCardView(padding = 0)`: 个人信息 / 登录设备 / 社交账号 / Passkey管理 / 授权应用 / 扫码登录 — `:117-169`
- `logout card` — full-width `HamCardView`, centred destructive label, `HamButton` → `vm.logout()`; **no confirmation dialog** — `:171-187`
**Values:** | Element | iOS | Android | normative |
|---|---|---|---|
| Chrome: bg / toolbar / title / back | — | `ham_bg_b1` #FFF9F9F9 / #000000; toolbar same @ **0.95f**, 42dp; 16sp Bold max 300dp maxLines 1; `ChevronLeft` `ham_blue` #007AFF, start 16dp | `surface.primary`; @ 0.95, 42 + status bar; `bodyBold` 17 `text.primary`; `accent`, 16 inset | `NavigationView.kt:70,90,116,130-164`; `colors.xml:65` |
| Avatar / nickname | — | 108dp `CircleShape`, `Crop`, crossfade, unlabelled; placeholder bg #EDEEEF + #888888 glyph @ pad 12; spinner 20dp ⌀ / 2dp / #888888 · 24sp weight **forced Normal** | 108 circular, labelled; `surface.tertiary` bg, `text.tertiary` glyph; 20 / 2 `text.secondary` · `title` 28 Bold | `:91,94,96-114,254-259`; `Font.kt:18`; `HamLoadingProgressBar.kt:20-30` |
| Card / row geometry / row text | — | r16 / `ham_bg_b2` #FFFFFFFF / pad 0 outer + v2 + h16 inner / **no elevation** · row pad v4, spacedBy 12, ~36dp · plate 28dp, r6, bg #888888, glyph #FFFFFF · label 16sp Normal #FF000000 · chevron 24dp #888888 · divider 1dp `ham_lightGray` #EDEEEF / #0F0E0F, inset 0 | `radius.card` / `surface.secondary` / h16 / none · `space.2` + `space.4` + 36 · `icon.sm` 20 on brand @ 0.1 plate, r6 · `bodyBold` 17 `text.primary` · `icon.xs` 12 `text.secondary` · `surface.tertiary`, pad v4 | `:117-120,197-216`; `Card.kt:51-55`; `Divider.kt:17-24` |
| Logout: label / pad / press | — | 16sp `ham_red` → `R.color.red` **#F44336** / v8 only, centred, ~37-40dp / alpha 1→0.25f while touching | `body` 17 `text.danger` #FF3B30 / #FF453A / v8, ≥44 tap target / 0.25 | `:178-184`; `Color.kt:63-64`; `colors.xml:40,72`; `Button.kt:42-44,76` |
**Strings:** 个人中心 `strings.xml:3`, 个人信息 `:4`, 登录设备 `:5`, 社交账号 `:6`, Passkey管理 `:7`, 扫码登录 `:8`, 退出登录 `:9`, 授权应用 `:53` — all resource-backed, no hardcoded CJK.
**States:** `logged in` — `:88-114` · `logged out` no branch: grey placeholder, nickname `orEmpty()` (zero-height text), all rows and 退出登录 still enabled — `:69-190` · `avatar loading`/`error` spinner then `PersonView()`; no page-level loading or error — `:96-103` · **iOS gap:** iOS mirrors this anatomy; the six-row set plus the destructive card is the parity contract.
### Personal info / edit info (个人信息) — platforms: Android
**Purpose:** edit the profile — replace the avatar through the system photo picker and edit the nickname, committed by a text 保存 button in the nav bar.
**Layout:**
```text
◀ 个人信息 · 保存 (nav trailing, 14sp #007AFF, end margin 8dp)   :65-75; NavigationView.kt:171-182
 ⬤ avatar 108dp, wholly tappable + ⬤36 ✎ badge BottomEnd, bg #888888, pad 6, glyph #FFFFFF   :82-113
 ↕32 · Column pad v32 only, h0 (edge-to-edge) · bottom spacer = nav-bar inset + 80dp   :79-80; Spacer.kt:19-31
 ══ HamDivider 1dp #EDEEEF, full-bleed ══   :118
  8dp │ 昵称 hint #FF888888 (offset y −2dp) · 16sp value · bg #FFFFFFFF, ~40dp   :119-127
 ══ HamDivider 1dp #EDEEEF, full-bleed ══   :128
```
**Blocks / anatomy:**
- `avatar editor` — `HamButton` → `PickVisualMedia(ImageOnly)`; 108dp circular crop, crossfade; `error` → `PersonView()`; **no local preview** before the server round-trip — `:82-115`
- `nickname field` — the only editable field, `HamTextField` bound to `vm.nickname`, `singleLine = true`, no max-length filter, no counter, no clear button, no inline error — `:117-129`
- `save` — nav-bar trailing text button, always enabled, no loading state; no bottom button, no keyboard-action save; back discards edits silently via `LocalNavController` — `:65-75`; `NavigationView.kt:76-81`
**Values:** | Element | iOS | Android | normative |
|---|---|---|---|
| Save label / slot / validation | — | `headline` **14sp** Normal, `ham_blue` #007AFF / nav `Row`, end margin 8dp · blank → 用户名输入有误; > 20 → 用户名不能大于20个字符; gRPC → `showGrpcError`; success → 保存成功 with **no pop-back** | `body` 17 `accent` / trailing action, 16 inset, ≥44 target · same messages, then pop back | `:71-72`; `NavigationView.kt:171-182`; `UserCenterInfoViewModel.kt:130-156` |
| Padding / dividers / avatar | — | v32, **h0** · 1dp #EDEEEF full-bleed above and below the field · 108dp circle + badge `CircleShape` bg #888888, 24dp glyph #FFFFFF, pad 6, `Alignment.BottomEnd` | v32, field block edge-to-edge · `surface.tertiary`, pad v4 · 108 circular, `surface.tertiary` badge, `icon.md` | `:79,87-89,105-112,118,128` |
| Picker limits / upload | — | ≤ 4×1024×1024 bytes; `png\|PNG\|jpg\|JPG\|jpeg\|JPEG`; **silent abort** on breach; `ByteArray(2 * 1024 * 1024)` chunks under `ShowModalLoading` | reject with an inline error toast; chunked upload under a visible modal | `UserCenterInfoViewModel.kt:68,72,83-85,114` |
| Field box, text, hint, caret, cap | — | bg `ham_bg_b2` #FFFFFFFF, pad 8dp, **no radius, no border** — the custom `boxModifier` drops both · text 16sp `ham_text_primary` · hint `ham_text_secondary` Gray #FF888888, `offset(y = -2dp)` · caret `SolidColor(ham_blue #007AFF)` · 20 enforced at save only | `surface.secondary`, `space.3`, r8, 1px `surface.tertiary` · `body` 17 `text.primary` · placeholder `text.tertiary` · caret `accent` · 20 with a counter and an input filter | `:122-124` vs `TextField.kt:42-49`; `TextField.kt:52,67,74-79`; `UserCenterInfoViewModel.kt:139` |
**Strings:** 个人信息 `strings.xml:4`, 保存 `:15`, 昵称 `:16`, 用户名输入有误 `:17`, 用户名不能大于20个字符 `:18`, 保存成功 `:19`. No hardcoded CJK; gRPC bodies are server-supplied — `UserCenterInfoViewModel.kt:101,151`.
**States:** `loaded` — `:86-127` · `logged out` no guard: placeholder avatar, empty hint, save still callable — `:91,99-101` · `uploading` global modal, save has no in-flight state — `UserCenterInfoViewModel.kt:83-85` · `error` silent abort on size/extension, toasts otherwise, no inline field error — `:68-74,133-153` · `success` toast + refresh, screen stays — `:154-155`.
### Login devices (登录设备) — platforms: Android
**Purpose:** list every device holding a session, mark the one in hand, and revoke the others remotely.
**Layout:**
```text
◀ 登录设备 · toolbar 42dp · AnimatedContent(loadState, fadeIn + fadeOut, label = "")   :57-60
 Loading → Box fillMaxSize centre, 20dp ⌀ / 2dp / #888888   :62-66
 Loaded → 16dp pad · HamCardView r16, pad 16 · rows spacedBy 8, no row padding · bottom = inset + 80dp   :68-69; Card.kt:48,53; Spacer.kt:19-31
  ⬤24 设备名 16sp Bold   [下线 chip]  │16dp → ~56dp   :96-128
     device_id (prefix stripped) 12sp · yyyy-MM-dd HH:mm 12sp · 本机 12sp, unstyled   :101-102,124
  ── 1dp #EDEEEF, between rows only ── · Unload / LoadedWithError → else {} → blank page   :74-76,82
 row: spacedBy 8 · icon 24dp untinted · Column w1f · trailing slot · Spacer(16dp)   :95,127
```
**Blocks / anatomy:**
- `device card` — one `HamCardView`, rows 8dp apart, divider after every row but the last; **no row is tappable as a whole** — `:67-81`
- `device row` — icon inferred from the id (`IOS`→`PhoneIphone`, `AND`→`PhoneAndroid`, else `Smartphone`), name, id line, timestamp; **no IP address is displayed** — `:96-103`
- `kick` — `下线` chip, immediate, **no confirmation**; a 20dp spinner replaces the chip while that row revokes; **no pull-to-refresh or retry** — `:106-122`; `UserCenterDeviceViewModel.kt:42-44,62-77`
**Values:** | Element | iOS | Android | normative |
|---|---|---|---|
| Card: pads / radius / bg / elevation | — | 16dp outer + 16dp inner (default) / 16dp / `ham_bg_b2` / **none** | `space.5` / `radius.card` / `surface.secondary` / none | `:68`; `Card.kt:48,51-55` |
| Row: gap / padding / height | — | `spacedBy(8.dp)`, **no vertical padding**, ~56dp | `space.3` / v8 / 56 | `:69,95` |
| Icon / name / id / time | — | 24dp, **no tint** (inherits content colour = black) / 16sp Bold #FF000000 / 12sp `ham_text_primary` / 12sp, `yyyy-MM-dd HH:mm` | `icon.sm` 20 `text.primary` / `bodyBold` 17 / `caption` 12 `text.secondary` / same | `:96-102`; `Font.kt:27-28`; `DateExt.kt:77-79` |
| 下线 chip / spinner / gutter / 本机 | — | 12sp `ham_blue` #007AFF on `ham_blue` @ **0.15f**, r6, pad 4 · 20dp ⌀ / 2dp / #888888 · `Spacer(16.dp)` inside the card · 本机 12sp, no colour / background / padding / border · divider 1dp #EDEEEF, inset 0 | `tint.chip`: `accent` @ 0.10 bg + accent text, r6, pad h6/v4 · 20 / 2 `text.secondary` · `space.5` · `caption` 12 `text.secondary`, no chrome · `surface.tertiary`, pad v4 | `:74-76,107,115-120,124,127` |
**Strings:** 登录设备 `strings.xml:5`, 下线 `:11`, 本机 `:12`, 设备已删除 `:13`. No hardcoded CJK; `user_center_no_data` (暂无数据, `:42`) exists but is unused here.
**States:** `loading` centred spinner — `:62-66` · `loaded` rows with 本机 and 下线 — `:67-81` · `empty` **no empty state**: an empty white card (~32×32dp of padding) — `:68-79` · `error`/`unload` `else -> {}` → blank page, no retry — `:82` · `revoking` chip → spinner; `revoke error` chip restored + gRPC toast, row stays — `:106-108`; `UserCenterDeviceViewModel.kt:69` · **iOS gap:** iOS mirrors this screen; add the empty state, the error/retry state, and a revoke confirmation to both.
### Social accounts (社交账号) — platforms: Android
**Purpose:** show which third-party login providers are bound to the account, and bind more.
**Layout:**
```text
◀ 社交账号 · toolbar 42dp · AnimatedContent(loadState, fadeIn + fadeOut)   :124-128
 Loading → fillMaxSize centre, 20dp ⌀ / 2dp / #FF888888   :130-134
 Loaded → 16dp pad · HamCardView r16, pad 0 · content pad v2/h16, rows 4 apart   :137-140
  ┌────┐ QQ              已登录   ›24 → ~34dp; card = screen − 64dp; 5 rows ≈ 210dp   :217-253
  │ic26│ row pad v4, spacedBy 12, w1f spacer before trailing · 微信 / Github / Apple / 自强 identical   :256-422
  └────┘ 1dp #FFEDEEEF / #FF0F0E0F between rows only, inset 0 · else {} → blank page   :201-203,209
 order = server `CCKVKey.ValidLoginType`, not the enum order; `LoginType.None` renders nothing   :142-143,199
```
**Blocks / anatomy:**
- `provider rows` — one per valid login type in **server order**; `LoginType.None` renders nothing and also skips its divider — `:142-143,199`
- `row` — 26dp icon chip + 16sp name + `Spacer(weight 1f)` + trailing slot, wrapped in `HamButton`; bound → 已登录 16sp #FF888888, no chevron, tap is a no-op though the press alpha still animates — `:220-249`
- `bind transports` — QQ native SDK (no sheet); Wechat / Github / Ziqiang via `FloatViewManager.showFloatView { LoginSheetView }`; **Apple has no bind path**, it only toasts — `UserCenterSocialAccountViewModel.kt:87-235`; `:180-187`
- `unbind` — **none**: `unbind()`'s body is entirely commented out and it is never called — `:237-249`
**Values:** | Element | iOS | Android | normative |
|---|---|---|---|
| Card / content / row / icon geometry | — | 16dp outer, pad 0 inner; content v2 + h16, rows 4 apart; row pad v4, spacedBy 12, ~34dp · icon box `size(26.dp)`, `RoundedCornerShape(6.dp)`, inner pad 2dp — **Ziqiang 4dp** | `space.5` / 0; h16, `space.2`; v4 + `space.4` + 34 · `icon.sm` 20 plate, r6, pad 2 | `:137,139-140,220,222,229-232,268-271,310-313,352-355,391-398` |
| Plates: QQ / Wechat / Github / Apple | — | QQ bg `ham_qq` #FF12B7F5, Wechat bg `ham_wechat` #FF58BE6A, both glyph `ham_white` #FFFFFFFF · Github / Apple bg `ham_text_primary` (#000000 / #FFFFFF), glyph `ham_black`/`ham_white` by `isSystemInDarkTheme()`; Apple always `login_apple_dark` | brand-social plates, white glyph · invert with `text.primary`; the light asset in light mode | `:231,270,306-312,348-354`; `Color.kt:9,12` |
| Ziqiang / name / 已登录 / chevron / spinner / divider / press | — | glyph hardcoded `Color(0xff01579b)`, bg `color.copy(alpha = 0.1f)` · name 16sp Normal `ham_text_primary` · 已登录 16sp overridden to `ham_text_secondary` Gray #FF888888 · unbound chevron 24dp (Material default) #FF888888 · row spinner 20dp ⌀ / 2dp / #FF888888 · divider 1dp #FFEDEEEF / #FF0F0E0F, inset 0 · press alpha 1→0.25f, bound rows still animate · **no `Loading` set** before `awaitWechatLogin()` | `ham_darkBlue` (`Color.kt:67`), brand @ 0.10 · `body` 17 `text.primary` · `body` 17 `text.secondary` · `icon.xs` 12 `text.secondary` · 20 / 2 · `surface.tertiary`, pad v4 · 0.25, bound rows must not animate · set `Loading` before every await | `:202,234,237-249,389,393,397`; `Button.kt:42-44,76`; `UserCenterSocialAccountViewModel.kt:133` vs `:91,170,209` |
**Strings:** 社交账号 `strings.xml:6`, QQ `:47`, 微信 `:48`, Github `:49`, Apple `:50`, 自强 `:51`, 已登录 `:22`, 设备暂不支持该方式登录 `:21`, 登录成功 `:23`. No hardcoded CJK.
**States:** `page loading` centred spinner — `:130-134` · `loaded` — `:136-207` · `error`/`unload` blank page, no retry — `:209` · `empty` card with 0 rows (~4dp) — `:137-144` · `row bound` / `unbound idle` / `binding` / `bind failed` (chevron returns, toast only) — `:236-249` · `Apple` permanently unbindable — `:180-187` · **iOS gap:** iOS mirrors this surface and must additionally ship an unbind affordance with a confirmation.
### Passkey config (Passkey管理) — platforms: Android
**Purpose:** explain Passkey, register a credential on this device, list the account's Passkeys, and delete them.
**Layout:**
```text
◀ Passkey管理 · toolbar 42dp · Column pad h16 + top16, bottom 0, blocks 16 apart   :62-64
 A explanation, 8 apart: 12sp gray intro · 支持设备： 1.GMS 2.鸿蒙 3.ColorOS   :66-102
  需要 [点击这里 #007AFF + underline] 前往密码管理设置。 (#F44336) — only this span is tappable   :83-97
 B add: r12, bg #FFFFFFFF, fillMaxWidth, pad v16, centred → ~56dp · ➕24 + 4dp + 添加Passkey 16sp #007AFF   :114-136
 C 已注册的Passkey 16sp Bold · loading → 20dp spinner (left-aligned) · empty → 暂无数据 12sp gray · else blank   :140-159,170
  ┌ r12 pad16: name 12sp 2-line ellipsis · 16dp · 删除 16sp #F44336 ┐ → ~64dp, rows 8 apart   :181-207
 No enclosing card: the button and each row are independent r12 surfaces on surface.primary   :63-64
```
**Blocks / anatomy:**
- `explanation` — 12sp intro plus a `ClickableText` support list built from 7 concatenated resources; only `点击这里` is actionable (`getStringAnnotations("clickable")`, exact offset, no slack) — `:66-111`
- `add button` — always visible and always enabled, no eligibility gate; taps call `vm.registerPasskey()` — `:114-136`
- `registered list` — header always shown, then `AnimatedContent`; rows are r12 surfaces separated by 8dp and surface contrast only, no divider, no elevation; row = name (12sp, 2 lines, ellipsis), created date, 删除 in `text.danger` with **no minimum tap target, no confirmation, no undo** — `:138-207`
- `eligibility` — resolved off-screen, asynchronously, by `PasskeyProviderManager` (HMS → GMS → OPPO → default); 点击这里 uses the hardcoded `defaultProvider`, so HMS/OPPO devices take the Google path, and it is a **silent no-op below Android 34** — `PasskeyProvider.kt:36-82`; `GooglePasskeyProvider.kt:75-85`
**Values:** | Element | iOS | Android | normative |
|---|---|---|---|
| Root padding / gaps / intro / header / spinner / empty | — | h16 + top16, bottom 0; blocks 16, explanation 8, rows 8 · 12sp `ham_text_secondary` Gray #FF888888, lineHeight unspecified · header 16sp Bold `ham_text_primary` · list spinner 20dp ⌀ / 2dp / #FF888888, **left-aligned** · empty 暂无数据 12sp `ham_text_secondary` | `space.5` / `space.6` / `space.3` / `space.3` · `footnote` 13 `text.secondary` · `bodyBold` 17 `text.primary` · centred 20 / 2 · `footnote` 13 `text.secondary` | `:63-65,68-69,105,139-159,161`; `Font.kt:34-35` |
| Warning / link spans | — | warning `ham_red` → `R.color.red` **#F44336**, both themes · link `ham_blue` #007AFF / #0A84FF + `TextDecoration.Underline`, `pushStringAnnotation("clickable","")` | `text.danger` #FF3B30 / #FF453A · `text.link` = `accent`, underlined, ≥44 tap target | `:83,87-90,95`; `Color.kt:63-64` |
| Add: radius / bg / pad / height / icon | — | `RoundedCornerShape(12.dp)` / `ham_bg_b2` #FFFFFFFF / #FF0F1010 / v16, `Alignment.Center` / ~56dp · `Icons.Rounded.Add` 24dp `ham_blue` + 4dp + 16sp label, both `ham_blue` | `radius.6` / `surface.secondary` / v16 / 56 · `icon.md` 24 `accent` + `space.2` + `body` 17 `accent` | `:118-133` |
| Row: box, name, date, 删除 | — | r12 / `ham_bg_b2` #FFFFFFFF / #FF0F1010 / pad 16dp / ~64dp (the name column has no internal spacing) · name · 12sp **`ham_text_primary`** (same as the name), `yyyy-MM-dd HH:mm:ss` in `%s创建` · `body` 16sp `ham_red` #F44336, no minimum tap target | `caption` 12 `text.primary` · `caption` 12 `text.secondary` · `body` 17 `text.danger`, ≥44 | `:187-207`; `DateExt.kt:81-83`; `strings.xml:43` |
**Strings:** Passkey管理 `strings.xml:7`; `Passkey 是一种安全、便捷的无密码身份验证技术…` `:31`; 支持设备： `:32`; GMS lines `:33-34`; 点击这里 `:35`; 前往密码管理设置。 `:36`; 鸿蒙 `:37`; ColorOS `:38`; 添加Passkey `:40`; 已注册的Passkey `:41`; 暂无数据 `:42`; `%s创建` `:43`; 删除 `:44`; 注册Passkey失败 `:45`. No hardcoded CJK; the Passkey **name** is a raw server value and the date uses a hardcoded pattern, neither localisable — `:187`; `DateExt.kt:82`.
**States:** `list loading` left-aligned spinner — `:150-152` · `empty` 暂无数据 — `:154-159` · `loaded` one row per Passkey — `:160-168` · `list error`/`unload` blank, **not even a toast** — `:170` · `delete success` row removed after refetch; `delete failure` toast and **no refresh** — `UserCenterPasskeyConfigViewModel.kt:74-85` · `register cancelled` reports 注册Passkey失败 though nothing failed (`PasskeyException.cancel` never inspected) — `:62-66`; `PasskeyProvider.kt:16-22` · `ineligible device` UI identical, failure only as a toast — `PasskeyProvider.kt:72-75` · **iOS gap:** iOS mirrors this surface; add a delete confirmation and a cancellation path that stays silent instead of raising an error toast.
### Logout / deactivate account (注销该账号) — platforms: Android
**Purpose:** end the session, and permanently delete the account — only the logout half ships; the deactivate half is unreachable `SyncLogoutView` code.
**Layout:**
```text
SHIPPED, inside 个人中心:
 ╭──────── 退出登录 16sp #F44336, centred ────────╮ r16, pad v8, ~37-40dp, width = screen − 32   :171-187
  tap → vm.logout() immediately — no dialog, no spinner, no toast, no error state   UserCenterMainViewModel.kt:21-24
DEAD CODE, SyncLogoutView.kt (no route):
 ◀ 退出登录 · toolbar 42dp · full-bleed CardButton, RectangleShape (r0), bg ham_bg_b2, M3 Card elev 1dp   :42,119-121
  退出登录  [◌ 25dp] → 40dp, pad h16/v8 · 注销该账号 ▶24 disclosure, rotate 0f→90f · cards abut, no divider   :44-70,110
  ▼ AnimatedVisibility(fade + expand) + animateContentSize → panel ≈ 88dp   :71-75
   │ pad v8/h16 · warning 12sp × ~2 lines · [◌ 25dp] ┌ 确定 ┐ r8, 40dp min, right-aligned, flush   :75-104
```
**Blocks / anatomy:**
- `auto-dismiss` — the stack is popped only by the `LoginStateChange` observer (current route contains `user-center`); the view performs no navigation — `LoginView.kt:105-113`
- `row 1 (dead)` — 退出登录 with a trailing 25dp spinner and **no re-tap guard**, so `logout()` can fire repeatedly — `SyncLogoutView.kt:44-55`; `SyncLogoutViewModel.kt:35-45`
- `deactivate panel (dead)` — warning paragraph plus a right-aligned 确定; spinner and button are mutually exclusive, so double-submit is prevented; no post-success transition — `:71-104`; `SyncLogoutViewModel.kt:47-65`
**Values:** | Element | iOS | Android | normative |
|---|---|---|---|
| Shipped label / container | — | 16sp `ham_red` #F44336 / r16 `ham_bg_b2`, pad v8 only, centred | `body` 17 `text.danger` / `radius.card`, `surface.secondary`, v8, ≥44 tap target | `:174,178-184` |
| CardButton: shape / bg / elev / pad | — | `RectangleShape` (0dp) / `ham_bg_b2` #FFFFFFFF / #ff0f1010 / M3 `Card` default **1.dp** / h16 + v8 → 40dp | `radius.card`, 16 margin / `surface.secondary` / none / h16 + v8 | `:119-125` |
| Row label colour / type | — | `androidx` `Color.Red` **#FFFF0000**, unthemed / no `style` → M3 `bodyLarge` 16sp, 24sp lineHeight, 0.5.sp tracking | `text.danger` / `body` 17 | `:46-50,62`; `HamTheme.kt:87-93` |
| Spinner / disclosure arrow | — | `CircularProgressIndicator` `size(25.dp)` / `Icons.Default.KeyboardArrowRight` 24dp, no tint, `animateFloatAsState` 0f→90f spring | 20 / 2 `text.secondary` / `icon.xs` 12, rotate 90 | `:53,57,65-69,84` |
| Panel / warning / 确定 | — | `Surface(ham_bg_b2)` fillMaxWidth, pad v8 + h16 · warning 12sp Normal `ham_text_primary` · 确定 r8, `Color.Red.copy(alpha = 0.1f)` = #1AFF0000, content `Color.Red`, pad h16 + v2, M3 min height 40dp, label 14sp Bold | `surface.secondary`, `space.5` · `footnote` 13 `text.secondary` · `radius.4`, `tint.subtle` over `surface.secondary`, h16 + v12, height 48, `bodyBold` 17 `text.danger` | `:72-100` |
**Strings:** 退出登录 `strings.xml:9` (nav title `SyncLogoutView.kt:42`, row `:47`, shipped `UserCenterMainView.kt:181`), 注销该账号 `:25`, `注销该账号后，您将无法使用同步功能，您的一切个人信息将被清除，是否继续？` `:26`, 确定 `:27`, 成功 `:28`, 已注销账号 `:29`, 个人中心 `:3`. No hardcoded CJK; the error text is server-supplied (`response.error!!.message`) and the arrow's `contentDescription` is a hardcoded `""` — `SyncLogoutViewModel.kt:42,54`; `SyncLogoutView.kt:67`.
**States:** `idle` both load states `Unload`, panel collapsed — `SyncLogoutViewModel.kt:31-33` · `logout loading` 25dp spinner, label still tappable — `:36`; `SyncLogoutView.kt:52-54` · `deactivate loading` 确定 swapped for the spinner — `:48`; `:83-87` · `error` toast only, back to idle — `:42,54` · `logout success` **none** — no toast, no navigation, no dialog anywhere · `deactivate success` toast 成功 / 已注销账号, panel stays open — `:58-64` · **iOS gap:** iOS mirrors this anatomy; deactivation must ship as a reachable surface on both clients, and neither confirms logout today.
### Authorized apps (授权应用) — platforms: Android
**Purpose:** list the third-party/SSO apps the account has granted access to, with scopes, grant date and a confirmed revoke action; paged 10 at a time.
**Layout:**
```text
◀ 授权应用 · header 36dp (HamNavigationView2) · bg ham_bg_b1 @ 0.75f when list ≠ ∅   :80-83; NavigationView.kt:197,286-291
 LazyColumn 16 apart · pad 16 sides + bottom · top pad = statusBarHeight() + 16 + 36   :166-171
 ┌ ┌────┐ App name 16sp Bold, 1 line ellipsis       ┐   :214-240
 │ │48dp│ 描述 12sp gray, 2 lines ellipsis (gap 2)  │   :243-250
 │ │r10 │ [scope][scope] 11sp gray on 15% (gap 6)  │   :255-272 · FlowRow 4dp/4dp
 │ └────┘ 授权于 2026-04-16 12sp gray (gap 6) · 取消授权 13sp #F44336 (gap 8, 12dp icon gap) │ :276-305 · r16, pad16 + row pad12 = 28 · ~104dp min, ~183dp full
 ↕16 · tail paging spinner [◌ 20dp] · prefetch at the 2nd-to-last item (===)   :176-194
 revoke dialog: ╔ 取消授权 24sp ═ 确定要取消对该应用的授权吗？ 16sp ═ 取消 · 确定(red) ═╗ r28 · tonalElev 6dp · 280–560dp · margin 48dp · scrim black @ 0.32f   :116-148
```
**Blocks / anatomy:**
- `nav` — `HamNavigationView2` with `ignoreTopSafeArea = vm.appList.isNotEmpty()`, so the page flips from stacked to overlay mode the instant the first page arrives — `:80-83`
- `branches` — `Loading` + empty → centred spinner; `LoadedSuccessfully` + empty → 暂无授权应用; non-empty → list. A fourth case (empty + `LoadedWithError`) matches none and falls through to `else -> {}` — `:84-112`
- `app card` — 48dp r10 icon (+placeholder), name, then description, scope chips and grant date, each conditional on non-empty data so height varies; 取消授权 opens the dialog, it does not revoke — `:207-305`
- `paging` — `PAGE_SIZE = 10`, VM guards concurrent runs; only 确定 in the dialog revokes, 取消 and scrim-tap clear `revokeTargetApp` — `AuthorizedAppsViewModel.kt:34,90`; `:116-148`
**Values:** | Element | iOS | Android | normative |
|---|---|---|---|
| Header / list / card / icon | — | 36dp, `ham_bg_b1` @ **0.75f** when non-empty else solid · top pad `statusBarHeight() + 16 + 36`, sides/bottom 16 · `spacedBy(16.dp)`, `userScrollEnabled = false` + `Modifier.bouncy` (resistance 0.2f) · card r16, bg #FFFFFFFF / #FF0F1010, outer 16 + inner Row 12 · icon 48dp r10 `Crop` crossfade, placeholder bg `ham_gray` @ 0.2f + `Icons.Rounded.Apps` 28dp #888888 | 42 (§1.3), `surface.primary` @ 0.95 · `space.5` + 42 + status bar · `space.5`, native overscroll · `radius.card`, `surface.secondary`, `space.5` · `icon.lg` 32 in a 48 plate, `radius.5`, `surface.tertiary` placeholder | `NavigationView.kt:197,288-290`; `:164-171,210,216-220,229,319-327`; `Card.kt:48,53`; `BounceScrollView2.kt:61-73` |
| Name / description / scope chip | — | name 16sp Bold `ham_text_primary`, 1 line ellipsis, 2dp gap · description 12sp `ham_text_secondary` #888888, 2 lines ellipsis · chip **11.sp hardcoded** (`caption2` is the token), `ham_text_secondary`, bg `ham_gray` @ 0.15f, r4, pad h6 + v2, FlowRow 4dp both axes, 6dp above | `bodyBold` 17, `space.1` · `caption` 12 `text.secondary` · `caption2` 11 `text.secondary`, `tint.muted` (`text.secondary` @ 0.10), `radius.2`, h6 + v4 | `:236-269`; `Font.kt:52` |
| Date / revoke / spinners / empty | — | hardcoded `SimpleDateFormat("yyyy-MM-dd", Locale.getDefault())`, 12sp `ham_text_secondary` · 取消授权 **13.sp hardcoded** `ham_red` #F44336 · row spinner 14dp ⌀, 2dp stroke, `ham_red`, 6dp gap · page spinner 20dp ⌀ / 2dp / `ham_gray`, centred · empty 16sp `ham_text_secondary` | locale-aware `yyyy-MM-dd`, `caption` 12 · `footnote` 13 `text.danger` · 20 / 2, `space.2` · 20 / 2 `text.secondary` · `body` 17 | `:87,95-96,191,276-305` |
| Dialog: shape / elev / width / scrim / title / body / buttons | — | M3 default r28 · tonalElevation **6dp** · min 280dp, max 560dp, margin 48dp · scrim `Color.Black` @ 0.32f · stock M3 `colorScheme`, **HamTheme not applied** · title `title` 24sp Bold, body `body` 16sp (M3 `onSurfaceVariant`) · 确定 `ham_red` #F44336, 取消 default M3 `primary` with no explicit colour | r28 · none · 280–560 · black @ 0.5, tap to dismiss · `surface.secondary` / `text.primary` · `title2` 22 Bold, `body` 17 · 确定 `text.danger`, 取消 `text.link` = `accent` | `:117-147`; `HamTheme.kt:55` |
| Divider / bottom safe area | — | none anywhere · **bottom safe area not applied**, only 16dp content padding | none; child-screen bottom rule | `:166,170` |
**Strings:** 授权应用 `strings.xml:53`, 暂无授权应用 `:54`, 取消授权 `:55` (**overloaded** — row label and dialog title), 确定要取消对该应用的授权吗？ `:56`, 已取消授权 `:57`, 授权于 %s `:58`, 确定 `:27`.
**Localisation gaps:** 取消 is `android.R.string.cancel` — the framework string, absent from `strings.xml`, not app-localisable, rendered in the system locale (`AuthorizedAppsView.kt:144`); `网络异常，请稍后重试` is hardcoded at `ToastManager.kt:251` and `:254`; app name, description, scopes and the icon `contentDescription` are raw server data (`:235,245,261,228`), the placeholder glyph's is hardcoded `null` (`:324`).
**States:** `first-page loading` centred 20dp spinner — `:85-89` · `paging` extra 20dp spinner item — `:188-194` · `empty` centred 暂无授权应用, no illustration or retry — `:91-99` · `loaded` variable-height cards — `:101-109` · `error (mid-list)` red toast, list stays, no inline error or retry — `AuthorizedAppsViewModel.kt:76-80` · `error (first load)` **blank page**, recovery only by leaving — `:111` · `revoking` 14dp spinner, row tap suppressed, VM enforces global single-flight — `:291-300`; `AuthorizedAppsViewModel.kt:90` · `revoke error` toast, card stays — `:98-101` · `revoked` row removed + success toast 已取消授权 — `:104-111` · **iOS gap:** iOS mirrors this surface, including the revoke confirmation dialog.
### Route table (Android user center)

| Route string | Screen | Destination | file:line |
|---|---|---|---|
| `user-center` | nested graph, `startDestination = user-center/main` | `navigation { }` | `UserCenterNavGraph.kt:22`; `UserCenterRoute.kt:8` |
| `user-center/main` | User center main (个人中心) | `UserCenterMainView` | `UserCenterNavGraph.kt:23-25`; `UserCenterRoute.kt:10` |
| `user-center/device` | Login devices (登录设备) | `UserCenterDeviceView` — `navController` declared but unused | `:26-28`; `UserCenterRoute.kt:11` |
| `user-center/social-account` | Social accounts (社交账号) | `UserCenterSocialAccountView` | `:29-31`; `UserCenterRoute.kt:12` |
| `user-center/edit-info` | Personal info (个人信息) | `UserCenterInfoView` — no nav arg, back via `LocalNavController` | `:32-34`; `UserCenterRoute.kt:13` |
| `user-center/passkey` | Passkey config (Passkey管理) | `UserCenterPasskeyConfigView` | `:35-37`; `UserCenterRoute.kt:14` |
| `user-center/authorized-apps` | Authorized apps (授权应用) | `AuthorizedAppsView` | `:38-40`; `UserCenterRoute.kt:15` |
| `QrCodeRoute.Scan(SCAN_TO_LOGIN, autoPopBack = false)` | 扫码登录 (typed route, outside the graph) | `QrCodeScanView` | `UserCenterMainView.kt:161-166`; `QrCodeNavGraph.kt:17-22` |
| — | **no logout route exists**; `SyncLogoutView` is unreachable dead code | — | `UserCenterRoute.kt:8-15`; `UserCenterNavGraph.kt:22-41` |

Entries into `user-center/main`: `MyViewUserCenterCard.kt:68`, `CourseCenterBriefCardView.kt:59`. Exit: `LoginView.kt:105-113` pops to `MainRoute` on `LoginStateChange` when the current route contains `user-center`. All six destinations use the string `composable(route)` overload — no arguments, no deep links, no `popUpTo`/`launchSingleTop`/`restoreState`.
### Shared tokens (Android user-center stack)

| Token | Resolved value | file:line |
|---|---|---|
| `HamFontStyle` — title / body / bodyBold / headline / headlineBold / caption / caption2 | 24sp Bold · 16sp Normal `ham_text_primary` · 16sp Bold `ham_text_primary` · 14sp, default weight, **no colour** · 14sp Bold · 12sp `ham_text_primary` · 11sp | `Font.kt:18`, `:24-25`, `:27-28`, `:30`, `:32`, `:34-35`, `:52` |
| `Color.ham_text_primary` / `ham_text_secondary` | #FF000000 / #FFFFFFFF · Compose `Gray` #FF888888 — **hardcoded, no night variant** | `Color.kt:24-26,28-29`; `colors.xml:63`; `values-night/colors.xml:30` |
| `Color.ham_blue` (`accent`) / `ham_red` (`text.danger`) | #007AFF / #0A84FF · `R.color.red` → **#F44336** both themes (correct #FF3B30 unused) | `Color.kt:21-22,63-64`; `colors.xml:79,40,72`; `values-night/colors.xml:46` |
| `Color.ham_gray` / `ham_bg_b1` / `ham_bg_b2` / `ham_lightGray` | #888888 · #FFF9F9F9 / #FF000000 · #FFFFFFFF / #FF0F1010 · #FFEDEEEF / #FF0F0E0F | `Color.kt:69-70,31-33,35-37,39-41`; `colors.xml:24,65,66,68`; `values-night/colors.xml:32-35` |
| `Color.ham_white` / `ham_black` / `ham_qq` / `ham_wechat` | #FFFFFFFF / #FF000000 (fixed, no dark variant) / #FF12B7F5 / #FF58BE6A | `Color.kt:47-51`, `:9`, `:12` |
| `TOOLBAR_HEIGHT` / `statusBarHeight()` / `HamBottomSpacer` | 42.dp · platform `status_bar_height` → dp · nav-bar inset + 80.dp | `ScreenExtension.kt:56`, `:31-41`; `Spacer.kt:19-31` |
| `HamDivider` / `HamCardView` | 1.dp `ham_lightGray` · r16, bg `ham_bg_b2`, default padding 16.dp, no elevation | `Divider.kt:17-24`; `Card.kt:42,48,51-55` |
| `HamButton` press / `HamLoadingProgressBar` / `HamTextField` | alpha 1f → 0.25f, `animateFloatAsState`, `Surface(Transparent, contentColor = ham_blue)` · 20.dp ⌀, 2.dp stroke, `ham_gray` · r8, bg `ham_bg_b2`, 1.dp border `ham_lightGray`, pad 8.dp, caret `ham_blue` | `Button.kt:42-44,76-79`; `HamLoadingProgressBar.kt:20-29`; `TextField.kt:42-49,67` |
| `LoadState` / date formats | `Unload`, `Loading`, `LoadedSuccessfully`, `LoadedWithError` · `yyyy-MM-dd HH:mm` / `yyyy-MM-dd HH:mm:ss` | `LoadState.kt:3-8`; `DateExt.kt:77-79`, `:81-83` |
### Cross-screen findings

- **No logout confirmation on either path** — `UserCenterMainView.kt:171-173` → `UserCenterMainViewModel.kt:21-24`; `SyncLogoutView.kt:44-55`. Normative: logout and every revoke are confirmed by an alert with 确定 / 取消.
- **Account deactivation has no reachable UI** — `SyncLogoutView.kt` is dead code; `UserCenterRoute.kt:8-15` defines no logout constant and `UserCenterNavGraph.kt:22-41` registers no such destination; `strings.xml:25-29` are defined but unread.
- **`ham_red` resolves to Material #F44336, not `text.danger` #FF3B30 / #FF453A, with no dark variant** — `Color.kt:63-64` → `colors.xml:40`; the correct hex sits unused at `colors.xml:72`. Affects 退出登录, 下线, 删除, 取消授权 and the two Passkey warning spans. `ham_text_secondary` is likewise a hardcoded `Gray` with no night value — `Color.kt:28-29`.
- **Every `else -> {}` branch renders a blank page** — device `UserCenterDeviceView.kt:82`, social `:209`, passkey list `:170`, authorized apps first-load error `:111`. Normative: an error state with a message and a retry, plus an empty state, on every list screen.
- **Destructive actions fire without confirmation** — device 下线 `UserCenterDeviceView.kt:110-112`, Passkey 删除 `UserCenterPasskeyConfigView.kt:201-207`.
- **Social accounts cannot be unbound** — `UserCenterSocialAccountViewModel.kt:237-249` is commented out and never called; `UserCenterSocialAccountView.kt:180-187` additionally renders Apple as bindable though it can never bind. **Wechat sets no `Loading` before its web login**, so only its row shows no spinner — `UserCenterSocialAccountViewModel.kt:133` vs `:91,170,209`.
- **Ziqiang's glyph colour is the literal `Color(0xff01579b)`** rather than the identical `ham_darkBlue` (`Color.kt:67`) — `UserCenterSocialAccountView.kt:389,393`; its icon padding is 4dp against 2dp for the other four (`:398`), so it renders smaller. **The nickname field's custom `boxModifier` drops the field's r8 radius and 1dp border** — `UserCenterInfoView.kt:122-124` vs `TextField.kt:42-49`.
- **Save has no disabled or in-flight state and does not pop back after success** — `UserCenterInfoView.kt:65-75`; `UserCenterInfoViewModel.kt:130-156`. The 20-char cap is a save-time toast only (`:139`); the avatar picker aborts silently above 4 MB or outside `png|PNG|jpg|JPG|jpeg|JPEG` (`:68-74`).
- **The main header is not tappable** — no shortcut from avatar or nickname into edit-info — `UserCenterMainView.kt:81-115`. **Non-token font sizes** — 11.sp scope chips and 13.sp 取消授权 (`AuthorizedAppsView.kt:262,303`); `SyncLogoutView.kt:46-50` declares no `style` and inherits M3 `bodyLarge` (16sp / 24sp / 0.5.sp tracking).
- **Untranslatable user-facing text** — 取消 is the framework `android.R.string.cancel` (`AuthorizedAppsView.kt:144`); `网络异常，请稍后重试` is hardcoded at `ToastManager.kt:251,254`; dates use hardcoded patterns (`AuthorizedAppsView.kt:278`, `DateExt.kt:82`); app name, description, scopes and gRPC error bodies are server-supplied.
- **The revoke dialog is not themed** — stock M3 `colorScheme` because `HamTheme` is not applied on the user-center path (`AuthorizedAppsView.kt:117-147`; `HamTheme.kt:55`); its scrim is black @ 0.32f against the spec's 0.5 and 取消 takes M3 `primary` instead of `accent`. Authorized apps also applies no bottom safe area, only 16dp content padding — `AuthorizedAppsView.kt:170` vs `Spacer.kt:19-31`.
- **Three screens use the `@Deprecated("")` `HamNavigationView`** at 42dp (`NavigationView.kt:62-63`; `UserCenterMainView.kt:73`, `UserCenterInfoView.kt:65`, `UserCenterDeviceView.kt:57`) while `AuthorizedAppsView` uses `HamNavigationView2` at 36dp. Normative: one 42dp header.
- **Dead code** — `UserCenterRoutes` sealed class with zero subclasses (`UserCenterRoute.kt:18-20`); `SyncLogoutView.kt`; six duplicate drawables in `feature/user-center/src/main/res/drawable/` (Kotlin reads `core.ui.R.drawable.*`, `UserCenterSocialAccountView.kt:225,264,306,348,391`); `HamSheet`, `WebViewCompose`, `TextButton`, `ham_blue` imported unused at `:71,:61,:27,:64`. `HamNavigationView`'s `if (toolbar != {})` / `if (titleView != {})` always evaluate true (a fresh lambda per call), so an empty right-slot `Row` with an 8dp end margin is always composed — `NavigationView.kt:165,170`.
- **Copy inconsistency** — `Passkey管理` has no space after the Latin word (`strings.xml:7`) while `Passkey 是一种…` has one (`:31`); `取消授权` is overloaded as both a row label and a dialog title (`:55`).


## 10. Auth and sign-in (登录与授权)

Normative: "build it this way". Values measured from source; **iOS is the baseline** where the clients
disagree. Tokens per `docs/design-system.md` §2 — `ham_bg_b1` = `surface.primary`, `ham_bg_b2` =
`surface.secondary`, `ham_text_t1` = `text.primary`, `ham_text_t2` = `text.secondary`, `ham_blue` =
`accent`. `pt` and `dp` are 1:1.

### Login (登录) — platforms: both

**Purpose:** the modal account gate — sign in with Passkey or a cloud-configured provider after ticking
the agreement; dismissing signs nothing in.

```text
┌ root: full-screen overlay, centred card — NOT a sheet, NOT a route ─────────┐
│ scrim black @0.30 (iOS) | @0.75 (Android) · tap/Spacer → dismiss            │
│  ┌ card ─ max width 468 (iOS) | screen−48 (Android) ────────────────────┐   │
│  │ 登录Ham 17/Bold · Spacer · [xmark]     · subtitle 12 text.secondary  │   │
│  │ gap 8 · [☐] 我已阅读并同意 用户使用协议 (link, accent)               │   │
│  │ gap 20 · [🔑 通过Passkey登录] fill · h56 · r30 · gray@0.3             │   │
│  │ gap 20 · 或者选择以下登录方式 12 secondary, centred (if list ≠ ∅)    │   │
│  │ gap 8 · row gap 16: provider circles, centred                        │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│ loading: black @0.75 scrim + 64×64 r16 plate + spinner  (iOS only)          │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Blocks / anatomy:** `root` overlay — iOS `.overlay(alignment:.center)` on `ContentView`, shown/hidden by `.ham_loginShow` / `.ham_loginDismiss` (`ContentView.swift:41`, `LoginView.swift:14,26,32`, `HamAccountManager.swift:56,61`) · Android `Box(fillMaxSize, Center)` above `HamNavHost`, driven by `NotificationChannel.ShowLoginDialog` (`MainView.kt:89`, `LoginView.kt:138-141`, `LoginViewModel.kt:227-232`) — **not** a NavHost destination on either (`LoginViewRoute.kt:9` unregistered) · `scrim` (`LoginView.swift:16-17`, `LoginView.kt:145-149`) · `card` (`LoginView.swift:174-187`, `LoginView.kt:151-156`, `Card.kt:48,53`) · `title + close` (`LoginView.swift:82-93`, `LoginView.kt:159-175`) · `subtitle` cloud-overridable via CCKV `.loginTip` (`LoginView.swift:73,94-96`, `CloudConfigKey.swift:24`) · `agreement row` (`:245-269`, `LoginView.kt:186-197,351-364`) · `Passkey button` (`LoginView.swift:334-352`, `LoginView.kt:202-228`) · `divider label + provider row` (`LoginView.swift:118-150`, `LoginView.kt:235-252`) · `loading` (`LoginView.swift:47-60`; Android uses global `LoadingModalView`, `MainView.kt:90-92`).

| Element | iOS | Android | normative |
|---|---|---|---|
| Presentation | centred overlay, no detents | centred overlay, no detents | **centred overlay** (no sheet, no route) — `ContentView.swift:41`, `MainView.kt:89` |
| Scrim | black **@0.30** | black **@0.75** | **black @0.30** — `LoginView.swift:16`, `LoginView.kt:145` |
| Card width · outer inset | 468 cap (`500−32`) · 16 | screen−48 · **24** | **468 cap · 16** — `LoginView.swift:170-171`, `LoginView.kt:156` |
| Card radius · padding · bg | **24** · 16 · `bg_b2` | 16 · 16 · `bg_b2` | **24** · `space.5` 16 · `surface.secondary` — `LoginView.swift:183`, `Card.kt:48,53` |
| Title · gap · subtitle | 17 Bold · 6 · 12 `t2` | 16 sp Bold · **0** · 12 sp #888888 | **`bodyBold` 17 · `space.3` 8 · `caption` 12 `text.secondary`** — `LoginView.swift:81,84,95-96`, `LoginView.kt:162,175-181` |
| Close · checkbox | `xmark` ~17 `t1` · `square`/`checkmark.square.fill` **12** | `Close` **24** · `CheckBox`/`…OutlineBlank` 16 | **`icon.sm` 20 `text.primary` · 20 `accent` when on, 44 target** — `LoginView.swift:89-90,247,268`, `LoginView.kt:169-172,192-195` |
| Agreement gap · link | `spacing:0` · `Color.blue`, underlined | 4 · `ham_blue`, markdown-regex | **`space.2` 4 · `text.link` underlined, tap target ≥44** — `LoginView.swift:255,258-259`, `LoginView.kt:186,359-362` |
| Agreement→Passkey gap | 8 | 20 (4+12+4) | **`space.5` 16** — `LoginView.swift:107`, `LoginView.kt:183,201` |
| Passkey · label · icon | **56** · **30** · gray@0.3 · icon 18 · label 18 Bold · gap 2 | ~40 · 10 · gray@0.3 · icon 24, y+1 · label 16 Normal · gap 4 | **48 (§3.4) · `radius.4` 8 · `tint.subtle` · `icon.sm` 20 · `bodyBold` 17 · `space.2` 4** — `LoginView.swift:341-347,433`, `LoginView.kt:208-228` |
| Divider label · row gap | 12 gray, `padding(.top,16)` · row 16 | 12 #888, `spacedBy(8)` · row 16 | **`caption` 12 `text.secondary`, 16 above / 8 below · row `space.5` 16, centred** — `LoginView.swift:119-126`, `LoginView.kt:237-252` |
| Loading · unsupported | in-card: scrim 0.75 + 64×64 r16 plate · `该版本暂不支持登录` 12 gray replaces the stack | global `LoadingModalView` · rows collapse, Passkey remains | **global modal spinner · the whole stack is replaced by one `caption` message** — `LoginView.swift:47-60,100-104`, `MainView.kt:90-92`, `LoginView.kt:245` |

**Strings:** `登录Ham` — iOS **HARDCODED** `LoginView.swift:83` (key exists `Localizable.strings:756`); Android `strings.xml:17` `common_login_title` · `登录Ham后，你可以使用校内功能以外的其他功能` — `Localizable.strings:757`, `strings.xml:18` · `我已阅读并同意` `I_AGREE` `Localizable.strings:82`; Android merges it into `common_privacy_agree_text` `strings.xml:19` with markdown `[用户隐私协议](url)` parsed at runtime (`LoginView.kt:341`) · `用户使用协议` / `用户隐私协议` TERM title — `Localizable.strings:83`, `LoginView.kt:351-353` · `或者选择以下登录方式` — iOS **HARDCODED** `LoginView.swift:119`; Android `strings.xml:21` · `通过Passkey登录` — iOS **HARDCODED** `LoginView.swift:343`; Android `strings.xml:20` · `该版本暂不支持登录` — iOS **HARDCODED** `LoginView.swift:101`; **absent on Android** · `请同意《用户使用协议》` toast iOS `LoginVM.swift:56` / `Localizable.strings:420`; Android `请阅读用户隐私协议` `LoginViewModel.kt:213` — **two different strings for the same gate**, norm to iOS.

**States:** `hidden` (`LoginView.swift:14`, `LoginView.kt:88`) · `loading` — spinner covers the gRPC round-trip only (`LoginView.swift:415-419`, `LoginViewModel.kt:170-188`) · `agreement unticked` — every button stays enabled and toasts instead of disabling (`LoginVM.swift:54-56`, `LoginViewModel.kt:207-219`; **no disabled styling anywhere**) · `unsupported` — `validLoginType` empty (`LoginView.swift:100-104`, CCKV `ham_validLoginType` `CloudConfigKey.swift:8`) · `provider web flow` — iOS `SFSafariViewController` sheet (`LoginView.swift:156-169`), Android `LoginSheetView` float view (`LoginViewModel.kt:106-139`) · `error` — gRPC toasts; iOS swallows `GRPCStatus.cancelled` (`ToastUtils.swift:96-98`) and shows nothing for non-GRPC Passkey errors (`LoginVM.swift:263`); Android swallows non-`GrpcException` (`LoginViewModel.kt:149-159`) · `success` — card dismissed with no confirmation (`HamAccountManager.swift:61`, `LoginViewModel.kt:189`).

**Divergence:** scrim 0.30 vs 0.75 and card radius 24 vs 16 — adopt 0.30 / 24. Android has no `该版本暂不支持登录` state and no Apple chip; iOS has three dead sites (`LoginView.swift:222-242,271-311,40-44`) including a WeChat flag no view observes (`LoginVM.swift:145`).

### Provider buttons (第三方登录) — platforms: both

**Purpose:** the cloud-gated third-party sign-in row beneath Passkey.

**Blocks / anatomy:** `Passkey` full-width button above the row, **not** gated by `validLoginType` (`LoginView.swift:109`, `LoginView.kt:202`) · `provider row` — one icon-only circular chip per configured provider, centred, `HStack(spacing:16)` (`LoginView.swift:126-146`, `LoginView.kt:252`) · no chip carries a text label or an accessibility label on either platform.

| # | Provider | Icon | Label | Brand colour | Action |
|---|---|---|---|---|---|
| 1 | Passkey | `person.badge.key.fill` 18 / `login_passkey` 24 | `通过Passkey登录` 18 Bold | `text.primary` on gray@0.3 | `doSocialLogin(.passkey)` → `PasskeySignInHelper.signIn()` `LoginView.swift:336`, `LoginView.kt:203` |
| 2 | QQ | `QQ-2` PNG 24 in 35 circle / `login_qq` 22 in 32 circle | none | **#12B7F5** | QQ SDK → avatar download → `doLogin(.qq)` `LoginView.swift:316`, `LoginView.kt:253-265` |
| 3 | Apple | `applelogo` 24, fg `.systemBackground` | none | circle `.primary` (inverted) | `authorizeByApple` → `doLogin(.apple)` `LoginView.swift:357` — **absent on Android** |
| 4 | WeChat | `login/wechat`, pad 4 → 27 / pad 5 → 22 | none | **#58BE6A** | iOS sets `showWechatLoginView` with no view bound (**dead**, `LoginVM.swift:145`); Android opens the web sheet `LoginView.kt:291-303` |
| 5 | GitHub | `login/github` ↔ `login/github_light`, 35, **no circle** / 32 bare | none | none — full-colour mark | OAuth web sheet → deep link → `doLogin(.github)` `LoginView.swift:375`, `LoginView.kt:277-283` |
| 6 | Ziqiang (自强 / CAS) | `login/ziqiang` SVG, pad 4 → 27 / pad 5 → 22 | none | **#01579B @0.10** / `ham_lightGray` #EDEEEF | CAS web sheet → `doLogin(.ziqiang)` `LoginView.swift:403`, `LoginView.kt:310-322` |

| Element | iOS | Android | normative |
|---|---|---|---|
| Code order · gate | Passkey · QQ · Apple · WeChat · GitHub · Ziqiang · `showOpenType.contains` + `QQManager.installed()` | Passkey · QQ · GitHub · WeChat · Ziqiang · `validLoginType.contains(label)` | **Passkey · QQ · Apple · WeChat · GitHub · Ziqiang, cloud-gated per provider including Passkey** (`LoginView.swift:109-116,127-145`, `LoginView.kt:245-322`) |
| Chip · glyph · pad · theme | **35** · 24 (QQ/Apple), 27 (WeChat/Ziqiang), 35 (GitHub) · `.padding(4)` · GitHub swaps assets, others single asset | **32** · 22, GitHub 32 · `.padding(5)` · GitHub swaps on `isSystemInDarkTheme()` | **35 frame · 24 glyph (GitHub 35) · `space.2` 4 · 44 min target (§3.4) · every chip ships a light and a dark asset** |

**Strings:** none per chip — the only copy is `或者选择以下登录方式`; `Passkey` is deliberately untranslated on both. **States:** `enabled` always (the agreement gates at tap time) · `pressed` — Android subtree alpha 1→0.25 (`Button.kt:76`); iOS none · `hidden` — provider absent from the cloud list, or QQ not installed (`LoginView.swift:112`).

**Divergence:** Android omits Apple although `LoginType.Apple` exists (`HamUserInfoUnbindRequest.kt:21-28`); it ships on iOS and is normative. Ziqiang chip background is #01579B@0.10 on iOS vs `ham_lightGray` on Android — adopt `tint.brand`.

### CAS settings (信息门户设置) — platforms: both

**Purpose:** bind, re-validate, and unbind the university 信息门户 (CAS) session.

```text
┌ nav bar: 信息门户设置 · back ── 42 + status bar (§6) ──────────────────────┐
│ ScrollView · h-pad 16 · cards 8 apart · bg surface.primary                 │
│ ┌ 登录状态 ─── r16 pad16 surface.secondary ─────────────────────────────┐  │
│ │ 登录状态 17/Bold · 信息门户的登录状态 12 secondary · 8                 │  │
│ │ ◌ spinner | 已登录 12 | 登录状态失效 12 text.danger | 未登录 12 · 8    │  │
│ │ 重新登录   body, text.link, no background                             │  │
│ └───────────────────────────────────────────────────────────────────────┘  │
│ ┌ 其它设置 ── bound only ──────────────────────────────────────────────┐   │
│ │ 其它设置 · 8 · 退出登录   body, text.danger, no confirm              │   │
│ └──────────────────────────────────────────────────────────────────────┘   │
└ CAS sheet overlays at the 85% detent ───────────────────────────────────────┘
```

**Blocks / anatomy:** `card 1 登录状态` — title, subtitle, status slot, action (`CasSettingView.swift:21-68`, `CasSettingMainView.kt:39-74`) · `card 2 其它设置` — 退出登录 (`CasSettingView.swift:71-89`, `CasSettingMainView.kt:77-84`) · `login sheet` — `NavigationView { CasMobileLoginView }` with a leading xmark (`CasSettingView.swift:98-113`) / 85 % `HamSheet` with 信息门户 + 关闭 (`CasSettingView.kt:46-85`) · `validation on appear` — `fastLogin()` against `bus.whu.edu.cn`, valid iff the redirect location contains `ticket` (`CasSettingView.swift:121-148`, `CasContext.kt:42-56`) · **no native credential fields exist on this screen on either platform.**

| Element | iOS | Android | normative |
|---|---|---|---|
| Nav bar | **title BLANK** — no `.navigationTitle` anywhere; system 44 | `信息门户设置` 16 Bold · 36 + status bar · back → `popBackStack()` | **`信息门户设置`, `bodyBold` 17 · 42 + status bar (§6) · back** — `Route.swift:197-198`, `CasSettingMainView.kt:32`, `NavigationView.kt:197,326` |
| Screen bg · h-pad · card gap · card | `bg_b1` · 16 · VStack default · r16 pad16 `bg_b2` | `bg_b1` · 16 · 8 · r16 pad16 `bg_b2` | **`surface.primary` · `space.5` 16 · `space.3` 8 · Card §3.1** — `CasSettingView.swift:19-20,93,97`, `CasSettingMainView.kt:33-38`, `Card.kt:48,53` |
| Card title · subtitle · gap | 17 `body.bold()` · 12, colour unset → primary · 8 | 16 Bold · 12 #888888 · 8 (`Card.kt:85`) | **`bodyBold` 17 · `caption` 12 `text.secondary` · `space.3` 8** — `CasSettingView.swift:23-29`, `Card.kt:81-85` |
| Status text · spinner | 12; valid `Color.primary`, invalid stock `.red` #FF0000; stock `ProgressView` | 12; valid `text_primary`, invalid `ham_red` #F44336; `CircularProgressIndicator` 20 / stroke 2 `ham_gray` | **`caption` 12; valid `text.primary`, invalid `text.danger` #FF3B30; spinner 20 / 2 `text.secondary`** — `CasSettingView.swift:33,37,40-41`, `CasSettingMainView.kt:50-60` |
| Status animation | `withAnimation` 0.35 | `AnimatedContent` default, pad bottom 8 | **`AnimatedContent`, pad bottom 8** — `CasSettingView.swift:115,124,141`, `CasSettingMainView.kt:44-47` |
| Action · logout | `重新登录` 17 accent · `退出登录` stock `.red` | `重新登录` / `登录` 16 `ham_blue` · `退出登录` 16 `ham_red` | **action `body` 17 `text.link`, label always `重新登录`; logout `body` 17 `text.danger`** — `CasSettingView.swift:46-50,86`, `CasSettingMainView.kt:65-74,78-84` |
| Unbound · card 2 | 未登录 12; card 2 bound only (`enable`, `:70`) | status slot **omitted**, label 登录; card 2 always | **未登录 12 `text.primary`, label `重新登录`; card 2 bound only** — `CasSettingView.swift:54-56,70`, `CasSettingMainView.kt:43,77` |

**Strings:** `信息门户设置` nav title — Android `cas/strings.xml:4`, **iOS missing** · `登录状态` — iOS **HARDCODED** `CasSettingView.swift:23` (`Localizable.strings:762`), Android `cas/strings.xml:5` · `信息门户的登录状态` — HARDCODED `:25` (`:557`), Android `:6` · `已登录` HARDCODED `:36` (`:643`) / `:7` · `登录状态失效` HARDCODED `:39` (`:763`) / `:8` · `未登录` HARDCODED `:55` (`:714`) — **no Android equivalent, Android omits the slot** · `重新登录` HARDCODED twice `:49,:62` (`:370` and duplicate `:408`) / `:9` · `其它设置` HARDCODED `:73` (`:570`) / `:11` · `退出登录` localized `:85` (`Localizable.strings:227`) / `:12` · `登录成功` toast localized `:119` (`Localizable.strings:196`, duplicate `:421`) — **Android has no success toast** · iOS entry-point copy 管理信息门户设置 / 登录信息门户 / 使用校内服务的前提 HARDCODED `MyUserCenterCard.swift:48,52`.

**States:** `validating` — spinner, action still tappable (`CasSettingView.swift:32-33`, `CasSettingMainView.kt:50`) · `bound + valid` → 已登录 · `bound + expired` → 登录状态失效 in `text.danger`, no auto-open of the sheet · `unbound` → 未登录, card 2 hidden · `validation threw` — iOS catches only `ResponseError`, so any other throw pins the spinner **forever** (`CasSettingView.swift:137-143`); Android treats every exception as invalid (`CasContext.kt:49-55`) · `login success` — `.ham_casLoginSuccess` flips to 已登录 + toast 登录成功 (`:114-120`); Android refreshes then hides (`CasSettingView.kt:81-84`) · `logout` — one tap wipes the credential with **no confirmation** on either (`:79-83`, `CasSettingMainView.kt:78-84`).

**Divergence:** iOS nav title is blank and 8 of 10 strings are hardcoded (`Text("中文")` performs no lookup); Android omits the unbound status slot and labels the action 登录. Adopt Android's title + iOS's status/label behaviour, and wrap all copy in `String(localized:)` — the keys already exist.

### CAS web login (信息门户登录 / WebView) — platforms: both

**Purpose:** the actual portal authentication surface — a full-bleed remote CAS page whose cookies and typed credentials are harvested for the bus, library, sport, pay and course modules.

**Blocks / anatomy:** `A/B branch` — remote CCKV flag `rnConfig["enable"]["RNCasMobileLogin"]` selects `RNCasLoginView` (RN module `RNCasMobileLogin`) else the `WKWebView` / `WebViewCompose` path; missing config defaults to WebKit (`CasMobileLoginView.swift:16-27`, `CasMobileLoginView.kt:43-58`) · `nav bar` — owned by the presenter (`CasSettingView.swift:99-111`) / in-view with title 信息门户 + 关闭 (`CasMobileLoginView.kt:48-55`, `CasSettingView.kt:60-79`) · `full-bleed web content`, `.ignoresSafeArea(.bottom)` (`CasMobileLoginView.swift:30`) · `injected JS` — binds `#mobileUsername`, `#mobilePassword`, `#load`, removes `.social-aut-login`, sets the 学号 placeholder and the 13-character rule (`CasJsInjector.swift:14,16-30`, `CasMobileLoginWebView.kt:57-83`) · `result capture` — cookie sweep then `successCallback` / `NotificationChannel.ReactNativeCasMobileRequestSuccess` (`CasMobileLoginWebView.swift:97-129`, `CasMobileLoginViewModel.kt:47-61`).

| Element | iOS | Android | normative |
|---|---|---|---|
| Presentation · chrome | `.sheet` default `.large`, or pushed from `IntroView` · system radius ≈12, no detents declared | 85 % sheet, `BackHandler` hides it · 24 top corners, black @0.5 × progress | **sheet at the 85 % detent (§3.7) + drag handle 36×5 · top radius 24 · scrim black @0.5** — `CasSettingView.swift:98`, `CasSettingView.kt:36`, `Sheet.kt:65,92,110-115` |
| Nav title · dismiss | **none** · leading `xmark`, no tint | 信息门户 16 Bold · trailing `关闭` 14 `ham_blue` | **信息门户, `bodyBold` 17 · trailing `关闭`, `body` 17 `text.link`, 44 target** — `CasMobileLoginView.kt:49`, `CasSettingView.swift:102-110`, `CasSettingView.kt:73-77` |
| Login URL | `cas.whu.edu.cn/authserver/login?service=…mobile/callback?appId=985180443&login_type=mobileLogin` | identical, **HARDCODED** `CasMobileLoginView.kt:63` | **identical; move to a build-config constant** — `CasMobileLoginWebView.swift:29`, `CasMobileLoginView.kt:63` |
| Cookies · cache · bridge | `CASTGC` + `JSESSIONID` on `cas.whu.edu.cn` · `removeAllCachedResponses` + cookie sweep, `.nonPersistent()` · `jsMessenger` at `atDocumentStart` | non-blank cookie containing both · `CookieManager.removeAllCookies {}` · `jsMessenger` on `onPageFinished` | **`CASTGC` + `JSESSIONID` · clear cookies on every open · bridge `jsMessenger`** — `CasMobileLoginWebView.swift:43-44,49,80-113`, `CasJsInjector.swift:38-44`, `CasMobileLoginView.kt:77-78`, `CasMobileLoginWebView.kt:92-96` |
| Credentials | `username`/`password` empty unless the `#load` tap fires; plaintext password persisted and pushed to the watch | posted to the bridge on `change`; plaintext persisted | **stop persisting the plaintext password** — `CasMobileLoginView.swift:37,39`, `CasJsInjector.swift:61`, `CasMobileLoginViewModel.kt:39-45` |
| Success · failure · progress | cookie → `fastLoginCookie` → notify → `ZSGXService.login/updateUserInfo` → refresh → dismiss · `ToastUtils.showError`, **sheet stays open**, non-`ResponseError` escapes silently · **no indicator** | `saveLoginResult` → LSKV + `addAccount(CAS)` + `initUserInfo` → refresh → hide · no native handling · **no indicator** | **validate then dismiss; refresh state on every dismissal, not only on success; toast + inline retry; add a progress bar (§5.4)** — `CasMobileLoginView.swift:33-52`, `CasSettingView.kt:81-84`, `WebViewCompose.kt:80-85` |

**Strings:** iOS renders **zero** strings — all copy comes from the remote page; the intro entry point hardcodes 从信息门户验证 and 将进入武汉大学信息门户网页验证你的身份 (`IntroView.swift:159-160`). Android: `信息门户` `cas/strings.xml:3` · `关闭` `:13` · `学号` `:25` and `请输入正确的学号` `:26` (both injected into the page). iOS additionally hardcodes `解析文档失败` and `请重新登录信息门户` as `ResponseError.message` (`CasRequestHelper.swift:83,109`).

**States:** `loading` — no native indicator on either · `credentials entered` — mirrored over the bridge · `login tapped` → `#load` hook validates · `wrong password / captcha / locked` — rendered by the CAS page; **no native copy or retry** · `cookies obtained` → result emitted, final redirect cancelled (`CasMobileLoginWebView.swift:118-125`) · `service chain failed` → toast, sheet stays · `user dismisses` → no notification, state not refreshed.

**Divergence:** iOS shows a leading xmark with a blank title at full height and has no 学号/请输入正确的学号 validation; Android titles the sheet, adds `关闭` and a `BackHandler`, and blocks non-13-character IDs before submit. Adopt the Android shell shape.

### SSO authorization sheet (授权请求) — platforms: both

**Purpose:** the OAuth2/OIDC consent sheet shown when a third-party app deep-links in — fetch the client's metadata and scope list, let the user grant optional scopes, optionally remember the choice, mint an auth code, then open `redirect_uri`.

```text
┌ sheet: 85% detent · r24 top · scrim black@0.5 · drag handle 36×5 (§3.7) ──┐
│ NAV 授权请求 · 17/Bold centred, no back · ✕ trailing                      │
│ ┌ ScrollView ─ h-pad 16 (A only) ───────────────────────────────────────┐ │
│ │ 32 · app icon 64×64 r14 (placeholder gray@0.15 + Apps 32) · 8         │ │
│ │ app name title3 20/Bold · 8 · description 12 caption ≤5 lines         │ │
│ │ 32 · 请求以下权限： 12 text.secondary, leading · 4                    │ │
│ │ ┌ scope card: gray@0.08, r12, pad 12, rows 8 apart ────────────────┐  │ │
│ │ │ ☑ (必选) desc │ ☑ (已授权) desc │ ☐ desc        check 20        │  │ │
│ │ └─────────────────────────────────────────────────────────────────┘  │ │
│ │ 200 spacer · 1px scroll marker                                       │ │
│ ├── gradient bg_b1 0→.8→.9→1, 32 tall ─────────────────────────────────┤ │
│ │ 下次自动授权                     [ Switch ]       caption, pad 20/6   │ │
│ │ ┌ 授权 ── fill · r12 · accent@0.15 · h48 (§3.4) ──────────────────┐   │ │
│ │ 16 · 取消   body, text.link, no background    ·   bottom pad 32    │   │ │
│ └────────────────────────────────────────────────────────────────────┘ │
└ B: no scope list, no switch — check 48 + 你已授权过该应用，是否继续授权？ ──┘
  C: no content — spinner only, self-dismisses
```

**Blocks / anatomy:** `nav bar` — inline title 授权请求 + close (`SSOAuthorizationSheet.swift:60-71`, `SSOAuthorizationSheet.kt:129-131`) · `app info header` — 64×64 r14 icon, name, conditional description (`:287-326`, `:380-419`) · `permissions label` (`:143-147`, `:236-241`) · `scope card` (`:152-162`, `:243-259`) · `200 spacer + scroll marker` (`:165-180`, `:262-263`) · `gradient overlay` (`:187-197`, `:267-282`) · `auto-authorize toggle` (`:201-204`, `:284-287`) · `授权 / 取消` (`:436-459`, `:527-559`) · `loading` (`:92-101`, `:186-193`).

**The three variants — exact selection logic** (`SSOAuthorizationViewModel.swift:262-277`, `SSOAuthorizationViewModel.kt:232-248`):

| # | Variant | Selection condition | Renders |
|---|---|---|---|
| **A** | Full consent `.showConsent` / `ShowConsent` | `can_auto_authorize == false` **OR** `pref.autoAuthorizeEnabled == false` | icon + name + description, 请求以下权限：, selectable scope list, 下次自动授权 switch, 授权 / 取消 |
| **B** | Simplified consent `.showSimplifiedConsent` | auto-auth flag **AND** `autoAuthorizeEnabled` **AND** last authorize **> 24 h ago** (or never) | icon + name + description, check icon 48, 你已授权过该应用，是否继续授权？, 授权 / 取消 — **no scope list, no switch** |
| **C** | Silent auto-authorize | auto-auth flag **AND** `autoAuthorizeEnabled` **AND** `isWithin24Hours(appId)` | **nothing** — `authorize()` fires programmatically (`VM:267`, `kt:238`), sheet self-dismisses |

Two stacked gates: the server flag `can_auto_authorize` and the client preference. A remembered preference can only **downgrade** A to B/C, never upgrade.

**Scope rows:**

| Row | Tag | Selectable | Text colour | Checkbox |
|---|---|---|---|---|
| Mandatory (`isRequired`) | `(必选)` | **locked on** — cannot be unticked | `text.secondary` | `accent` @0.6, disabled |
| Already granted | `(已授权)` | **locked on** | `text.secondary` | `accent` @0.6, disabled |
| Optional | none | user-toggleable, starts **selected** | `text.primary` | `accent` when on, `text.secondary` when off |

`isLocked = isRequired || alreadyGranted` (`SSOAuthorizationSheet.swift:349`, `SSOAuthorizationSheet.kt:449`); `isRequired` wins the tag. Deselection is blocked in three places (`SSOAuthorizationViewModel.swift:192,194`, `SSOAuthorizationSheet.swift:353,390`). Every scope starts `isSelected = true` (`VM:257`, `kt:227`). **Only the description renders — the raw scope key (`openid`, `profile`) is never shown, and there is no per-scope icon.**

**Account selector:** **does not ship.** Neither client offers one — iOS has zero matches for `accountselector|accountswitch|switchaccount` and acts on the single `accountContext` (`SSOAuthorizationViewModel.swift:58,101-105`); Android likewise — the sealed state has no account case (`SSOAuthorizationViewModel.kt:66-75`) and no switcher row exists in either consent variant (`SSOAuthorizationSheet.kt:196-366`). Normative: the sheet authorises the **currently signed-in account**, with no switcher. When no account is signed in, prompt login and resume — Android's `WaitingForLogin` state (`SSOAuthorizationViewModel.kt:140-146,162-171`) is normative; iOS dismisses silently, which is a defect.

**Action-button enable logic:** `canAuthorize = hasSelectedScopes && hasScrolledToBottom` (`SSOAuthorizationSheet.swift:114-125`, `SSOAuthorizationSheet.kt:207-213`). `hasSelectedScopes` is true from the first frame (all scopes default on), so **the scroll gate is the real gate**. Variant B hardcodes `authorizeEnabled = true` (`:272`, `kt:360`). 取消 is always enabled.

**Remember-my-choice persistence:**

| Aspect | iOS | Android | normative |
|---|---|---|---|
| Model | `SSOAuthPreference { autoAuthorizeEnabled = false; lastAuthorizeTime: Date? }` | `SSOAuthPreference(appId, autoAuthorizeEnabled = false, lastAuthorizeTime: Instant?)` | **identical shape** — `SSOAuthPreferenceManager.swift:11-14`, `kt:13-17` |
| Store | one JSON dict keyed by `client_id`, `UserDefaults(AppGroup)` key `sso_authPreferences` | LSKV map keyed by `appId`, `LocalStorage.Key.SSOAuthPref` | **one JSON map keyed by client id** — `swift:27-39`, `LocalStorageKey.swift:117`, `kt:22-25` |
| Decode failure | yields `[:]` → Variant A | default() → Variant A | **degrade to Variant A, never crash** — `swift:28-31`, `kt:27-28` |
| Toggle write | **immediately on tap**, before any authorize | **immediately on tap** (`kt:356-359`) | **persist on tap** — flipping then cancelling keeps the value — `swift:45-51`, `VM:199-202` |
| Timestamp write | after a successful `getAuthCode` | always on success (`kt:305`) | **stamp only on success** — `swift:53-59`, `VM:148` |
| 24 h window | `timeIntervalSince < 86400`, false when nil | `(now − last) < 24.hours`, false when nil | **strict `< 86 400 s`, nil = outside** — `swift:67-70`, `kt:44-47` |
| Forget | `removePreference(_:)` defined, **never called** | `removePreference(appId)` defined, **never called** | **ship a per-app revoke in authorized-apps** — `swift:61-65`, `kt:40-42` |

| Element | iOS | Android | normative |
|---|---|---|---|
| Detent | default `.large`, no `presentationDetents` anywhere | **85 %**, single detent, 100 dp drag slack | **85 % (§3.7)** — `Sheet.kt:205,213` |
| Radius · scrim · bg · handle | system ≈12 · none declared · `bg_b1` · **no handle** (single detent) | 24 top · black @0.5 × progress · `bg_b1` · **no handle** — invisible 32 dp grab zone | **24 top · black @0.5 · `surface.primary` · 36×5 pill `text.secondary`@0.4 in a 32-tall header (§3.7)** — `Sheet.kt:65,92,116,130-144` |
| Nav bar · close | ~44 · `授权请求` inline · `xmark` `t1` trailing | 36, status-bar inset suppressed · 16 Bold · no back button, no `BackHandler` | **42 + inset (§6) · `bodyBold` 17 · `icon.sm` 20 `text.primary` close; back dismisses** — `SSOAuthorizationSheet.swift:60-68`, `kt:129-130`, `NavigationView.kt:197` |
| App icon · placeholder | 64×64 r**14** · gray@0.15 + `app.fill` 32 | 64×64 r14 · gray@0.15 + `Apps` 32 | **64×64 · 14 · `text.secondary` @0.15 + 32 glyph** — `swift:303-304,334-338`, `kt:389-390,430-438` |
| App name · header rhythm | `.title3.bold()` **20 semibold** · 8/8/8 · top spacer 32 | `title` **24 Bold** · 8/8/8 · top pad 32, block spacing 32 | **`title3` 20 / Bold `lineLimit(1)` · 8/8/8 · top 32** — `swift:131,140,293,311-313`, `kt:224,226,401-407` |
| Description · permissions label | 12 `t2`, centred, ≤5 lines, h-pad 20 · label 12 `t2` leading h-pad 20 | 12 #888, centred, ≤5, conditional · label 12 `text_secondary` h-pad 16 | **`caption` 12 `text.secondary`, centred, `maxLines 5`; label leading, h-pad 16** — `swift:143-147,318-322`, `kt:236-241,412-419` |
| Scope card | gray @**0.08**, r12, pad 12, rows 8, no divider | gray @0.08, r12, pad 12, rows 8, no divider | **`text.secondary` @0.08, `radius.6` 12, pad 12, gap `space.3` 8, no divider** — `swift:159-161`, `kt:247-251` |
| Checkbox · scope text | 20 · locked `accent`@0.6 · on `accent` · off `t2` · 12 ≤2 lines, `t2` locked / `t1` unlocked | 20 · disabled-checked `accent`@0.6 · on `accent` · off `gray`[@0.6] · 12 ≤2 lines, same split | **`icon.md` 20 · `accent`@0.6 locked · `accent` on · `text.secondary` off; `caption` 12, `maxLines 2` + ellipsis** — `swift:359-363,374-383`, `kt:466-471,477-484` |
| Tag parens | ASCII `( )`, **not localized** | ASCII `( )`, **not localized** | **full-width `（）`, localized** — `swift:369-371`, `kt:451-452` |
| Bottom spacer · marker | 200 · 1 px, `minY < screenHeight`, **one-way latch** | 200 · `maxValue == 0 \|\| value >= maxValue` | **200 · recompute each frame (no latch) + a visible hint on the disabled button** — `swift:165-180`, `kt:210,263` |
| Gradient · switch | 32, `bg_b1` 0→.8→.9→1 · native `Toggle`, hidden label | 32, same stops · `HamSwitch`, checked track `ham_green` **#4CAF50** | **32, `surface.primary` 0→.8→.9→1 · native switch (§3.6)** — `swift:188-197,406-410`, `kt:271-278,494-509` |
| 授权 button · label | r12, v-pad 12, fill, `accent`@0.15 / off gray@0.1 · 17 semibold `accent`/gray | r12, v-pad 12, fill, `ham_blue`@0.15 / gray@0.1 · 16 Bold `ham_blue`/`ham_gray` | **48 · `radius.6` 12 · `tint.subtle` → `tint.muted` · `bodyBold` 17** — `swift:438-446`, `kt:532-548` |
| 取消 · action group | 17 `accent`, no bg · v-pad 12, h-pad 20, gap 16, bottom 32 | 16 `ham_blue`, no bg · identical | **`body` 17 `text.link` · gap `space.5` 16, h-pad 20, bottom 32** — `swift:213,454-459`, `kt:281,521-523,553-559` |
| Variant B | `checkmark.circle.fill` 48, stock `.green` · message 17 `t1`, h-pad 32, gap 12 | `CheckCircle` 48, `ham_green` #4CAF50 · message 16, h-pad 32, gap 12 | **48 `feedback.success` #34C759 / #30D158 · `body` 17 centred, h-pad 32, gap 12** — `swift:240-250`, `kt:322-337` |
| Spinner · redirect · retry | stock `ProgressView` · allow-list `http`/`https`/`ham` · ≤1 retry on `PERMISSION_DENIED` + `13008` | `CircularProgressIndicator` 40/4 `ham_blue` · same allow-list · ≤1, same predicate | **centred spinner `accent` · allow-list `http`/`https`/`ham` else toast 无法打开回调链接 · ≤1 retry** — `swift:96`, `VM:159-164,291-307`, `kt:187-192,317-322,364-384` |

**Strings:** `授权请求` `sso_auth_title` — `Localizable.strings:903` / `auth/strings.xml:25` · `授权` `:904` / `:26` · `取消` `:905` / `:27` · `下次自动授权` `:906` / `:28` · `已授权` `:907` / `:29` · `必选` `:908` / `:30` · `你已授权过该应用，是否继续授权？` `:909` / `:31` · `请求以下权限：` `:914` / `:36` · `网络错误，请稍后重试` `:912` / `:34` · `无法打开回调链接` `:913` / `:35`. **Unused on both:** `无法识别该应用` (`:910` / `:32`), `请先登录后再进行授权` (`:911` / `:33`), `请至少选择一项权限` (`:915` / `:37`).

**States:** `idle` / `redirecting` — nothing rendered (`swift:56-57`, `kt:177-179`) · `loading` / `authorizing` — spinner only, and **the fetch, silent-authorize and mint phases are visually identical** (`swift:35-36`, `kt:166-168`) · `waitingForLogin` — **Android only**; shows the global login dialog and resumes (`kt:140-146`) · `not scrolled to bottom` — 授权 disabled with no explanatory hint · `content shorter than viewport` — Android enables immediately; iOS's latch never resets (`swift:121,178`, `kt:210`) · `gRPC error` — toast, sheet dismisses, no retry (`VM:279-284`, `kt:207-212`) · `missing nonce` — toast, sheet stays (`VM:123-127`, `kt:266-270`) · `bad redirect scheme` — toast 无法打开回调链接, dismiss · `success` — `updateLastAuthorizeTime`, open `redirect_uri`, dismiss; **Android's 取消 only hides the sheet and never notifies the third party** (`kt:150-152`).

**Divergence:** iOS has no scroll-progress source (a one-way latch), no `WaitingForLogin`, and dismisses silently when not signed in; Android has no visible drag handle on any sheet and uses #4CAF50 rather than `feedback.success`. Adopt Android's scroll computation + login gate and iOS's detent-free defaults are to be replaced by the 85 % detent in both.

### QR code login (二维码登录) — platforms: both

**Purpose:** the confirmation surface the phone shows after the user scans a desktop QR code.

```text
┌ nav bar 42 + inset · bg surface.primary @0.95 · [<]16 · 二维码登录 centred ┐
│ Column · h-pad 16 · spacedBy 8 · centred · non-scrolling                  │
│   64 spacer                                                               │
│   [confirm] Computer 72, no bg, no tint  |  [success] Done 48 in a 64     │
│                                             circle, accent, pad 8         │
│   8                                                                       │
│   [confirm] 确定在电脑上登录Ham吗？ (server message || fallback) 16     │
│   [success] 登录成功  24 Bold                                             │
│   8 + 32 (inside the button) = 40                                         │
│   [ 确认登录 | 返回 ]  fill · r12 · accent@0.1 · pad-v 12 · 16 Bold      │
│ bottom spacer = nav-bars inset + 80                                       │
└ ticket arrives as ham://qrcode-login?ticket=… ────────────────────────────┘
```

**Blocks / anatomy:** `toolbar` — `HamNavigationView(title = "二维码登录", isBounceScroll = false)`, deprecated overload (`QrCodeLoginView.kt:49`, `NavigationView.kt:62-63`) · `top spacer` 64 (`:54,:83`) · `state icon` (`:55-63`, `:84-86`) · `message` — `loginInfoResponse.message.ifEmpty { 确定在电脑上登录Ham吗？ }` (`:87-90`) · `primary button` — 返回 (success) or 确认登录 (confirm), mutually exclusive (`:65-77`, `:93-105`) · `bottom spacer` (`NavigationView.kt:108`, `Spacer.kt:19-31`).

| Element | iOS | Android | normative |
|---|---|---|---|
| Route · toolbar · nav title | `qrCodeLogin` — **not audited in this range** | `qrcode-login?ticket={ticket}` · 42 + inset, `bg_b1` @**0.95** · `二维码登录` 16 Bold, max 300 dp, 1 line | **pushed route with a `ticket` arg · 42 + inset (§6) · `bodyBold` 17** — `LoginNavGraph.kt:18-23`, `NavigationView.kt:97,112,116,160-163` |
| Content h-pad · gap · top spacer | unmeasured | 16 · 8 · **64** | **`space.5` 16 · `space.3` 8 · `space.7` 32 above the icon (64 only with a hero)** — `QrCodeLoginView.kt:51-54,80-83` |
| Success icon · confirm icon | unmeasured | `Done` 48 in a 64 circle `ham_blue`, tint raw `White` · `Computer` **72**, **no tint** → black on black in dark mode | **`icon.xl` 64 in `accent`, white glyph · `icon.hero` 72 in `text.primary`** — `:55-63`, `:84-86` |
| Success title · message | unmeasured | 24 Bold · 16 Normal, server `message` else fallback | **`title2` 22 / Bold · `body` 17** — `:64`, `:87-90` |
| Button · top gap | unmeasured | fill · r12 · **accent@0.1** · pad-v 12 · 16 Bold · gap 8 + 32 **inside** the button | **48 · `radius.6` 12 · `tint.subtle` · `bodyBold` 17 · gap `space.7` 32 outside the target** — `:69-75,97-103` |
| Bottom spacer | unmeasured | nav-bars inset + 80 | **nav inset + 80 (§1.3)** — `NavigationView.kt:108`, `Spacer.kt:19-31` |

**Strings:** `二维码登录` `auth/strings.xml:7` · `登录成功` `:8` · `返回` `:9` · `确认登录` `:10` · `确定在电脑上登录Ham吗？` `:11` (fallback only). iOS `QrCodeLoginView.swift:28,30,54` are hardcoded (Part 01 §0.4). The server-supplied `message` is rendered verbatim with no localisation (`qr_login.proto:41`).

**States:** `initial` — `loginInfoResponse == null` renders an **empty page**, no spinner (`QrCodeLoginView.kt:50,79`) · `confirm requested` — `REQUEST_CONFIRM` (2) renders the computer icon + 确认登录 (`:92-105`) · `confirming` — **no in-flight state**, the button can be spam-tapped (`QRCodeLoginViewModel.kt:64`) · `success` — check + 登录成功 + 返回 (`:50-78`) · `error` — `LoadedWithError` is never read by the UI; toast only (`QRCodeLoginViewModel.kt:69-73`) · `expired / invalid / failed` — **no dedicated UI**: the message branch renders with the button suppressed, leaving only the toolbar back chevron (`:92`); no countdown, no refresh (`qr_login.proto:31-32`).

**Divergence:** iOS values are unmeasured in this audit range — the Android shape above is normative pending an iOS pass. The expired-ticket state ships no UI on Android and must be added.

### Scan code (扫一扫) — platforms: both

**Purpose:** the in-app camera scanner whose `ham://` results route to QR login, SSO authorize or a deep link.

**Blocks / anatomy:** `camera preview` — full-bleed · `scan frame` — centred reticle · `result dispatch` — Android forwards every `ham://` scan as `OnReceivedDeepLink` (`QrCodeScanToLoginHandler.kt:16-25`), then `DeepLinkHandler` maps host → route (`DeepLinkHandler.kt:20-24,55-59`); routes `qrcode/scan` (`QrCodeViewRoute.kt:10-13`) and `scanCode` (iOS, `Route.swift:96`).

**GAP — not audited.** The audit range ends at `usercenter.md:8542` with CAS settings: **no scan section exists in the source notes and no iOS or Android value was measured.** Normative requirements that need no measurement: preview full-bleed behind a 42 + inset nav bar titled 扫一扫; reticle and controls at the 44 minimum target (§3.4); `text.primary` chrome; a permission-denied state with copy and a 去设置 action; torch and album as `icon.md` 24 icon buttons.

### Cross-screen defects (auth stack)

- **Kill the three dead iOS login sites** — `githubLoginWebView()` (`LoginView.swift:222-242`), `wechatLoginWebView()` (`:271-311`; unreachable — `LoginVM.swift:145` sets a flag no view observes), `cancelLogin()` (`:40-44`); route WeChat through the same web sheet as GitHub/Ziqiang.
- **Gate Passkey on `validLoginType`** like every other provider (`LoginView.swift:109`, `LoginView.kt:202`) · **add the missing disabled/error affordances**: the agreement toasts instead of disabling (`LoginVM.swift:54-56`, `LoginViewModel.kt:207-219`), `该版本暂不支持登录` has no Android equivalent (`LoginView.kt:245`), the QR confirm has no in-flight state (`QRCodeLoginViewModel.kt:64`) and QR expiry no UI (`qr_login.proto:31-32`).
- **Give iOS CAS settings a nav title and localise its 8 hardcoded strings** (`CasSettingView.swift:23,25,36,39,49,55,62,73`; keys already at `Localizable.strings:643,557,762,763,370,714,408,570`); de-duplicate `重新登录` `:370`/`:408` and `登录成功` `:196`/`:421`.
- **Stop persisting the CAS plaintext password** and stop delivering empty credentials when a session is established without a `#load` tap (`CasMobileLoginView.swift:37,39`, `CasJsInjector.swift:61`, `CasMobileLoginWebView.swift:121-122`).
- **iOS `CasSettingView` catches only `ResponseError`** — any other throw pins the spinner forever (`:137-143`); Android should distinguish a network error from an expired session instead of collapsing both into 登录状态失效 (`CasContext.kt:49-55`) · **unify the two reds** #FF0000 / #F44336 into `text.danger` #FF3B30 / #FF453A (`CasSettingView.swift:86`, `CasSettingMainView.kt:80`).
- **Add a drag handle to every sheet** (§3.7) — `HamSheet` ships only an invisible 32 dp grab zone (`Sheet.kt:63,130-144`); iOS declares no detents at all (`MainSheetView.swift:22`, `CasSettingView.swift:98`).
- **iOS SSO: replace the one-way scroll latch with a live scroll computation** and give the disabled 授权 a visible reason (`SSOAuthorizationSheet.swift:121,176-179`, `kt:207-212`); localise the ASCII scope parens (`swift:369-371`, `kt:451-452`); adopt Android's `WaitingForLogin` resume and surface the unused `请先登录后再进行授权` (`Localizable.strings:911`, `auth/strings.xml:33`).
- **Ship `removePreference`** — a per-app revoke in authorized-apps; dead on both (`SSOAuthPreferenceManager.swift:61-65`, `kt:40-42`) — and make Android's 取消 notify the third party of denial (`SSOAuthorizationSheet.kt:150-152`).
- **Add the missing Apple chip on Android** (`LoginType.Apple` exists, `HamUserInfoUnbindRequest.kt:21-28`) · **audit 扫一扫** — no measured values exist for either platform in this range.


## 11. Shared components (共享组件)

### Error view (错误视图) — platforms: both
**Purpose:** Terminal failure state: one icon, one headline, an optional hint, one primary action.
**Layout:**
```text
┌──────────────────────────────────────────────────────────────┐
│ ⊗ 64 feedback.error · 更新失败 28/Bold · <hint> caption 12   │
│ (32) · [ 返回 ] 48 tall r12 · pad 16 · surface.primary · slide-in+fade 200 ms │
└──────────────────────────────────────────────────────────────┘
```
**Blocks:** icon — `ErrorView.swift:31-33`, `ErrorView.kt:76-84`; title — `ErrorView.swift:35`, `ErrorView.kt:85`; hint, omitted when empty — `ErrorView.swift:37-40`, `ErrorView.kt:86`; action — `ErrorView.swift:47-53`, `ErrorView.kt:90-105`.
**Values:**
| Element | iOS | Android | normative |
|---|---|---|---|
| Icon / title / hint | `xmark.circle.fill` 64, white on `.red` — `ErrorView.swift:31-33`; `.title` — `:35`; `.caption` 12 — `:39` | `Icons.Rounded.Close` 64, pad 8, white on `ham_red` — `ErrorView.kt:77-83`; 24sp Bold — `:85`; `body` 16sp — `:86` | `icon.xl` 64 white on `feedback.error`; `title` 28 / Bold; `caption` 12 / `text.secondary` |
| Spacing / action / container | stack 4 — `:30`; icon→action 32 — `:42`; implicit height, maxWidth 350, blue @ 0.15, r12 — `:49-53`; no padding, `ham_bg_b1Color`, no animation — `:57-58` | stack 8.dp — `:74`; gap 16.dp — `:88`; 48.dp tall, fillMaxWidth + 4.dp inset, `ham_blue` @ 0.15f, r12 — `:92-97`; 16.dp padding, inherits bg, `slideInVertically{it/2}` + fade after 200 ms — `:71,56-67` | `space.3` 8 stack; `space.7` 32; action 48 tall, `radius.6` 12, `tint.subtle`, `bodyBold` / `accent`, to the 16 screen margin, cap 350; `space.5` 16, `surface.primary`, slide-in + fade 200 ms |
**Strings / states:** action label defaults to `返回` (hardcoded) — `ErrorView.swift:22`; Android `common_done` 完成 — `ErrorView.kt:60`, `strings.xml:7`; caller titles `更新失败` `CourseUpdateByCasView.swift:42`, `请求失败` `CourseCenterView.swift:52`, `预约失败` (hardcoded) `LibraryBookErrorView.swift:24`, `更新失败` (hardcoded) `ScoreUpdateByCasView.swift:43`. Static; the hint is the only variable content.
**Divergence:** iOS hides the hint when empty and caps the action at 350; Android always renders the message and full-bleeds the action.

### Success view (成功视图) — platforms: both
**Purpose:** Terminal success state, with a confetti animation and one haptic.
**Layout:**
```text
┌──────────────────────────────────────────────────────────────┐
│░ confetti Lottie, full bleed ░│
│ ✓ 64 white on feedback.success · 验证成功 28/Bold · <hint> caption 12 │
│ (32) · [ 返回 ] 48 tall r12 · pad 16 · surface.primary · fade+rise 0.8 s │
└──────────────────────────────────────────────────────────────┘
```
**Blocks:** confetti — `SuccessView.swift:41`, `SuccessView.kt:52-54,65-70`; icon — `SuccessView.swift:43-45`, `SuccessView.kt:83-91`; title — `:47`, `:92`; hint — `:50-51`, `:93`; injected content, iOS only — `SuccessView.swift:32-37,54-56`; action — `SuccessView.swift:60-65`, `SuccessView.kt:97-106`.
**Values:**
| Element | iOS | Android | normative |
|---|---|---|---|
| Lottie / delay / haptic | `CongratsLottieView` — `:41`; 0.8 s — `:87`; `ImpactManager` `.heavy` — `:91` | `lottie/lottie_congrats.json`, speed 1f, no restart — `:53,68-70`; 0.5 s — `:60`; `DeviceManager.doVibrate` — `:62` | one shared asset; 0.8 s lead-in; one haptic |
| Icon / title / hint | `checkmark.circle.fill` 64, white on `.green` — `:43-45`; `.title` — `:47`; `.caption` — `:51` | `Icons.Rounded.Done` 64, pad 8, white on `ham_blue` — `:83-91`; 24sp Bold — `:92`; `body` 16sp — `:93` | `icon.xl` 64 white on `feedback.success`; `title` 28 / Bold; `caption` 12 / `text.secondary` |
| Spacing / action / container | stack 4 — `:42`; gap 32 — `:58`; maxWidth 350, blue @ 0.1, r8 — `:69,72,74`; pad 32, `ham_bg_b1Color`, `.spring()` + offset y 100→0 — `:78-81,84` | stack 8.dp — `:81`; gap 16.dp — `:95`; 48 tall, `ham_blue` @ 0.15f, r12 — `:98-101`; pad 16, inherits, `slideInVertically{it/2}` + fade — `:74,78` | `space.3` 8; `space.7` 32; 48 tall, `radius.6` 12, `tint.subtle`, `bodyBold` / `accent`, cap 350; `space.5` 16, `surface.primary`, spring fade + rise 100 |
**Strings / states:** iOS action label `返回` is hardcoded — `SuccessView.swift:67`; callers `验证成功` / `你可以开始使用图书馆了` `LibraryIntroView.swift:20`, `发布成功` / `已成功发布课程评分与评论` `CourseDetailCreateReviewView.swift:39`, `更新成功` `CourseUpdateByCasView.swift:40` and (hardcoded) `ScoreUpdateByCasView.swift:41`, `ScoreUpdateViewSuccessCell.swift:17`; Android default `common_done` 完成 — `SuccessView.kt:57`. Appear (haptic + Lottie) → content animates in → dismissed.
**Divergence:** iOS's icon is green, Android's is `ham_blue` with green reserved for the toast — `ToastManager.kt:114`; normative is `feedback.success`. iOS caps the action at 350 with r8.

### Empty view (空状态) — platforms: both
**Purpose:** The shared placeholder rendered by any screen, card, or list with no content.
**Layout:**
```text
┌──────────────────────────────────────────────────────────────┐
│ ◌ 64 text.tertiary · 无日程 body 17 · <body> caption 12      │
│ [ action ] 48 tall r12 · centred · pad 16 · inherits parent card │
└──────────────────────────────────────────────────────────────┘
```
**Blocks:** icon — one glyph naming what is missing; title — one line naming the state; body — optional guidance; action — optional recovery step.
**Values:**
| Element | iOS | Android | normative |
|---|---|---|---|
| Component | absent; empties authored per module — `LibrarySelectSeatView.swift:243,248` | absent; empties authored beside each screen | one shared component, built once and adopted on both |
| Icon / title / body / action / gaps / container | per module | per module | `icon.xl` 64 `text.tertiary`; `body` 17 / Regular / `text.secondary`; `caption` 12 / `text.tertiary`; action 48 tall, `radius.6` 12, `tint.subtle`, `bodyBold` / `accent`; gaps 8 / 4 / 16; inherits the parent card (§3.9), centred, padding `space.5` 16 |
**Strings / states:** none shared today; each module writes its own (`无日程`, `无群组`, the library seat empty) — one shared key per state, title and body separate. Each content slot renders exactly one of loading, empty (this), or error.

### Loading (加载中) — platforms: both
**Purpose:** Three forms cover every wait: inline spinner, skeleton for known-shaped content, blocking modal for a committed action.
**Layout:**
```text
(a) spinner   (b) skeleton      (c) blocking modal
┌────────┐ ┌──┬──┬──┬──┐ ░░░░░░░░░░░░░░░░░░░░
│   ◌    │ │▢ │▢ │▢ │▢ │ ░ ┌────────┐ ░ 72 box r12 · scrim black @0.65
│ <label>│ ├──┼──┼──┼──┤ ░ │   ◌    │ ░ above content, below toast
└────────┘ └──┴──┴──┴──┘ ░ └────────┘ ░
20, stroke 2 · label 17   min cell 70, gap 10, cell r8
```
**Blocks:** spinner — iOS stock `ProgressView()` at ~30 sites — `CourseCenterView.swift:49`, `CasSettingView.swift:33`, `SportOrderView.swift:26`, `LibraryDetailBookView.swift:22`, `LibraryHistoryView2.swift:86`, `CourseScoreCourseDetailView.swift:31`; Android `HamLoadingProgressBar.kt:19-30`; determinate — `SportQuickOrderView.swift:35`, `LibraryQuickBookView2.swift:123`; skeleton — `LibrarySelectSeatView.swift:345-372`, reused at `LibraryDetailBookViewBody.swift:55`; blocking modal — `LoadingModalView.kt:26-40`.
**Values:**
| Element | iOS | Android | normative |
|---|---|---|---|
| Spinner / stroke / colour / label / determinate | system default; no label; `ProgressView(value:)`, nil → indeterminate — `SportQuickOrderView.swift:35`, `LibraryQuickBookView2.swift:123` | 20.dp / 2.dp / `ham_gray` — `HamLoadingProgressBar.kt:21-23`; print module only, `print_uploading` 16sp — `PrintShareFileLoadingView.kt:32-37` | 20 / 2 / `text.secondary`; optional `body` 17 / `text.secondary`, gap `space.3` 8; determinate fill `accent`, track `surface.tertiary`, height 4, `radius.1` 2 |
| Skeleton grid / cell | `LazyVGrid` `.adaptive(minimum: 70)`, spacing 10, 48 cells — `LibrarySelectSeatView.swift:347-350`; cell r8, gray @ 0.2, vertical pad 4, `.redacted(.placeholder)` — `:357-368` | — | min cell 70, gap 10, count from the loaded layout; cell `radius.4` 8, `surface.tertiary`, pad 4, placeholder redaction |
| Modal scrim / box | — ; inline, in the content slot | `ham_black` @ 0.65, tap-absorbing — `LoadingModalView.kt:28-32`; 72 square, r12, `ham_bg_b1` — `:34-35` | black @ 0.65, tap-absorbing; 72 square, `radius.6` 12, `surface.secondary` |
**Strings / states:** `加载中` (hardcoded) — `InnerWebView.swift:31`; `正在更新` — `CourseUpdateByCasView.swift:35`; `print_uploading` — `PrintShareFileLoadingView.kt:32-37`. Loading (this) / loaded / failed → Error view with 重试.
**Divergence:** iOS has no blocking modal and no labelled spinner; Android has no skeleton. All three forms ship on both.

### Toast (轻提示) — platforms: both
**Purpose:** Transient top-anchored status bar for a fired action.
**Layout:**
```text
┌──────────────────────────────────────────┐
│ ← status-bar inset →                     │
│ ┌──────────────────────────────────────┐ │
│ │ ⊙ 36 · 标题 body 17/Bold · 内容 cap 12 │ │ r8 · pad 16 · feedback.* · gap 5
│ └──────────────────────────────────────┘ │ full width · fade + slide · 3300 ms
└──────────────────────────────────────────┘
```
**Blocks:** top inset — `ToastView.swift:101,135-142`, `ToastManager.kt:157`; bar — `ToastView.swift:83,100,131`, `ToastManager.kt:155-166`; type icon — `:87`, `:174`; title — `:91`, `:181-185`; content — `:95-96`, `:188-193`; dismiss gestures — tap `:118-120`, `:160-164`, drag-up `:121-128`.
**Values:**
| Element | iOS | Android | normative |
|---|---|---|---|
| Radius / padding / icon / gap / title / content | 0 (full-bleed `Rectangle`) — `:105-106`; top = safe-area top, h+b 16 — `:101-102`; 36 — `:87`; 5 — `:85`; `.semibold` 17 — `:91`; `.caption` 12, 2-line limit — `:95-96` | 12.dp — `:166`; 16.dp all round — `:169`; 32.dp — `:174`; 8.dp — `:170`; `bodyBold` 16sp — `:181-185`; `body` 16sp, no limit — `:188-193` | `radius.4` 8 (§3.8); 16 all round + status-bar inset; 36; 5; `body` 17 / Bold; `caption` 12 / Regular, 2-line limit |
| Duration / animation / dismiss | 3 s + 0.3 s — `:37`; opacity + `move(edge: .top)` — `:108-117`; tap yes — `:118-120`; drag-up yes, `translation.height < 10` — `:121-128` | 2000 ms — `:150,202`; alpha 0→1 fade only — `:154,159`; tap yes — `:160-164`; drag-up no | 3300 ms; fade + slide from top; tap and drag-up dismiss |
| Types / fill / presentation | info, warning, success, error, normal — `ToastType.swift:6-12`; `feedback.*` opaque, white fg, `.primary` on normal — `ToastType+UI.swift:12,14,16,18,20,28,30`; probe `UIHostingController` `sizeThatFits` then window — `ToastManager.swift:49-52,61-65` | Normal, Success, Error — `ToastManager.kt:64-68`; `0xFFEDEEEF` / `ham_green` / `red` — `:104-105,113-114,122-123`; Snackbar with a grafted `ComposeView` — `:139-144,201` | info, warning, success, error, neutral; `feedback.<type>` opaque, white fg; width = screen width, height = content |
**Strings / states:** iOS legacy action `了解更多` is dead code — `ToastView.swift:261`, handler commented out `:247-267`; Android toasts are hardcoded Kotlin — `遇到了错误` `:224`, `网络异常，请稍后重试` `:251,254`. Transient; an empty title and content drops the toast — `ToastManager.swift:31-33`.
**Divergence:** iOS ships a full-bleed square bar, five types, 3 s, slide and drag dismiss; Android ships a 12dp rounded bar, three types, 2 s, fade only.

### Bottom sheet (底部弹层) — platforms: both
**Purpose:** The primary modal container — hosts Intro, Pay, SSO, and the changelog.
**Layout:**
```text
┌──────────────────────────────────────────┐
│░ scrim black @ 0.5, tap to dismiss     ░│
│ ╭────────────────────────────────╮       │ top corners 24 · surface.primary
│ │ ▬ handle 36×5 ▬ 32 · content 85 % │    │ over-drag 100 · dismiss past ¼ height
└─┴────────────────────────────────┴───────┘
```
**Blocks:** scrim — `Sheet.kt:90-101`; drag-handle header — `Sheet.kt:63,131-134`; content box — `Sheet.kt:104-128`; app-level sheets (pay, SSO, changelog) — `MainSheetView.swift:11-57`.
**Values:**
| Element | iOS | Android | normative |
|---|---|---|---|
| Radius / detent | system ≈12; system detent — no `presentationDetents` anywhere — `MainSheetView.swift:11-57` | 24.dp, top only — `:65,112-113`; 85 % of screen height — `:205,213` | platform default, Android 24 top-only; 85 %, overridable (§3.7) |
| Handle / scrim / buffer / physics | system indicator; system scrim | invisible 32-tall strip — `:63,133`; black @ 0.5 × modalProgress — `:92`; over-drag 100.dp — `:62`; dismiss past sheetHeight / 4 — `:174-175`; below rest `+Δ/2` — `:166`; above rest `+Δ×2/(abs(c)+1)` — `:168` | 36 × 5 pill, `text.secondary` @ 0.4, centred in a 32-tall header (§3.7); black @ 0.5, tap to dismiss; buffer 100; dismiss past ¼ of height; same curves |
| Background / hidden state | `surface.primary` | `ham_bg_b1` — `:116`; content not rendered when hidden — `:85-87`; `Animatable.animateTo` under a `Mutex` — `:184-200` | `surface.primary`; content not rendered when hidden |
**Strings / states:** none structural; callers supply their own 关闭 / 取消 — `IntroView.kt:184`, `CasSettingView.kt:60-79`. Hidden / shown (offset 0) / dragging.
**Divergence:** iOS uses the platform sheet at system detents with a free drag indicator; Android hand-rolls the 85 % sheet. Android renders the 36 × 5 pill.

### Banner / notification bar (横幅) — specified, shipped on neither platform
**Purpose:** In-layout alert card for a message that belongs to a specific place on screen.
**Layout:**
```text
┌──────────────────────────────────────────┐
│ (pad 16) ┌────────────────────────────┐  │ pad 12 · r8 · feedback.* · white fg
│ │ ⊙ <title> bodyBold · <detail> 15 │    │ gap 2 · spring in from the top
│ └────────────────────────────────────┘  │ auto-dismiss 3 s · tap dismisses
└──────────────────────────────────────────┘
```
**Blocks:** tinted card — `Banner.swift:113-115`; type icon — `:83-92,102`; title — `:106`; detail — `:108`; auto-dismiss — `:125-127`; tap to dismiss — `:121-123`.
**Values:**
| Element | iOS | Android | normative |
|---|---|---|---|
| Padding / radius / fill / type | outer 16 — `:118`; card 12 — `:113`; r8 — `:115`; opaque type tint — `:114`, `:69-80`; white fg, all types — `:112` | — | `space.5` 16 outer; `space.4` 12 card; `radius.4` 8; `feedback.<type>` opaque; white fg |
| Title / detail / gaps / motion / glyphs | `.bold()` — `:106`; 15 `.light` — `:108`; title→detail 2 — `:104`; icon→text default `HStack` — `:102`; `.spring()` + `move(edge: .top)` + opacity — `:119-120`; 3 s — `:125`; info blue `info.circle`, success green `checkmark.seal`, warning yellow `exclamationmark.octagon`, error red `xmark.octagon` — `:70-79,83-92` | — | `bodyBold` 17 / Bold; `subheadline` 15 / Regular; gaps `space.1` 2 and `space.3` 8; spring in from the top; 3 s auto-dismiss; tap dismisses; identical type mapping |
**Strings / states:** none shipped; the preview carries `123` / `456` — `Banner.swift:136`. Visible / hidden, driven by the `isShow` binding.

**Correction.** This component does **not** ship. `Banner` is constructed in exactly one place —
its own SwiftUI preview, `Banner.swift:136` — and nothing else in the app builds a
`BannerDataModel`. Android has no banner component at all (`find . -iname '*banner*'` returns
only `SportMainViewBannerCard` and `LibraryMainViewBannerCard`, which are announcement *content*
cards, not notification banners). Everything above is therefore a **specification of an
unbuilt component**, not a description of shipped UI. Treat it as the target if the banner is
ever built; do not treat it as evidence that either platform has one today.

### Text field (输入框) — platforms: both
**Purpose:** One shared single-line and multi-line text input, used by every form on both platforms.
**Layout:**
```text
┌──────────────────────────────────────────┐
│ r8 · surface.secondary fill · 1px surface.tertiary border · pad 8 · body 17 │
│ ┌────────────────────────────────────┐   │ caret accent · multi-line minHeight 30
│ │ <hint when empty> / <value>        │   │ fills the available width
│ └────────────────────────────────────┘   │
└──────────────────────────────────────────┘
```
**Blocks:** box — fill, border, radius — `TextField.kt:42-49`; hint — shown while empty — `:74-79`; value — `:52`; password transformation — `:70-71`.
**Values:**
| Element | iOS `TextEdit` | iOS `TextEditorApproach` | Android `HamTextField` | normative |
|---|---|---|---|---|
| Radius / border / fill / padding | 4 — `TextEdit.swift:27`; 1pt `Color.lightGray` — `:28`; none; 6 — `:25` | plate 12, editor 6 — `:28,37`; none; gray @ 0.3 — `:27`; editor 9, placeholder 16 — `:33,39` | 8.dp — `TextField.kt:42,47`; 1.dp `ham_lightGray` — `:44-48`; `ham_bg_b2` #FFF9F9F9 / #ff0f1010 — `:43`, `colors.xml:66`, `values-night/colors.xml:33`; 8.dp — `:49` | `radius.4` 8; 1 `surface.tertiary`; `surface.secondary`; `space.3` 8 |
| Text / min height / hint / caret / lines | system default; — ; native placeholder; system caret; 1, intrinsic | system default; 30 — `:36`; `.gray` @ 0.8 — `:31-32`; system; unbounded | `body` 16sp — `:52`; — ; `ham_text_secondary` #888888, offset (−2).dp — `:76,78`; `ham_blue` #007AFF — `:67`, `colors.xml:79`; `Int.MAX_VALUE`, `singleLine = true` default — `:51,55`, `fillMaxWidth` — `:62`, `PasswordVisualTransformation` — `:70-71` | `body` 17 / Regular; 30 per line, multi-line; `text.tertiary`, no offset hack; `accent`; 1 or unbounded by variant; fills the available width; password supported |
**Strings / states:** hint is caller-supplied, default `""` — `TextField.kt:50`; preview `测试` — `:89`. Empty (hint) / focused (1 border in `accent`) / error (`text.danger` border + message) / disabled (`tint.muted`) / password.
**Divergence:** iOS's two text components are unused and are deleted, replaced by the shared field — `TextEdit.swift:10`, `TextEditorApproach.swift:10`. Android's live sites adopt it unchanged: `CourseEditViewInfoCell.kt:35,45,55`, `CourseScoreSearchResultView.kt:72`, `CourseCommentView.kt:373`, `SportPayView.kt:112`, `UserCenterInfoView.kt:119`, `PrintShareFilePrepareView.kt:238,262`, `ColorPicker.kt:163`.

### Segmented picker (分段选择器) — platforms: both
**Purpose:** Two-to-five-way mutually exclusive choice, rendered as a track with a sliding thumb.
**Layout:**
```text
┌──────────────────────────────────────────┐
│░ track text.secondary @ 0.40 · r6 · pad 2░│
│ ░┌────────┐░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ thumb r6 surface.tertiary, width−2
│ ░[lbl 0][lbl 1][lbl 2] 1f · x animated ░│ scrolls when segments overflow
└──────────────────────────────────────────┘
```
**Blocks:** track — `HorizontalPicker.kt:47-60`; thumb — `:64-71`; labels — equal weights, caller-supplied — `:73-85`.
**Values:**
| Element | iOS `SegmentedPicker` | Android `HamHorizontalPicker` | normative |
|---|---|---|---|
| Track / thumb | none — the caller's `selection()` view — `:54-60`; thumb is a caller-supplied view sized to the measured frame — `:55-56`; vertical pad 3.0 when `isShowIndicator` — `:82` | `ham_gray` @ 0.4f (#888888) — `:56`; r6 — `:57`; pad 2.dp — `:59`; `Spacer` width = measuredWidth / count − 2.dp, full height, `ham_lightGray`, r5 — `:64-71`; `animateDpAsState` on x — `:65,69` | `text.secondary` @ 0.40 (§3.6); `radius.3` 6; `space.1` 2; thumb `surface.tertiary`, radius 6 (= track radius), animated x-offset |
| Segments / selection / overflow / label | spacing 0 — `:66,86`; index, `Binding<Data.Index?>` — `:18`; `isScroll` → `ScrollView` — `:29,64-84`; caller-supplied, `isShowIndicator` default false — `:30` | weight 1f each — `:75`; key, `PickerItem<T>` — `:43,63`; none; caller-supplied lambda — `:79`, always shown | spacing 0, equal weights; by key; scroll on overflow; `body` 17 / Regular, `text.primary` selected, `text.secondary` unselected, indicator always shown |
**Strings / states:** caller-supplied; live Android call site `SportMainViewQuickOrderCard.kt:142`. Selected / unselected / disabled.
**Divergence:** iOS selects by index with a caller-supplied indicator; Android selects by key with a fixed thumb. Normative is key-based with a fixed thumb at radius 6 (§3.6).

### Divider (分割线) — platforms: both
**Purpose:** A 1-thick separator between stacked rows or inline items.
**Layout:**
```text
horizontal: ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬  fill width × 1, surface.tertiary
vertical:   ▮ 1 wide, height from the row (16 in colour cells)
```
**Blocks:** horizontal — `Divider.kt:16-24`, iOS stock `Divider()`; vertical — `Divider.kt:26-34`.
**Values:**
| Element | iOS | Android | normative |
|---|---|---|---|
| Thickness / colour / inset | SwiftUI hairline, system separator — `ScheduleView.swift:371`, `CourseAddTimeSettingView.swift:36,53`, `CourseScoreSingleCard.swift:62`; ad-hoc insets, e.g. `.padding(.top, 12)` — `ScheduleView.swift:371` | 1.dp `DividerDefaults.Thickness` — `Divider.kt:21,31`; `ham_lightGray` #EDEEEF / #0F0E0F — `:17,27`, `colors.xml:68`, `values-night/colors.xml:35`; none | 1, `surface.tertiary`, 0 inset from the container edge, vertical padding `space.2` 4 (§3.1) |
| Vertical use | — | `height(16.dp)` — `CourseEditViewColorCell.kt:83`, `CourseMainViewDetailCourseScoreCard.kt:168`, `CourseScoreResultView.kt:363` | 1 wide, height from the row |
**Strings / states:** none.
**Divergence:** three values disagree today — the unused `ham_divider` #F4F4F4 / #0C0C0C (`colors.xml:67`, `values-night/colors.xml:34`), `ham_lightGray` in `HamDivider`, and the iOS system separator.

### Image crop (裁剪图片) — platforms: both
**Purpose:** Crop and orient an image to a fixed aspect ratio before upload.
**Layout:**
```text
┌──────────────────────────────────────────┐
│ 返回 / ✕                          完成    │ toolbar over a full-bleed surface
│   ┌──────────────────────────────────┐   │ frame white stroke 2 · guidelines on
│   │  <image, scaled>                 │   │ outside masked black @ 0.5, even-odd
└───┴──────────────────────────────────┴───┘ reset · rotate · flip H/V · confirm
```
**Blocks:** crop surface — `MantisImageCropView.swift:33-40`, `ImageCropActivity.kt:42-46`; toolbar — `MantisImageCropView.swift:84`, `ImageCropView.swift:180-203`, `ImageCropActivity.kt:90-104`; mask — `ImageCropView.swift:165-168`.
**Values:**
| Element | iOS (Mantis) | iOS legacy `CropImageView` | Android | normative |
|---|---|---|---|---|
| Toolbar | `[.reset]` only — `MantisImageCropView.swift:84` | 返回 + 完成, hardcoded — `ImageCropView.swift:184,199` | rotate +90, flip-H, flip-V, crop — `ImageCropActivity.kt:97-100` | reset, rotate, flip-H, flip-V, cancel, confirm |
| Guidelines / mask / stroke / ratio / shape | Mantis default; `PresetFixedRatioType`, multi-preset — `:21`; `CropShapeType`, `.rect` default, circle supported — `:13,20` | none; black @ 0.5 `eoFill`, no hit-testing — `:166-168`; white 2pt — `:173`; fixed `cropSize`; rect | `Guidelines.ON` — `:42`; library default; fixed `(ratioX, ratioY)`, `setFixedAspectRatio(true)` — `:45-46`; rect | on; black @ 0.5 even-odd; white 2; caller-supplied fixed ratio; rect or circle by caller |
| Zoom / drag / output / failure | Mantis default; `UIImage` via `onCrop` — `:15,58-60`; delegate methods empty — `:66-73` | clamp `[0.01, 2]` — `:80-85`; min scale from crop size — `:94-95`; tap-zoom +0.1 — `:113-116`; drag clamped to ±(size·scale − crop)/2 — `:128-147`; `UIImage?` — `:43,196`; `guard let` — `:237-239` | library default; PNG quality 100 temp-file `Uri` — `:64-86`; exceptions swallowed — `:67-69,78-80` | as measured; in-memory image on iOS, temp-file `Uri` on Android; failure surfaces a toast |
**Strings / states:** iOS legacy 返回 / 完成 are hardcoded — `ImageCropView.swift:184,199`; Android uses `image_crop_title` — `:48` and `R.menu.image_crop_menu` — `:91`. Loading (Android `setImageUriAsync` is async with no indicator — `:44`) / editing / failed → toast.

### Webview screens (网页视图) — platforms: both
**Purpose:** Three patterns — inner webview, in-app full-text reader, external browser — selected by the remote-config entry's action type (§5.4).
**Layout:**
```text
inner:                full-text:             external:
┌─────────────────┐  ┌─────────────────┐   ┌─────────────────┐
│ <title | 加载中> │  │ <title> headline│   │ in-app browser  │
│  [webview]      │  │ <markdown body> │   │ sheet, system   │
│  progress 4     │  │      查看详情 ↗  │   │ chrome          │
└─────────────────┘  └─────────────────┘   └─────────────────┘
```
**Blocks:** inner webview — `InnerWebView.swift:9-68`, `PlainWebView` `:70-138`, Android `WebViewCompose.kt`; nav title — `InnerWebView.swift:31,35-36,63,65`; full-text view — `LibraryMainViewBannerFullTextCell.swift:11-41`, tappable cell `:43-79`; external browser — `SafariViewController.swift:11-63`.
**Values:**
| Element | iOS | Android | normative |
|---|---|---|---|
| JS / windows / storage / images / media / dark mode / background | true / true — `EducationCASWebView.swift:34-35`, `WebContainer.swift:30`; domStorage, loadImages, media gesture not set; not set; not set (white flash) | true / true / true / true / false — `WebViewCompose.kt:51-55`; `FORCE_DARK_ON/OFF` by theme, API ≥ Q — `:70-73`; `ham_bg_b1` — `:65,69` | true / true / true / true / false; follows the system theme; `surface.primary` |
| Cookies / reload / back / progress / error | cleared on create — `EducationCASWebView.swift:27-28,36-40`, `.nonPersistent()` — `CasMobileLoginWebView.swift:49`; `.returnCacheDataElseLoad` — `EducationCASWebView.swift:51`, `WebContainer.swift:47`; `allowsBackForwardNavigationGestures = true` — `:20,115`, in-page toolbar commented out — `:39-60`; no progress, no error | default persistent store; reload skipped when the URL is unchanged — `:90-94`; system back; no progress, no error | non-persistent for CAS, default otherwise; reload only when the URL changes; edge-swipe back plus an in-page back control when the page has history; 4-tall bar, `radius.1` 2, `surface.tertiary` track; error with 重试 |
| Full-text title / date / body / affordance | `.headline` limit 2 — `:22-24`; `.body` `ham_text_t2Color` limit 1 — `:27-30`; `Text(.init(content))` — `:34`; 查看详情 + `arrow.up.right`, bottom-trailing offset (−16, −4), `.blue` — `:66-75`; `isMarkdown` parsed but unused — `:59` | no equivalent | identical on both, and `isMarkdown` is honoured |
| External link | `SFSafariViewController`, dismiss delegate — `SafariViewController.swift:25-27,42-44,62` | `Intent(ACTION_VIEW)` — `AboutView.kt:258,291,448`, `MyViewLinkCard.kt:90,116`, `MyPromotionView.kt:58`, `ScoreJsCalcDetailView.kt:218`, `VersionUpdateSheet.kt:153,186` | in-app browser sheet on both |
**Strings / states:** `加载中` is hardcoded — `InnerWebView.swift:31`. Loading (title text plus the progress bar) / loaded / failed → error with 重试.

### React Native container (RN 容器) — platforms: both
**Purpose:** Hosts a React Native module inside a native screen.
**Layout:**
```text
┌──────────────────────────────────────────┐
│ [RN root view — fills the box]           │
│  starting → shared spinner · failed → Error view + 重试
└──────────────────────────────────────────┘
```
**Blocks:** container — `RNContainerView.swift:24-30`, `ReactNativeContainer.kt:13-24`, `ReactNativeContainerRendererImpl.kt:18-38`; host — `RNViewManager` `:74-90`, `RNHost` `RNContainer.kt:29-66`; bundle — `RNContainerView.swift:98-104`; native modules — `rn/module/`, `app/.../rn/components/custom/`.
**Values:**
| Element | iOS | Android | normative |
|---|---|---|---|
| Sizing / startup | `.frame(maxWidth:.infinity, maxHeight:.infinity)` — `CasMobileLoginWebView.swift:15`, `CourseUpdateByCasView.swift:24`; `ScoreJsCalcView.swift:19` ignores top + bottom; sync, async variant `RNContainerAsyncView` — `RNContainerView.swift:33-55` | `Modifier.fillMaxSize()` — `CasMobileLoginView.kt:57`, `ScoreJsCalcView.kt:40`, `CourseSettingViewUpdateCourseSheet.kt:170`; `RNHost.initHost` on `Dispatchers.IO`, `show` seeded from `RNHost.inited` — `RNCommonView.kt:21-34` | fills the available box; initialise off the main thread |
| Lifecycle / bundle | singleton factory, `RCTAppDependencyProvider` — `:74-90`; debug dev server, release `HotUpdater.bundleURL()`, prebuilt `Ham/iOS/ui/rn/main.jsbundle` — `:98-104` | `onHostResume` / `onHostDestroy` via `DisposableEffect` — `ReactNativeContainerRendererImpl.kt:27-32`; `RNInitHelper` / `IRNManager` | resume / pause / destroy wired to the host; debug dev server, release OTA |
| Modules (6 each) / gap while starting | `RNCasMobileLogin`, `RNCas`, `RNCommon`, `RNEducation`, `RNLog`, `RNScoreCalc` — Swift + `.mm`; empty `ZStack` — `RNContainerView.swift:40-44` | `RNCommonModule`, `RNEducationModule`, `RNLogModule`, `RNCasMobileLoginModule`, `RNCasModule`, `RNScoreCalcModule`; nothing until `show` flips — `RNCommonView.kt:31`, `delegate.reactRootView!!` — `ReactNativeContainerRendererImpl.kt:35` | identical names and contracts; shared Loading spinner, no non-null assertion |
**RN-backed screens:** headless bootstrap `ContentView.swift:37` / `RNCommonView.kt:32` — `RNCommon`; CAS login `CasMobileLoginWebView.swift:14` / `CasMobileLoginView.kt:57` — `RNCasMobileLogin`; update course `CourseUpdateByCasView.swift:23` / `CourseSettingViewUpdateCourseSheet.kt:170` — `RNFetchCourseView`; update score `ScoreUpdateByCasView.swift:24` / `ScoreMainViewUpdateScoreSheet.kt:156` — `RNFetchScoreView`; score JS calc `ScoreJsCalcView.swift:18` / `ScoreJsCalcView.kt:40` — `RNScoreCalcView`.
**Strings / states:** `正在更新` `CourseUpdateByCasView.swift:35`, `更新成功` `:40`, `更新失败` `:42`, `选择计算方式` `ScoreJsCalcView.swift:20`. Starting (shared spinner) / ready / failed (Error view + 重试) / RN disabled by the CCKV kill-switch → native captcha fallback — `CourseUpdateByCasView.swift:11,21,25-31`.
**Divergence:** iOS gates RN behind the CCKV key `.rnComponentConfig` with a native captcha fallback; Android hosts the same modules with no fallback.

### Shared-component gaps to close

| Component | Status | Normative action | file:line |
|---|---|---|---|
| Empty view | absent on both | build once, adopt on both | `LibrarySelectSeatView.swift:243,248` |
| Banner | zero production call sites | adopt on both, or delete | `Banner.swift:56,136` |
| `NotificationService` | uncalled | delete | `Banner.swift:10-54` |
| iOS text field | zero production call sites | delete both, adopt one shared field | `TextEdit.swift:10`, `TextEditorApproach.swift:10` |
| iOS segmented picker | zero production call sites | ship one shared picker on both | `SegmentedPicker.swift:10` |
| Skeleton | iOS only | build on Android | `LibrarySelectSeatView.swift:345-372` |
| Blocking loading modal | Android only | build on iOS | `LoadingModalView.kt:26-40` |
| Labelled spinner | print module only | promote to the shared component | `PrintShareFileLoadingView.kt:32-37` |
| RN container loading / error | absent on both | add spinner and error to the container | `RNContainerView.swift:40-44`, `RNCommonView.kt:31` |
| Divider colour | three disagreeing values | one `surface.tertiary` token | `Divider.kt:17,27`, `colors.xml:67` |
| Toast types | 5 on iOS, 3 on Android | 5 on both | `ToastType.swift:6-12`, `ToastManager.kt:64-68` |
| Sheet drag pill | iOS only, system-provided | Android renders the 36 × 5 pill | `Sheet.kt:63,133` |
| Success icon colour | green vs `ham_blue` | `feedback.success` on both | `SuccessView.swift:45`, `SuccessView.kt:88` |
| Toast strings | hardcoded Kotlin | move to `strings.xml` | `ToastManager.kt:224,251,254` |

### Intro / connect screen (连接页)

Specified in **Part 3 — Authentication** (§ CAS verification gate / Intro view): `IntroView.swift:35-164`, `core/ui/.../intro/IntroView.kt:66-245`.
Its anatomy — dismiss control, logo → link → module icon row, title, subtitle, choice card, CAS row — belongs to that flow.
Nothing is duplicated here.


## 12. Standalone screens (独立页面)

Normative ("build it this way"). Condensed from `/tmp/audit/shared.md`, plus direct source measurement for §4b
and §6 (Android) — neither was measured in the audit. Every value carries one `file:line`. Platform columns are
**evidence**; `normative` is the requirement. `Divergence:` lines are migration tasks.

**Prefixes** — `IOS/` `repos/ham-ios/Ham/iOS/` · `AND/` `repos/ham-android/android/` ·
`ANDCORE/` `AND/core/ui/…/com/nowcent/ham/common/ui/` · `CASSTR:` `AND/feature/cas/…/values/strings.xml` ·
`MYSTR:` `AND/feature/my/…/values/string.xml` · `AUTOSTR:` `AND/feature/automatic/…/values/strings.xml` ·
`APPSTR:` `AND/app/…/values/strings.xml` · `LS:` `Ham/zh-Hans.lproj/Localizable.strings`. Basenames: `.swift` =
iOS, `.kt` = Android. pt and dp are 1:1.

**Tokens** — card r16 / pad16 / `ham_bg_b2` (`ANDCORE/container/card/Card.kt:53,48,42`; iOS
`CardView.swift:23,22,128`) · `HamSheet` 85 % (`ANDCORE/container/Sheet.kt:205`) · type
(`ANDCORE/config/Font.kt`): `title` 24 Bold `:18` · `title2` 20 Bold `:20` · `title3` 16 Bold `:22` · `body` 16
`:49` · `bodyBold` 16 Bold `:27` · `caption` 12 `:34`; iOS semantic `.title` 28 · `.title2` 22 · `.body` 17 ·
`.caption` 12 · `ham_blue` #007AFF `colors.xml:79` · `ham_red` #FF3B30 `:72` · `ham_text_primary` `:63` ·
`ham_text_secondary` `:64` · `ham_bg_b1` #F9F9F9 `:65` · `ham_bg_b2` #FFFFFF `:66` · `ham_brand_bus`
=`R.color.ham_brown` (`Color.kt:87`) · `ham_brand_pay` #BF360C (`Color.kt:72`; iOS `Color+Ham.swift:35`) ·
toolbar 42.dp (`ScreenExtension.kt:56`) on `ham_bg_b1@0.95` (`NavigationView.kt:70`) · shared button = accent@0.15
fill, accent text, r12, pad-v 16, `fillMaxWidth`.

**Selection rule.** On a *tokenised* property (colour, type, radius) the **Android** token wins; where Android
*omits* a treatment iOS has (subtitle, state, guard) the **iOS** value wins; where the difference is a platform
capability (Siri vs AlarmManager, App Store vs APK) the two are spec'd separately and the gap is called out.

**Spec'd elsewhere** (shared components, do not repeat): intro shell, error/success view, empty, loading, toast,
bottom sheet, text field, segmented picker, divider, image crop, webview, RN container.

---

### 1. Bus (校车) — platforms: both
**Purpose:** Full-screen campus-bus web app (`https://bus.whu.edu.cn/mobile/#/`) opened with a CAS ticket,
with `navigator.geolocation` patched in; gated by the shared connect sheet until CAS is authorised.
**Entry:** iOS route `.bus` → `BusView()` (`IOS/ui/main/Route.swift:72,218`); Android
`AND/feature/my/…/ui/MyView.kt:45` → `BusView(navController)`.
**Layout:**
```
┌ ◀ 校巴 · toolbar h 42 · no bounce ───────────────────────┐
├ WebView fillMaxSize · +"?ticket=" · geolocation on finish ┤
└─────────────────────────────────────────────────────────┘
░ overlay: connect sheet 连接校巴 · COND !isLogin · +300 ms
    routes CAS → success → dismiss · failure → error view
```
**Blocks:** 1 **WebView** — iOS `InnerWebView(url:title:webView:)` over a `WKWebView` built in `init`, delegate
evaluates the geolocation shim on `didFinish` (`BusView.swift:35,43`, `:17-19`); Android
`WebViewCompose(fillMaxSize, url = vm.url)` (`BusView.kt:54`). 2 **Connect gate** — iOS `.sheet(isPresented:)`
(`BusView.swift:47-58`); Android `BusIntroView(introSheetState)` (`BusView.kt:58`) auto-shown **500 ms** after
entry when unauthenticated (`:44-46`). 3 **Auto-dismiss** — gate dismissed while still unauthenticated →
dismiss after **300 ms** (iOS `:59-65`) / pop the back stack (Android `:36-40`; intro `BackHandler`
`bus/intro/BusIntroView.kt:38-42`). 4 **Ticket** — `CasClient(service: "https://bus.whu.edu.cn/mobile/%23%2F")
.fastLogin()`, accept only if the redirect `contains("ticket")` (`BusView.swift:88-91`; `BusViewModel.kt:26`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Toolbar | none — title passed to `InnerWebView` `BusView.swift:43` | `HamNavigationView` 42.dp `BusView.kt:50` | **42.dp toolbar, no bounce scroll** |
| Nav title | 校巴 `BusView.swift:43` (`LS:731`) | `cas_bus_title` 校巴 `BusView.kt:51` (`CASSTR:21`) | 校巴 |
| Intro icon / brand | `bus.fill` `BusIntroView.swift:20` / `.ham_brand_bus` (`Color+Ham.swift:40`) | `Icons.Filled.DirectionsBus` `BusIntroView.kt:48` / `ham_brand_bus` (`Color.kt:87`) | bus glyph in `ham_brand_bus` |
| Intro title + rows | 连接校巴, CAS row only `BusIntroView.swift:21,25` (`LS:835`) | `cas_connect_bus` 连接校巴, CAS row only `BusIntroView.kt:47,84-86` (`CASSTR:22`) | 连接校巴, CAS row only |
| Intro subtitle | 使用前，Ham需要验证你的在校信息 `BusIntroView.swift:22` (`LS:547`) | **none** `BusIntroView.kt:44-50` | **使用前，Ham需要验证你的在校信息** |
| Gate delay | 300 ms `BusView.swift:70` | 500 ms `BusView.kt:44` | **300 ms** |
| Success screen | none — straight to the webview `BusIntroView.swift:28-30` | `cas_verify_success` 验证成功 + message `BusIntroView.kt:62-63` | **shared success view on both** |
| Failure | `ToastUtils.showError` `BusView.swift:92-94` | intro routes to error view `BusIntroView.kt:70-81` | error view inside the sheet |

**Strings:** 校巴 · 连接校巴 · 使用前，Ham需要验证你的在校信息 · 验证成功 · 验证失败 (`CASSTR:21-23`; `LS:731,835,547`).
**States:** loading — none (blank until the ticket resolves); unauthenticated — connect sheet; fetch failure —
error view in the sheet.
**Divergence:** iOS has no toolbar on the webview and no bus success/error screen; Android has both and opens
the gate at 500 ms.
> **Copy bug (normative: fix).** `cas_bus_success_message` = 你可以开始使用图书馆了 ("**library**") renders on the
> **bus** success screen (`CASSTR:23`, `BusIntroView.kt:63`). Must read 你可以开始使用校巴了.

---

### 2. Pay (付费 · 珞珈E卡) — platforms: both
**Purpose:** Campus E-card web app opened from a service URL; gated by the shared connect sheet.
**Entry:** iOS app-level sheet `IOS/ui/main/component/MainSheetView.swift:19` → `PayView()`; Android function
grid `AND/feature/my/…/component/function/MyViewFunctionComponentView.kt:146` → `PaySheetView`.
**Layout:**
```
┌ HamSheet 85 % · toolbar ignores status-bar height ────────┐
│ ◀ 珞珈E卡                                            h 42 │ ◀ = web back, ✕ = close
├─ WebView fillMaxSize · no bounce · bottom inset ignored  ─┤
└──────────────────────────────────────────────────────────┘
░ overlay: connect sheet 连接珞珈E卡 · COND !isLogin
```
**Blocks:** 1 **Connect gate** — iOS `PayIntroView` inline in the same `VStack`, `dismiss()` on cancel
(`PayView.swift:27-37`); Android `PayIntroView(dismissAction)` owning its `HamSheet` (`PaySheetView.kt:62`;
`pay/intro/PayIntroView.kt:39,42-44`). 2 **WebView** — iOS `PlainWebView` (`PayView.swift:39`); Android
`PayWebView` (`PaySheetView.kt:56`). 3 **Nav chrome** — iOS two leading buttons, `chevron.backward` →
`webview.goBack()` then `xmark` → dismiss (`:45-58`); Android relies on `HamNavigationView`'s back button, start
margin 16.dp (`NavigationView.kt:134`). 4 **Insets** — Android `ignoreStatusBarHeight = true`
(`PaySheetView.kt:47,53`); iOS `.ignoresSafeArea(.bottom)` (`:66`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Presentation | inline branch in a `VStack` `PayView.swift:26-65` | `HamSheet` 85 % `PaySheetView.kt:34,47` | **85 % sheet** |
| Nav title | E卡, hardcoded over the page title `PayView.swift:41` | `cas_e_card_title` E卡 `PaySheetView.kt:49` (`CASSTR:16`) | **E卡, fixed — discard the page title** |
| Toolbar chrome | `◀` web-back + `✕` close `PayView.swift:48,56`; bottom inset ignored `:66` | back only, margin 16.dp `NavigationView.kt:134`; `ignoreStatusBarHeight` `PaySheetView.kt:53`; bounce off `:54` | **web-back then close; ignore the status-bar height, keep the bottom inset; bounce off** |
| Intro icon | `creditcard.fill` `PayIntroView.swift:21` | `Icons.Rounded.CreditCard` `PayIntroView.kt:56` | card glyph in `ham_brand_pay` |
| Intro title | 连接珞珈E卡, subtitle 使用前，Ham需要验证你的在校信息 `PayIntroView.swift:22-23` (`LS:836`) | `cas_connect_e_card` 连接E卡, no subtitle `PayIntroView.kt:55` (`CASSTR:15`) | **连接珞珈E卡 + subtitle** |
| Success / failure | neither — `ToastUtils.showError` `PayView.swift:87-89` | `cas_verify_success` + `cas_e_card_success_message` `PayIntroView.kt:68-75`; error view `:76-87` (`CASSTR:19`) | **shared success + error views on both** |

**Strings:** 珞珈E卡 / E卡 · 连接珞珈E卡 · 使用前，Ham需要验证你的在校信息 · 验证成功 · 你可以开始使用E卡了 · 验证失败
(`CASSTR:15-19`; `LS:497,836`).
**States:** unauthenticated — connect sheet; authenticated — webview; failure — error view. No loading state.
**Divergence:** iOS is an inline branch with its own web-back button and no success/error screen; Android is an
85 % sheet with both. iOS says 珞珈E卡, Android 连接E卡. iOS ignores the bottom safe area, Android the top.

---

### 3. Privacy (隐私) — platforms: both
**Purpose:** Blocking first-run consent gate — the app is unusable until the policy is accepted.
**Entry:** iOS from the login screen (`IOS/ui/common/login/LoginView.swift:106,245`) and `PrivacyView2.swift:101`;
Android as an activity `MainActivity.kt:66` → `PrivacyActivity.kt:15`.
**Layout:**
```
┌ scrim black@0.5 · fillMaxSize ────────────────────────┐
│ ┌ dialog · w 300 · r16 · pad16 · ham_bg_b2 ────────┐  │
│ │ 隐私协议                             title 24    │  │
│ │ 您需要同意隐私协议才能继续使用Ham。       body 16  │  │
│ │ Ham不会在未经允许的情况下收集…           caption  │  │
│ │ 总的来说，Ham会获取下列信息：            caption  │  │
│ │ [必须] 设备标识… [可选] 查给分时上传的成绩 caption  │  │
│ │                         查看隐私协议 ›   accent  │  │
│ └──────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────┘
░ policy sheet 85 %: WebView {DocHost}/privacy/ · 拒绝 · 接受
```
**Blocks:** 1 **Lead** 您需要同意隐私协议才能继续使用Ham。 (iOS `PrivacyView.swift:22`; Android merges it into
`privacy_message`, `PrivacyView.kt:104-107` — **unmerge**). 2 **Legal paragraph** — the 个人信息保护法 text,
caption 12 secondary (iOS `:24-25`). 3 **Data list** — 总的来说，Ham会获取下列信息： + `[必须]`/`[可选]` rows with
embedded `\n`, iOS only (`:26-27`) — **required on both**. 4 **Policy link** — 查看隐私协议 opens an 85 % sheet
hosting a webview (`PrivacyView.swift:36-52`; `PrivacyView.kt:108-117,128`); URL = `CCKVKey.DocHost` +
`/privacy/`, fallback `https://whu-ham.github.io` (`PrivacyView.kt:129-130,166`). 5 **Actions** — accept →
`vm.didAgreePrivacy()` / `ColdStartManager.shared.onPrivacyAgreed()` (`PrivacyView.kt:151-162`;
`PrivacyView.swift:64-68`); decline → dismiss the sheet only on Android (`:139-149`), **suspend the app** on iOS
(`UIApplication.shared` ← `#selector(NSXPCConnection.suspend)`, `:56-62`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Container + scrim | full-screen `VStack`, no scrim `PrivacyView.swift:16` | scrim `Black@0.5` + 300.dp modal `PrivacyView.kt:58-69` | **modal dialog w 300 over black@0.5** |
| Dialog radius / bg / pad | — | **10**.dp / `ham_bg_b2` / 16 `PrivacyView.kt:91-93` | **16 / `ham_bg_b2` / 16** |
| Title | `.title` + `.semibold` `:20-21` | `title` 24 Bold `:100-101` | **24 Bold** |
| Lead sentence | body default `:22` | merged into body | **body 16, unmerged** |
| Legal paragraph + gap | `.caption` 12, 15 spacer above `:23-25` | body 16, merged `:106` | **caption 12 `ham_text_secondary`, gap 12** |
| Data list | `[必须]`/`[可选]` `:26-27` | **absent** | **`[必须]`/`[可选]` list, caption 12** |
| Link | `.caption`, leading `:35-42` | body 16, `ham_blue`, `align End` `:110-115` | **caption 12, accent, align End** |
| Actions | agree 同意 blue `:67`; decline 退出Ham `.red`, suspends the app `:60-62` | accept 接受 `ham_blue` `:159`; decline 拒绝 `ham_red`, dismisses the sheet only `:139-149` | **accept 同意 accent; decline 退出Ham `ham_red`, suspends the app** |
| Policy URL | hardcoded `PrivacyView.swift:47` | `CCKVKey.DocHost` `:129-130` | **`DocHost` + `/privacy/`, hardcoded fallback** |

**Strings:** 隐私协议 · 您需要同意隐私协议才能继续使用Ham。 · 〈legal paragraph〉 · 总的来说，Ham会获取下列信息： ·
[必须] 设备标识，用于推送服务、统计数据与记录崩溃数据 · [可选] 查给分时上传的成绩 · 查看隐私协议 · 退出Ham / 拒绝 ·
同意 / 接受 (`APPSTR:12-16`).
**States:** agreed / not agreed (`vm.isAgreePrivacy`, `PrivacyView.kt:64`). No loading, no error state.
**Divergence:** every iOS string is **hardcoded** — none in `Localizable.strings`. iOS is a full-screen page with
a quit-app affordance; Android is a 300.dp dialog whose reject/accept live in the policy sheet, and it omits the
data list. A third `privacy_help` variant exists twice (`APPSTR:25`, `feature/my/…/string.xml:3`) and a second
policy URL (`orangeboychen.github.io/whu-ham/privacy/`, `AND/feature/auth/…/strings.xml:19`) — consolidate both.

---

### 4a. Version changelog (更新日志) — platforms: both
**Purpose:** Post-update "what's new", shown once per installed version.
**Entry:** iOS `MainSheetViewModel.checkVersionLog()` on `didBecomeActive`, only when
`VersionManager.shared.isVersionUpdated()` (`IOS/ui/main/component/MainSheetViewModel.swift:158-178`), presented
from `MainSheetView.swift:41-48`; both sheet dismissal and the button call `setChangeLogRead()` (`:42,46`).
Android `AND/feature/my/…/setting/about/VersionManager.kt:98-108` as a float view when **no** newer version
exists **and** stored `LocalStorage.Key.AppVersion` ≠ current code (written immediately, so it fires once).
**Layout:**
```
┌ HamSheet 85 % · pad top 32 · bottom = nav-bar height ─────┐
│              ▤ document glyph                      72    │ accent
│              更新日志                          title 24   │
│  1.2.3  2026-01-31     title2 20 + caption 12 sec, gap 8 │ date hidden if ≤ 0
│  <changelog title>                          bodyBold 16  │
│  <changelog content markdown>               body 16      │ scrolls on overflow
│                (48 above the button)                     │
│  ┌ 好 · accent@0.15 · r12 · pad-v 16 · fillMaxWidth ───┐ │
└──────────────────────────────────────────────────────────┘
```
**Blocks:** 1 **Hero** — iOS `wand.and.stars` 56pt `.blue` (`VersionChangeLogView.swift:19-21`); Android
`Icons.Rounded.Description` 72.dp `ham_blue` (`VersionInfoSheet.kt:66-71`). 2 **Title** — iOS 本次更新日志
`.title.bold()` (`:22-24`); Android `version_info_title` 更新日志 (`VersionInfoSheet.kt:72-76`, `MYSTR:72`).
3 **Version + date row** — Android only: `title2` 20 + `Date(updateTime).toYYYYMMDD()` caption secondary,
`Bottom`-aligned, spaced 8, date COND `updateTime > 0` (`:97-115`). 4 **Body** — Android renders structured
`title` + `content` (`bodyBold` / `body` via `MarkdownText`, `:116-129`); iOS renders one raw markdown string
through `Text(.init(changeLog))` in a `ScrollView` (`:27-33`). 5 **Confirm** — dismisses and marks read.
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Hero glyph / size / colour | `wand.and.stars` `:19` / 56 / `.blue` `:20-21` | `Icons.Rounded.Description` `:67` / 72.dp / `ham_blue` `:69-70` | **document glyph, 72, accent** |
| Title font + copy | `.title.bold()` / 本次更新日志 (`:23-24`, `LS:722`) | `title` 24 Bold / 更新日志 (`:74`, `MYSTR:72`) | **24 Bold / 更新日志** |
| Version + date row | absent | `title2` + `caption`, gap 8 `:97-115` | **present**, date hidden when ≤ 0 |
| Body source + markdown | one raw markdown string via SwiftUI `Text(.init(_:))` `:29` | structured `title` + `content` via `MarkdownText` `:116-129`, `:42,124` | **structured title + content, markdown, body 16** |
| Body container + scrolling | grey@0.1 r10 pad 12, `ScrollView` `:32,35-36`, `:27` | none, not scrollable `:91` | **none — text on the sheet; scrolls on overflow** |
| Gap above button | `Spacer()` `:38` | 48.dp `:90` | **48** |
| Button visuals | `Color.blue` fill / `.white` text, r10, pad 16, default font, 去使用 (`:42-46`, `LS:593`) | `ham_blue@0.15` fill / `ham_blue` text, r12, pad-v 16, `bodyBold` 16, 好 (`:141-149`, `MYSTR:74`) | **accent@0.15 fill, accent text, r12, pad-v 16, fillMaxWidth, bodyBold 16, 好** |
| Sheet padding | 16 all round `:55` | top 32, bottom = nav-bar height `:62` | **top 32, bottom = nav-bar height** |

**Strings:** 更新日志 · 好 · 未知版本 · 标题 / 副标题 (preview placeholders, `MYSTR:72-76`); iOS 本次更新日志 · 去使用
(`LS:722,593`).
**States:** **empty — do not present the sheet** (Android renders nothing when `changeLogBody == null`,
`VersionInfoSheet.kt:58`; iOS today shows an empty grey box). No loading, no error.
**Divergence:** iOS has no version/date row, wraps the body in a grey r10 card, uses a solid blue button, and
shows an empty container on a null payload.

---

### 4b. Version update sheet (发现新版本) — platforms: Android only
**Purpose:** Prompt to install a newer build, escalating below the minimum supported version, offering Play Store
and/or direct-APK download.
**Entry:** `AND/feature/my/…/setting/about/VersionManager.kt:84-93` — a `FloatViewManager` float view when
`newVersionInfo.versionCode > current`; `usePlayStore` from CCKV `EnableGooglePlayStoreDownload`.
**Layout** (`AND/feature/my/…/version/VersionUpdateSheet.kt:63-225`):
```
┌ HamSheet 85 % · pad top 32 · bottom = nav-bar height ─────┐
│     ↻ Update glyph 72 · 发现新版本 title 24 · accent       │
│  当前版本过低，请及时更新  title3 16 ham_red · COND low-ver │
│  1.2.3  2026-01-31   title2 20 + caption 12 sec, gap 8    │
│  <changelog title> bodyBold 16 │ <content> body 16  COND  │ plain text
│                    (48 above · button gap 8)              │
│  ┌ 从PlayStore下载  COND usePlayStore ─────────────────┐  │ tinted pill
│  ├ 从浏览器下载 ─── COND updateUrl non-empty ──────────┤  │
│  └ 关闭 ────────── always · text-only accent ─────────┘  │
└──────────────────────────────────────────────────────────┘
```
**Blocks:** 1 **Hero** `Icons.Rounded.Update` 72.dp `ham_blue` (`:72-77`). 2 **Title** 发现新版本 `title` 24 Bold
(`:78-82`). 3 **Hard-block warning** — COND `App.VERSION_CODE < minVersionCode`, `title3` 16 `ham_red`
(`:83-89`). 4 **Version + date row** — `title2` 20 + `Date(updateTime).toYYYYMMDD()` caption secondary, spaced 8;
row COND versionName non-empty, date COND `updateTime > 0` (`:111-129`). 5 **Changelog** — `title` bodyBold +
`content` body, each COND non-empty (`:130-143`), **plain `Text`, not markdown** (unlike §4a). 6 **Buttons**
`Column` spaced 8 (`:150`): Play Store → `ACTION_VIEW` forced onto `com.android.vending` with an
`ActivityNotFoundException` fallback (`:151-180`); browser → `ACTION_VIEW` on `updateUrl` (`:182-206`); 关闭 →
`dismissAction` (`:208-221`).
**Values:**

| Element | Android | normative |
|---|---|---|
| Hero glyph / size / colour | `Icons.Rounded.Update`, 72.dp, `ham_blue` `:73-76` | update glyph, 72, accent |
| Title + warning | 发现新版本 `title` 24 Bold `:80-81`; 当前版本过低，请及时更新 `title3` 16 `ham_red` `:86-87` | **title 24 Bold; low-version warning 16 `ham_red`** |
| Version / date | `title2` 20 + `caption` 12 secondary, gap 8 `:118-125` | as §4a |
| Changelog | `bodyBold` 16 + `body` 16, plain `Text` `:133,140` | **plain text, no markdown** |
| Gap above buttons / between | 48.dp `:103` / 8.dp `:150` | 48 / 8 |
| Buttons | downloads r12, `ham_blue@0.15`, pad-v 16, `bodyBold` accent, fillMaxWidth `:168-177`; 关闭 accent text, pad-v 16, no fill `:210-219` | downloads = shared tinted pill; 关闭 = text-only accent button |
| Sheet padding | top 32, bottom = `navigationBarHeight()` `:68` | as §4a |

**Strings:** 发现新版本 · 当前版本过低，请及时更新 · 从PlayStore下载 · 从浏览器下载 · 关闭 (`MYSTR:66-70`).
**States:** `getVersionResponse == null` → **nothing renders** (`:64`, no else branch — never present an empty
update sheet). No loading or error state; the caller logs and returns on request failure
(`VersionManager.kt:60-71`).
**Divergence:** **iOS has no equivalent** — no update prompt anywhere in the iOS tree, only the post-update
changelog (§4a). No iOS norm is invented; file it as a parity gap (iOS today cannot prompt at all).

---

### 5. Debug (调试) — platforms: both
**Purpose:** Developer-only switch between the production backend and a custom HTTP/gRPC endpoint; debug builds
only.
**Entry:** iOS `#if DEBUG` row on the My tab (`IOS/ui/my/component/card/MyViewSettingCard.swift:121`, gated
`:124`) and `#if DEBUG` route `.debug` (`IOS/ui/main/Route.swift:103,245`); the view is itself wrapped in
`#if DEBUG` (`DebugView.swift:11,120`). Android nav route in the My graph
(`AND/feature/my/…/ui/MyView.kt:55`) — **not** build-gated.
**Layout:**
```
┌ ◀ Debug · scroll · pad 16 · cards spaced 16 · bg ham_bg_b1 ─┐
│ ┌ card Environment · pad 16 · inner gap 8 ───────────────┐ │
│ │ Production                              [switch]       │ │ bodyBold 16
│ │ Using production server config…   caption 12 secondary  │ │
│ └────────────────────────────────────────────────────────┘ │
│ ░ COND !useProductionEnv — animated reveal (slide + fade) ░ │
│ ┌ card HTTP Server · inner gap 12 ───────────────────────┐ │
│ │ Changes take effect on the next HTTP request.  caption  │ │
│ │ Base URL · [outlined field r12, URL keyboard]   bodyBold│ │
│ │ (4) · Save                        bodyBold 16 accent    │ │
│ └────────────────────────────────────────────────────────┘ │
│ ┌ card gRPC Server — same shape, + Use TLS (mTLS) [switch]│ │
│ │ …next gRPC request · Base URL · field · (4) · Save      │ │
│ └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```
**Blocks:** 1 **Environment toggle** — flips `debug_useProductionEnv` /
`LocalStorage.Key.DebugUseProductionEnv`, default **true** (`DebugView.swift:14-15,35`; `DebugView.kt:44,94-97`);
label reads Production/Test (`DebugView.swift:37`). 2 **Conditional reveal** — iOS plain `if`
(`DebugView.swift:50`); Android `AnimatedVisibility` (`:102`) — **the animation is normative**. 3 **Base-URL
fields** — storage values seeded into a separate editing buffer on appear (`:114-117`; Android
`remember(currentValue)` `:61-62`), `.trim()`ed on save (`:68,99`; `:134,193`). 4 **Save** — per card, writes the
trimmed value back. 5 **Defaults** — HTTP `https://api.ham.nowcent.cn`, gRPC `https://api.ham.nowcent.cn:4443`
(`:18,21`; `:47,50`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Container / card | `ScrollView` + `VStack(16)`, pad 16, `ham_bg_b1Color` `DebugView.swift:30-31,109,112`; card r16/`ham_bg_b2`/pad16 `CardView.swift:23,22,128` | `HamNavigationScrollView` + `Column(16.dp)`, pad 16, `ham_bg_b1` `DebugView.kt:64-68`; card r16/`ham_bg_b2`/pad16 `Card.kt:53,42,48` | scrolling column, pad 16, gap 16, card token |
| Card title | `.semibold` `ham_text_t1Color` `CardView.swift:60-61` | `bodyBold` 16 `ham_text_primary` `Card.kt:81` | **bodyBold 16** |
| Env-card inner gap | 12 `:34` | 8.dp `:74` | **8** |
| Server-card inner gap | 12 `:52,78` | 12.dp `:108,149` | 12 |
| Status label | `.bold()` `ham_text_t1Color` `:38-39` | `bodyBold` 16 `:82-83` | bodyBold 16, `ham_text_primary` |
| Helper text | `.caption` 12 `ham_text_t2Color` `:43-44` | `caption` 12 `ham_text_secondary` `:91` | caption 12, secondary |
| Field | `.roundedBorder` `:62` | `OutlinedTextField` r12 `:120,125` | **outlined, r12, single-line** |
| Field keyboard | `.keyboardType(.URL)`, no autocapitalise/correct `:63-65` | unconfigured | **URL keyboard, no autocorrect** |
| Field placeholder | `https://api.ham.nowcent.cn` `:61` | same `:127` | the current default value |
| Save + spacer above | `.bold()` `.blue`, no spacer `:71-72` | `bodyBold` 16 `ham_blue`, 4.dp spacer `:131,138-139` | **bodyBold 16 accent, 4 above** |
| Reveal | none | `AnimatedVisibility` `:102` | **animated reveal** |
| Build gating | `#if DEBUG` `:11,120` | none (comment only `:39`) | **compile-gated to debug builds** |

**Strings:** all English, all hardcoded on both — `Environment` · `Production` · `Test` ·
`Using production server config. HTTP/gRPC settings are locked.` · `Using custom server config. You can edit
HTTP/gRPC settings below.` · `HTTP Server` · `Changes take effect on the next HTTP request.` · `Base URL` ·
`Save` · `gRPC Server` · `Changes take effect on the next gRPC request.` · `Use TLS (mTLS)` · `Debug`
(iOS `DebugView.swift:33,37,41-42,51,53,57,71,77,79,94,113`; Android `DebugView.kt:64,71,81,87-88,105,110,116,137,146,151,157,179,196`).
**States:** production (server cards hidden) / editable. No loading, empty, or error state.
**Divergence:** iOS is compile-gated, Android is not (normative: gate it); iOS uses a plain `if`, no URL
keyboard, 12 env-card spacing and `.roundedBorder`. Both files were AI-generated on the same date and share copy.

---

### 6. Automatic / Siri shortcuts (自动化) — platforms: both, different features
**Purpose:** Settings for unattended automation. **The two platforms ship different capabilities under one
label** — iOS donates a Siri Shortcut for library booking; Android schedules an alarm that auto-books a seat.
Only the entry label and the hint card are shared.
**Entry:** iOS route `.automatic` → `MySiriView()` (`Route.swift:99,220`) and a My-tab row
(`MyViewSettingCard.swift:49`); Android `MyViewRoute.AUTOMATIC_SETTING` from the settings hub
(`AND/feature/my/…/setting/SettingView.kt:57`).
**Layout (iOS `IOS/ui/siri/MySiriView.swift:11-25` / Android `AND/feature/automatic/…/ui/AutomaticSettingView.kt:50-125`):**
```
iOS (pad 16)                │ Android (scroll · pad 16 · gap 16)
┌ [nav inline] 自动化 ────┐  │ ┌ hint card · pad 12 · Alarm icon 128 ───┐
│ 通过Siri预约图书馆  cap │  │ │ 提示                  bodyBold accent │
│ ┌ [ Add to Siri ] h 60 ┐│  │ │ (8) 通过选定时间，实现自动预约图书馆…  cap│
│ └─────────────────────┘│  │ │ 要正常使用自动化功能，需开启本应用"自启动"…│
│ (Spacer)               │  │ └───────────────────────────────────────┘
└────────────────────────┘  │ ┌ card · gap 16 · animateContentSize ────┐
                            │ │ 图书馆 · 自动快速预约 [switch] · ░COND░ │
                            │ │ 时间 [time picker]    slide + fade      │
                            │ └───────────────────────────────────────┘
```
**Blocks (iOS):** 1 explanatory caption 通过Siri预约图书馆, `.caption` `.secondary` (`:13-15`).
2 `SiriButtonView(shortcut: .libraryQuickBookIntent)` at `height 60` (`:16-17`) — the system Add-to-Siri
control; *tap* donates the shortcut. 3 spacer (`:18`); logging tag `AutomaticView` (`:20`). Related:
`IOS/ui/siri/components/MyHanmuRunShortcutController.swift`.
**Blocks (Android):** 1 **Hint card** — `HamCardView(padding 12, icon = Icons.Rounded.Alarm, size = 128.dp)`
(`:55-59`): 提示 in `bodyBold` `ham_blue` (`:60-64`), 8.dp gap (`:65`), `automatic_hint_description` caption
(`:66-70`), `automatic_tip` caption on autostart + notification permission (`:71-75`). 2 **Library card** —
`Column.animateContentSize()` spaced 16 (`:78-82`): 图书馆 `bodyBold` (`:84-88`); row 自动快速预约 `body` +
`Spacer(1f)` + `HamSwitch` → `vm.updateEnableLibrary(it)` (`:89-99`); `AnimatedVisibility(enableLibrary)` with
`slideInVertically + fadeIn` (`:101-105`) revealing 时间 + `HamTimePickerButton(date = libraryTime)` →
`vm.updateLibraryTime(it)` (`:106-116`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Nav title | 自动化 (`LS:35`) | 自动化操作 (`AUTOSTR:4`) | **自动化操作** |
| Explainer | caption 12 secondary, bare text `:13-15` | hint card — `bodyBold` accent title + 2× caption `:55-76` | **hint card, title + 2 caption lines** |
| Hint icon / pad | none | `Icons.Rounded.Alarm` 128.dp `:57-58` / 12.dp `:56` | alarm glyph 128 / pad 12 |
| Controls | Add-to-Siri, h 60 `:16-17` | `HamSwitch` `:96-98` + `HamTimePickerButton` `:113`, revealed slide+fade `:101-105` | platform-native (not merged) |
| Screen pad / gap | 16 / — `:21` | 16.dp / 16.dp `:52-53` | 16 / 16 |

**Strings:** iOS 自动化 (`LS:35`) · 通过Siri预约图书馆 (`LS:59`). Android 自动化操作 · 提示 ·
通过选定时间，实现自动预约图书馆或完成每日计划。 · 〈autostart / notification tip〉 · 图书馆 · 自动快速预约 · 时间
(`AUTOSTR:3-9`), plus alarm notification copy for running / success `已预约%1$s` / ignored (not-logged-in,
no-seat) / failed (`AUTOSTR:10-19`).
**States:** iOS — donated / not. Android — automation on / off; alarm outcomes surface as notifications. No
loading or empty state.
**Divergence:** no shared layout, strings or states — different features behind one name. iOS has no enable
switch or time picker; Android has no Siri surface.

---

### 7. Share sheet (分享) — platforms: iOS only
**Purpose:** Bottom sheet offering QQ share and the system share sheet for a URL.
**Entry:** app-level overlay `IOS/ui/ContentView.swift:40`; triggered by the `.ham_share` notification, only for
`type == "url"` with a `SocialShareUrlModel` payload (`ShareView.swift:29,31-39`).
**Layout** (`IOS/ui/common/share/ShareView.swift:12-42`, sheet body `:72-123`):
```
┌ scrim black@0.5 · ignoresSafeArea · tap → dismiss ─────────┐
│ ┌ sheet · surface · r16 top corners · pad 16 + bottom 56 ┐ │
│ │ 分享到                                          body    │ │ :80
│ │  ┌──────┐  ┌──────┐                    HStack :82      │ │
│ │  │  QQ  │  │  ⬆   │     56×56 circles  :91,:106        │ │
│ │  └──────┘  └──────┘     QQ #12B7F5 · system gray@0.25   │ │
│ └────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────┘
  transition .move(edge: .bottom) :24 · .ignoresSafeArea :121
```
**Blocks:** 1 **Scrim** `black@0.5` full-bleed, tap → `reset()` (`:16-20`). 2 **Title** 分享到 (`:80`).
3 **QQ** — COND `UIApplication.shared.canOpenURL("mqq://")` (`:83`); asset `QQ-2` `.aspectRatio(.fit)` inner pad 8,
framed 56×56 on a `Circle()` filled `#12B7f5` (`:87-96`); *tap* → `shareUrlToQQ(model:)`. 4 **System** —
`square.and.arrow.up.fill` 24pt `ham_text_t1Color`, 56×56 on `gray@0.25` (`:103-110`); *tap* →
`shareUrlToSystem(model:)` (`:44-56`). 5 **Dismiss** — `reset()` clears show/type/data with animation (`:58-64`).
**Values:**

| Element | iOS | Android | normative |
|---|---|---|---|
| Scrim + transition | `black@0.5` `:16`, `.move(edge: .bottom)` `:24` | — | black @ 0.5 tap-to-dismiss; slide in from the bottom |
| Sheet surface / radius / padding | `Color.white`, r16 top corners, pad 16 + bottom 56 `:115-119` | — | **`ham_bg_b2` (adaptive), r16 top corners, pad 16 + bottom 56** |
| Icon container | 56×56 `Circle()` `:91,106` | — | 56×56 circle hit target |
| QQ fill / glyph | `#12B7F5` / asset `QQ-2`, inner pad 8 `:87-94` | — | QQ brand blue, inner pad 8 |
| System fill / glyph | `gray@0.25` / SF symbol 24, `ham_text_t1Color` `:103-109` | — | **`ham_text_secondary`@0.15 fill, `ham_text_primary` glyph 24** |

**Strings:** 分享到 (`:80`) — move to `Localizable.strings` (hardcoded today).
**States:** shown / hidden; the QQ target is hidden when QQ is not installed. No loading, error or empty state.
**Divergence:** **Android has no equivalent** — it fires the system share intent directly
(`AND/feature/print/…/PrintShareFileLoadingView.kt`, `MyViewLinkCard.kt:90,116`). No Android norm is invented;
parity would mean an Android in-app share row.
> **Bug (normative: fix).** `Color.white` (`:118`) and `ham_text_t1Color` (`:105`) are not adaptive — dark mode
> renders a white sheet with light-mode foreground.

---

## Appendix — audit sections outside this spec

Standalone screens in `/tmp/audit/shared.md` not named in this file's brief. **Not spec'd here**; each needs its
own section or another agent's slice.

| § | Screen | Platforms | Note |
|---|---|---|---|
| 1 | Intro / connect (连接页) | both | shared shell — belongs to the shared-components slice |
| 12 | About (关于) | both | `IOS/ui/about/AboutView.swift:11-53` + `feature/my/…/setting/about/AboutView.kt`; logo plate, version, links, 10-tap dev unlock |
| 21 | Scan code / QR login (扫码) | both | scanner + QR-login confirmation — two surfaces |
| 22 | SSO authorization sheet | both | `shared.md:1513-1641` — the largest unclaimed standalone |
| 23 | Login screen (登录Ham) | both | `shared.md:1642-1747` |
| 24 | CAS settings (信息门户设置) | both | `shared.md:1748-1844` |
| 25a | Settings hub (设置) | Android only | `SettingView.kt:40-90`; iOS inlines the rows on the My tab |
| 25b, 25c, 25i | Language / widget / promotion settings | Android only | no iOS equivalent for language (iOS follows the system locale) or widget settings |
| 25d | Print / share-file flow | both | iOS `PrintPrepareView` (330 ln) + `PrintFileSourcePicker` + `PrintActionExtension`; Android `feature/print/` 5 surfaces, incl. the only spinner+label pairing in the codebase |
| 25e, 25g, 25h | Social-account binding, authorized apps, passkey config | iOS only | `SyncSocialAccountView.swift:23-90`; `IOS/ui/sso/AuthorizedAppsView.swift`; Android has passkey *login* only (`LoginView.kt:202-232`) |
| 25f | User center / profile info | both | flagged in the audit as **not extracted** — needs a follow-up pass |
| 26 | Cross-cutting findings | — | 17 ranked findings (dead iOS components, inverted success/error icon colours, the r8/r12 button triple standard, unlocalised iOS strings, unused `ham_divider` token) — belong in the top-level spec |
