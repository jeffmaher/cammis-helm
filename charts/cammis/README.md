# cammis

Helm Chart for deploying the entire CA-MMIS Modernization stack.

This can be thought of as a manifest file, referencing all the applications and their corresponding versions to be installed. These are all specified in the `requirements.yaml` file.

## Setup Instructions

### Dependencies

- Kubernetes Cluster
- Tiller installed into your Kubernetes Cluster
- Helm configured to Tiller in your Kubernetes Cluster
- Have Docker authenticated with the CA-MMIS Dev ECR repo

### Steps

1. Navigate into the base of the parent directory via a terminal (i.e. `cammis-helm/charts`)
1. Install dependencies: `helm dep up cammis`
1. Deploy (Kubernetes Deployment and Service): `helm install cammis -n cammis`
1. Verify:
    - `helm ls`
    - `kubectl get all`
1. Delete: `helm delete --purge cammis`
