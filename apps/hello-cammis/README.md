# hello-cammis

Helm Chart for deploying [hello-cammis](https://github.com/ca-mmis/hello-cammis) to a Kubernetes cluster.

## Setup Instructions

### Dependencies

- Kubernetes Cluster
- Tiller installed into your Kubernetes Cluster
- Helm configured to Tiller in your Kubernetes Cluster
- Have Docker authenticated with the CA-MMIS Dev ECR repo

### Steps

1. Navigate into the base of this directory via a terminal
1. Deploy (Kubernetes Deployment and Service): `helm install hello-cammis -n hello-cammis`
1. Verify:
    - `helm ls`
    - `kubectl get all`
    - Go to: `http://<cluster url>:3000` (only currently tested on a localhost cluster, so `localhost` for now)
1. Delete: `helm delete --purge hello-cammis`

## hello-cammis Local Development

_Note: This documentation can eventually get moved into hello-cammis's Git repo if this PR is accepted_

This chart can be used to enable local development of hello-cammis by install all the dependencies needed. To get setup for local development, do the following:

1. Install Docker for Desktop
1. Enable Kubernetes
1. Install Helm
1. Install Helm dependencies: `helm dep up cammis-helm/hello-cammis`
1. Install all the dependencies: `helm install hello-cammis -n hc --set tags.localDev=true`
1. For your local installation of the hello-cammis code, set the following for environment variables:
    - `HELLO_CAMMIS_DATA_HOST`: `localhost`
    - `HELLO_CAMMIS_DATA_PORT`: `8000`
