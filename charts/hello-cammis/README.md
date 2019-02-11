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
