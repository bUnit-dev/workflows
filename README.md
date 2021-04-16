# workflows

## Changes to code or documentation for next release:

1. Build of mainline
2. Create PR, include update to changelog.md
   1. build
   2. test
   3. build docs
   4. build nuget packages

## Create a release:

1. Trigger prepare-release workflow (input: versionIncrement, releaseImmediately):
   1. nbgv prepare-release --nextVersion $version
   2. update changelog.md with new version (nbgv get-version) using keep-a-changelog-new-release
   3. commit
   4. build
   5. test
   6. build docs
   7. build nuget packages
   8. push release branch
   9. create pull request for release branch
      1. If releaseImmediately, merge PR immediately.
   10. on RELEASE pull request merge create github release
      2. get version
      3. read changelog from https://github.com/mindsers/changelog-reader-action
      4. create github release with version and description
   11. on RELEASE pull request merge, push latest mainline to docs_live
   12. on GITHUB release
      5. 


## Updates to live docs

1. On push (merge of PR or direct push)
   1. build docs and push to GH Pages
   2. merge with FF back to main

## Create emergency patch to existing release

1. Create branch of tag for the release to fix
2. 

steps:

&version = nbgv get-version
ensure changelog has matching major.minor.patch set
nbgv tag
git push