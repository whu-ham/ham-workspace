# Android `test` build type

The Android release pipeline can only be dispatched as `beta` or `release`. Both are real
releases: they bump the committed version metadata, tag `ham-android`, upload the AAB to Google
Play internal testing, announce the build on Telegram/Discord, and start the E2E suite. There is
no way to prove the pipeline still builds without shipping something.

This adds a third `test` build type whose only job is to prove the build works.

Workspace issue: whu-ham/ham-workspace#23. Two submodules get a PR:

- `whu-ham/ham-android` — dispatch input, completion handling, docs
- `whu-ham/whu-ham.github.io` — build, GitHub Release, cleanup, E2E gating, download page

## Dispatch and build

`Release Build` in `ham-android` gains `test` as a `build_type` option. `test` does not consume
a version code: it reuses the committed `android/metadata/versionCode` and `versionName`, where
`beta` and `release` default to `current + 1`.

`Android Release Build` in `whu-ham.github.io` builds `test` with the fastlane `build` lane —
the same artifacts as `beta` (`assembleGithubRelease` + `bundlePlayStoreRelease`) but with no
`upload_to_play_store`, so nothing reaches Google Play. `ORG_GRADLE_PROJECT_versionType` is
`test`, which only labels telemetry: `VERSION_TYPE` is read by analytics and Crashlytics, and no
server or client branches on it.

## GitHub Release

Tag and title are `TEST-<UTC timestamp>-<short commit>`, for example
`TEST-20260923064500-82909132`. The timestamp makes the tag unique; the commit names the source
it was built from. Created as a prerelease, so it never becomes the latest release, and published
rather than drafted, so the cleanup step can find it through the releases API and a maintainer can
download the APK.

The body states that the package is a build verification artifact, not an official release, and
must not be used in production or redistributed, and that it was never uploaded to Google Play.

## What a `test` build does not do

- No version metadata commit and no `v<name>.<code>` tag in `ham-android`.
- No Telegram or Discord announcement.
- No auto E2E run: `android-e2e-auto.yml` starts on every successful `Android Release Build`
  run, so the build type travels in the E2E context artifact and the suite skips `test`.
- No listing on the public download page, which reads the same releases API.

## Cleanup

Every `beta` and `release` run deletes all `TEST-` releases and their git tags. Test builds are
throwaway: they exist only until the next real build, and nothing links to them.

`TEST-` tags are also excluded when the beta release notes resolve the previous public release,
so a test build cannot be reported as the version a beta is based on.
