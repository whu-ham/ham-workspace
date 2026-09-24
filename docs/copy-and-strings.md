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

> **Updated 2026-09-24.** This section was written when iOS used `SCREAMING_SNAKE_CASE` and, mostly,
> the Chinese text itself as the key. That world ended with `34a59411 refactor(i18n): move to a
> String Catalog and one structured key convention (#175)`, which deleted every
> `Ham/{zh-Hans,en,ja}.lproj/Localizable.strings` and replaced them with `Ham/Localizable.xcstrings`
> (817 keys, four locales). The measured state is now:

| | iOS | Android |
| --- | --- | --- |
| Convention | `<area>.<screenOrOwner>.<thing>` — dotted lower-camelCase, always prefixed | `snake_case`, loosely module-prefixed |
| Example | `course.main.deleteThisClass` (删除这节课) | `library_reserve` |
| Keys | **817**, of which **0** contain an underscore | (unchanged) |
| Module prefix | always — 34 areas; `common` alone is 311 of 817 | inconsistent (`common_`, `library_`, `sport_`, but also bare `select_color`, `picker_confirm`) |
| Keys are Chinese | **13 of 817 (1.6%)**, all deliberate | 0 |

Before the migration, **777 of 898 (87%) iOS entries used the Chinese text itself as the key** —
765 of 898 unique keys, where the entry is literally `"预约" = "预约"`. That meant rewording any
string silently changed its key and orphaned the other locales, keys could not be searched or
grouped by module, and a typo in the copy became a permanent identifier. All three failure modes
are gone on iOS.

What remains:

- **13 keys are still bare Simplified Chinese** — `总馆` · `信息分馆` · `工学分馆` · `医学分馆` ·
  `专业教育必修` · `专业教育选修` · `通识教育必修` · `通识教育选修` · `公共基础必修` · `跨学院专业课`
  · `羽毛球` · `乒乓球` · `健身房`. Each carries the comment *"Data alias: a server value looked up
  through this table at runtime."* They are server enum values used as lookup keys, so they are
  Chinese **by design**, not by neglect — but they still cannot be reworded safely.
- **Android still ships unprefixed keys** (`select_color`, `picker_confirm`, `not_set`).
- **The two platforms still spell the same slot differently**, so the catalogs cannot be
  byte-diffed. See §4.3.

### 4.2 The standard

**One namespace, two spellings.** Both platforms fill the same three slots in the same order; only
the separator and the letter case differ.

| | iOS | Android |
| --- | --- | --- |
| Shape | `<area>.<screenOrOwner>.<thing>` | `<module>_<element>[_<qualifier>]` |
| Separator | `.` | `_` |
| Case | lowerCamelCase per segment | snake_case |
| Segments | 2 or 3 — 377 keys have 2, 427 have 3 | 2 or 3 |

| Part | Rule | Examples |
| --- | --- | --- |
| `<area>` / `<module>` | the feature area | iOS: `common` (311) · `course` (83) · `library` (59) · `score` (57) · `coursescore` (51) · `status` (50) · `shared` (44) · `print` (29) · `schedule` (29) · `widget` (24) · `sport` (13) · `sso` (11) · `usercenter` (11) · `sync` · `watch` · `about` · `my` · `siri` · `main` · `bus` · `pay`. Android: `library`, `sport`, `score`, `course`, `coursescore`, `schedule`, `status`, `my`, `user_center`, `common` |
| `<screenOrOwner>` | the owning view or view-model, lowerCamelCase | `printPrepareView`, `userCenterView`, `sSOAuthorizationSheet` |
| `<thing>` / `<element>` | what it is | `loadError`, `noPrinterFound`, `logInAgain` |

Two area names changed in the migration and must be used in their new form on iOS: **`user_center`
is now `usercenter`**, and **`cas` is now `sso`** (with `usercenter` taking the scan/login screens).
Four areas are new and were not in the previous list: `shared`, `print`, `widget`, `watch`.

