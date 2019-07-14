This repository manages the continuous deployment of the [Pangeo](http://pangeo.io/) Cloud Federation
JupyterHub Kubernetes clusters using [hubploy](https://github.com/yuvipanda/hubploy).
It contains scripts to automatically redeploy when the image definition or
chart parameters are changed.

Changing the image will typically take ~20 minutes, and changing a Helm config variable ~1 minute.

# Clusters

Name   | Cloud: region      |  Staging URL                     | Production URL
--     |-                   |-                                 |-
dev    | GCP: us-central1-b | https://staging.hub.pangeo.io    | https://hub.pangeo.io
ocean  | GCP: us-central1-b | https://staging.ocean.pangeo.io  | https://ocean.pangeo.io
hydro  | GCP: us-central1-b | https://staging.hydro.pangeo.io  | https://hydro.pangeo.io
nasa   | AWS: us-east-1     | https://staging.nasa.pangeo.io   | https://nasa.pangeo.io
esip   | AWS: us-west-2     | https://staging.esip.pangeo.io   | https://esip.pangeo.io
icesat | AWS: us-west-2     | https://staging.icesat.pangeo.io | https://icesat.pangeo.io

# Build Status

Branch | Build
-- |-
staging | [![CircleCI](https://circleci.com/gh/pangeo-data/pangeo-cloud-federation/tree/staging.svg?style=svg)](https://circleci.com/gh/pangeo-data/pangeo-cloud-federation/tree/staging)
prod | [![CircleCI](https://circleci.com/gh/pangeo-data/pangeo-cloud-federation/tree/prod.svg?style=svg)](https://circleci.com/gh/pangeo-data/pangeo-cloud-federation/tree/prod)

# Setup

## Setup a Kubernetes Cluster

The first step to using this automation is to create a Kubernetes cluster and
install the server-side component Tiller on it (used for deploying applications
with the Helm tool). Scripts to do so using Google Cloud Platform can be found [here](https://github.com/pangeo-data/pangeo/tree/master/gce/setup-guide). For other cloud providers (e.g. AWS, Azure), follow the [Zero-to-JupyterHub](https://zero-to-jupyterhub.readthedocs.io/en/latest/create-k8s-cluster.html) guide.

## Install git-crypt

You will need to install
[`git-crypt`](https://www.agwa.name/projects/git-crypt/). `git-crypt` is used
to encrypt the secrets that are used for deploying your cluster. Please read this [HOW GIT-CRYPT WORKS](https://www.agwa.name/projects/git-crypt/) if new to it.

# Configure this repository

Once you have a cluster created, you can begin customizing the configuration.

* Create a fork of this repository in GitHub and clone your fork. (Note: the default branch is staging.)
* Request a git-crypt symmetric key from the maintainers of this repo to be used for your deployment secrets files.
* Initialize git-crypt using the unlock command.
  * `git-crypt unlock /path/to/your.key`
* Copy the deployments/dev directory to a directory with your deployment name (we'll use `foobar` as our deployment name from here on).
  * `cp -r dev foobar`
* **Add your deployment's secrets directory to .gitattributes.** IMPORTANT: before pushing to GitHub ensure encryption with `git-crypt status | grep secrets`
* Configure the JupyterHub config files. These are found in `deployments/foobar/config`.
* Configure the `hubploy.yaml` config file.
* Configure the deployment secrets found in `deployments/foobar/config`.
  * Information on what needs to be in hubploy.yaml and in the secrets directory can be found [here](docs/readme-secrets.md).
* Configure your deployments image. This is found in `deployments/foobar/image`.
  * Edit the files in the binder directory to change the contents of the user Docker image. The specification for these files comes from [repo2docker](https://repo2docker.readthedocs.io/en/latest/).
  * Add or modify the README.md and Jupyter notebooks. These will be in each user's home directory.

## Push your setup to GitHub and let HubPloy do the rest

* Make a commit to and push to your fork. Issue a Pull Request to this repo's `staging` branch.  

# Common Issues

* `Error: could not find a ready tiller pod`
  * Some times it takes a while to download the new tiller image, wait and try again.
* `Error: UPGRADE FAILED: "example.pangeo.io-staging" has no deployed releases`
  * If your first deploy of an application fails. Run `helm delete dev-staging --purge` anywhere you have run `gcloud container clusters get-credentials`

# Related Projects

- [Pangeo](http://pangeo.io/): main website for the Pangeo project.
- [Pangeo Helm Chart](https://github.com/pangeo-data/helm-chart): A simple helm chart that wraps the jupyterhub helm chart to support horizontal compute scaling with Dask.
- [Zero to JupyterHub](https://zero-to-jupyterhub.readthedocs.io/en/latest/): A tutorial to help install and manage JupyterHub on a cloud with Kubernetes.
- [HubPloy](https://hubploy.readthedocs.io/en/latest/): a suite of commandline tools & a python library for continuous deployment of JupyterHub on Kubernetes (with Zero to JupyterHub)
- [Repo2Docker](https://repo2docker.readthedocs.io/en/latest/): a tool to build, run, and push Docker images from source code repositories that run via a Jupyter server.
