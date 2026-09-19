# All-courses list — verification record

Companion to `tasks/course-all-list.md`. Records what was built, what was run, and the layout
parity between the two clients, with the side-by-side captures this file references.

Client pull requests: whu-ham/ham-ios#167 · whu-ham/ham-android#131.
Workspace issue: whu-ham/ham-workspace#18.

## Where the button is — the same slot on both clients

Both clients put the button in the timetable's own header bar, as the leading control, immediately
left of the previous-week control:

- **iOS** — that bar is `CourseViewHeaderView` (previous week, week label, next week).
- **Android** — the same bar is `CourseMainViewWeekChoiceView`; "week switcher" is only its internal
  name, and it carries exactly the same previous-week button, week label and next-week button.

So the two screenshots below are the same control in the same place, not two different features.

## Layout parity (side-by-side, iOS left / Android right)

### 1. Timetable header — the leading all-courses button

![Timetable header, iOS vs Android](assets/parity-1-header.png)

The button is the leading control of the timetable header, left of the previous-week chevron, on
both. iOS draws `list.bullet`, Android draws `TableRows` — the same list mark in both icon sets.

![Timetable, full screen](assets/parity-2-timetable-full.png)

#### Measured geometry (not eyeballed)

| | iOS | Android |
| --- | --- | --- |
| Leading button frame | x = 10 pt, y = 56 pt, 48 x 46 pt on a 402 x 874 pt screen → x = 2.5% of the width, y = 6.4% of the height | list icon ink measured at x = 3.7-8.0% of the width, y = 4.8-6.5% of the height |
| Accessibility | identifier `course_header_all_course_list`, label "List" | test tag `course_week_all_course`, content description `course_all_course_title` |
| What sits to its right | the previous-week chevron | the previous-week chevron, measured at x = 12.8-14.3% of the width, i.e. immediately right of the button |

The iOS numbers come from an XCUI query run on the simulator against the running app
(`app.descendants(matching: .any).matching(identifier: "course_header_all_course_list").firstMatch`
→ `exists=true frame=(10.0, 56.0, 48.0, 46.0) label=List`), so the button in the capture above is
that element and not some other header icon. The Android numbers come from the capture's pixels.

## 2. The all-courses page

![All-courses list, iOS vs Android](assets/parity-3-all-course-list.png)

Both render: a navigation title (全部课程 / All Courses), one section header (进行中 / In progress),
one card per course with the same internal spacing, and the same 7-cell weekday strip.

### 3. One row, at readable size

![Row detail, iOS vs Android](assets/parity-4-row-detail.png)

Row content and order are identical: name + status badge on the first line, teacher · room on the
second, the seven weekday cells on the third (filled with the course colour where the course meets),
and week range + credit on the fourth.

### 4. Swiping up reveals the courses already taken — proven on both platforms

**iOS, before and after the swipe.** The case captures the page, asserts the already-taken course is
*absent* before the gesture (`then(.showsNone([Seed.courseAlreadyTaken]))`) and *present* after it,
then captures the page again.

![iOS before and after the swipe](assets/parity-5-ios-swipe-before-after.png)

**The same gesture on both platforms.** Left: iOS after swiping up. Right: Android after swiping up.

![iOS and Android after the swipe](assets/parity-6-swipe-completed-ios-android.png)

**The already-taken section at readable size.**

![Already-taken section detail](assets/parity-7-completed-section-detail.png)

How the iOS side is made possible: the shared E2E seed pins the term to the current Monday, which is
week 1, where no course can be in the past. The suite now opts into `-e2e_courseTermWeeks 3`, which
starts the seeded term three weeks ago and adds two seeded courses — one that ended before the week
the app opens on (`E2E Done Course`), one that starts after it (`E2E Later Course`). Both sit on week
ranges the opening week does not cover, so the grid every other suite reads is unchanged, and only
this suite asks for them.

