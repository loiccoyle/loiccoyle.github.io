---
author:
  - Loïc Coyle
title: please use release-please
date: '2024-07-03'
description: 'Automate release, version bumping and CHANGELOG with release-please'
summary: 'Automate release, version bumping and CHANGELOG with release-please'
categories:
  - Posts
tags:
  - Workflow
ShowToc: true
TocOpen: false
---
# What is `release-please`?

[{{< ico bootstrap github >}} googleapis/release-please](https://github.com/googleapis/release-please) is a nifty piece of software that parses a repo's commits since the last release, looking for [Conventional Commits](https://www.conventionalcommits.org/) to generate the changelog and determine the next release version.

It then tracks the release's changes in a PR. When the PR is merged, the commit is tagged with the version number and it creates a Github release.

# How to set up `release-please`?

> I'll be using the [{{< ico bootstrap github >}} googleapis/release-please-action](https://github.com/googleapis/release-please-action) github action.

## Config

Add a `release-please-config.json` file to your repo, I usually put it in the `.github/` folder, with the following content.

```json
{
  "packages": {
    ".": {
      "release-type": "{{your release type}}"
    }
  },
  "$schema": "https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json"
}
```

> By defaults `release-please` will write changes to the `CHANGELOG.md` file. To change it add a `"changelog-path"` property.

Make sure to swap use the appropriate `release-type` of your repo.

> The full list of `release-types` is shown [here](https://github.com/googleapis/release-please-action?tab=readme-ov-file#release-types-supported)

If there isn't an appropriate `release-type` for your repo, you can always use `simple` and tell `release-please` where to update the version string by using the `extra-files` option paired with the `x-release-please-version` marker.

> For the full list of configuration options, see the [schema](https://github.com/googleapis/release-please/blob/main/schemas/config.json) and the [docs](https://github.com/googleapis/release-please/blob/main/schemas/config.json).

<details>
  <summary><b>Example using <code>extra-files</code> and <code>x-release-please-version</code> marker.</b></summary>

Let's assume we want to update the version string contained in the `src/file_containing_version.lua` file. We can add it to the `extra-files` property.

```json {hl_lines=[4,5]}
{
  "packages": {
    ".": {
      "release-type": "simple",
      "extra-files": ["src/file_containing_version.lua"]
    }
  },
  "$schema": "https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json"
}
```

And add the `x-release-please-version` marker in a comment on the line where the version string is in the `src/file_containing_version.lua` file.

```lua {hl_lines=[2]}
local M = {}
M.version = "1.2.3" -- x-release-please-version
```

With this `release-please` will know where to update the version number when a release is made.

</details>

## Manifest

`release-please` keeps track of the current version using a `.release-please-manifest.json` file. You'll need to add one to your repo. Let's add in the `.github/` folder.

```json
{
  ".": "{{your current version}}"
}
```

> If you're adding `release-please` to a fresh repo just use `"0.0.0"`.

## Workflow

The final step is to add the `release-please-action` to the repo's CI/CD workflows.

The simplest way is just to add this workflow snippet.

```yml
on:
  push:
    branches:
      - main
permissions:
  contents: write
  pull-requests: write

name: release-please
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
        with:
          config-file: .github/release-please-config.json
          manifest-file: .github/.release-please-manifest.json
```

If we want to run additional jobs when a release is made, say to publish the package to a package index, we can modify the workflow as follows:

```yml {hl_lines=[13,14,18,24,25,26]}
on:
  push:
    branches:
      - main
permissions:
  contents: write
  pull-requests: write

name: release-please

jobs:
  release-please:
    outputs:
      release_created: ${{ steps.release.release_created }}
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
        id: release
        with:
          config-file: .github/release-please-config.json
          manifest-file: .github/.release-please-manifest.json

  publish:
    needs:
      - release-please
    if: needs.release-please.outputs.release.release_created
    runs-on: ubuntu-latest
    steps:
      # ...
```

