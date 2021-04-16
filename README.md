# workflows

Goal:

1. Follow github flow, e.g. PRs into mainline branch.
2. Release of mainline
3. Keep a change log, whose version numbers are automatically updated when a release is pushed
4. Automatically create github releases, add tags to repo
5. Automatically push nightly builds on push to mainline to GPR
6. Allow for live documentation to be updated independently of releases
7. Allow for docs to be updated with a release.
8. Release workflows should be idempotent, be re-runable in case of errors

## common verification steps in PR / build on mainline
1. build
2. test
3. verify template
4. build docs
5. build nuget packages   

## Changes to code or documentation for next release:

1. Build of mainline
2. Create PR, include update to changelog.md 

## On merge to mail

1. Run code analyzers
2. build nuget packages
3. push nightly nuget packages to github package repository

## Create a release:

1. Trigger prepare-release workflow (input: versionIncrement, releaseImmediately):
2. nbgv prepare-release --versionIncrement versionIncrement
3. $version = nbgv get-version
4. git push main
5. git checkout release/($version)   
6. push release branch

## Release

1. build
2. test
3. verify template
4. build docs
5. build nuget packages   
6. update changelog.md with new version ($version) using keep-a-changelog-new-release
7.  Create version tag 
8.  get version
9.  read changelog from https://github.com/mindsers/changelog-reader-action
10. create github release with tag, title, and description
11. Build nuget packages
12. push nuget packages
13. merge release branch with origin:mainline
14. merge mainline to docs_live
15. merge release branch with origin:docs_live
16. delete release branch
   
## Updates to live docs

1. On push (merge of PR or direct push) to docs_live branch
   1. build docs and push to GH Pages
   2. merge with FF back to main
   3. if merge fails, create PR from docs_live to mainline