| | iOS | Android |
| --- | --- | --- |
| Before the swipe | `E2E Done Course` not on screen (asserted) | `E2E Done Course` below the fold (asserted) |
| Gesture | `when(.scrollUp)` — a drag from 75% to 35% of the window, repeated until the row appears | `performTouchInput { swipeUp() }`, repeated until the section appears |
| After the swipe | `Completed` / 已上过 section with `E2E Done Course` | `Completed` / 已上过 section with `E2E Done Course` |

## Source captures

| File | What |
| --- | --- |
| `assets/course-timetable-ios.png` | iOS timetable with the new leading button (1206x2622) |
| `assets/course-timetable-android.png` | Android timetable with the new leading button (1080x2400) |
| `assets/all-course-list-ios.png` | iOS all-courses page (1206x2622) |
| `assets/all-course-list-android-top.png` | Android all-courses page, top (1080x2400) |
| `assets/all-course-list-android-completed.png` | Android all-courses page after swiping up (1080x2400) |
| `assets/all-course-list-ios-before-swipe.png` | iOS all-courses page before the swipe (1206x2622) |
| `assets/all-course-list-ios-after-swipe.png` | iOS all-courses page after the swipe (1206x2622) |

## Verification actually run

| Check | Command | Result |
| --- | --- | --- |
| Android compile | `./gradlew :feature:course:compileDebugKotlin` | BUILD SUCCESSFUL (3m40s) |
| Android app + tests build | `./gradlew :app:assembleGithubDebug :app:assembleGithubDebugAndroidTest` | BUILD SUCCESSFUL |
| Android JVM unit tests | `./gradlew :feature:course:testDebugUnitTest` | BUILD SUCCESSFUL — 7 new cases + existing pass |
| Android E2E on emulator (Medium_Phone AVD, API 37) | `adb shell am instrument -w -e class com.nowcent.ham.e2e.suite.course.CourseAllCourseListE2eTest com.nowcent.ham.test/com.nowcent.ham.HiltTestRunner` | `OK (3 tests)`; `connectedGithubDebugAndroidTest`: 3 run / 0 failed |
| iOS app build | `xcodebuild -workspace Ham/Ham.xcworkspace -scheme "Ham (iOS)" -configuration Debug -destination 'platform=iOS Simulator,name=iPhone 17 Pro' build` | ** BUILD SUCCEEDED ** |
| iOS E2E on simulator (iPhone 17 Pro) | `xcodebuild -workspace Ham/Ham.xcworkspace -scheme "Tests iOS" -configuration Debug -destination 'platform=iOS Simulator,name=iPhone 17 Pro' -only-testing:Tests\ iOS/HamE2ECourseAllListTests test` | ** TEST SUCCEEDED ** — 3 run / 3 passed / 0 failed (66.2s), including `testSwipingUpRevealsTheCoursesAlreadyTaken` |

Both clients were started from `ham-workspace` pinned at `origin/main` (2c070fec) with every
submodule at its latest `origin/main`: ham-android 1b68e365, ham-ios 42e32927, ham-backend-go
253e206, ham-proto d63af3e, ham-rn 7578b30, ham-web b614546, whu-ham.github.io 4312d20.

## Key findings

- The earlier round's work was never committed or pushed, its iOS E2E test did not compile
  (`HeaderButton.allCourseList` did not exist), its Android graph file had a duplicate import, and it
  sat on stale submodule heads. All fixed in this round.
- Real test bugs found while verifying on-device: iOS `When.tapIdentifier` never expired the 4 s label
  cache (fixed in shared `Assertions.swift`); the third iOS case's `showsNone(["Mon", ...])` premise
  was wrong because each row's own week strip prints weekday names (rewritten to assert the section
  band); `Label.courseAllList` had to live under `Label`, not `Seed`.
- Android JVM unit tests needed `testOptions { unitTests { isReturnDefaultValues = true } }` because
  `displayColor` touches `android.graphics.Color.parseColor` through `toColorInt()`.
- Environment-only hiccups (resolved, no source change): stale Gradle cache for
  `:integration:social-login`; `Ham/Podfile.lock` checksum drift after `pod install` was left out of
  the commit to keep the PR diff clean.
