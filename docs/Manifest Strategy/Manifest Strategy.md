# Manifest Strategy

The first few sections of this document describe the basics of manifest files, but if you're already seasoned, feel free to jump to [Our Strategy](#our-strategy).

## What is a Manifest File?

Think of baking cookies. A batch of cookies consists of many ingredients, each independently edible. However, if you were to put in the wrong ingredients, the overall cookie might not be yummy. That's why cookie recipes exist, to tell you what to put into the batch to make highly delicious cookies.

Manifest files are cookie recipes for large systems. 

Modern systems are stood up by deploying small, loosely coupled applications together. These small applications are the ingredients. 

Here's an example manifest, with each line being a small application and its associated version (i.e. the ingredients). The file itself represents the entire system (i.e. the batch of cookies). In this application, all the small apps/ingredients represent a fully baked 3-tier web application. 

**Big System Manifest:**
```yaml
website: 4.0.8
backend_api: 1.0.1
data_layer: 2.1.3
```

## Manifest Versions

The next concept to absorb is that the manifest itself is versioned and there are different iterations of it, just like there are different ways to make a chocolate chip cookie.

For example, let's say we take the prior example and make that version `2.0.0` of Big System, but then there is a version `2.1.0` when the team added a few new features.

**Big System Manifest, `2.0.0`:**
```yaml
website: 4.0.8
backend_api: 1.0.1
data_layer: 2.1.3
```

**Big System Manifest, `2.1.0`:**
```yaml
website: 4.1.0
backend_api: 1.1.0
data_layer: 2.1.3
```

Note the change in the `website` and `backend_api` modules that included the new feature code changes.

## Environments

Drawing from our version `2.0.0` and `2.1.0` manifest files, let's say that the team is still working on their changes that compose version `2.1.0` and are testing them out.

- **Test Environment**: Big System Manifest `2.1.0`
- **Production Environment**: Big System Manifest `2.0.0`

The testers find a bug and the developers goes to work on a fix. The developers come up with a fix to something wrong in the `backend_api` and create patch release for that module at `1.1.1`. Then, they create a new version of the Big System Manifest to include that:

**Big System Manifest, `2.1.1`**
```yaml
website: 4.1.0
backend_api: 1.1.1
data_layer: 2.1.3
```

Note the updated `backend_api` compared to the `2.1.0` manifest.

Now the developers are gonna make pop into into the Test Environment for last verification before it gets released.

- **Test Environment**: Big System Manifest `2.1.1`
- **Production Environment**: Big System Manifest `2.0.0`

The testers verify the fix and it's ready, so production is updated to use the latest manifest:

- **Test Environment**: Big System Manifest `2.1.1`
- **Production Environment**: Big System Manifest `2.1.1`

## Benefits

Using manifest files gives release managers and product teams the following benefits:

