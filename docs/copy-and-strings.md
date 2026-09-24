# Ham copy and string standard

How to write and name every user-visible string in the Ham apps. This is the companion to
`design-system.md` — that document covers how things look, this one covers what they say and how
those words are stored.

Current divergences are in [§7 Current state](#7-current-state) at the end.

## Contents

- [1. Scope](#1-scope)
- [2. Terminology](#2-terminology)
- [3. Copy rules](#3-copy-rules)
- [4. String keys](#4-string-keys)
- [5. File organisation](#5-file-organisation)
- [6. Localization requirements](#6-localization-requirements)
- [7. Current state](#7-current-state)
- [8. Review checklist](#8-review-checklist)

---

## 1. Scope

Three things this document governs:

1. **Terminology** — which Chinese word to use for each concept, so the same action is never
   called two different things.
2. **Copy rules** — tone, punctuation, spacing, placeholders.
3. **String keys** — how strings are named and stored, so the two platforms' catalogs can be
   diffed, synced, and handed to a translator as one unit.

---

## 2. Terminology

For each concept there is **one approved word**. The forbidden column lists variants that exist
in the codebase today and must not be used in new copy.

**iOS is the baseline.** Where the two platforms word the same thing differently and this table
does not say otherwise, the iOS wording wins. The table overrides that default where the iOS
word is the weaker one — for example row 26, where all four sport-venue words are iOS's.

| # | Concept | ✅ Use | ❌ Not | Notes |
| --- | --- | --- | --- | --- |
| 1 | Book / reserve | **预约** | 预定, 预订, 订 | Applies to library **and** sport. |
| 2 | Cancel | **取消** | 返回, 关闭 | 返回 is back-navigation, a different action. 关闭 dismisses a surface. |
| 3 | Confirm | **确定** | 确认 | 确定 for buttons; 确认 only where 确定 would read ambiguously. |
| 4 | Refresh (re-fetch now) | **刷新** | 重新加载 | — |
| 5 | Fetch from portal | **获取** | 更新 | Use 获取 for the whole flow: entry, progress, **and** result. |
| 6 | Update (refresh stored data) | **更新** | 获取 | Reserved for timestamps (`%@更新`) and 更新基础数据. |
| 7 | Settings | **设置** | 配置 | 选项 is a legitimate noun in subtitles (图书馆预约选项), not a synonym. |
| 8 | Log in | **登录** | 登陆 | 登陆 is a typo. 连接 (connect a service) and 验证 (verify identity) are separate concepts — keep them. |
| 9 | Favourited seat | **收藏座位** | — | Distinct from 首选座位 (auto-booking target) and 想上 (course want-list). Do not merge. |
| 10 | Empty collection | **暂无X** | 没有X, 无X | 暂无历史记录, not 没有历史记录. |
| 11 | Prerequisite unmet | **未X** | 无X | 未设置, 未绑定. |
| 12 | Action failed | **X失败** | X错误 | 预约失败, 获取失败. |
| 13 | Load failed | **遇到了错误** | X异常 | Passive failures only. Never show 异常 to a user. |
| 14 | Loading | **正在X** | 加载中, 很快就好, 请稍等 | Pick one form; do not add a second reassurance line. |
| 15 | Deadline | **前** | 之前, 截止 | `请于%@前完成支付`. |
| 16 | Success | **成功** | — | 完成 is a separate "done" concept (完成 button, 已完成). |
| 17 | Retry | **重试** | 重新尝试, 再来一次 | — |
| 18 | Delete | **删除** | 移除, 清除 | 移除/清除 are for bulk or specific objects only. |
| 19 | Edit | **编辑** | 修改, 更改 | 更改 is scoped to time changes (更改预约时间). |
| 20 | Add | **添加** | 新增 | — |
| 21 | Today / tomorrow (picker) | **今天 / 明天** | 当天 / 隔天 | Pickers use 今天/明天. Read-only weather readouts use 今日/明日. |
| 22 | Session dead | **失效** | 过期 | 登录状态失效. 过期 is for timestamps (过期时间). |
| 23 | Other (as in "other settings") | **其他** | 其它 | 其它 is a variant form. |
| 24 | Numbers below ten | **1** | 一 | `还有不到1分钟`, not 还有不到一分钟. |

### 2.1 Further concepts

Mined from the full screen audit — 38 more concepts with more than one wording in the codebase.

| # | Concept | ✅ Use | ❌ Not | Notes |
| --- | --- | --- | --- | --- |
| 25 | A user-written course review | **评论** | 评价 | 评论 outnumbers it 3:1, names the API and analytics surface, and 评价 collides with the star rating 评分. The split is baked into file names, route names and resource keys. |
| 26 | The sport venue | **场馆** | 体育场所, 运动场馆, 运动场地 | Four words today. 场馆 is shortest and is what the proper-noun exception already uses. |
| 27 | Yesterday | **昨日** | 昨天 | The standard already mandates 今日/明日 for read-only readouts; a third member should match. |
| 28 | Location permission missing | **未获取地理位置权限** | 未获取地理权限 | 地理 alone is a truncation. |
| 29 | The library statistics feature | **数据统计** | 统计服务 | One feature, one name. |
| 30 | Captcha settings | **验证码识别设置** | 验证码设置 | The feature is OCR of the captcha, not captcha entry. |
| 31 | Average (course score) | **均分** | 平均 | Already the wording bound to a sort chip. |
| 32 | Grade / score (course score) | **成绩** | 分数 | 6:1 majority; the outlier sits on a card titled 成绩排行. |
| 33 | Counting people | **位同学** | 人 | 人 is bare. |
| 34 | Comment length | **字数** | 长度 | The unit shown is 字, so 字数 is the literal match. |
| 35 | "This course" | **这门课** | 这门课程, 该课程 | 该课程 is bureaucratic register. |
| 36 | Library account card | **账号信息** | 图书馆账号信息 | The card is already inside the library settings screen. |
| 37 | Booking-history screen title | **历史预约** | 历史记录 | Matches the entry-row label; 记录 is generic. |
| 38 | The course-review module name | **给分** | 课程评分 | 给分 is everything the user sees; 课程评分 survives only as a dead string. |
| 39 | "Want to take this course" | **想上** | 想上这门课, 想上课程 | Empty state should read 没有想上记录. |
| 40 | "Go to settings" link | **去设置** | 前往设置, 请前往"设置"开启 | The full sentences are a different construct, but the curly-quote form should be normalised. |
| 41 | "Go to X" — the bare verb | **前往X** | 去X | 去X reads as a verb phrase and cannot host a link style. Note 去 is also a real verb elsewhere (去支付, 去选择), so the collision is live. |
| 42 | Select, as a button noun | **选择** | 选取 | 选取 appears on exactly one screen. |
| 43 | Changing an existing booking | **更改** | 变更, 修改 | 变更 is administrative register. One screen currently mixes all three for the same action. |
| 44 | Second person | **你** | 您 | 82% of the corpus. Shifting to 您 on three surfaces is an unforced register break. |
| 45 | Course timetable, abbreviated | **课程表** | 课表 | The 重置/已重置 toasts are the outliers. |
| 46 | Available capacity | **可用** (capacity) / **剩余** (counts) | 空余, 多余, 余, 共剩余 | 多余 literally means *superfluous* and is simply wrong for "no seats left". |
| 47 | "Not found" | **没有找到X** | 未找到 | Two different negative verbs for the same state. |
| 48 | The time-window field | **开始时间 / 结束时间** | 时间, 持续时间 | Already the established pair on three screens; iOS's bare 时间 says nothing. |
| 49 | Seat attribute filters | **偏好** (section), **电源 / 靠窗** (rows) | 附加条件, 仅 | iOS ships an icon with 仅 and no object — unreadable without colour. |
| 50 | Failure noun | **错误** | 异常 | 异常 is a developer term leaking into user copy, in three bus/weather/print strings. |
| 51 | The legal document agreed at login | **用户使用协议** | 《用户使用协议》, 隐私协议, 用户隐私协议 | The two platforms currently link **different documents** under different names. |
| 52 | The automation feature | **自动化操作** | 自动化, 自动化功能 | Matches the hub row. |
| 53 | The scheduled library task | **自动快速预约** | 图书馆快速预约, 图书馆自动化 | Notifications should reuse the switch label. |
| 54 | Copy-course-log action | **复制课程信息** | 复制课程回包日志 | 回包 is network jargon. iOS got this right. |
| 55 | Widget refresh interval | **更新间隔** | 编辑小组件更新时间 | The control is a picker, not an editable field. |
| 56 | The course widget | **课程表小组件** | 课程与图书馆小组件 | The tip's 课程 is a half-word. |
| 57 | About screen title | **关于Ham** | 关于 | Unambiguous in a nav bar. iOS also keys the same word twice. |
| 58 | "Needs" in length counters | **需** (chip) / **需满足** (sentence) | 需要 | One counter must not carry both forms. |
| 59 | The module's statistics opt-in | **数据统计** | 统计服务 | See row 29. |
| 60 | Study-time total | **共计X分钟** | 累计学习X分钟 | The Android variant is dead. |
| 61 | Course-review module badge | **给分** | 课程评分 | See row 38. |
| 62 | Unbind a social account | **解绑** | — | Neither platform currently lets a user do this; the copy exists but the action does not. |

**Correction to row 9:** 收藏座位 and 首选座位 are distinct concepts, but the iOS entry point
mislabels one as the other — the home badge and settings card header read 收藏座位 while the
screen they open is titled 首选座位设置.

**Borderline, no change recommended:** 去支付 (action) vs 待付款 (state) — arguably two concepts,
both stable and used consistently within their own screen.


### Exceptions to rule 1

Two strings keep 场馆预约 because it is a **proper noun** — the WeChat mini-program's real name:

- 前往微信小程序"场馆预约"继续支付

Do not "correct" these to 场馆预约 → 场馆预约.

---

## 3. Copy rules

### Tone

**Plain and direct.** No exclamation marks, no emoji, no sentence-final particles in functional
copy.

| ✅ | ❌ |
| --- | --- |
| 今日课程已上完 | 本周的课程已经全部结束啦～辛苦啦！💪 |
| 获取天气数据时遇到了错误 | 哎呀，天气加载失败啦… |
| 外面下雪了，不出去看看吗 | 下雪啦～要不要出去看看呀？❄️ |

The cutesy register exists in the codebase but is unreachable dead code on both platforms. Do
not revive it.

### Punctuation

- **Labels, buttons, toasts, status text: no terminal period.**
- Prose (help text, consent dialogs, FAQ answers, privacy copy): ends with `。`
- Use full-width punctuation in Chinese text: `，。：？！`
- There is exactly one `！`-terminated toast in the codebase
  (`更改预约时间出现错误。预约已取消，请尽快手动预约！`). It is wrong — a toast should not
  carry both a `。` mid-sentence and a terminal `！`.

### Spacing

- **Colons in `label: value` are full-width with no space**: `登录时间：%@`
- **Pad Latin words with spaces; do not pad CJK**: `是否同意上传…到 Ham 服务器`. This is already
  the de-facto rule on both platforms — keep it.
- No stray half-width spaces inside otherwise-Chinese strings.

Today the codebase carries four different colon styles, sometimes within a single file. Fix on
sight:

| ❌ | ✅ |
| --- | --- |
| `登录时间: %1$s` | `登录时间：%1$s` |
| `综测成绩：%.6f  平均成绩` (two spaces) | `综测成绩：%.6f 平均成绩` |
| `专业必修: %.1f` | `专业必修：%.1f` |

### Placeholders

Placeholders are platform-native — `%@` and `%lld` on iOS, `%1$s` and `%d` on Android. That is
expected. What must match across platforms is the **word order and spacing around them**:

| ❌ mismatched | ✅ matched |
| --- | --- |
| iOS `%@更新` vs Android `%1$s 更新` | both `%@更新` / `%1$s更新` |
| iOS `登录时间：%@` vs Android `登录时间: %1$s` | both `登录时间：%@` / `登录时间：%1$s` |
| iOS `还有不到一分钟` vs Android `还有不到1分钟` | both `还有不到1分钟` |

Also check that the placeholder carries the **same information**, not just the same position —
iOS renders a bare `328m` where Android renders `距你328m`. Those are different strings, not a
formatting difference.

#### Number precision

Precision is part of the copy, not just the format. Today it drifts:

| Value | iOS | Android | Should be |
| --- | --- | --- | --- |
| GPA | `%.2f` | `%.6f` | `%.2f` |
| Credits | `%.1f` | `%.2f` | `%.1f` |
| Course rate average | `%.1f` | `%.2f` | `%.1f` |

A student sees `4.00` on one platform and `4.000000` on the other for the same course.

---

## 4. String keys

This is the part that is currently broken.

### 4.1 The problem

The two platforms' catalogs cannot be diffed, synced, or sent to a translator together:

| | iOS | Android |
| --- | --- | --- |
| Convention | `SCREAMING_SNAKE_CASE`, mostly no prefix | `snake_case`, loosely module-prefixed |
| Example | `DELETE_THIS_SCHEDULE` | `library_reserve` |
| Module prefix | none | inconsistent (`common_`, `library_`, `sport_`, but also bare `select_color`, `picker_confirm`) |
| Keys are Chinese | **777 of 898 (87%)** | 0 |

Worse, **85% of iOS entries use the Chinese text itself as the key** — 765 of 898 unique keys, where the entry is literally `"预约" = "预约"`. That
means:

- rewording any string silently changes its key, orphaning the other two locales;
- keys cannot be searched, sorted, or grouped by module;
- a typo in the copy becomes a permanent identifier.

### 4.2 The standard

**`snake_case`, module-prefixed, lowercase, ASCII only.**

```
<module>_<element>[_<qualifier>]
```

| Part | Rule | Examples |
| --- | --- | --- |
| `<module>` | the feature area | `library`, `sport`, `score`, `course`, `coursescore`, `schedule`, `status`, `my`, `cas`, `user_center`, `common` |
| `<element>` | what it is | `quick_book`, `reserve`, `settings`, `history` |
| `<qualifier>` | optional disambiguator | `cta`, `title`, `desc`, `error`, `empty`, `success`, `subtitle` |

Rules:

1. **ASCII only.** Never use Chinese, spaces, or punctuation in a key.
2. **Never use the copy text as the key.** The key describes the string's *role*; the value is
   the words. This is what makes rewording safe.
3. **Prefix by module, always** — including on iOS.
4. **Name the role, not the current wording.** `sport_order_cta`, not `sport_yuding`.
5. **Reserve suffixes** for predictable roles:

| Suffix | Use |
| --- | --- |
| `_title` | screen or card title |
| `_subtitle` | secondary line under a title |
| `_desc` | explanatory body |
| `_cta` | the primary button label |
| `_empty` | empty-state text |
| `_error` | failure text |
| `_success` | success text |
| `_confirm(_title/_desc)` | confirmation dialog |
| `_hint` | inline guidance |

### 4.3 Migration

- **All new keys on both platforms follow §4.2.** This is the rule that matters most and costs
  nothing.
- **Android**: keys are close already. Tighten the unprefixed ones (`select_color`,
  `picker_confirm`, `not_set`) into a module namespace as they are touched.
- **iOS**: this is the larger job. 777 keys need identifiers. Do it **module by module**, not in
  one pass — start with the modules that change most (sport, library, score). Because the
  current keys are the Chinese text, the migration is mechanical: generate the new key from the
  module + element, move all three locales together, then update the call sites.

A shared key namespace is what lets the two catalogs be diffed. Until iOS migrates, use the
**Chinese text** as the join key when comparing the two platforms.

---

## 5. File organisation

### 5.1 iOS has six catalogs, not one

This is easy to miss and is the source of several current defects. Strings are spread across
every app target, and each target has its own bundle — so **a string in the main catalog is
invisible to the widget and the Siri extension**.

| # | Catalog | Locale dirs | Entries (zh) | Key style |
| --- | --- | --- | --- | --- |
| 1 | `Ham/{locale}.lproj/Localizable.strings` | 3 | **898** | 85% Chinese-as-key |
| 2 | `Ham/{locale}.lproj/InfoPlist.strings` | 3 | 4 | `CFBundle*` |
| 3 | `Ham/SiriIntent/{locale}.lproj/Localizable.strings` | 3 | **2** | Chinese-as-key |
| 4 | `Ham/SiriIntent/{locale}.lproj/Intents.strings` | 3 | 26 | Xcode-generated (`956YUW`) |
| 5 | `Ham/Widget/{locale}.lproj/WidgetIntentConfiguration.strings` | 3 | 48 | Xcode-generated (`0yHcjK`) |
| 6 | `Ham/SiriIntentUI/{locale}.lproj/MainInterface.strings` | 3 | 2 | Interface Builder |

Catalogs 3–5 are **not** duplication for its own sake — Siri intent and widget-configuration
strings *must* live in their target's bundle, and their keys are generated by Xcode from the
intent definition. That part is platform-forced.

What is **not** forced, and is wrong:

- **Shared copy duplicated across catalogs.** The widget update-interval options (5分钟,
  10分钟, 15分钟, 30分钟, 1小时, 2小时) are defined in both catalog 1 and catalog 5. They can
  drift, and 10分钟 already appears four times inside catalog 5 alone.
- **Untranslated English in the zh-Hans catalogs.** `MDC7j9 = "Success Message"`,
  `iQFhaG = "Error Message"`, `jO5xID = "Error Message"` in
  `SiriIntent/zh-Hans.lproj/Intents.strings`. These are default values nobody translated.
- **Cutesy tone in the Siri catalog.** `未登录图书馆，先前往我的-图书馆登录哦～` violates
  [§3 tone](#3-copy-rules) and uses a `-` nav notation found nowhere else.

### 5.2 The rule

| Platform | Layout |
| --- | --- |
| iOS | One `Localizable.strings` per locale in `Ham/{zh-Hans,en,ja}.lproj/`, plus `Localizable.stringsdict` for plurals. Group with `// MARK: -` comments per module. Target-specific catalogs (Siri, widget) hold **only** strings that must live there. |
| Android | One `strings.xml` per Gradle module under `src/main/res/values/`, with `values-en` and `values-ja` siblings. |

Rules:

- **A string lives in exactly one catalog.** If two targets need the same copy, put it in the
  main catalog and have the target reference it — or accept that it cannot and define it once
  with a comment pointing at the twin. Never silently duplicate.
- **A string lives in the module that owns it.** Shared strings go in the shared module
  (`Ham/shared` on iOS, `core/ui` on Android) under a `common_` prefix.
- **No duplicate values within or across catalogs.** The same Chinese text defined twice will
  drift.
- **Generated keys stay generated.** Do not hand-edit `Intents.strings` or
  `WidgetIntentConfiguration.strings` keys — regenerate from the intent definition. But **do**
  translate every generated value, including the ones Xcode leaves as English defaults.
- **Mark non-translatable strings explicitly** — Android `translatable="false"`, iOS by keeping
  them out of the strings file entirely (as a Swift constant).

---

## 6. Localization requirements

### Locales

Both platforms support **zh-Hans (default), en, ja**. A new string must be added to all three in
the same commit. If a translation is genuinely unavailable, add the default value rather than
leaving the key absent — a missing key renders the raw key to the user.

### No hardcoded user-visible text

Every user-visible string lives in a strings file. No exceptions.

The current gap is lopsided and worth stating plainly:

A full-screen audit of every screen found **282 hardcoded CJK literal sites**, by module:

| Module | Hardcoded CJK literals |
| --- | --- |
| Sport | **76** |
| Library | **70** |
| My / user center | **67** |
| Shared | **27** |
| Status | **18** |
| Score + schedule | **21** |
| Course | **2** |
| CourseScore | **1** |
| **Total** | **282** |

This supersedes the earlier per-file grep (175 sites across 59 files) — that count missed
literals inside interpolations, model-level status names, and bare `Text(_: StringProtocol)`
calls, which do **not** localise even when a matching catalog key exists.

Notes:

- **Library is the worst module**: 42 distinct Chinese literals plus 3 `-` separators and 2
  `Hello, World!` stubs, across **20 files** (not 10). iOS hardcodes roughly 30 library labels
  that already exist in `Localizable.strings`.
- **Sport hardcodes its status names in the data model** (待付款 / 待使用 / 使用中 / 已使用 /
  已取消 / 已退款 / 未知), passed through `.localized` at the call site, so they resolve only if
  a matching catalog key happens to exist.
- **Android's Debug screen is absent from all `.lproj`/`values-*` files** — it is entirely
  English.
- Android's three include `Text("关闭")` in the shared bottom-sheet component, which surfaces in
  every language.
- **Server-driven copy with no fallback**: ten categories of user-visible text arrive from CCKV
  or the server with no catalog fallback on either platform — consent messages, external-service
  titles, want/comment card titles and hints, the comment-disabled reason, the not-found message,
  the comment input hint, and the course-centre card titles.

### Plurals

Use plural rules for any string containing a countable number.

| Platform | Mechanism | Current state |
| --- | --- | --- |
| iOS | `Localizable.stringsdict` | **en has 105 entries, zh and ja are empty stubs** |
| Android | `<plurals>` | **0 — none defined** |

Today the app relies on `%lld 分钟` style strings with no plural handling in either Chinese or
Japanese. Both are single-plural languages, so the practical impact is limited to English, but
the iOS asymmetry means the en build pluralises while zh and ja do not.

### Formatting

Numbers, dates, and times are formatted by the platform, never string-concatenated. Use
`Text(verbatim:)`-free interpolation so the platform can localise separators.

---

## 7. Current state

The gap list. Each entry is a task.

### 7.0 Findings from the full screen audit

**Scope — read this before using the counts below.** This audit covered **68** of the sections
in [`screens.md`](screens.md), which enumerates **129** `###` entries across 12 modules. The 129
counts analysis sections as well as screens — `screens.md` §1.2 "Card scoring and ordering",
§1.3 "Shared card container" and the per-module "Findings" and "Route table" sections are not
screens — so the real screen count is lower than 129 but **still well above 68**.

The practical consequence: **every "0 occurrences" claim below is scope-limited to the 68
sections that were audited.** It means "0 in what was checked", not "0 in the app". The 61
unaudited sections are concentrated in User center (21 sections) and Shared components (15),
which is exactly where the remaining copy problems are likely to be. Closing that gap is a
prerequisite to treating §7 as complete.

| Category | Count |
| --- | --- |
| Split terminology concepts | 38 (see [§2.1](#21-further-concepts)) |
| Wrong or nonsensical string values | 60 |
| Same UI element, different word across platforms | 92 |
| Placeholder / format inconsistencies | 36 |
| Tone breaches | 28 |
| Punctuation / spacing breaches | 49 (83 half-width-colon instances vs 32 correct) |
| Hardcoded CJK literals | 282 (see [§6](#6-localization-requirements)) |
| Missing translations, junk and dead strings | 101 |

**Ten most important:**

1. **评价 / 评论 / 评分 three-way split** in CourseScore — `创建评价` vs `课程评论` vs
   `已成功发布课程评分与评论`. Baked into file names, route names and resource keys.
2. **`cas_bus_success_message` = `你可以开始使用图书馆了`** — the 校巴 success screen tells users
   they can use the library.
3. **Privacy copy is self-contradictory** — `Ham承诺…不会收集您的任何个人信息。…如果您认为Ham应该收集您的个人信息，您可以终止使用本应用。`
   Shipped verbatim on both platforms.
4. **Nine iOS `String(localized:)` literals have no catalog entry and render as raw keys** —
   存在冲突的课程, 保存失败, 保存颜色失败, 已已保存背景颜色, 保存课程基础信息失败, 已保存基础信息,
   已重置课程时间. Note `已已保存` is itself a typo.
5. **Numeric precision drift** — GPA `%.2f` vs `%.6f`, credits `%.1f` vs `%.2f`, rate average
   `%.1f` vs `%.2f`. The same course reads differently per platform.
6. **`用户名输入有误` / `用户名不能大于20个字符`** on a field whose hint is 昵称.
7. **Key `本周课程已上完` → value `本周的课程已经全部结束啦～辛苦啦！💪`** — key/value contradiction
   plus four tone breaches in one string; the ja translation carries it too.
8. **Android's `feature/my`, `feature/auth` and `feature/cas` ship no `values-en` / `values-ja`**
   — widget, login, SSO consent and CAS are Chinese-only.
9. **Sport venue noun has four words** — 体育场所 / 场馆 / 运动场馆 / 运动场地.
10. **Colon chaos** — 83 half-width `CJK: ` instances against 32 correct; one row in the score
    screen mixes both styles.

**Clean areas worth protecting:** the library and user-center modules have zero emoji, zero `！`,
zero cutesy particles, and no terminal `。` on any label, button, toast or status string.

### 7.1 iOS catalog sprawl

Strings are spread across six catalogs (see [§5.1](#51-ios-has-six-catalogs-not-one)). Three
concrete defects fall out of it:

| # | Issue | Where |
| --- | --- | --- |
| 1 | Widget update-interval options duplicated — 5分钟/10分钟/15分钟/30分钟/1小时/2小时 exist in both the main catalog and the widget's | `Ham/zh-Hans.lproj/Localizable.strings` + `Widget/zh-Hans.lproj/WidgetIntentConfiguration.strings` |
| 2 | Untranslated English defaults sitting in the zh-Hans catalog — `Success Message`, `Error Message` ×2 | `SiriIntent/zh-Hans.lproj/Intents.strings` (`MDC7j9`, `iQFhaG`, `jO5xID`) |
| 3 | 15 duplicated values inside the 48-entry widget catalog — 10分钟 appears 4×, `确认一下，是指"10分钟"对吗？` appears 4× | `Widget/zh-Hans.lproj/WidgetIntentConfiguration.strings` |
| 4 | Cutesy tone and a `-` nav notation in the Siri catalog — `未登录图书馆，先前往我的-图书馆登录哦～` | `SiriIntent/zh-Hans.lproj/Localizable.strings` |
| 5 | 4 duplicated values in the 26-entry Siri intent catalog — `${errorMessage}` appears 5× | `SiriIntent/zh-Hans.lproj/Intents.strings` |

### 7.2 Terminology

| # | Issue | Where |
| --- | --- | --- |
| 1 | 预约/预定 split — library uses 预约, sport uses 预定, on **both** platforms | ~20 strings per platform |
| 2 | Sport order footer CTA: iOS 预约 vs Android 预定 — the only true cross-platform split | `SportOrderViewFooter.swift:31` vs `sport_reserve` |
| 3 | 获取/更新 inversion in the score fetch flow: iOS 更新成功/更新失败 vs Android 获取成功/获取失败 for the identical step | iOS `ScoreUpdateByCasView` vs Android `strings.xml:29-31` |
| 4 | Same inversion in the course timetable fetch flow | both platforms |
| 5 | 其他/其它 — iOS has both two lines apart; iOS sport says 其他设置, Android says 其它设置 | `Localizable.strings:568,570`; `SportSettingViewOtherCard.swift:15` |
| 6 | 没有历史记录 (iOS) vs 暂无历史记录 (Android) | `Localizable.strings:743` vs `library_no_history` |
| 7 | 加载异常 / 加载时遇到错误 / 加载时遇到了错误 — three spellings, one missing 了 | library + status |
| 8 | 今天/明天 vs 当天/隔天 — Android sport uses 明天/今天 where iOS uses 隔天/当天, which appear nowhere else | `sport_setting_day_tomorrow` vs `SportSettingViewStarredOrderCard.swift:38` |
| 9 | 还有不到一分钟 vs 还有不到1分钟 | `Localizable.strings:11` vs Android |
| 10 | 取消 vs 返回 vs 关闭 for dismiss actions | both |

### 7.3 Wrong values behind right-sounding keys

Three keys whose value contradicts their purpose. The full list of 60 is summarised in
[§7.0](#70-findings-from-the-full-screen-audit).

| Issue | Where |
| --- | --- |
| `"CONFIRM" = "更改"` — the confirm key renders "change" | `Localizable.strings:104` |
| `"CANCEL" = "返回"` — the cancel key renders "back" | `Localizable.strings:86` |
| `cas_bus_success_message` = 你可以开始使用图书馆了 — a 校巴 string that says 图书馆 | `feature/cas/.../strings.xml:23` |

Also from the screen audit: `未知倒序` (a sort chip reading "unknown descending order"),
`在课表空白处长按…在课程出长按…` (`课程出` is a typo for `课程处`, shipped on **both** platforms),
`评论长度不合法` (不合法 is administrative register; the sibling screen says 评论字数未符合要求),
and `服务器没有返回数据` (a protocol description, not user copy).

### 7.4 Junk and dead strings

101 items found: 22 junk, 30 dead, 10 key/value mismatches, 4 duplicate keys, 6 references to
non-existent keys, 9 blank-state placeholders, 20 missing translations. Highlights:

**iOS** — broken and test content in the shipped `zh-Hans` file:

| Line | Content |
| --- | --- |
| 483 | `, stadiumArea.lowPrice))起` — leaked Swift interpolation fragment |
| 503 | `deletePasskey - 删除passkey=\(passkeyId)` — debug log string |
| 504 | `inputJsonStr的示例数据为` — identifier leak |
| 711 | `期末考试2` — zero references |
| 744 | `测试` — zero references |
| 698 | `景山公园站` — orphan mock bus stop |
| 784 | `获取Sport Banner` — untranslated English inside a Chinese string |
| 863 | `高等书序二` — typo for 高等数学 |
| 616 | `啊啊啊啊…` (~230 chars) — test data |
| 487 | `1.先休息休息\n\n**2.ssss**…` — preview content |

**Android:**

| Where | Content |
| --- | --- |
| `feature/status/.../strings.xml:3` | `<string name="status_preview_title">测试</string>` |
| 4 files | Placeholder text shipped as content: `文件1`, `副标题`, `公告内容` |
| `feature/coursescore/.../strings.xml:41,42` | `文件`, `课程评分` — nonsensical and dead tab labels |

### 7.5 Missing translations

| Platform | zh keys | en keys | ja keys | Missing from ja | Missing from en |
| --- | --- | --- | --- | --- | --- |
| iOS (main catalog) | 898 | 894 | 883 | **17** | **6** |
| Android | ~919 | ~919 | ~919 | — | — |

iOS also carries 2 orphan keys in each of `en` and `ja` that have no zh source.

Most of the 17 iOS gaps are the junk strings above — deleting them closes most of the gap. The
genuine ones: 登录序列号, 获取课程评论错误, 获取Sport Banner, 高等数学, 发布于 今天.

### 7.6 Hardcoded strings

175 sites across 59 production files on iOS, 3 on Android. See [§6](#6-localization-requirements).

### 7.7 Unreachable copy

Both platforms ship a set of cutesy weather strings that nothing renders:

| iOS | Android |
| --- | --- |
| 下雨啦～出门记得小心脚下哦☔ | 下雨了，注意安全哦 |
| 下雪啦～要不要出去看看呀？❄️ | 外面下雪了，不出去看看吗 |
| 空气有点不太好，今天尽量少出门哦😷 | 空气不好，尽量少外出 |
| 天气超棒！快出去玩一玩吧～☀️ | 天气很好，快出去耍吧 |

Neither side composes any of them. Delete, and keep the plain register.

---

## 8. Review checklist

- [ ] Every user-visible string comes from a strings file — nothing hardcoded.
- [ ] The string lives in exactly one catalog. If a second target needs it, reference rather
      than duplicate — see [§5.1](#51-ios-has-six-catalogs-not-one).
- [ ] The key follows `<module>_<element>[_<qualifier>]` in snake_case, ASCII only.
- [ ] The key describes the string's **role**, not its current wording.
- [ ] Added to **all three** locales in the same commit.
- [ ] Terminology matches [§2](#2-terminology) — no forbidden variant.
- [ ] No terminal punctuation on labels, buttons, toasts, or status text.
- [ ] Colons are full-width with no space; Latin words padded, CJK not.
- [ ] Placeholder word order and spacing match the other platform.
- [ ] Countable numbers use a plural rule.
- [ ] The same string exists on the other platform with the same meaning — or the difference is
      listed in [§7](#7-current-state).
