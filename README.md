# Ham Workspace

Shared coordination workspace for the HAM backend and clients. Each product repository is a Git submodule under `repos/`.

## Start here

```sh
git clone --recurse-submodules <workspace-repository-url>
cd ham-workspace
git submodule update --init --recursive
```

Read `AGENTS.md` for repository ownership and cross-repository change rules.

## Layout

- `repos/`: independently versioned product repositories
- `docs/`: architecture, API contracts, and release guidance
- `tasks/`: scoped plans and acceptance criteria for cross-repository work
