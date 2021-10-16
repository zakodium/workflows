# workflows

Shared reusable GitHub workflows.

## Available workflows

### Release

The release workflow helps to automatically release packages that conform the
[conventional commits specification](https://www.conventionalcommits.org/en/v1.0.0/).

It uses the [Release Please Action](https://github.com/google-github-actions/release-please-action#release-please-action)
to maintain a release pull request. When the pull request is merged, the changelog
is updated and a release is created in GitHub. Additionally, it is possible to
publish the package on the npm registry.

Inputs:

* **npm**:
  * Pass `true` to enable the npm publish steps. In that case, the `npm-token`
    secret is mandatory.
  * Default: `false`
* **node-version**:
  * Version of Node.js used to run the npm publish steps.
  * Default: `16.x`
* **npm-setup-command**:
  * Command to run to setup the package before publishing to npm.
  * Default: `npm install`

Example usage:

````yml
name: Release

on:
  push:
    branches:
      - main

jobs:
  release:
    uses: zakodium/workflows/.github/workflows/release.yml@release-v1
    with:
      npm: true
    secrets:
      github-token: ${{ secrets.BOT_TOKEN }}
      npm-token: ${{ secrets.NPM_BOT_TOKEN }}
````
