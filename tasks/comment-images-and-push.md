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

1. `ham-proto` merges and is auto-tagged. — not done, and it is the
   maintainer's call, not the implementer's.
2. Each consumer bumps `PROTO_VERSION` to that tag, then merges.

For development the contract is published as a pre-release tag cut from the
branch — `v1.0.12-beta.1`, following the repo's existing `v1.0.4-beta.1` /
`-beta.2` convention — and all three consumers pin it. That is enough to build
and test without touching `main`.

`ham-proto#12` was merged once without review; `ham-proto#13` (open) rolls it
back and `ham-proto#14` (open) re-proposes the contract. `v1.0.11` is the tag
the auto-tag workflow cut from that merge, and it still exists.

Before a pre-release tag existed, every consumer generated code from `v1.0.10`,
which predates the contract. That is why CI failed on its first run with
`undefined: coursedetailpb.CourseDetailCourseCommentImage` while the same code
built locally — locally the generated code came from the branch.

## 8. Adversarial review log

Two rounds, run before the pull requests were finalised.

**Round 1 — contract review** (before any consumer was written) found seven
gaps, all fixed as contract comments so no consumer had to change: image order
is display order; `width`/`height` are authoritative from the upload; `edited`
is defined as `update_time` being present; the detail response always fills
`course_info` and `self_comment`; upload carries `filename` on the first chunk
only and signals moderation failure as an RPC error.

**Round 1 — implementation review** (four finders: backend, Android, iOS,
cross-repo consistency) produced 26 findings.

**Round 2** tried to refute each one. Nine were refuted, including three the
finders had ranked as real: a push failure turning a persisted like into a
client error (already swallowed by then), an unsound OSS URL prefix check
(already rewritten to parse both URLs), and uneven image grid rows (short rows
are padded with spacers). One claim was disproved outright: iOS's push type
does match the backend's `course_commentLiked`.

Survivors, all fixed:

| Repo | Finding |
| --- | --- |
| backend | No decoded-size budget before `image.Decode` — a small PNG declaring huge dimensions OOMs the server |
| backend | A moderation-rejected image was left in OSS as a public-read object |
| backend | Two mappers dropped images / `update_time` / `edited`, so a user's own image comment rendered as bare text |
| backend | `FileExceedMaxSize` mapped to gRPC `Internal`, colliding with `SystemException` |
| android | Push type was `course_comment_liked`; the server sends `course_commentLiked`, so the tap would silently do nothing |
| android | Comment detail screen never refetched after an edit (iOS did) |
| android | Client size cap was 8 MB against the server's 5 MB and iOS's 4 MB |
| android | Only `GrpcException` was caught around the streaming upload, leaving the editor stuck uploading |
| android | Documentation claimed a comment row opens the detail screen; nothing did |
| ios | A `#Preview` outside its `#if DEBUG` region breaks the Release build |
| both | One e2e case and two Android unit assertions did not pin what they claimed |

### Round 3 — a second pass over the fixed code

Round 2 confirmed the survivors above and fixed them. A third pass ran over the
result, because the fix commits themselves were unreviewed and neither client
can be compiled locally. It found two compile breakers and one DoS guard that
did not hold:

| Repo | Finding |
| --- | --- |
| android | `import androidx.compose.foundation.layout.matchParentSize` — `matchParentSize` is a member extension of `BoxScope`, not a top-level declaration, so the import cannot resolve and `:feature:coursescore` fails to compile. Deleted; the other 10 call sites resolve it off the receiver |
| ios | `Google_Protobuf_Timestamp` used in a file-scope `#Preview` without `import SwiftProtobuf` — nothing re-exports it, so Debug and Release both fail. Import added |
| backend | The decode budget counted pixels, not bytes. A 16-bit RGBA PNG allocates 8 bytes/pixel, so a 0.37 MB upload passed both guards and allocated ~382 MB. Now bounded in bytes |
| backend | The like push and a DB read ran inside the redis lock, stalling the same user's opposite action |
| backend | Three newly added comments were Chinese, which the repo's language policy forbids |
| backend | `isOwnOSSImageURL` accepted `..` path segments, so the prefix scope was escapable |
| android | An oversized image was reported to the user as "rejected by moderation" |
| android | The course-centre comment card dropped images and the edited marker that iOS renders |
| android | `UpdateCourseCommentResponse.success` was discarded |
| ios | Full-resolution JPEG transcode ran on the main actor |
| ios | A second pick during an in-flight upload could exceed nine images |
| ios | Only `GRPCStatus` was caught around upload, so a non-GRPC failure dropped the rest silently |
| ios | Edit mode gated on a guessed 20-character minimum when the config was never loaded |
| ios | Three additional hard compile errors at HEAD: non-exhaustive `catch` in `async` funcs (`submitRate`, `submitComment`, `updateComment`), verified with `xcrun swiftc` |

Round 3 also refuted three claims: the `course` deep link really does reach the
Course tab (`MainBodyViewModel` switches the tab); the TPNS tap callback the
branch implements is the current non-deprecated selector; `pendingURL` was not
deleted by this branch — it landed on `main` after the branch point.

### Verification status

All three consumers pin `v1.0.12-beta.1`, cut from the contract branch. The
backend is verified for real against that tag — `make install-all`,
`go build ./...`, `go test ./...` (75 packages) and `golangci-lint run ./...`
are all clean.

Neither client has been compiled. `E2E_AUTHORING.md` (Android — one shared AVD
and build directory) and `E2E_CONVENTIONS.md` (iOS — the orchestrator builds
centrally, and concurrent xcodebuild runs collide) both forbid running the
build. Every symbol in the two client changes was therefore verified by reading
its definition, not by compiling, and each client's own codegen convention was
confirmed separately (Wire emits snake_case, SwiftProtobuf upper-cases
acronyms). The first real build should be treated as part of the review.
