# GitHub Workflows

A collection of reusable GitHub workflows.

## Dependabot PR post-processing

This workflow performs some extra operations on Dependabot PRs:

* it adds the `internal` label to PRs for the selected dependencies (or a custom label name)
* it enables auto-merge for these same PRs
* it updates the `kotlin-js-store/yarn.lock` and `kotlin-js-store/wasm/yarn.lock` files in PRs that update 
  Kotlin-related dependencies

Typical setup:

```yaml
name: dependabot-pr-post-processing

on:
  pull_request:
    types: [opened, synchronize, reopened]

# We don't need any permissions here, because they are handled in the PAT.
# We need the PAT because Dependabot workflows don't have enough permissions to process the PRs properly.
permissions:
  contents: none

jobs:
  dependabot-pr-post-processing:
    uses: joffrey-bion/gh-workflows/.github/workflows/dependabot-pr-post-processing.yml@main
    with:
      internal-dependencies: |
        gradle-wrapper
        com.gradle.develocity
        com.vanniktech.maven.publish
        org.hildan.github.changelog
        org.jetbrains.dokka
        org.jetbrains.kotlinx.binary-compatibility-validator
        ru.vyarus.github-info
    secrets:
      # Needs:
      #   contents:write to commit yarn.lock files
      #   pull-requests:write to label PRs
      #   issues:write to create the label if missing
      github-token: ${{ secrets.GH_PAT_FOR_DEPENDABOT }}
```

With a `dependabot.yml` setup like this:

```yaml
version: 2
updates:
  # Maintain dependencies for GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "daily"
    labels:
      - internal
      - dependencies

  - package-ecosystem: "gradle"
    directory: "/"
    schedule:
      interval: "daily"
    labels:
      - dependencies
```