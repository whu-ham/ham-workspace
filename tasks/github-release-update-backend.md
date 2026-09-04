# Backend-owned GitHub Release update metadata

## Goal

Move GitHub Release discovery and download-link selection to `ham-backend-go`. Android clients continue using the existing version endpoint and consume the final metadata and download URL returned by the backend.

## Scope

- Resolve the applicable GitHub Release on the backend.
- Return stable Releases for non-beta clients and Pre-releases for beta clients.
- Restrict release versions to `x.y.z`; use `vX.Y.Z` for stable tags and `vX.Y.Z-betaN` for beta tags.
- Read the public release repository from backend configuration, defaulting to `whu-ham/whu-ham.github.io`.
- Support the official GitHub source and the configured `gh-proxy.org` sources when constructing the final URL.
- Select the exact `Ham-Github.apk` asset and return its final download URL through the existing version response.
- Preserve existing client-facing fields for version name, version code, changelog, minimum version, update URL, and info URL.

## Acceptance Criteria

- Stable clients receive only the latest valid stable Release.
- Beta clients receive only the latest valid Pre-release.
- Invalid tags, missing assets, duplicate `Ham-Github.apk` assets, unavailable sources, and malformed repository configuration produce explicit backend errors or a safe no-update response.
- The returned update URL points to `Ham-Github.apk` through the selected source.
- Existing Android clients can update without implementing GitHub API parsing or source selection.
- Backend tests cover channel filtering, strict version parsing, repository fallback, source URL construction, asset selection, and failure cases.
- Android workflow publishes exactly one `Ham-Github.apk` asset for public stable and beta Releases.

## Non-goals

- No Android GitHub API client or GitHub source selector.
- No changes to unrelated clients unless they consume the shared version contract and require compatibility adjustments.
- No arbitrary user-entered proxy URLs.

## Related work

- Workspace issue: `whu-ham/ham-workspace#4`
- Backend Sub-issue: `whu-ham/ham-backend-go#83`