Rules:

1. **ASCII only.** Never use Chinese, spaces, or punctuation in a key. The 13 data-alias keys above
   are the only sanctioned exception, and each must carry the `Data alias` comment.
2. **Never use the copy text as the key.** The key describes the string's *role*; the value is the
   words. This is what makes rewording safe, and it is now true of 804 of 817 iOS keys.
3. **Prefix by area, always** — including on iOS, where the catalog now enforces it.
4. **Name the role, not the current wording.** `sport_order_cta`, not `sport_yuding`.
5. **Do not reserve suffixes.** An earlier revision of this section prescribed `_title`,
   `_subtitle`, `_desc`, `_cta`, `_empty`, `_error`, `_success`, `_confirm`, `_hint`. Neither
   platform works that way: of 817 iOS keys, **0** end in `Title`, `Subtitle`, `Cta`, `Success`,
   `Confirm`, `Hint` or `Message`; 5 end in `Error`, 3 in `Empty`, 3 in `Desc`. Put the role word
   inside the segment — `loadError`, `noPrinterFound`, `uploadFailed` — where it reads the same in
   both spellings.

### 4.3 Migration

- **iOS: done.** `34a59411` converted all 898 entries to 817 structured keys in one pass, added
  **zh-Hant** as a fourth locale, and moved plurals into the catalog. Do not reintroduce Chinese
  or `snake_case` keys.
- **Android: the remaining work.** Tighten the unprefixed keys (`select_color`, `picker_confirm`,
  `not_set`) into a module namespace as they are touched.
- **Diffing the two catalogs** no longer needs the Chinese text as a join key. Normalise
  mechanically — `.` ↔ `_`, and lower-camelCase ↔ snake_case per segment — then diff. The residual
  mismatches are the finding: 未登录 is `common.notLoggedIn` on iOS but `sport_setting_not_logged_in`
  on Android, so the same string sits in two different area namespaces.

---

## 5. File organisation

### 5.1 iOS has five catalogs, not one

This is easy to miss and is the source of several current defects. Strings are spread across
every app target, and each target has its own bundle — so **a string in the main catalog is
invisible to the widget and the Siri extension**.

| # | Catalog | Locale dirs | Entries (zh) | Key style |
| --- | --- | --- | --- | --- |
| 1 | `Ham/Localizable.xcstrings` | **4** | **817** | 1.6% Chinese-as-key (13 data aliases) |
| 2 | `Ham/{locale}.lproj/InfoPlist.strings` | 4 | 4 | `CFBundle*` |
| — | ~~`Ham/SiriIntent/{locale}.lproj/Localizable.strings`~~ — **deleted by `34a59411`**; its two strings moved into catalog 1. `Ham/SiriIntent/{locale}.lproj/Intents.strings` (29 generated IDs) survives below. | — | — | — |
| 3 | `Ham/SiriIntent/{locale}.lproj/Intents.strings` | 4 | 26 | Xcode-generated (`956YUW`) |
| 4 | `Ham/Widget/{locale}.lproj/WidgetIntentConfiguration.strings` | 4 | 48 | Xcode-generated (`0yHcjK`) |
| 5 | `Ham/SiriIntentUI/{locale}.lproj/MainInterface.strings` | 4 | 2 | Interface Builder |

Catalogs 3–4 are **not** duplication for its own sake — Siri intent and widget-configuration
strings *must* live in their target's bundle, and their keys are generated by Xcode from the
intent definition. That part is platform-forced.

What is **not** forced, and is wrong:

- **Shared copy duplicated across catalogs.** The widget update-interval options (5分钟,
  10分钟, 15分钟, 30分钟, 1小时, 2小时) are defined in both catalog 1 and catalog 4. They can
  drift, and 10分钟 already appears four times inside catalog 4 alone.
