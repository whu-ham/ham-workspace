# Course comments: edit/delete, like push, and moderated images

Tracks `whu-ham/ham-workspace#20`.

Extends the course comment feature across four submodules. `ham-web` and
`ham-rn` have no comment surface today, so they are out of scope.

| Submodule | Change |
| --- | --- |
| `ham-proto` | Add image, edit, delete, detail and upload contracts |
| `ham-backend-go` | Implement the RPCs, image moderation and the like push |
| `ham-android` | Edit/delete UI, comment detail screen, image grid, push tap routing |
| `ham-ios` | The same surface as Android |

## 1. Contract (`ham-proto`)

All changes land in `course_detail/course_detail_comment.proto` and
`course_detail/course_detail_rpc.proto` — the current contract. The legacy
top-level `course_comment.proto` is untouched.

```proto
message CourseDetailCourseCommentImage {
  string image_url = 1;
  int32 width = 2;   // 0 means unknown; lets one-image comments keep their aspect
  int32 height = 3;
}
```

- `CourseDetailCourseCommentBody.image_list` (field 2) — up to 9 images.
- `CourseDetailCourseCommentItem.update_time` (field 9, optional) and
  `.edited` (field 10) — an edited flag avoids clients comparing timestamps.
- `CreateCourseDetailCourseCommentBody.image_list` (field 2).
- `DeleteCourseCommentRequest.comment_id` — the message already existed but was
  empty, so delete was unimplementable.
- New `UpdateCourseCommentRequest/Response`, `GetCourseCommentDetailRequest/
  Response`, `UploadCourseCommentImageRequest/Response`.
- New services `UpdateCourseCommentService`, `GetCourseCommentDetailService`,
  `UploadCourseCommentImageService` (client-streaming, mirroring
  `ModifyUserAvatar`). `DeleteCourseCommentService` was already declared.

`course_center/course_center_course_comment.proto` reuses
`CourseDetailCourseCommentItem`, so comment history gets images for free.

## 2. Moderation: images are reviewed before they can be posted

There is no moderation queue in the backend today, and adding one (pending
state, admin review, visibility filtering on every list query) is a much larger
change than the requirement needs. Decision: **moderate synchronously, twice.**

1. `UploadCourseCommentImage` writes the bytes to OSS and calls
   `moderation.Client.TestImage(url)`. A rejected image never returns a URL, so
   the client cannot attach it.
2. `CreateCourseComment` / `UpdateCourseComment` re-validate every URL: it must
   start with `oss.Client.GetURLPrefix()` (so a client cannot smuggle in an
   arbitrary external URL) and must pass `TestImage` again.

Any failure returns `errorx.BadRequest`. Text keeps using the existing
`TestText` check. Because images are only ever persisted after passing, no
`status` column and no visibility filtering in the list queries is needed.

## 3. Backend (`ham-backend-go`)

- `CourseCommentUpdateTime` column + `Deleted` handling: delete uses the
  existing gorm soft delete (`DeletedAt`); edit sets `UpdateTime`.
- Images are stored as a JSON column on `course_comment` (a single comment can
  only ever hold 9 images and is always read whole — a join table buys nothing).
- Repository: add `Update`, and a `FindByID` variant that still returns
  soft-deleted rows is *not* needed — soft-deleted comments must disappear.
- Ownership check in the use case: `contextx.GetContext(ctx).UserID()` must
  equal `courseComment.UserID`, otherwise `errorx.Forbidden`.
- Like push: add a `push.Type` constant and inject `push.Client` into
  `CourseCommentLikeUseCase`. Push only on a new like, not on unlike, and never
  to the author (self-likes are already rejected).
- Errors use `errorx.GenerateAppErrorWithMessage`; handlers return them bare.

## 4. Push payload and deep link

The push envelope is already `{"app":"ham","type":...,"data":...}` on both
clients. New type `course_commentLiked`, data:

```json
{ "commentId": "123456789", "courseName": "高等数学" }
```

Deep link URI: `ham://comment/<commentId>`.

- Android: `VALID_DEEPLINK_HOST` is a hard allow-list in `DeepLinkHandler`, so
  the `comment` host must be added there, plus a branch that builds the typed
  route. TPNS tap handling (`XGPushReceiver.onNotificationClickedResult`) is an
  empty stub today and must be filled.
- iOS: add a `comment` host branch to `DeepLinkManager.handle(url:)` returning a
  new `Route.commentDetail(...)` case. `xgPushDidReceiveNotificationResponse` is
  empty today and must be filled.

`android/docs/05-ai-development-guide.md` flags deep link and cold start
changes as needing human review — call this out in the PR description.

## 5. Image layout

Two different surfaces, two different rules.

**Comment detail** — everything the comment carries, up to 9:

| Count | Layout |
| --- | --- |
| 1 | single image, original aspect, bounded to ~60% width |
| 2 | 2 columns |
| 4 | 2x2 |
| 3, 5–9 | 3 columns |

**Comment list rows** — capped at **3** images in a single row of three, with a
`+N` overlay on the last visible thumbnail when the comment has more. Feed rows
stay scannable and the tap target ("open the comment") stays obvious; the detail
page carries the full set.

Neither client has an image grid component today, so each builds one.

## 6. Tests

- `ham-backend-go`: Ginkgo + gomock unit tests for update/delete/detail/upload
  and the like push; repo tests against in-memory SQLite.
- `ham-android`: JUnit unit tests for the new ViewModel logic; Compose e2e
  suites under `app/src/androidTest/.../e2e/suite/coursescore/`.
- `ham-ios`: XCUITest e2e cases under `Ham/Tests iOS/`, following
  `E2E_CONVENTIONS.md` (register new files with `register_tests_file.py`, add
  `PbTestFixtures` entries for every new gRPC path).

## 7. Merge order

1. `ham-proto` merges and is auto-tagged.
2. Each consumer bumps `PROTO_VERSION` to that tag, then merges.

Consumer branches are developed against the proto feature branch; the
`PROTO_VERSION` bump to the released tag happens after step 1.
