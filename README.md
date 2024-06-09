# deploy-pages

![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/actions-mn/deploy-pages)
[![GitHub actions](https://github.com/actions-mn/deploy-pages/actions/workflows/test.yml/badge.svg)](https://github.com/actions-mn/deploy-pages/actions/workflows/test.yml)

Meta action to deploy GitHub Pages with PR deployment ability in one step

If you don't need PR deployment use other actions like [`actions/deploy-pages@v1`](https://github.com/actions/deploy-pages)

- [Description](#description)
- [Prerequisites](#prerequisites)
- [Scenario](#scenario)
- [Complete example](#complete-example)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## Prerequisites

1. Make sure that [GitHub Pages enabled in settings](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site#creating-your-site)
1. Site artifact must be built before. `actions-mn/build-and-publish@v1` can be used for this
1. Define all required `permissions` in workflow like this

   ```yml
   permissions:
     pull-requests: write
     contents: read
     pages: write
   ```
1. Use PAT token that has permissions for pull request comments and git push

## Scenario

Typically, you can utilize it by providing only one parameter `token`

```yml
...
+ uses: actions-mn/deploy-pages@main
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
```

> *Important note*. Make sure you pass `token`, that is allowed to do GitHub Pages deployment

Except `token`, you also can pass:

- `artifact-name` - the default value is the same as [`actions-mn/build-and-publish`](https://github.com/actions-mn/build-and-publish)
- `source-dir` - source directory where 
- `preview-branch` - GitHub Pages branch
- `umbrella-dir` - directory in the GitHub Pages branch where PR deployments are stored

## Complete example

```yml
name: github-pages-deployment

on:
  push:
    branches:
    - main
  pull_request:
    types:
    - opened
    - reopened
    - synchronize
    - closed

permissions:
  pull-requests: write
  contents: read
  pages: write

jobs:
  build:
    if: ${{ github.event.action != 'closed' }}
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Setup Metanorma
      uses: actions-mn/setup@v1

    - name: Cache Metanorma assets
      uses: actions-mn/cache@v1

    - name: Metanorma generate site
      uses: actions-mn/build-and-publish@main
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
        destination: artifact
        agree-to-terms: true

  deploy-gh-pages:
    runs-on: ubuntu-latest
    needs: build
    steps:
    - name: Deploy to GitHub Pages
      uses: actions-mn/deploy-pages@main
      with:
        token: ${{ secrets.PAT_TOKEN }}

  clean-gh-pages:
    if: ${{ github.event.action == 'closed' }}
    runs-on: ubuntu-latest
    steps:
    - name: Deploy to GitHub Pages
      uses: actions-mn/deploy-pages@main
      with:
        token: ${{ secrets.PAT_TOKEN }}
```