- **Untranslated English in the zh-Hans catalogs.** `MDC7j9 = "Success Message"`,
  `iQFhaG = "Error Message"`, `jO5xID = "Error Message"` in
  `SiriIntent/zh-Hans.lproj/Intents.strings`. These are default values nobody translated.
- **Cutesy tone in the Siri catalog.** `未登录图书馆，先前往我的-图书馆登录哦～` violates
  [§3 tone](#3-copy-rules) and uses a `-` nav notation found nowhere else.

### 5.2 The rule

| Platform | Layout |
| --- | --- |
| iOS | One `Ham/Localizable.xcstrings` holding all four locales; plurals inline as `substitutions`; grouping is by the dotted key prefix, not comments. Target-specific catalogs (Siri, widget) hold **only** strings that must live there. |
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

iOS supports **four** locales — `Ham/{zh-Hans,zh-Hant,en,ja}.lproj/`, and `Localizable.xcstrings`
is `sourceLanguage: en` with all four present on every one of its 817 keys. Android supports
**three** — `values` (zh-Hans), `values-en`, `values-ja`. **zh-Hant is now an iOS-only locale**, which
is a divergence this document did not previously have to record. A new string must be added to every
locale the platform ships, in the same commit. If a translation is genuinely unavailable, add the
default value rather than leaving the key absent — a missing key renders the raw key to the user.

### No hardcoded user-visible text

Every user-visible string lives in a strings file. No exceptions.

The current gap, re-measured on iOS `main` `9a9c6a3f` (2026-09-24) with a Swift lexer that walks
each non-test file skipping comments and string delimiters, and that classifies a CJK literal as
hardcoded unless it is routed through the localization system (`String(localized:)`,
`NSLocalizedString`, `.localized`), is a `Log.*` argument, or is a parse / compare key:

| Module | Hardcoded CJK literals |
| --- | --- |
| Sport | **13** |
| Library | **40** |
| My / user center | **6** |
| Shared | **4** |
| Status | **0** |
| Score + schedule | **23** |
| Course | **0** |
| CourseScore | **0** |
| Everything else (Widget, print, privacy, CAS, pay, bus, sync, …) | **64** |
| **Total, all of `Ham/`** | **150** across **55** files |
| &nbsp;&nbsp;of which fixture / preview / debug-only | 22 |
| **Production sites** | **128** across **47** files |

The two earlier counts (**282**, and the **175**-site grep before it) are **withdrawn**: applying
this same rule to the pre-migration `811c03f6` yields 339, so neither number is reproducible
there either. The migration `34a59411` removed the single largest cause — model-level status
names that were CJK literals resolved through `.localized` at the call site.

Notes:

- **Library is still the worst module** — 40 sites, 31 distinct literals, 12 files, including 2
  `Hello, World!` stubs (`LibraryEducationQuickLoginView.swift:12`,
  `LibraryMainViewBannerImageCover.swift:12`). 32 of the 40 already exist as a `zh-Hans` value
  somewhere in `Ham/Localizable.xcstrings` under a new dotted key — they are unreferenced values,
  not missing keys. Across all 150 sites, 90 have a matching catalog value.
- **Sport's status names are no longer literals** — `SportOrderDetail.swift:23-35` maps every case
  to `String(localized:)` with a dotted key (`common.unknown`, `common.pendingPayment`,
  `common.pendingUse`, `common.inUse`, `common.used`, `shared.bookingVO.canceled`,
  `common.refunded`). The surviving model-level status names are **library** booking states:
  `LibraryModel.swift:49-56` hardcodes 8 (预约 / 履约中 / 暂离 / 已结束 / 已取消 / 失约 / 早退 /
  未签退).
- **Two live bugs**: `Ham/Widget/Course/CourseWidget.swift:321` passes `今天` / `明天` to
  `String(localized:)`, and no such keys exist in the catalog (keys are dotted), so the widget
  renders the raw literal — use `status.today` / `status.tomorrow`.
  `Ham/shared/basic/location/LocationManager.swift:192` passes a whole Chinese sentence as the
  `withPurposeKey:` argument, but that argument is an **`InfoPlist.strings` key** — the sentence is
  a *value* there, under `NSLocationWhenInUseUsageDescription`
  (`Ham/zh-Hans.lproj/InfoPlist.strings:7`). The lookup misses, so the prompt is untranslatable.
- **Android's Debug screen is absent from all `.lproj`/`values-*` files** — it is entirely
  English (`DebugView.kt:64,71,81,105,137,179,196`), which is why it contributes no CJK site.
- **Corrected 2026-09-24**: the often-cited Android `Text("关闭")` is **not** a shipped string — it
  sits inside `private fun Preview()` in `core/ui/container/Sheet.kt:293`. Android's shipped
  hardcoded CJK is overwhelmingly in the model layer, not in composables: 79 sites across 19 files,
  led by `BookingVO.kt` (15 booking-status names), `CCKVDefaultValue.kt` (14 server-default titles,
  some with a ja twin inline), `ResponseBean.kt` (14 error strings), `DateTimeUtils.kt` (8 weekday
  labels), `HamResponse.kt` (5), `BiometricUtils.kt` (4) and `ToastManager.kt` (3). See
  [§7.6](#76-hardcoded-strings).
- **Server-driven copy with no fallback**: ten categories of user-visible text arrive from CCKV
  or the server with no catalog fallback on either platform — consent messages, external-service
  titles, want/comment card titles and hints, the comment-disabled reason, the not-found message,
  the comment input hint, and the course-centre card titles.

### Plurals

Use plural rules for any string containing a countable number.

| Platform | Mechanism | Current state |
| --- | --- | --- |
| iOS | in-catalog `substitutions` + `NSStringPluralRuleType` | **8 keys** (`status.hours`, `status.minutesRemaining`, `status.overMinutes`, `status.hoursRemaining`, `status.pendingSchedules`, `status.pendingSchedulesThisWeek`, `status.studiedMinutesInTotal`, `library.history.totalMinutes`) — `Localizable.stringsdict` no longer exists |
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
| Hardcoded CJK literals | 150 (see [§6](#6-localization-requirements)) |
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

Strings are spread across five catalogs (see [§5.1](#51-ios-has-five-catalogs-not-one)). Three
concrete defects fall out of it:

| # | Issue | Where |
| --- | --- | --- |
| 1 | Widget update-interval options duplicated — 5分钟/10分钟/15分钟/30分钟/1小时/2小时 exist in both the main catalog and the widget's | `Ham/Localizable.xcstrings` + `Widget/zh-Hans.lproj/WidgetIntentConfiguration.strings` |
| 2 | Untranslated English defaults sitting in the zh-Hans catalog — `Success Message`, `Error Message` ×2 | `SiriIntent/zh-Hans.lproj/Intents.strings` (`MDC7j9`, `iQFhaG`, `jO5xID`) |
| 3 | 15 duplicated values inside the 48-entry widget catalog — 10分钟 appears 4×, `确认一下，是指"10分钟"对吗？` appears 4× | `Widget/zh-Hans.lproj/WidgetIntentConfiguration.strings` |
| 4 | ~~Cutesy tone and a `-` nav notation in the Siri catalog — `未登录图书馆，先前往我的-图书馆登录哦～`~~ **resolved**: that catalog was deleted by `34a59411`; the string no longer ships. | — |
| 5 | 4 duplicated values in the 26-entry Siri intent catalog — `${errorMessage}` appears 5× | `SiriIntent/zh-Hans.lproj/Intents.strings` |

### 7.2 Terminology

| # | Issue | Where |
| --- | --- | --- |
| 1 | 预约/预定 split — library uses 预约, sport uses 预定, on **both** platforms | ~20 strings per platform |
| 2 | Sport order footer CTA: iOS 预约 vs Android 预定 — the only true cross-platform split | `SportOrderViewFooter.swift:31` vs `sport_reserve` |
| 3 | 获取/更新 inversion in the score fetch flow: iOS 更新成功/更新失败 vs Android 获取成功/获取失败 for the identical step | iOS `ScoreUpdateByCasView` vs Android `strings.xml:29-31` |
| 4 | Same inversion in the course timetable fetch flow | both platforms |
| 5 | 其他/其它 — iOS still ships **both** spellings under two separate keys: `common.otherSettings` = 其他设置 is now **unused** (its only consumer, `SportSettingViewOtherCard.swift`, was deleted by `53a17721` #101), while `common.otherSettings2` = 其它设置 is the CAS row (`CasSettingView.swift:73`) and matches Android's `cas_other_settings`; Android also has `sport_setting_other_title` = 其它设置 | `Localizable.xcstrings`: `common.otherSettings` / `common.otherSettings2`; `CasSettingView.swift:73`; `cas/.../values/strings.xml:11`, `sport/.../values/strings.xml:57` |
| 6 | 没有历史记录 (iOS) vs 暂无历史记录 (Android) | `Localizable.xcstrings`: `common.noHistory` vs `library_no_history` |
| 7 | 加载异常 / 加载时遇到错误 / 加载时遇到了错误 — three spellings, one missing 了 | library + status |
| 8 | 今天/明天 vs 当天/隔天 — Android sport uses 明天/今天 where iOS uses 隔天/当天, which appear nowhere else | `sport_setting_day_tomorrow` vs `SportSettingViewStarredOrderCard.swift:38` |
| 9 | 还有不到一分钟 vs 还有不到1分钟 | `Localizable.xcstrings`: `status.lessThanAMinute` vs Android |
| 10 | 取消 vs 返回 vs 关闭 for dismiss actions | both |

### 7.3 Wrong values behind right-sounding keys

Three keys whose value contradicts their purpose. The full list of 60 is summarised in
[§7.0](#70-findings-from-the-full-screen-audit).

| Issue | Where |
| --- | --- |
| `"CONFIRM" = "更改"` — the confirm key renders "change" | `Localizable.xcstrings`: `library.modifybooking.confirm` (en `Confirm`) |
| `"CANCEL" = "返回"` — the cancel key renders "back" | `Localizable.xcstrings`: `common.dismiss` (en `Cancel`); 返回 is also `common.back` (en `Back`) |
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
| iOS (main catalog) | **817** | **817** | **817** | **0** | **0** |
| Android | **976** | **968** | **968** | **0** | **0** |

**iOS** — every one of the 817 keys carries all four locales (`en`, `ja`, `zh-Hans`, `zh-Hant`);
the old 898 / 894 / 883 counts and the 17 + 6 gaps belonged to the `.strings` files deleted by
`34a59411`. 22 ja values are byte-identical to their `zh-Hans` value — most are correct Japanese
(同意, 名称, 医学, 工学, 保存, 今日, 明日, 昨日, 成功, 早退), so only a human pass can separate
those from real gaps. The five genuine gaps listed before are **all translated now**:
`登录序列号`, `获取课程评论错误`, `获取Sport Banner`, `高等数学`, `发布于 今天`. No orphan keys
remain — a key cannot exist in one locale only.

**Android** — 976 `<string>` entries in `values/` across 24 files against 968 in each of
`values-en/` and `values-ja/`. All 8 apparent gaps are legitimate: `translatable="false"` data and
format strings (`library_default_building_info`, `library_location_format`), language names
(`language_english`, `language_japanese`, `language_simplified_chinese`), the proper noun
`app_name`, and two strings that exist in the module that owns them (`automatic_tip` in
`feature/automatic`, `privacy_help` in `feature/my`) as well as in `app`.

### 7.6 Hardcoded strings

150 sites across 55 files on iOS (128 in production code across 47 files; 22 in fixtures,
previews and debug-only views) and **79 sites across 19 files on Android**, measured with the same
rule on both: a CJK literal counts unless it is routed through the platform's localization system,
is a log/diagnostic argument, is a parse / compare / mapping key, or sits inside a `@Preview`
composable. See [§6](#6-localization-requirements).

Android's 79, by file:

| File | Sites |
| --- | --- |
| `core/foundation/…/bean/library/BookingVO.kt` | 15 |
| `core/configuration/…/CCKVDefaultValue.kt` | 14 |
| `core/foundation/…/bean/ResponseBean.kt` | 14 |
| `core/foundation/…/utils/DateTimeUtils.kt` | 8 |
| `core/network/…/standard/response/HamResponse.kt` | 5 |
| `core/foundation/…/utils/BiometricUtils.kt` | 4 |
| `core/ui/…/toast/ToastManager.kt` | 3 |
| `data/cas/…/model/BusLine.kt` | 3 |
| 11 more files | 13 |

Excluded from that 79: 30 log arguments, 17 parse/compare literals, 12 mapping branches
(`"信息馆" -> stringResource(…)`), 12 preview-only literals, and one 51-entry keyword table
(`WeatherTranslationHelper.kt`, which maps server weather strings to `R.string` ids).

The 12 heaviest iOS sites:

| Site | Copy |
| --- | --- |
| `Ham/shared/business/library/service/jsq/LibraryModel.swift:49-56` | 8 booking status names (预约 / 履约中 / 暂离 / 已结束 / 已取消 / 失约 / 早退 / 未签退) |
| `Ham/shared/basic/model/vo/response/Response.swift:53,55,89,91,93,95,101` + `Ham/shared/basic/request/response/HamResponse.swift:35,37,39,41` | 11 error strings |
| `Ham/shared/business/education/EducationRequestHelper.swift:43,65,79,85,108,127` | 6× 登录失败 + 6× 请重新登录 |
| `Ham/shared/business/cas/CasRequestHelper.swift:109,115,146,153,160` | 5× 解析文档失败 |
| `Ham/iOS/ui/library/main/component/LibraryMainViewCurrentBookingCard.swift:61,63,65,67` | 4 hardcoded map queries (`武汉大学信息学部图书馆`, …) |
| `Ham/iOS/ui/my/component/card/MyUserCenterCard.swift:48` | `Text(vm.isCasLogin ? "管理信息门户设置" : "登录信息门户")` |
| `Ham/iOS/ui/user-center/scan/QrCodeLoginView.swift:28,30,54` | 确认登录 / 返回 / 确定在电脑上登录Ham吗 |
| `Ham/iOS/ui/schedule/insert/ScheduleInsertView.swift:41,142,279`, `ScheduleInsertMoreDataView.swift:43`, `group/ScheduleGroupEditView.swift:38` | TextField prompts and navigation titles (输入日程名称 / 添加日程 / 编辑日程 / 输入地点(可选) / 名称) |
| `Ham/iOS/ui/score/ScoreViewModel.swift:51,59,72` | FaceID `localizedReason` and the two failure strings (保护你的成绩数据 / 请重新验证 / 你已开启成绩保护，请开启生物认证权限) |
| `Ham/iOS/ui/library/book/detail-book/component/LibraryDetailBookViewHeader.swift:44`, `LibrarySelectSeatView.swift:111` | `Text(LocalizedStringKey(… ?? "请选择图书馆"))` — the literal is a `String`, so the wrapper does not localise it |
| `Ham/iOS/ui/sport/select-area/common/SportSelectAreaViewHeader.swift:25` | `Text(LocalizedStringKey(em.selectedSportType?.title ?? "请选择运动类别"))` |
| `Ham/iOS/ui/library/main/component/LibraryMainViewCurrentBookingCard.swift:111` | `Text(expand ? "收起" : "展开")` |

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
      than duplicate — see [§5.1](#51-ios-has-five-catalogs-not-one).
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
