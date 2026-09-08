# Contributing

## Branching

Work off a short-lived feature branch cut from `main`. Open a PR early and mark it as a Draft
if it's still in progress.

## Commit convention

Commit header: `type(scope): short imperative`, <= 72 chars. Body (if present): 1-2 factual
sentences (what/why) — no emojis, issue refs, or co-authors.

Types: `feat`, `fix`, `refa`, `perf`, `docs`, `ci`, `chore`, `test`, `build`.

## Pull requests

Fill out the PR template. Keep each PR scoped to one logical change — one action added or
changed, or one cross-cutting fix. CI must pass before merge.

## Code style

Each action lives in its own top-level folder with an `action.yml` and a `README.md` documenting
its inputs/outputs and usage. Keep actions composable and reusable across the repos that consume
them — avoid baking in assumptions specific to any one consumer repo.

## Documentation

If you add or change an action's inputs, outputs, or behavior, update its `README.md` in the
same PR. New actions also need an entry in the root `README.md`'s action table.
