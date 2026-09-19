# All-courses list — verification record

Companion to `tasks/course-all-list.md`. Records what was built, what was run, and where the
captured evidence lives, so the two client pull requests can be reviewed against a fixed set of
facts.

Client pull requests: whu-ham/ham-ios#167 · whu-ham/ham-android#131.

Workspace issue: whu-ham/ham-workspace#18 · Workspace PR: whu-ham/ham-workspace#19

## What was done

1. **New worktree on latest main.** `ham-workspace` fetched, and a fresh worktree created at
   `.worktrees/task-7-course-all-list-r2` on branch `feat/course-all-list-mobile-r2` from `origin/main`
   (2c070fec). Every submodule was then fetched and checked out at latest `origin/main`:
   ham-android 1b68e365, ham-backend-go 253e206, ham-ios 42e32927, ham-proto d63af3e, ham-rn 7578b30,
   ham-web b614546, whu-ham.github.io 4312d20.

2. **Feature implemented in both clients** (the previous round's unmerged work was ported onto the new
   latest main, re-reviewed, and fixed):
   - **iOS (whu-ham/ham-ios#167, commit d856ac8f)** — `CourseViewHeaderView` (legacy + iOS 26 variants)
     gains a leading `list.bullet` button (`course_header_all_course_list`) pushing
     `Route.courseAllList` → new `CourseAllListView` / `CourseAllListViewModel`, plus a
     `CourseService.getCurrentSemesterCourseInfo()` and 3 new localizable keys in en/zh-Hans/ja.
   - **Android (whu-ham/ham-android#131, commit 6c6b42e2c)** — `CourseMainViewWeekChoiceView` gains a
     leading `TableRows` button (`course_week_all_course`) navigating `CourseRoutePath.ALL_LIST` → new
     `AllCourseListView` / `AllCourseListViewModel`, route registration in `CourseGraph`, 3 locale
     files, module README updated, JVM unit tests added.
   - **Workspace (whu-ham/ham-workspace#19)** — scoped task doc `tasks/course-all-list.md`.
   - Every functional commit carries `Workspace-Issue: whu-ham/ham-workspace#18`.

## Page behaviour (both clients, identical layout)

One row per course (grouped by course id, fallback name + instructor): name, status badge, teacher ·
room, a 7-cell weekday strip filled where the course meets (same course colour as the grid), week
range, credit. Sections ordered 进行中 → 未开始 → 已上过 (in progress / not started / already taken),
so already-taken courses are the last section, reached by scrolling/swiping up. Row order inside a
section: earliest weekday, earliest period, name. Read-only over the local store — no network,
backend, or protobuf change.

## Verification actually run

| Check | Command | Result |
| --- | --- | --- |
| Android compile | `./gradlew :feature:course:compileDebugKotlin` | BUILD SUCCESSFUL (3m40s) |
| Android app + tests build | `./gradlew :app:assembleGithubDebug :app:assembleGithubDebugAndroidTest` | BUILD SUCCESSFUL |
| Android JVM unit tests | `./gradlew :feature:course:testDebugUnitTest` | BUILD SUCCESSFUL — 7 new cases + existing pass |
| Android E2E on emulator (Medium_Phone AVD, API 37) | `adb shell am instrument -w -e class com.nowcent.ham.e2e.suite.course.CourseAllCourseListE2eTest com.nowcent.ham.test/com.nowcent.ham.HiltTestRunner` | `OK (3 tests)` — also `connectedGithubDebugAndroidTest`: 3 run / 0 failed |
| iOS app build | `xcodebuild -workspace Ham/Ham.xcworkspace -scheme "Ham (iOS)" -configuration Debug -destination 'platform=iOS Simulator,name=iPhone 17 Pro' build` | ** BUILD SUCCEEDED ** |
| iOS E2E on simulator (iPhone 17 Pro) | `xcodebuild -workspace Ham/Ham.xcworkspace -scheme "Tests iOS" -configuration Debug -destination 'platform=iOS Simulator,name=iPhone 17 Pro' -only-testing:Tests\ iOS/HamE2ECourseAllListTests test` | ** TEST SUCCEEDED ** — 3 run / 3 passed / 0 failed (61.5s) |

## Evidence artifacts (real UI, captured from the running apps)

iOS (1206×2622, exported from `/tmp/task7-ios-e2e.xcresult`):
- `evidence/ios/course-timetable-ios.png` (232,135 B) — timetable with the new leading button left of the previous-week chevron
- `evidence/ios/all-course-list-ios.png` (184,716 B) — pushed "All Courses" page: In progress section, 3 rows, each with 7-cell strip, week range, credits
- `evidence/ios/xcresult-raw/` — raw xcresult attachment export + manifest

Android (1080×2400, PixelCopy captures pulled off the emulator):
- `evidence/android/course-timetable-android.png` (101,816 B) — timetable with the new leading button left of the previous-week button
- `evidence/android/all-course-list-android-top.png` (169,526 B) — "All Courses" page top: In progress rows with 7-cell strips
- `evidence/android/all-course-list-android-completed.png` (175,615 B) — after swipe-up: In progress → **Not started** → **Completed** (already taken) last

All paths are relative to `/Users/orangeboy/Projects/ham-workspace/.worktrees/task-7-course-all-list-r2/`.

## Key findings

- The previous round's work was never committed or pushed, its iOS E2E test did not compile
  (`HeaderButton.allCourseList` did not exist), its Android graph file had a duplicate import, and it
  sat on stale submodule heads. All fixed in this round.
- Real test bugs found while verifying on-device: iOS `When.tapIdentifier` never expired the 4 s label
  cache (fixed in shared `Assertions.swift`); the third iOS case's `showsNone(["Mon", …])` premise was
  wrong because each row's own week strip prints weekday names (rewritten to assert the section band);
  `Label.courseAllList` had to live under `Label`, not `Seed`.
- Android JVM unit tests needed `testOptions { unitTests { isReturnDefaultValues = true } }` because
  `displayColor` touches `android.graphics.Color.parseColor` through `toColorInt()`.
- Environment-only hiccups (resolved, no source change): stale Gradle cache for
  `:integration:social-login`; `Ham/Podfile.lock` checksum drift after `pod install` was **left out of
  the commit** to keep the PR diff clean.

## Recommended next action

Review and merge whu-ham/ham-ios#167 and whu-ham/ham-android#131, then merge the workspace doc PR
whu-ham/ham-workspace#19 (optionally bumping the two submodule pointers once the client PRs are
merged) so whu-ham/ham-workspace#18 closes.