- Visibility over what's deployed (and when, assuming manifest and configs are version controlled)
- Easy to see how big changes are to both individual modules and the overall system based on [SemVer version numbering](https://semver.org)
- Human readable
- Computer readable, making deployment automation simpler
- Upgrades and rollbacks become easier, since it's easier to see what changed

## Our Strategy

### Manifest Format

The CA-MMIS Platform will use a [Helm](https://helm.sh) Chart `requirements.yaml` as the format for its manifest files. This file is not only human and computer readable, but by keeping using this format, it also makes it negligible to go from describing the composition of the system to automating the deployments using Helm.

An early example, will give you an idea of what this looks like:
- [`requirements.yaml`](https://github.com/ca-mmis/cammis-helm/blob/45e91f64c1270570203f3f2b1d6e3cbca0f8e09d/apps/cammis/requirements.yaml)
- [Full `cammis` Helm Chart](https://github.com/ca-mmis/cammis-helm/tree/45e91f64c1270570203f3f2b1d6e3cbca0f8e09d/apps/cammis)

The version numbers contained in the manifest are the version of each app's deployment package (also Helm Charts), and we intend to keep parity between Helm Chart versions and Docker image versions.

### Deploying a Manifest to a Long Lived Environment

For each environment, there will be a 1:1 relationship with a `CI-deploy-X` GitHub repo. For example, Dev would be `CI-deploy-dev` and Stage would be `CI-deploy-stg`.

Within each `CI-deploy-X` there will be a `deployment.yml` file. One of the keys within it will be the CA-MMIS manifest version. For example, the proposed sample below indicates deploying version `1.0.0` to the Stage environment:

**CI-deploy-stage/deployment.yaml**
```yaml
infrastructure-private: 5.1.0
cammis: 1.0.0
```

When the `master` branch is updated, CircleCI will do a Helm installation or upgrade of the indicated CA-MMIS manifest, depending on whether an environment is new or existing.

In the case of an initial install, this command might be an example:

`helm install cammis:1.0.0 -n cammis`

And an upgrade (i.e. a version other than 1.0.0 is deployed in the cluster already and this bumps it to a different version) might looks like this:

`helm upgrade cammis cammis:1.0.0`

### Deploying a Review Environment

This would be largely the same as the [prior section](#deploying-a-manifest-to-a-long-lived-environment), but would be in a `CI-review-X` repository.

An important thing to note here is that there is a big of a "chicken and the egg" problem here between a new Helm version and deploying a new copy of the code. Since the manifest references existing Helm Charts, a new version of a chart hasn't been cut yet. However, in most cases, a deployment configuration doesn't need to be made to test a new copy of the code and the existing chart can be overridden to deploy a different Docker image than what is encoded in the manifest.

For example, let's say our file looks like this:

**CI-review-dev/deployment.yaml**
```yaml
infrastructure-private: 5.1.0
cammis: 1.0.0
```

and the manifest itself looks like this:

**cammis, `1.0.0`**
```yaml
dependencies:
  - name: hello-cammis-data
    version: 1.0.0
    repository: https://ca-mmis.github.io/cammis-helm
  - name: hello-cammis
    version: 3.0.0
    repository: https://ca-mmis.github.io/cammis-helm
```

Working with a new feature branch for the hello-cammis project (let's call the branch `api_change`), a review environment would be spun up with the cammis `1.0.0` which references the `3.0.0` Helm Chart for hello-cammis. However, the job actually injects a new image, that overrides the existing deployment instructions.

`helm install cammis:1.0.0 --set global.helloCammis.imageVersion=review-api_change`

This should work most of the time. However, if a configuration change also needs to happen, a new CA-MMIS manifest version may need to be cut to enable the deployment of the review environment. This shouldn't happen too often, but when it does, the development team should work with the PATS team to coordinate how this will happen.

An example might be that an interim CA-MMIS manifest is cut to test the change. If we use the above, we might make a manifest version that looks like this: `1.0.0-api_change` and the environment config looks like:

**CI-review-dev/deployment.yaml**
```yaml
infrastructure-private: 5.1.0
cammis: 1.0.0-api_change
```

and the manifest looks like:
**cammis, `1.0.0-api_change`**
```yaml
dependencies:
  - name: hello-cammis-data
    version: 1.0.0
    repository: https://ca-mmis.github.io/cammis-helm
  - name: hello-cammis
    version: 3.0.0-api_change
    repository: https://ca-mmis.github.io/cammis-helm
```

### Local Developer Setup

When developers need to setup a local environment, they can just use the CA-MMIS manifest to create a local Kubernetes cluster that mirrors what they're building against. For example, if a hello-cammis dev is working against all the CA-MMIS manifest `1.0.0` dependencies, they can do a local install and divert the hello-cammis port to something they won't use (i.e. `13123`):

`helm install cammis:1.0.0 --set tags.localDev=true --set global.helloCammis.port=13123`

Then, for their local hello-cammis setup, they can configure any files or environment variables to point to things spun up by Helm locally. Each project's 

### Blue/Green Deploys with a Manifest

If or when CA-MMIS gets to the point where it wants to do blue/green deploys (i.e. deploy a new manifest quietly in the background, then switch out the old one after the new one is up and verified), Helm can help with this process.

For example, let's say the blue deployment in an environment is CA-MMIS manifest `1.0.0`:

`helm install cammis:1.0.0 -n hello-cammis-blue --set global.helloCammis.name=hello-cammis --set global.helloCammisData.name=hello-cammis-data`

The team wants to deploy version `1.1.0` as the green deployment, the team would change any relevant namespacing that impacts service discovery by arguments issued with the Helm command. An possible command might be:

`helm install cammis:1.1.0 -n hello-cammis-green --set global.helloCammis.name=hello-cammis-green --set global.helloCammisData.name=hello-cammis-data-green`

Once verified, the old environment might be torn down:

`helm delete --purge hello-cammis-blue`

And the new one switched into the common namespace:

`helm upgrade --set global.helloCammis.name=hello-cammis --set globals.helloCammisData.name=hello-cammis-data hello-cammis-green cammis:1.1.0`

In all of the commands above, note that the deployment names themselves are `hello-cammis-(color)`, but that set name values don't contain the color when it's is going to be the one actively receiving traffic from normal routes. You can see that the green name values start with `*-green`, but then it's removed when the deployments are swapped.

These are just examples of how this might happen, but may be implemented in a different way should a more robust service management tool (such as Istio) be employed.