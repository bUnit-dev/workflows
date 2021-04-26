# Workflows

The workflows have the goal of automating as much as possible, while keeping workflows flexible to handle odd cases when needed.

**NOTE:** whenever I write mainline below, I am referring to whatever is the default branch in a GitHub repository, e.g. `main`.

**The goals are:**

1. Follow GitHub flow, e.g. PRs into mainline branch.
2. Mainline has latest preview/nightly code in unreleased form, and new preview releases is automatically pushed to GitHub Package Repository when source code changes in mainline.
3. `stable` branch contains latest released code to NuGet.org.
4. Use "[Keep a change log](https://keepachangelog.com/en/1.0.0/), where mainline has new items in "Unreleased" section, that are automatically moved to a "released versioned" section when released.
5. Use "[Nerdbank.GitVersioning](https://github.com/dotnet/Nerdbank.GitVersioning)" to correctly stamp versions, repository references, and commit references in .NET assemblies of both releases and prereleases.
7. Automatically create GitHub releases and add tags on new release.
8. Allow for live documentation to be updated independently of releases from stable.
9. Allow for changes to docs to be updated with a release.
10. Release workflows should be idempotent, be re-runable in case of errors.

The workflow examples are located in the usual place in this repository under `.github/workflows`, and are built with the settings defined in the [`version.json`](https://github.com/bUnit-dev/workflows/blob/main/version.json) in mind.

The following sections will list what should happen in the different scenarios involved.

## Changes to code or documentation for next release stored in mainline:

1. Locally, make changes from mainline branch. remember to update CHANGELOG.md.
2. Push to mainline or create a PR targeting mainline, if not authorized to push directly.

## On push/pull request merged on mainline

1. Automatically trigger `verification` workflow.
2. Automatically trigger `release-preview` workflow if `verification` workflow successed.

## On pull request (created, updated) to mainline

1. Automatically trigger `verification` workflow.

## Prepare a new release:

Manually trigger `prepare-release` workflow, specifying if it is a `minor` or `major` release. This does the following:

1. Check that release contains changes, if not, abort. This is verified by looking at the "Unreleased" section in the CHANGELOG.md.
2. Increment version.json on mainline, e.g. 1.1-preview to 1.2-preview. Push mainline branch to origin.
3. Create release branch and set version.json in that to non preview, e.g. 1.1. Push release branch to origin.
4. Create pull request for release branch towards `stable`.

Creating the PR ensures `verification` workflow runs, and gives the option to make small adjustments before release if needed.

TODO: 

- Make it possible to specify that the release PR should be accepted automatically for immidiate release.
- Use some tooling like https://github.com/fsprojects/SyntacticVersioning/ or changelog analyser to automatically determine if the release is a minor or major release.

## Release:

Automatically trigger `release` workflow on release PR merged with stable, or if it is triggered manaully. This does the following:

1. Check that release contains changes, if not, abort. This is verified by looking at the "Unreleased" section in the CHANGELOG.md.
2. Update changelog: move content from "unreleased" section to an "[x.y.c] yyyy-mm-dd" section
3. Commit CHANGELOG.md to `stable` branch
4. Building library in release mode
5. Upload library to NuGet.org repository
6. Push `stable` branch to origin
7. Create GitHub release from commit with CHANGELOG.md changes on stable. The release is created with the title and content from the CHANGELOG related to the release.
8. Merge `stable` with mainline and push to origin
9. If merge between `stable` and mainline fails, then create a PR from `stable` to mainline instead.

The order of these steps are important as they make sure that GitVersioning correctly stamps the generated assemblies/nuget packages with the right version and github commit hash from when the changelog was updated.

## Updates to live docs on `stable`

If a library has a documentation site, the version of the docs should match the latest released version of the library. This, the source for the docs located on the `stable` branch is where the live documentation website is built from.

Since it is not uncommon that fixes and changes to the docs need to happen between releases, changes to the docs sources are the only changes allowed to be pushed to `stable` outside of normal release pull requests. When this happens, the following should happen:

1. Automatically trigger `docs-deploy` workflow, that builds and uploads docs to whatever hosting solution it uses, e.g. GH Pages.
2. Merge `stable` with mainline branch, such that updated docs are visible in the mainline branch.
3. If merge between `stable` and mainline fails, then create a PR from `stable` to mainline instead.
