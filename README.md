# charts

This readme describes how to use Chart Releaser Action to automate releasing charts through GitHub pages. Chart Releaser Action is a GitHub Action workflow to turn a GitHub project into a self-hosted Helm chart repo, using `helm/chart-releaser` CLI tool.

`cr` (chart-releaser) is a tool designed to help GitHub repos `self-host` their own chart repos by adding Helm chart artifacts to GitHub Releases named for the chart version and then creating an `index.yaml` file for those releases that can be hosted on GitHub Pages (or on S3 bucket, on any static site!).

Currently, Github Action Workflow can create GitHub Releases from a set of charts packaged up into a directory and create an `index.yaml` file for the chart repository from GitHub Releases.

## Handling chart dependies

Unfortuntely the `cr` won't automatically add repositories for dependencies, and this needs to be added to the pipeline, prior to running the releaser.

## Using with private repos

When using `cr` on a private repository, `helm` is unable to download the chart package files. When you give `helm` your `username` and `password` it uses it to authenticate to the repository (the index file). The index file then tells Helm where to get the tarball. If the tarball is hosted in some other location (Github Releases in this case) then it would require a second authentication (which Helm does not support). The solution is to host the files in the same place as your index file and make the links relative paths so there is no need for the second authentication.

### Prerequisites

Have a Github token with the right permissions (SSO enabled for entreprise) and Github Pages configured.

## Appendix

### Configure repository

Create a Git repository under your GitHub organization. You could give the name of the repository as helm-charts (*-charts), though other names are also acceptable. The sources of all the charts can be placed under the `main` branch. The charts should be placed under /charts directory at the top-level of the directory tree.

There should be another branch named `gh-pages` to publish the charts. The changes to that branch will be automatically created by the Chart Releaser Action described here. However, you can create that `gh-branch` and add `README.md` file, which is going to be visible to the users visiting the page.

You can add instruction in the README.md for charts installation like this (replace <alias>, <orgname>, and <chart-name>):

### Example README.md

```bash
## Usage

[Helm](https://helm.sh) must be installed to use the charts.  Please refer to
Helm's [documentation](https://helm.sh/docs) to get started.

Once Helm has been set up correctly, add the repo as follows:

  helm repo add <alias> https://<orgname>.github.io/helm-charts

If you had already added this repo earlier, run `helm repo update` to retrieve
the latest versions of the packages.  You can then run `helm search repo
<alias>` to see the charts.

To install the <chart-name> chart:

    helm install my-<chart-name> <alias>/<chart-name>

To uninstall the chart:

    helm uninstall my-<chart-name>
```

The charts will be published to a website with URL like this:

```bash
https://<orgname>.github.io/helm-charts
```

### Configure Github Pages

### Create Github Token with right permission

### Github Action Workflow

Create GitHub Actions workflow file in the main branch at .github/workflows/release.yml

```yaml
name: Release Charts

on:
  push:
    branches:
      - main

jobs:
  release:
    permissions:
      contents: write
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "$GITHUB_ACTOR"
          git config user.email "$GITHUB_ACTOR@users.noreply.github.com"

      - name: Run chart-releaser
        uses: helm/chart-releaser-action@v1.6.0
        env:
          CR_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
```

The above configuration uses @helm/chart-releaser-action to turn your GitHub project into a self-hosted Helm chart repo. Checking each chart in your project, and whenever there's a new chart version, creates a corresponding GitHub release named for the chart version, adds Helm chart artifacts to the release, and creates or updates an `index.yaml` file with metadata about those releases, which is then hosted on GitHub pages.

Note: The Chart Releaser Action is almost always used in tandem with the Helm `Testing Action` and `Kind Action`.