# cammis-helm

This repo acts as web server that serves up Helm charts for CA-MMIS applications. The actual Helm Repo is stored within the `gh-pages` branch and serves traffic from `https://ca-mmis.github.io/cammis-helm`.

## Using this as a Helm Repo

You can add this to your list of Helm repos like this:

`helm repo add cammis-helm https://ca-mmis.github.io/cammis-helm`

After that, charts can be installed like this:

`helm install cammis-helm/<chart name> -n <install name>`

## Developers

If you're adding and modifying charts, the next few sections have directions on common operations.

### How to Add/Modify a Helm Chart

The first part of this is adding/modifying the source for a chart.

1. Clone this Git repo and move into 
1. Create a branch from `master`
1. Create/modify a Helm chart in the [apps](apps) folder
1. Commit and push your changes
1. Create a pull request to `master` and have someone peer review it
1. Merge to `master`

From here, the Helm chart can be used by others only if they pull down this repo and directly reference the source code, which is not ideal.

### Adding to Helm Chart Repo

_Note: If this approach is appetizing to the team, this part would be automated and not run by humans_

This packages a chart and makes a it referenceable to anyone that has added `https://ca-mmis.github.io/cammis-helm` as an Helm Chart Repo, without pulling down the source code directly. This also makes it really easy for folks to reference different versions.

1. Clone this Git repo, but don't move into it yet (i.e. stay in the directory it is cloned to). If it's already cloned, switch it to the `master` branch and move up to its parent directory.
1. Package the chart: `helm package cammis-helm/apps/<chart>`.
1. Change the Git repo to the `gh-pages`: `git -C cammis-helm checkout gh-pages`
1. Move the produced `tgz` file into the Git repo.
1. Re-index the cammis-helm repo: `helm repo index cammis-helm --url https://ca-mmis.github.io/cammis-helm`
1. Commit and push all the changes in the `gh-pages` branch: `git -C cammis-helm add . && git -C cammis-helm commit -m "Added <chart> <version>"`

