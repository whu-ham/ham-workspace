# Ham UI parity

The work list for bringing the two clients' interfaces into line.

This is **not** the specification. The specification is `screens.md` (129 sections — 120 screens
plus 9 analysis sections — each with an
`iOS` / `Android` / `normative` column per element) plus `design-system.md` (the tokens). This
document is the gap list derived from them: for every page, what each platform currently renders,
what the agreed value is, and which side moves.

Wording divergences are **not** here — see `copy-and-strings.md` §7. Behaviour divergences are
**not** here — see `logic-parity.md`.

## Contents

- [1. How to read this](#1-how-to-read-this)
- [2. Summary by module](#2-summary-by-module)
- [3. Status dashboard](#3-status-dashboard)
- [4. Course timetable](#4-course-timetable)
- [5. Schedule](#5-schedule)
- [6. Library](#6-library)
- [7. Sport](#7-sport)
- [8. Score](#8-score)
- [9. CourseScore](#9-coursescore)
- [10. My and user center](#10-my-and-user-center)
- [11. Auth and sign-in](#11-auth-and-sign-in)
- [12. Shared components](#12-shared-components)
- [13. Standalone screens](#13-standalone-screens)
- [14. Cross-cutting](#14-cross-cutting)
- [15. Motion](#15-motion)
- [16. Accessibility](#16-accessibility)
- [17. Non-content states](#17-non-content-states)
- [18. Screens already at parity](#18-screens-already-at-parity)
- [19. Patterns](#19-patterns)

---

## 1. How to read this

One row per divergence:

| Page | Element | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- | --- |
| … | … | current value | current value | the value in `screens.md` | which platform moves, and to what |

Severity markers:

- 🔴 **visible without comparison** — a user switching devices notices immediately.
- 🟠 **visible side by side** — needs the two screens next to each other.
- 🟡 **only measurable** — extractable from code, hard to see.

`Change` names the platform and the target value. Where both platforms must move, both are
named. Where the agreed value is a new choice neither side has made, it is marked **[new]**.

---

## 2. Summary by module

| Module | Screens | Notes |
| --- | --- | --- |
| Status dashboard | 10 | Card container differs, so every card inherits it. All 7 cards exist on both — an earlier "2 of 7 missing on Android" was wrong |
| Course timetable | 7 | Grid metrics differ on all four axes |
| Schedule | 5 | Countdown semantics differ; group controls and hero content differ |
| Library | 12 | Largest numeric spread; print ships on both — only the arrow size and share-hint copy differ |
| Sport | 13 | Card order is swapped; decoration sizes differ throughout |
| Score | 12 | Function-card fill and score-row typography are the visible ones |
| CourseScore | 8 | Closest module — the filter chip already matches exactly |
| My and user center | 31 | Header collapse, plus three Android-only screens |
| Auth and sign-in | 8 | SSO sheet and authorized apps already match |
| Shared components | 15 | Sheet, toast and text field affect every screen |
| Standalone | 8 | Bus, pay, privacy, changelog, debug, automation, share |

---

## 3. Status dashboard

| Element | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- |
| Card radius | 16 | 12 | **16** | Android → 16. Affects all 5 Android cards. 🔴 |
| Card header padding | 12 all sides | v12 / h16 | **12 all sides** | Android → 12 horizontal. 🟠 |
| Card content padding | 12 (parameter, bus passes 0) | 16 | **12** | Android → 12, and add a `padding` parameter so the bus card can pass 0. 🟠 |
| Cards rendered | 7 (weather, library, course, bus, sport, schedule, casAlert) | **7** — same seven; all six types composed in the card-order loop, `StatusView.kt:231-253`, CAS card above it `:229` | **7** | none 🔴→✅. Previously read as "5 — no sport, no schedule"; that was wrong. `StatusViewCardType` lists `Sport` (`StatusViewCardScoreManager.kt:40`) and `StatusSportCard.kt` is 212 lines of real UI. **The real work is the container** (radius 16 vs 12, padding 12 vs 16), which every card inherits — see the three rows above. |
| CAS alert card | Solid red pill, white text, 信息门户登录失败 + 重新登录 | Message only, no button | **Message + 重新登录 button** | Android: add the action. 🔴 |
| Weather — temperature | 36, rounded | 28 | **36 rounded** | Android → 36 with rounded figures. 🔴 |
| Weather — attribution | Links Apple's WeatherKit attribution | **none** | **Credit the data source** | Android: add a source credit. This is compliance, not cosmetics. 🔴 |
| Weather — no permission | 未获取地理位置权限 | 未获取地理权限 | **未获取地理位置权限** | Android. 🟡 |
| Weather — error | Shows raw `error.localizedDescription` (unlocalised system text) | 获取天气数据时遇到了异常 | **遇到了错误** | Both: iOS must stop surfacing system text; Android must drop 异常. 🔴 |
| Library — sub-minute | 还有不到一分钟 | 还有不到1分钟 | **还有不到1分钟** | iOS. 🟡 |
| Library — extra states | 不久后, 已结束 | absent | **both present** | Android: add. 🟠 |
| Course — all done | 本周的课程已经全部结束啦～辛苦啦！💪 | 今日课程已上完 | **今日课程已上完** | iOS: replace. Wrong scope (本周 vs 今日) and wrong register. 🔴 |
| Course — view toggle | 周视图 / 日视图 (labels *current* mode) | 切换到日视图 / 切换到周视图 (labels *target*) | **label the target** | iOS. 🔴 — opposite semantics, so one platform is always wrong. |
| Bus — distance | `328m` (bare) | 距你328m | **距你328m** | iOS: add the prefix. 🟠 |
| Bus — states | 已到站, 即将到达 | 到达 only | **both** | Android: add 即将到达. 🟠 |
| Bus — load error | 获取地理位置异常 / 更新校巴信息异常 | 加载时遇到了错误 | **遇到了错误** | Both: drop 异常. 🟠 |
| Pull to refresh | hand-rolled pill, one screen: drag past **128 pt** → 已请求刷新, release fires `onRefresh()` — `StatusUpdateView.swift:15,57-63` | native `rememberPullToRefreshState`, one screen, **300 px** — `StatusContainerView.kt:157`; **+56 `BounceScrollView` sites that bounce without refreshing** | **present on every list that can go stale** | Both, 🟠: refresh exists on exactly one screen each, at thresholds differing >2×. Android must stop bouncing where it will not refresh; both must extend the gesture past the status dashboard. |

---

## 4. Course timetable

| Element | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- |
| Period rail width | 42 | 32 | **42** | Android → 42. Shifts every column. 🔴 |
| Weekday row height | 36 | 48 | **36** | Android → 36. 🔴 |
| Grid cell radius | 10 | 12 | **10** | Android → 10. 🟠 |
| Grid cell padding | 2 | 4 | **2** | Android → 2. 🟠 |
| Today cell fill | Solid `surface.tint` | `accent` @ 0.15 | **solid `surface.tint`** | Android. 🟠 |
| Empty cell fill | Solid `surface.tertiary` | `text.secondary` @ 0.15 | **solid `surface.tertiary`** | Android. 🟠 |
| Course name | 11 Bold | 12 Bold | **12 Bold** | iOS → 12. 🟡 |
| Instructor / location | 11 | 12 | **12** | iOS → 12. 🟡 |
| Weekday date label | 11 | 12 | **12** | iOS → 12. 🟡 |
| Header height | 50 | 56 | **50** | Android → 50. 🟠 |
| Header horizontal padding | 10 | 12 | **10** | Android → 10. 🟡 |
| Dark-mode overlay | black @ 0.25 | absent | **present** | Android: add. 🟠 |
| Background image | `VisualEffectBlur` behind cells | alpha only, no blur | **blur behind cells** | Android: add. 🟠 |
| Drag threshold | 60pt | 20px | **one value** | Android → 60. 🟡 |

See `logic-parity.md` §2 for the course-colour hash divergence — the same course is a different
colour on each platform, which is a logic bug with a visible UI symptom.

---

## 5. Schedule

| Element | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- |
| Countdown target | `end` once the item has started | `begin` always | **`end` once started** | Android: use `end` after start. 🔴 — during a 10:00–12:00 event at 11:00 both show "60" but iOS means remaining and Android means something else. |
| Countdown size | 50 Bold | 50 Bold | **50 Bold** | already matches |
| Countdown panel | `#01579B`, flips when passed | present | **`#01579B`** | verify the flip on Android. 🟠 |
| Hero — location | shown | absent | **shown** | Android: add. 🟠 |
| Hero — related course | shown | absent | **shown** | Android: add. 🟠 |
| Hero hides on scroll | yes, for long lists | no | **yes** | Android: add. 🟠 |
| Group tab badges | count badges present | absent | **present** | Android: add. 🟠 |
| Group panel preview | next upcoming schedule | arbitrary first schedule | **next upcoming** | Android: pick the next, not the first. 🟠 |
| Detail popover | 300 wide | — | **300 wide** | Android: match. 🟠 |
| List row spacing | 16 | 8 | **8** | iOS → 8. 🟡 |
| Time picker state | 24h correct | seeded from 12-hour `Calendar.HOUR`, rendered as `HH:mm` | **24h** | Android: fix — PM times reopen as AM. This is a 🟠 bug, not just a value. |

Schedule alarms and calendar integration are **absent on Android entirely** — that is a capability
gap, recorded in `logic-parity.md` §7.

---

## 6. Library

| Element | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- |
| Gap between cards | 8 (implicit) | 16 | **8** | Android → 8. 🔴 |
| Banner height | 200 | 180 | **200** | Android. 🔴 |
| Banner title | 28 Bold | 24 Bold | **28 Bold** | Android. 🟠 |
| Banner watermark icon | 36 @ 0.25 | 72 @ 0.20 | **36 @ 0.25** | Android. 🟠 |
| Banner watermark grid | 6 × 9 | 5 × 10 | **6 × 9** | Android. 🟡 |
| Banner page indicator | present when >1 page | absent | **present** | Android. 🟠 |
| Analytics — group name | 17 (no style set) | 12 | **12** | iOS: set `.font(.caption)`. 🔴 |
| Analytics — bar height | 30 | 20 | **30** | Android. 🟠 |
| Analytics — percentage | 17 (no style set) | 20 | **17** | Android → 17, iOS: set it explicitly. 🟠 |
| Booking status bar | v8 | v12 | **v8** | Android → 8. 🟠 |
| Booking — 添加到系统日历 | commented out | present | **present** | iOS: enable. 🟠 |
| Booking — 在地图打开 | works | **no-op** | **works** | Android: implement. 🔴 |
| Quick-book seat number | 28 Bold | 32 (raw literal) | **28 Bold** | Android: use the token. 🟠 |
| Quick-book location | 12 | 16 | **12** | Android. 🟠 |
| Quick-book seat badge | 收藏座位 / 上次预约, orange / green | absent | **present** | Android: add. 🟠 |
| Quick-book button radius | 10 | 12 | **10** | Android. 🟡 |
| Quick-book button fill | `accent` @ 0.10 | `accent` @ 0.25 | **@ 0.15** | Both → 0.15. 🟠 |
| Quick-book item spacing | 0 | 12 | **8** | Both → 8. 🟡 |
| Function row height | 150 | 142 | **150** | Android. 🟠 |
| Function small arrow | 8 | 20 | **12 [new]** | Both → 12. 🟠 |
| Function subtitle | 11 | 12 | **12** | iOS → 12. 🟡 |
| Settings — account row | 账号信息 / 你的图书馆系统认证信息 | 图书馆账号信息 / 用以预约座位… | **账号信息** + iOS subtitle | Android. 🟡 |
| Captcha row | 验证码设置 | 验证码识别设置 | **验证码识别设置** | iOS. 🟡 |
| Print card | present | present | **present on both** | Both, small: unify the share-hint string (iOS `或者通过其他应用分享文件到Ham中打印`), iOS arrow **12** vs Android `20.dp`, Android must stop re-fetching the printer list on every recomposition, and Android's `打印任务` card is dead code. 🟡 |
| Print drop zone · printer row | `168`/r 16 · pad 16/r 16/48 | `168.dp`/r 16 · pad 16/r 16/48 | **identical** | none — already at parity. |
| History grouping | grouped by date with totals | flat list | **grouped by date** | Android. 🟠 |

---

## 7. Sport

| Element | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- |
| **Card order** | banner → orders → divider → quick-order | banner → quick-order → orders | **iOS order** | Android: swap. 🔴 |
| Banner height | 200 | 180 | **200** | Android. 🟠 |
| Banner title | 28 | 24 | **28** | Android. 🟠 |
| Banner watermark | 36 @ 0.25, 6 × 9 | 56 @ 0.20, 5 × 10 | **36 @ 0.25, 6 × 9** | Android. 🟡 |
| Area badge | 32, 22 rounded bold | 36, 24 bold | **32, 22 rounded bold** | Android. 🟠 |
| Current-order — watermark | 128 @ 0.15 | absent | **128 @ 0.15** | Android: add. 🟠 |
| Current-order — footer gap | 8 top / 16 sides | 12 all | **8 / 16** | Android. 🟡 |
| Pay button radius | 6 | 8 | **8** | iOS. 🟡 |
| Pay button font | 17 | 14 | **17** | Android. 🟡 |
| Quick-order star badge | fill @ 0.15, icon ~12 | fill @ 0.25, icon 18 | **@ 0.15, icon 12** | Android. 🟠 |
| Quick-order button | v-padding 16, fill @ 0.25, label-colour text | v-padding 12, fill @ 0.20, brand text | **v-padding 16, fill @ 0.25, brand text** | Both. 🟠 |
| Day picker | native segmented | custom (track r6, thumb r5) | **one control** | Android: fix the thumb/track mismatch; iOS: adopt if a shared control is wanted. 🟠 |
| Function row height | 150 | 160 | **150** | Android. 🟠 |
| Function large icon | 64 | 72 | **64** | Android. 🟠 |
| Function trailing icon | 32 @ 0.75 | 56 @ 0.65 | **32 @ 0.75** | Android. 🔴 |
| Bulletin list screen | present | **absent** (banner links to detail) | **present** | Android: add. 🟠 |
| Order success dismiss | 返回 (outlined) | 完成 (filled) | **完成** | iOS. 🟡 |
| Order error | 预定失败, red icon, 64 | grey icon, generic message | **预定失败**, red | Android. 🔴 |
| Order loading text | 正在预约 / 很快就好 | bare spinner | **正在预约** | Android: add. 🟠 |
| Venue placeholder | 请选择运动类别 | 请选择场所 | **请选择运动类别** | Android. 🟡 |
| Favourite day entry | 隔天 / 当天 | 明天 / 今天 | **明天 / 今天** | iOS. 🟡 |
| Other-settings row | 其他设置 | 其它设置 | **其他设置** | Android. 🟡 |
| Login / expire time | `登录时间：%@` (full-width, no space) | `登录时间: %1$s` (ASCII + space) | **full-width, no space** | Android. 🟡 |

---

## 8. Score

| Element | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- |
| Function-card background | `text.secondary` @ 0.10 (translucent) | **solid `#EDEEEF`** | **@ 0.10 translucent** | Android. 🔴 |
| Function-card radius | 16 | 12 | **16** | Android. 🟠 |
| Score row — instructor | 12 (caption) | 16 (same as course name) | **12** | Android. 🔴 |
| Score row — colour bar | 35 tall, radius 6 | 28 tall, radius 3 | **35 / 6** | Android. 🟠 |
| Score row — spacing | 15 | 8 | **8** | iOS. 🟠 |
| Score row — value | 22 | 24 | **22** | Android. 🟡 |
| GPA | 28 | 24 | **28** | Android. 🟠 |
| Semester divider padding | 12 | 4 | **8 [new]** | Both. 🟡 |
| Year-row spacing | 5 | 4 | **4** | iOS. 🟡 |
| Watermark | 220 @ 0.10, offset (60, 20) | 240 @ 0.15, offsetY 64 | **220 @ 0.10** | Android. 🟡 |
| Chevron | 8 | 24 | **12 iOS / 16 Android [new]** | Both. 🟠 |
| Pinned select header | `VisualEffectBlur` | flat `text.secondary` @ 0.25 over `surface.primary` | **one treatment** | Android: add blur, or agree on flat. 🟠 |
| Select-mode spacer | 96 | 92 | **96** | Android. 🟡 |
| Gap before semester list | 24 | 8 | **8** | iOS. 🟡 |
| Stat card in select mode | hidden | always shown | **hidden** | Android: hide. 🟠 |
| Comment-button box | 32, radius 8 | 28, radius 8 | **32** | Android. 🟡 |
| Update sheet — result | 更新成功 / 更新失败 | **获取成功 / 获取失败** | **获取成功 / 获取失败** | iOS: change the verb. See `copy-and-strings.md` §2 row 5. 🟠 |
| Update sheet — progress | 正在更新 | spinner only | **正在更新** | Android: add. 🟠 |
| Last-updated format | `%@更新` | `%1$s 更新` | **no space** | Android. 🟡 |

Numeric precision (GPA `%.2f` vs `%.6f`, credits `%.1f` vs `%.2f`) is a copy-and-format issue —
`copy-and-strings.md` §3.

---

## 9. CourseScore

The closest module to parity. The filter chip is already an exact match on all eight metrics —
use it as the reference when standardising elsewhere.

| Element | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- |
| Filter chip | r6, h6 / v4, @ 0.10, 12 | identical | **identical** | none ✅ |
| Search placeholder | 输入课程名或授课人 | 输入关键词 | **输入课程名或授课人** | Android. 🟡 |
| Average precision | `%.1f` | `%.2f` | **`%.1f`** | Android. See `copy-and-strings.md` §3. 🟡 |
| Band labels | client-side | client-side, different splits | **one rule** | TBD — see `logic-parity.md` §4. 🟡 |
| TOP1 badge | all-caps Latin, untranslated | same | **localise or drop** | Both. 🟡 |
| Sort chip | includes 未知倒序 | — | **drop the case** | iOS. 🟠 |
| Comment length error | 评论长度不合法 | 评论字数未符合要求 | **评论字数未符合要求** | iOS. See `copy-and-strings.md` §7.3. 🟡 |

---

## 10. My and user center

| Element | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- |
| Header collapse threshold | 50 | — | **50** | Android: confirm it collapses. 🟡 |
| Header avatar | 108 | 108 | **108** | matches ✅ |
| Header title when signed out | remote-config title or 未登录 | absent | **未登录** | Android: add. 🟠 |
| Profile card (logged out) | 点击登录 + 不登录也可以使用校内功能哦 | **absent** | **present** | Android: add. 🔴 |
| CAS rows | 管理信息门户设置 / 登录信息门户 / 使用校内服务的前提 | **absent** | **present** | Android: add. 🟠 |
| Settings rows | 自动化 · 使用指南 · 反馈 · 关于 | 自动化操作 only | **all four** | Android: add 使用指南, 反馈, 关于. 🟠 |
| Function grid | 8 module tiles + link rows | 8 module tiles | **identical set** | matches ✅ |
| Board card | remote-config markdown | same | **identical** | matches ✅ |
| Promotion card | remote-config | same | **identical** | matches ✅ |
| Settings hub screen | **absent** (a card, not a screen) | present | **decide** | Product decision — see §14. 🔴 |
| Language screen | absent | present | **Android-only is correct** | platform rule, no change |
| Widget settings screen | absent | present | **Android-only is correct** | platform rule, no change |
| User-center title | absent | 个人中心 | **个人中心** | iOS: add. 🟡 |
| Device last-seen | 最近上线：%@ | absent | **present** | Android: add. 🟡 |
| Social unbind | 解绑 | absent | **present** | Android: add the action (copy exists, action does not). See `logic-parity.md` §8. 🟠 |
| Authorized apps | matches | matches | **identical** | none ✅ |
| SSO sheet | matches | matches | **identical** | none ✅ |
| Deactivate account | reachable, call commented out | implemented, screen absent | **both working** | See `logic-parity.md` §8. 🔴 |
| Logout confirmation | confirmed | **none** | **confirm** | Android: add. 🟠 |

---

## 11. Auth and sign-in

| Element | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- |
| SSO authorization sheet | v-padding 12, radius 12, @ 0.15, bold, brand text, grey @ 0.1 disabled | identical | **identical** | none ✅ — the reference implementation for buttons |
| Authorized apps list | identical | identical | **identical** | none ✅ |
| Login scrim | **0.3** — `Color.black.opacity(0.3)`, `LoginView.swift:16` | **0.75** — `Color.ham_black.copy(alpha = 0.75f)`, `LoginView.kt:143` | **0.5** | **both** — 🟠. Previously read as 0.5 / 0.5 "matches"; both values were wrong, and the row that claimed no work was needed was the wrong one. |
| QR scan reticle | from the Huawei ScanKit AAR | same | **identical** | matches ✅ — not themeable from this repo |
| QR scan — torch / album | absent | absent | **decide** | Both: consider adding. 🟡 |
| CAS settings | no credential fields; remote webview + injected JS | same | **identical** | matches ✅ |
| QR scan localization | `values-en` / `values-ja` present | — | **localise** | Both: SSO and CAS are Chinese-only. 🟠 |

---

## 12. Shared components

These affect every screen, so they are worth doing first.

| Component | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- |
| Sheet radius | system (~12) | 24 | **keep per platform** | documented, not changed |
| **Sheet drag handle** | system provides one | **none** | **36 × 5 pill, `text.secondary` @ 0.4** | Android: add. 🔴 — users cannot tell the sheet is draggable. |
| Sheet detent | system | 85% | **one rule** | verify |
| Toast radius | **0** (plain rectangle) | 12 | **8 [new]** | Both → 8. 🟠 |
| Toast subtitle | 12 | **16** (same as title) | **12** | Android. 🟠 |
| Toast duration | 3300 ms | 2000 ms | **3300 ms** | Android. 🟠 |
| Toast icon | 36 | 32 | **36** | Android. 🟡 |
| Toast icon → text gap | 5 | 8 | **5** | Android. 🟡 |
| Toast error colour | `#FF3B30` | `#F44336` | **`feedback.error` #FF3B30** | Android. 🟠 |
| Toast types | 5 | 3 | **5** | Android: add warning and info. 🟠 |
| Toast background | adaptive | hardcoded `#EDEEEF` | **adaptive** | Android: use the token. 🟠 |
| Text field | **no shared component in use** — `TextEdit.swift:11` has **0** call sites, 14 hand-built `TextField(` | one `HamTextField`, **no error slot**, bypassed by the flagship schedule editor (`InsertEditHomeView.kt:146-165`) | **Android's spec, plus an error slot** | iOS: port it (radius 8, border 1px, padding 8, placeholder `text.tertiary`, caret `accent`) and add a persistent accessibility label, since an Android hint disappears when the field has text. Android: add the error slot and stop bypassing it. 🟠 |
| Segmented picker | native | custom, thumb r5 ≠ track r6 | **thumb radius = track radius** | Android: fix. 🟠 |
| Divider colour | system, no token, **76 raw `Divider()` and no component at all** | `surface.tertiary`, shared `HamDivider` at **66** sites (`Divider.kt:17-24`) | **`surface.tertiary`, one component per platform** | iOS: add the token *and* the component — 76 unowned call sites is 76 chances to drift. 🟠 |
| Divider vertical padding | 4 (varies 0/4/8/10/12) | same spread | **4** | Both: collapse outliers. 🟡 |
| Empty state | inline text in a card | bare full-screen column | **one component** | Both: build one. 🟠 |
| Error / success views | Lottie + icon + text | Lottie + icon + text, different colours | **one set** | Align: success icon should be green on Android (currently blue). 🟠 |
| Loading view | ad-hoc per screen | ad-hoc per screen | **one component** | Both: build one. 🟠 |
| Card | padding 16, radius 16 | identical | **identical** | none ✅ |
| Card header → body gap | 8 | 8 | **8** | matches ✅ — the one spacing value both platforms agree on |
| **Search debounce** | **none — a gRPC call per keystroke** (`CourseScoreSearchViewSearchBar.swift:33-35`) | none needed — fetches on submit only (`CourseScoreSearchViewModel.kt:151-165`) | **300 ms debounce, or submit-only** | iOS: debounce. Either way, no request per character. 🟠 |
| **Score-result filter gate** | ships to everyone | behind A/B flag `enableCourseScoreResultFilter` (`CourseScoreSearchViewModel.kt:89-92`) | **ships to everyone or to no one** | Android: resolve the flag; an affordance behind an experiment is not a design decision. 🟠 |
| **Paging footer** | `searchResultLoadState` computed, **never rendered** (`CourseScoreResultViewBody.swift:22-41`) | same — computed, never rendered (`CourseScoreResultView.kt:221-227`) | **trailing loading row + end-of-list marker** | Both: render what you already compute. iOS already does this correctly for want/comment history. 🟠 |
| **Result count** | never, even in nav titles (`CourseCenterSelfWantPageView.swift:68`) | in the nav title for want/rank (`想上历史(%1$d)`) and a filtered seat count in library | **show it on any filtered or searched list** | iOS: add; both: add it to the score result, the list that needs it most. 🟡 |
| **Logout confirmation** | **none**, 2 entry points | **none**, 3 entry points | **confirm** — a platform-native alert, cancel + action | Both: add. 11 of 13 destructive actions are also unconfirmed; see `design-system.md` §4.6. 🔴 |

---

## 13. Standalone screens

| Screen | Element | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- | --- |
| Bus | distance prefix | absent | 距你 | **距你** | iOS. 🟠 |
| Bus | arrived / arriving | both | 到达 only | **both** | Android. 🟠 |
| Pay (E卡) | pay-info initial state | collapsed | **expanded** | **collapsed** | Android. 🟡 |
| Pay | method labels | 8 labels | identical | **identical** | matches ✅ |
| Privacy | agreement text | self-contradictory | same text | **rewrite** | Both — see `copy-and-strings.md` §7.0. 🔴 |
| Version changelog | one-shot launch modal | sheet | **one rule** | Product decision | 🟠 |
| Version update flow | **absent** | present, non-blocking | **present** | iOS: add. 🔴 |
| Debug | 16 hardcoded English strings | absent from all locale files | — | **localise or gate** | Both: at minimum mark it debug-only. 🟡 |
| Automation / Siri | one Siri shortcut, never donated | full alarm scheduler | — | See `logic-parity.md` §5. 🔴 |
| Share sheet | present | **absent** | **decide** | Product decision. 🟠 |
| About | 关于 | 关于Ham | **关于Ham** | iOS. 🟡 |
| About | no update check | — | — | **add** | iOS. 🟡 |

---

## 14. Cross-cutting

Divergences that repeat across many screens. Fixing these once closes dozens of per-screen rows.

| # | Pattern | iOS | Android | Screens affected | Agreed |
| --- | --- | --- | --- | --- | --- |
| 1 | **Chevron size** | 8pt — 22 sites across 8 files | Material default 24dp | ~20 | **12 iOS / 16 Android [new]** — caption-paired, so the platforms differ on purpose; neither current value was chosen |
| 2 | **Tint alpha** | 0.10 / 0.13 / 0.25 depending on screen | 0.15 / 0.20 / 0.25 | ~15 | **0.15** for `tint.subtle`, 0.10 for chips |
| 3 | **Watermark size** | 36, 128, 156, 220 — hand-rolled, outside the primitive | 56, 60, 72, 128, 144, 172, 240, 250 | ~20 | **128 @ 0.15**, bottom-end (16, 16), fixed frame not font size |
| 4 | **Hero value size** | 28 / 36 / 50 / 72 depending on screen | 24 / 28 / 32 | ~12 | Use the type scale: 28 for card heroes, 50 for schedule, 36 for weather |
| 5 | **Icon sizes** | unsized in many places, inheriting ~17 | explicit, but inconsistent (25, 30, 36, 56, 72) | ~25 | Use the icon scale: 12 / 20 / 24 / 32 / 64 / 72 |
| 6 | **Card gap in a scroll column** | 8 (implicit, unauthored) | 8 or 16 depending on screen | ~10 | **8** |
| 7 | **Colon style** | full-width on some screens, ASCII + space on others | four styles, sometimes in one file | ~49 strings | **full-width, no space** (see `copy-and-strings.md` §3) |
| 8 | **Function-row height** | 150 (library, sport) | 142 / 160 | 4 | **150** |
| 9 | **Secondary-text colour** | adaptive | hardcoded `Gray`, no dark variant | every screen | **adaptive `text.secondary`** |
| 10 | **Module brand vs accent** | raw `Color.blue`/`.green` at call sites | tokens, but two resolve wrong | ~10 | Use `accent` for interactive, `brand.*` only in the three identity places |
| 11 | **Icon *metaphor*** | court / grid / filled-circle | ball / school / bare glyph | see `design-system.md` §2.8 | 5 of 8 function-grid keys, 1 of 3 tab icons, 4 of 6 status-card headers agree today. Sizes are already specified — the **glyph** is not |
| 12 | **Lazy vs eager lists** | `ScrollView { VStack }` by default; lazy only for the schedule list and 2 `List`s needing `swipeActions` | `LazyColumn` (17 sites) for the score and schedule lists | ~10 | **lazy above ~30 items** — iOS's score, library, sport and status columns are all eager today |
| 13 | **Function-launcher grid** | horizontal `ScrollView` of hand-laid `HStack` rows (`MyViewFunctionCard.swift:49-67`) | `LazyHorizontalStaggeredGrid` (`MyViewFunctionComponentView.kt:203-214`) | 1, but config-driven | **the real grid on both** — same cloud config, one of the two is a reimplementation |
| 14 | **Schedule row gap** | 12 | 16 | 1 | **12** |
| 15 | **Result count on a filtered list** | never | nav title for want/rank, filtered seat count in library | 3 | **show it** |

Item 10 is the root cause of several rows above: iOS writes `Color.blue` literally in the library
module and `Color.green` in the sport module, bypassing the tokens.

---

## 15. Motion

There is no motion scale on either platform today — see `design-system.md` §2.9. Every row below
is a value that must be written down for the first time on at least one platform.

| Behaviour | iOS | Android | Agreed | Change |
| --- | --- | --- | --- | --- |
| **Page push / pop** | no duration anywhere — bare `withAnimation`, `Navigation.swift:91` | 400 ms in, 300 ms out — `NavHost.kt:45,55` | **300 ms symmetric**, ease-out in / ease-in out | Android 400 → 300; iOS write an explicit 300. Also dedupe `NavHost.kt` — the two overloads repeat the values |
| **Sheet duration** | system default | not written — `Animatable` default spring, `Sheet.kt:187` | **300 ms** | both write it |
| **Sheet detents** | **0** uses of `presentationDetents` | 85 % height | declare detents explicitly | iOS |
| **Sheet scrim** | system default | `0.5 × dragProgress`, `Sheet.kt:92` — no independent animation | **max alpha 0.5, animating on its own curve** | Android decouple from drag |
| **Toast dwell** | **3.3 s** — `ToastView.swift:37` | **2.0 s** — `ToastManager.kt:150` | **3.0 s** | both |
| **Toast exit** | slides and fades | **fades only** — `ToastManager.kt:154` | slide **and** fade in and out | Android add the slide |
| **Toast radius** | **0** — plain rectangle, `ToastView.swift:105` | 12 dp | **12** | iOS |
| **Tab switch** | animated + **heavy haptic** — `MainTabView.swift:32` | `fadeIn`/`fadeOut`, no haptic | **instant content swap, no haptic** | both remove the animation; iOS remove the haptic |
| **Button press** | none | **whole button dims to alpha 0.25** — `Button.kt:48` | native press treatment | Android delete the dim |
| **Loading → content** | **hard swap** at 51 `ProgressView` sites | crossfade, spec almost never given | **300 ms crossfade** | iOS add; Android pin the spec |
| **List item insert / remove** | none | `.animateItem()` in one file | **150 ms both** | iOS add |
| **Pull-to-refresh** | hand-rolled, one screen, **128 pt** threshold — `StatusUpdateView.swift:15,57-63`; **0** `.refreshable` | native, one screen, **300 px** — `StatusContainerView.kt:157`; **+56 `BounceScrollView`** no-ops | **on every stale-able list**, one threshold, ~64 dp | Both — see §4.7 |
| **`.fade` transition** | pod's **asymmetric** fade at 13 sites — fade in, cut out | n/a | first-party symmetric `AnyTransition.fade` | iOS — this is the highest-severity motion defect |
| **Reduced motion** | not honoured | not honoured | **honour on both** | both |

---

## 16. Accessibility

Both platforms fail in the *same* direction, so this is not a parity list — it is one shared
work item. See `design-system.md` §2.10 for the rules.

| Item | iOS | Android | Work |
| --- | --- | --- | --- |
| Icons labelled | **2** `accessibilityLabel` of 236 `Image(systemName:)` in `iOS/` — 4 production-visible modifiers in the whole app; 3 of the 4 are in `PrintPrepareView.swift`, the only accessible screen in either app | **7** real strings of 290 `contentDescription` in `Android/` — 271 `null`, 8 `""` | label every meaningful icon on both |
| Row / group merging | **0** `accessibilityElement(children:)` | **1** `mergeDescendants` | merge the row so decorative icons need no label of their own |
| State semantics | 1 `accessibilityAddTraits` | **0** `toggleable`, **0** `selectable` | Android: expose toggle/selected state |
| Announcements | **0** | **0** | announce toasts and loading on both |
| Test identifiers | **3** `accessibilityIdentifier`, all `#if HAM_E2E` — a release build has none | **7** `testTag` | both — this doubles as the audit trail for the above |
| Touch targets | unenforced | unenforced, 0 `minimumInteractiveComponentSize` | audit below-minimum targets; start with the timetable cell |
| Large text clipping | ~184 fixed-width frames | ~202 fixed heights | replace fixed heights on text containers with minimum heights |
| Contrast | `text.secondary` #8E8E93 → 3.26 on white; `text.tertiary` @60% → **1.92** | `text.secondary` #888888 → 3.37 / 3.54; brand text on brand tint: sport **2.42**, score **1.92** | both — new greys that clear 4.5:1, `text.tertiary` as a solid, a `brand.<module>.text` per module, darker toast fills. See `design-system.md` §2.10 "Contrast". |

---

## 17. Non-content states

See `design-system.md` §3.10. Each row is a state that is absent or wrong on **both** platforms,
so unlike most of this document these are not divergences — they are shared gaps.

| State | iOS | Android | Work |
| --- | --- | --- | --- |
| Shared loading component | **none** — `ProgressView` repeated ~40× | `HamLoadingProgressBar`, 61 sites | iOS build one; Android keep |
| Skeleton where shape is known | **one** — `.redacted` seat grid | **none** | Android add; iOS extend |
| Shared empty component | **absent** | **absent** | build one per platform to §3.9 |
| Error default action | **返回** — `ErrorView.swift:22` | **完成** — `ErrorView.kt:60` | **重试** on both |
| Retry in the shared error component | **no** | **no** | add to both |
| Offline | none — reachability never drives UI | **none** — no connectivity check at all | persistent banner on both |
| Session expired | none — `AuthPbInterceptor` is a pass-through | none — no auth interceptor | its own state and copy on both. **Both already solve it once, in the library module** (`LibraryRetryLoginView`) — generalise that, do not invent a second pattern |
| Rate-limited / blocked | none | none | surface the server's message |
| **Route to system Settings** | **none** — 0 hits for `UIApplication.open` in `iOS/` | **1** — `AlarmPermissionSheet.kt:143`, exact alarms only | add on both; required by the permission-denied state |
| **Permission denied** | scan toasts 相机权限未开启 / 请前往"设置"开启, then cannot navigate | **scan has no denial handling at all** — Scan Kit reports to nobody, so a denied camera yields a black screen forever (`QrCodeScanView.kt:205-208`) | render the state on both: Android build it, iOS add the destination |
| **Retry offered** | **4 screens** — `CourseScoreCourseDetailErrorView.swift:33`, `StatusBusCard.swift:31`, `StatusWeatherCard.swift:58`, `PrintPrepareView.swift:45` | **6 screens**, 5 of 7 retry strings wired | only 4 of 14 comparable cases match — audit the rest |
| **Retry mislabelled as a dismissal** | course center: `ErrorView(title: 请求失败) { vm.doRequest() }` under the default **返回** | print: same shape under the default **完成** | pass the label — a one-argument fix each |
| **Booking seat-list failure** | **backs out only** — `LibraryBookErrorView.swift:22-25` | **retries** — `library_retry` → `vm.fetchSeatList()`, `LibraryBookView.kt:134-141` | iOS add the retry; this is the sharpest single divergence in the module |
| **Error vs empty** | fall through to an empty list — identical pixels | `when (loadState) { … else -> {} }` — identical pixels | distinguish them on both |
| Pull-to-refresh | hand-rolled overscroll, **128 pt**, `StatusUpdateView.swift:15,57-63` — **0** `.refreshable` | native, **300 px**, `StatusContainerView.kt:157`; **+56 `BounceScrollView`** no-ops | refresh exists on one screen each at thresholds >2× apart; migrate iOS to `.refreshable`, fix Android's threshold, and make the 56 bouncing screens refresh or stop bouncing |

The Android error text 网络异常，请稍后重试 (`ToastManager.kt:247`) promises a retry the UI does
not offer. It is shown at ~70 call sites against 13 `ErrorView` sites — the dominant pattern.

---

## 18. Screens already at parity

Do not re-audit these.

| Screen / element | Status |
| --- | --- |
| Card primitive — padding 16, radius 16, background, header→body gap 8 | identical |
| List row — icon plate 40 @ 0.1, icon 20, gap 8, title 17, subtitle 12 | identical |
| Filter chip (CourseScore) — all eight metrics | identical |
| SSO authorization sheet — all variants | identical |
| Authorized apps list | identical |
| Function grid on 我的 — 8 tiles, module colours | identical |
| Board card and promotion card (remote-config driven) | identical |
| Sport pay screen — all 8 labels | identical |
| Course long-press menu — all 6 items | identical |
| Library home function rows and banner title | identical |
| Term/semester mapping, GPA table, both F2 formulas | identical (logic, `logic-parity.md` §11) |
| Search, course matching, ranking | identical (server-side) |
| Card elevation — none on either platform | identical (both flat) |
| Settings rows separated by a divider; cards separated by spacing | identical (both) |
| Validation timing — on submit only | identical (both) |
| Required fields — never marked | identical (both, and wrong on both) |
| Cursor pagination triggered at the second-to-last item | identical (both) |
| The 5-or-7 × 13 timetable grid, hand-offset rather than lazy | identical (both) |

---

## 19. Patterns

Five behaviours that cut across every screen. The rules are in `design-system.md`
[§4.5](design-system.md#45-forms-and-text-entry)–[§4.9](design-system.md#49-lists-grids-and-card-columns);
this is the verdict per pattern.

| Pattern | Do the platforms agree? | Divergences to close |
| --- | --- | --- |
| **Forms** ([§4.5](design-system.md#45-forms-and-text-entry)) | **yes** on every measured axis — placeholder-only fields, label rows with a chevron, submit-only validation, toast errors, no required marking, no keyboard type | The bug is *shared*: iOS has **no field component** (`TextEdit` 0 call sites), Android's `HamTextField` has **no error slot** and is bypassed by the flagship editor, and **within each app** two submit placements ship. Both sides move. |
| **Confirmation** ([§4.6](design-system.md#46-confirmation-and-destructive-actions)) | **yes** — 11 of 13 destructive actions unconfirmed on both, logout unconfirmed on both, the same one action confirmed on both | None to align; **both** sides must *add* confirmations. The largest single gap in either client. |
| **Data loading** ([§4.7](design-system.md#47-data-loading-and-caching)) | **mostly** — course/score/schedule local-first on both; library/sport network-first on both | The **status dashboard**: iOS shows a persisted snapshot immediately, Android starts empty. Plus pull-to-refresh — one screen each at 128 pt vs 300 px, and 56 Android screens that bounce without refreshing. |
| **Search, filter, sort, pagination** ([§4.8](design-system.md#48-search-filter-sort-pagination)) | **yes** on the model — the same three descending-only chips, cursor paging at the second-to-last item, no footer, no count | iOS requests **per keystroke**; Android gates the filter behind an A/B flag; comment page size differs (server vs 20); Android shows counts in nav titles, iOS never does. |
| **Lists, grids, card columns** ([§4.9](design-system.md#49-lists-grids-and-card-columns)) | **yes** on nearly every container — scrolling card column, bespoke timetable grid, lazy for the two long lists | iOS's score screen is eager where Android's is lazy; the function launcher is hand-laid on iOS and a real grid on Android; 76 raw iOS `Divider()` vs one Android component; schedule row gap 12 vs 16. |
