# Promotion card tabs

**Status:** implemented
**Repos:** `ham-ios`, `ham-android`

## Goal

The promotion card on the **My** tab listed entries vertically. With tabs it shows
a horizontally scrollable tab bar above the entries, so a single card can carry
more entries without growing taller.

The layout is driven entirely by the `promotion` cloud-config value; there is no
client-side switch. Payloads without tab data keep the old vertical layout.

## Contract

See [docs/promotion-cckv-contract.md](../docs/promotion-cckv-contract.md) for the
payload shape, layout rules, locale fallback, and tracking fields.

## Scope

| Repo | Change |
| --- | --- |
| `ham-ios` | `MyViewPromotionCard.swift` — payload model, tab bar, selected-tab entries. |
| `ham-android` | `MyPromotionParser.kt` (new), `MyPromotionTabRow.kt` (new), `MyPromotionViewModel.kt`, `MyPromotionView.kt`. |

Not affected: `ham-proto`, `ham-backend-go`, `ham-web`, `ham-rn`. The payload is
cloud-config JSON, not an API response, so no protobuf contract is involved.

## Validation

- Android: `./gradlew :feature:my:testDebugUnitTest` — 12 cases in
  `MyPromotionParserTest` covering both layouts, the single-tab fallback, tab
  filtering, locale fallback, and malformed entries.
- Android: `./gradlew :feature:my:compileDebugKotlin`.
- iOS: `xcodebuild -workspace Ham.xcworkspace -scheme "Ham (iOS)" -configuration
  Release -destination 'generic/platform=iOS' CODE_SIGNING_ALLOWED=NO build`
  (the command used by `.github/workflows/verify.yml`).

iOS has no unit test for the parser. Two things block it: the repo has no
synchronized folder group, so a new test file needs an entry in
`Ham.xcodeproj/project.pbxproj`; and the simulator build fails on
`mars.framework` being built for device (pre-existing, reproducible on an
unmodified tree), which is where XCTest would have to run.

## Open questions

- Whether a one-tab payload should show the bar anyway. Current behaviour: no,
  it renders as a vertical list.
