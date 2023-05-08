# workflows

Shared reusable GitHub workflows.

## Available workflows

* [`nodejs` (Node.js CI)](#nodejs-ci)
* [`release`](#release)
* [`npm-prerelease`](#npm-prerelease)

### Node.js CI

Generic workflow to run linters and tests for Node.js projects. It supports `eslint`,
`prettier`, and `check-types` linter steps and will run tests using a matrix of
Node.js versions.

Inputs:

* **node-version**:
  * Version of Node.js used to run the lint steps.
  * Default: `16.x`
* **npm-setup-command**:
  * Command used to setup the package before running other steps.
  * Default: `npm ci` if there is a `package-lock.json`, `npm install` otherwise.
* **lint-eslint**:
  * Whether to run the `eslint` npm script.
  * Default: `true`
* **lint-prettier**:
  * Whether to run the `prettier` npm script.
  * Default: `true`
* **lint-check-types**:
  * Whether to run the `check-types` npm script. This should be set to `true`
    for TypeScript projects.
  * Default: `false`
* **disable-tests**:
  * Disable the test matrix.
  * Default: `false`
* **node-version-matrix**:
  * Versions of Node.js to test on, as a JSON array.
  * Default: `'[16, 18, 20]'`
* **test-setup-command**:
  * Command used to setup the package before running npm tests. Will run
    between `npm-setup-command` and `npm-test-command`.
  * Default: `undefined`
* **npm-test-command**:
  * Command used to run the tests.
  * Default: `npm run test-only`
* **upload-coverage**:
  * Whether to run the Codecov action to upload coverage data.
  * Default: `true`

Example usage for a TypeScript project:
  
```yml
name: Node.js CI

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  nodejs:
    # Documentation: https://github.com/zakodium/workflows#nodejs-ci
    uses: zakodium/workflows/.github/workflows/nodejs.yml@nodejs-v1
    with:
      lint-check-types: true
```

### Release

The release workflow helps to automatically release packages that conform the
[conventional commits specification](https://www.conventionalcommits.org/en/v1.0.0/).

It uses the [Release Please Action](https://github.com/google-github-actions/release-please-action#release-please-action)
to maintain a release pull request. When the pull request is merged, the changelog
is updated and a release is created in GitHub. Additionally, it is possible to
publish the package to the npm and GitHub package registries.

Inputs:

* **npm**:
  * Pass `true` to enable the npm publish steps. In that case, the `npm-token`
    secret is mandatory.
  * Default: `false`
* **node-version**:
  * Version of Node.js used to run the npm publish steps.
  * Default: `16.x`
* **npm-setup-command**:
  * Command used to setup the package before publishing to npm.
  * Default: `npm ci` if there is a `package-lock.json`, `npm install` otherwise.
* **public**:
  * This option only affects scoped packages.
    Whether the package will be published to the public npm registry. If `false`,
    the package will be published only to the GitHub Package Registry (GPR).
    When publishing to GPR, the `github-token` must also have the `write:packages` scope.
  * Default: `false`

Example usage:

````yml
name: Release

on:
  push:
    branches:
      - main

jobs:
  release:
    # Documentation: https://github.com/zakodium/workflows#release
    uses: zakodium/workflows/.github/workflows/release.yml@release-v1
    with:
      npm: true
    secrets:
      github-token: ${{ secrets.BOT_TOKEN }}
      npm-token: ${{ secrets.NPM_BOT_TOKEN }}
````

### npm prerelease

This workflow allows to create an npm prerelease. The state of the repository
will be published as-is, using `$currentVersion-pre-$epoch` as a version number
and `pre` as the npm dist-tag. Scoped packages are not supported for security
reasons.

Inputs:

* **node-version**:
  * Version of Node.js used to run the npm publish steps.
  * Default: `16.x`
* **npm-setup-command**:
  * Command used to setup the package before publishing to npm.
  * Default: `npm ci` if there is a `package-lock.json`, `npm install` otherwise.

Example usage:

````yml
name: Prerelease package on npm

on:
  pull_request:
    types: [labeled]
  workflow_dispatch:

jobs:
  prerelease:
    # Documentation: https://github.com/zakodium/workflows#npm-prerelease
    uses: zakodium/workflows/.github/workflows/npm-prerelease.yml@npm-prerelease-v1
    secrets:
      github-token: ${{ secrets.BOT_TOKEN }}
      npm-token: ${{ secrets.NPM_BOT_TOKEN }}
````
