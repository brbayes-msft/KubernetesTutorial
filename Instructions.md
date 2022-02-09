# Kubernetes Tutorial Instructions
This tutorial walks through some of the basics of using Kubernetes. It utilizes Docker Desktop to run a local Kubernetes cluster. Before continuing with this tutorial, it is recommended to first go through the setup instructions in `Setup.md`.

## kubectl Overview
kubectl is the main CLI that is used for operating with Kubernetes. All operations in kubectl are structed as follows:

``` bash
kubectl [command] [TYPE] [NAME] [flags]
```

where command, TYPE, NAME, and flags are:

 - command: Specifies the operation that you want to perform on one or more resources, for example create, get, describe, delete.

 - TYPE: Specifies the resource type. Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms. For example, the following commands produce the same output:

``` bash
kubectl get pod pod1
kubectl get pods pod1
kubectl get po pod1
```

 - NAME: Specifies the name of the resource. Names are case-sensitive. If the name is omitted, details for all resources are displayed, for example kubectl get pods.

 - flags: Specifies optional flags. For example, you can use the -s or --server flags to specify the address and port of the Kubernetes API server. There is also -n or --namespace to specify a particular namespace for a resource.

More information can be found at https://kubernetes.io/docs/reference/kubectl/overview/.

## kubectl Cheat Sheet
1. List all namespace services: `kubectl get services`
2. Retrieve details on nodes: `kubectl get nodes`
3. Create a new namespace: `kubectl create namespace my-namespace`
4. Apply manifest to cluster: `kubectl apply -f deployment.yaml`
5. Get Logs: `kubectl logs my-pod-name` or `kubectl logs deployment/my-deployment-name`
6. Check Kubernetes events: `kubectl get events`

More commands can be found here at https://phoenixnap.com/kb/kubectl-commands-cheat-sheet

## Create a basic pod
For this exercise, we're going to use the pod template to create a basic pod, and then connect to it.  From the Templates folder, we are going to use the below commands:

``` bash
kubectl apply -f PodTemplate.yaml
kubectl get pods
kubectl port-forward pod/my-pod-example 8080:80
kubectl logs my-pod-example
```

## Create a deployment
For this exercise, we will create a manifest will two deployments, and two services, one for each deployment. The backend deployment will be a redis cache that will be used for storing data, while the frontend will be a UI. For this exercise, you will need to use the templates `DeploymentTemplate.yaml` and `ServiceTemplate.yaml`. The general specifications are included below, but this is left as an exercise for you to go through on your own. The final manifest can be found in the `FinalManifests` folder under `VotingMachine.yaml`.

 - Frontend
   - Image: mcr.microsoft.com/azuredocs/azure-vote-front:v1
   - Port: 80
   - Environment Variables:
     - REDIS: <!-- The name of the service for the Redis Cache -->
 - Backend
   - Image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
   - Port: 6379
   - Environment Variables:
     - ALLOW_EMPTY_PASSWORD: yes

### Hint
```
To simplify things, you can add multiple manifest files in a single yaml file by delimiting them with `---` on a new line.
```

After completing the manifest files, you should be able to deploy them using `kubectl apply`, and then using the `kubectl port-forward` command from earlier, except using the service name instead. (i.e. `kubectl port-forward service/azure-vote-front 8080:80`)

As a final step after getting the code running, try upping the replica count on the frontend deployment, apply the changes, and then use the `kubectl get deployments` command to see the deployment progress.

## Create a Helm Chart


## WordPress with Helm (Optional)
This is an optional exercise that walks through pulling a chart from a public source to run WordPress on Kubernetes.

``` bash
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install wp-release --set wordpressUsername=admin --set wordpressPassword=password --set mariadb.mariadbRootPassword=secretpassword bitnami/wordpress

kubectl port-forward service/wp-release-wordpress 8080:80
```

## Additional Resources
https://kubernetes.io/docs/tutorials/