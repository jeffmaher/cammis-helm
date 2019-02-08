# cammis-helm

This repo acts as web server that serves up Helm charts for CA-MMIS applications. The actual Helm Repo is stored within the `gh-pages` branch and serves traffic from `https://ca-mmis.github.io/cammis-helm`.

## How to Add/Update a Chart

For this example, we'll use the [hello-cammis-data Helm chart](https://github.com/ca-mmis/hello-cammis-data-helm).

1. Clone [hello-cammis-data-helm](https://github.com/ca-mmis/hello-cammis-data-helm)
1. Clone this repo and switch its branch to `gh-pages`
1. Package hello-cammis-data's Helm chart: `helm package hello-cammis-data-helm/hello-cammis-data`
1. Move the produced `tgz` file into `cammis-helm/`
1. Re-index the cammis-helm repo: `helm repo index cammis-helm --url https://ca-mmis.github.io/cammis-helm`
1. Commit all the changes in cammis-helm and push the changes
