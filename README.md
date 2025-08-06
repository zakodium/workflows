# workflows

Shared reusable GitHub workflows.

## Available workflows

* [`nodejs` (Node.js CI)](#nodejs-ci)
* [`release`](#release)
* [`typedoc`](#typedoc)
* [`npm-prerelease`](#npm-prerelease)

### Node.js CI

Generic workflow to run linters and tests for Node.js projects. It supports `eslint`,
`prettier`, and `check-types` linter steps, will run tests using a matrix of
Node.js versions, and verify that the package can be published to a registry
and imported.

#### Inputs

* **node-version**:
  * Version of Node.js used to run the lint steps.
  * Default: `22.x`
* **npm-setup-command**:
  * Command used to set up the package before running other steps.
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
* **disable-test-package**:
  * Disable testing that the package can safely be published to a registry
    and imported.
  * Default: `false`
* **node-version-matrix**:
  * Versions of Node.js to test on, as a JSON array.
  * Default: `'[20, 22, 24]'`
* **test-setup-command**:
  * Command used to set up the package before running npm tests.
    Will run between `npm-setup-command` and `npm-test-command`.
  * Default: `undefined`
* **npm-test-command**:
  * Command used to run the tests.
  * Default: `npm run test-only`
* **upload-coverage**:
  * Whether to run the Codecov action to upload coverage data.
    This requires to pass the `codecov-token` secret for private repos.
  * Default: `true` for public repos and `false` for private repos.
* **cwd**:
  * Usefully for monorepos. Set the working directory to run the steps in.
  * Default: `.`.

#### Secrets

* **codecov-token**
  A token for code coverage uploads.
  Necessary for private repos and public repos if the organization hasn't enabled [tokenless uploads](https://docs.codecov.com/docs/codecov-tokens#uploading-without-a-token).
* **env**
  Environment variables necessary to run the tests.
  Must be passed with the `KEY=value` format (one per line).
  Values will be automatically treated as secrets and redacted from the logs.

#### Example usage for a TypeScript project
  
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

The release workflow helps to automatically release packages that conform to the
[conventional commits specification](https://www.conventionalcommits.org/en/v1.0.0/).

It uses the [Release Please Action](https://github.com/google-github-actions/release-please-action#release-please-action)
to maintain a release pull request. When the pull request is merged, the changelog
is updated and a release is created in GitHub. Additionally, it is possible to
publish the package to the npm and GitHub package registries.

#### Inputs

* **github-app-id**
  * The id of a GitHub app used to publish the release.
    The `github-app-key` secret is mandatory instead of the `github-token` secret when this is set.
* **npm**:
  * Pass `true` to enable the npm publish steps. In that case, the `npm-token`
    secret is mandatory.
  * Default: `false`
* **node-version**:
  * Version of Node.js used to run the npm publish steps.
  * Default: `22.x`
* **npm-setup-command**:
  * Command used to set up the package before publishing to npm.
  * Default: `npm ci` if there is a `package-lock.json`, `npm install` otherwise.
* **public**:
  * This option only affects scoped packages.
    Whether the package will be published to the public npm registry. If `false`,
    the package will be published only to the GitHub Package Registry (GPR).
    When publishing to GPR, the `github-token` must also have the `write:packages` scope.
  * Default: `false`
* **release-type**:
  * Option passed to the [release-please action](https://github.com/googleapis/release-please-action?tab=readme-ov-file#release-types-supported).
    Set to the empty string to use a [release manifest config](https://github.com/googleapis/release-please/blob/main/docs/manifest-releaser.md).
  * Default: `node`

#### Secrets

* **github-token**
  A GitHub PAT passed to release-please and npm (if the package is published to GPR).
  Mandatory when `github-app-id` **is not** passed.
* **github-app-key**
  A GitHub app private key belonging to the app referenced by `github-app-id`.
  Mandatory when `github-app-id` **is** passed.
* **npm-token**
  A npm automation token with the permission to publish this package.

#### Example usage

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

### TypeDoc

The typedoc workflow can be used to generate a documentation website with [TypeDoc](https://typedoc.org/)
and publish it to GitHub pages.

#### Inputs

* **entry**:
  * Entry point of the API.
    Multiple entry points can be specified using spaces as separators.
* **name**:
  * Name of the package. It will be used as a documentation title.
    Defaults to `package.json`'s name field.
* **node-version**:
  * Version of Node.js used to run the build steps.
  * Default: `22.x`
* **npm-setup-command**:
  * Command used to set up the package before publishing to npm.
  * Default: `npm ci` if there is a `package-lock.json`, `npm install` otherwise.

#### Secrets

* **github-token**
  Token used to deploy to GitHub pages.

#### Example usage

````yml
name: TypeDoc

on:
  workflow_dispatch:
  release:
    types: [published]

jobs:
  typedoc:
    # Documentation: https://github.com/zakodium/workflows#typedoc
    uses: zakodium/workflows/.github/workflows/typedoc.yml@typedoc-v1
    with:
      entry: 'src/index.ts'
    secrets:
      github-token: ${{ secrets.BOT_TOKEN }}
````

### npm prerelease

This workflow allows to create a npm pre-release.
The state of the repository will be published as-is, using `$currentVersion-pre-$epoch` as a version number
and `pre` as the npm dist-tag.
Scoped packages are not supported for security reasons.

#### Inputs

* **node-version**:
  * Version of Node.js used to run the npm publish steps.
  * Default: `22.x`
* **npm-setup-command**:
  * Command used to set up the package before publishing to npm.
  * Default: `npm ci` if there is a `package-lock.json`, `npm install` otherwise.

#### Example usage

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
