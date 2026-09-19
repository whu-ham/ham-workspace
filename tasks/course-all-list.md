# Course timetable: all-courses list

Give the timetable a way to review every course of the term at once, instead of one week at a
time, on iOS and Android with the same layout.

Workspace issue: whu-ham/ham-workspace#18.

## Entry

A leading button in the timetable header, before the previous-week chevron:

- iOS — `CourseViewHeaderView` (both the legacy and the iOS 26 variant) gains an
  `Image(systemName: "list.bullet")` button with the accessibility identifier
  `course_header_all_course_list`, pushing `Route.courseAllList`.
- Android — `CourseMainViewWeekChoiceView` gains a `HamButton` with `Icons.Rounded.TableRows`
  and the test tag `course_week_all_course`, navigating to `CourseRoutePath.ALL_LIST`.

## The page

A read-only list over the courses the device already stores — no network, no backend or protobuf
change, because both clients already keep the whole semester locally.

- One row per course. A course that meets several times a week is one row, not one row per
  meeting: rows are grouped by course id (falling back to name + instructor when the id is
  empty).
- A row carries the course name, a status badge, the teacher and room, a strip of seven weekday
  cells filled where the course meets, the week range, and the credit. The cells use the same
  course colour the timetable grid uses, and the same readable foreground the grid picks for it.
- Rows are sectioned 进行中 / 未开始 / 已上过 — in progress, not started, already taken — in that
  order, so the courses still ahead of the student come first and the ones already taken are the
  last section, reached by scrolling up.
- A course whose week range cannot be placed on the timeline is treated as running rather than
  hidden, because the grid draws it too.

## Parity rules

The two clients must match on: the button's position (leading, before the previous-week
control), the section order, the row order inside a section (earliest weekday, then earliest
period, then name), the row content and its spacing, the seven-cell week strip with the same
weekday labels, the status badge colours (green / blue / secondary), and the dimming of rows
already taken. `AllCourseListViewModel.buildSections` and
`CourseAllListViewModel.buildSections` are the two halves of the same rule: same inputs, same
ordering, same status boundaries.

## Scope

`ham-ios` and `ham-android` only.
