# Ham Workspace

This repository coordinates the HAM product repositories. The repositories in `repos/` are Git submodules and remain independently versioned, tested, reviewed, and released.

## Repository map

| Submodule | Responsibility |
| --- | --- |
| `repos/ham-backend-go` | Backend services and HTTP/gRPC implementation |
| `repos/ham-proto` | Shared protobuf contracts and generated API definitions |
| `repos/ham-android` | Native Android client |
| `repos/ham-ios` | Native iOS client |
| `repos/ham-web` | Web client |
| `repos/ham-rn` | React Native client |

## Cross-repository work

- Start from the affected API/protobuf contract, then trace every consuming client.
- For API, authentication, error-code, or model changes, inspect `ham-proto`, `ham-backend-go`, and each affected client before editing.
- Keep changes inside the owning submodule. Do not make unrelated changes in other clients.
- Obey a submodule's own `AGENTS.md`; it overrides this file for work inside that submodule.
- Report changed submodule commits and validation separately. Never assume one client's build validates another client.

## Code and commit conventions

- Write code comments in English.
- Use English Conventional Commit messages, such as `feat(auth): add OAuth callback handling` or `fix(api): return validation error`.

## Workspace commands

```sh
git submodule update --init --recursive
git submodule foreach 'git status --short'
git submodule foreach 'git fetch --prune'
```

## Shared context

Keep cross-repository architecture, API contracts, and release checklists in `docs/`. Create a focused task document in `tasks/` before implementing a change that spans two or more submodules.
