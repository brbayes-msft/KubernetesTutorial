# Setup for Kubernetes Session
These steps walk through the tooling that will be necessary for this lab with Kubernetes, with all of the tooling being run locally. At least 16GB of RAM is recommended for this setup.

## Tools to Install
 - Azure CLI (`https://docs.microsoft.com/en-us/cli/azure/install-azure-cli`)
 - Kubernetes CLI (`az aks install-cli`)
 - Helm CLI (`https://helm.sh/docs/intro/install/`)
 - Docker Desktop (`https://www.docker.com/products/docker-desktop`)
 - Visual Studio Code (`https://code.visualstudio.com/`)
   - Kubernetes extension (`https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools`)

## Setup Kubernetes Locally
This section will walk you through setting up Kubernetes to run on your local machine. It is assumed that all of the above tools have been installed.

1) Open Docker Desktop and go to the Settings menu.
2) Select Kubernetes from the sidebar.
3) Check the box for `Enable Kubernetes`, and then click `Apply & Restart`.
4) It may take a few for the local Kubernetes cluster to start up. Wait for the `Starting...` progress bar under the `Enable Kubernetes` checkbox to disappear.
5) After the checkbox disappears, you should be able to open a terminal and run the command `kubectl get nodes`. It should show a single node labeled `docker-desktop`.