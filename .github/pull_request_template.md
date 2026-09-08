## Summary

Describe the change in 2-6 lines.

## Why

Explain the reason for this change.

## Change Type

- [ ] New action
- [ ] Change to an existing action
- [ ] Bug fix
- [ ] Docs only
- [ ] CI/build/tooling

## Affected Areas

- [ ] Core Actions (generate-version, git-tag, generate-badge, github-release)
- [ ] .NET Actions (dotnet-tool-install, dotnet-pack, dotnet-test, dotnet-docfx-*, dotnet-nuget-*, dotnet-cyclonedx)
- [ ] Supply Chain / Security (gh-sbom, sbom-publish, sbom-sign, install-apple-certificate)
- [ ] Utility Actions (normalize-arguments, normalize-path, debug-tree)
- [ ] Documentation

## Behavior And Compatibility

- [ ] Action inputs/outputs changed
- [ ] Breaking change for existing consumers
- [ ] No externally visible behavior change

If any box above is checked, describe impact:

## Tests

- [ ] Verified the action runs successfully in a consuming repo's workflow
- [ ] Not applicable (explain)

## Documentation

- [ ] Action's own README.md updated
- [ ] Root README.md action table updated (new actions only)
- [ ] Not applicable (explain)

## Checklist

- [ ] Commit header follows `type(scope): short imperative` and is <= 72 chars
- [ ] Commit type is one of: feat, fix, refa, perf, docs, ci, chore, test, build
- [ ] Commit body is 1-2 factual sentences (what/why), no emojis, refs, or co-authors
- [ ] Change is scoped to one logical unit of work
