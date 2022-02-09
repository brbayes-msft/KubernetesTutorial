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
Helm is an additional CLI that simplifies the process of deploying applications. It includes support for templating Kubernetes manifests.

For learning more about Helm, you can visit the Helm Docs at https://helm.sh/docs/.

For this part of the tutorial, we will take the deployment from the last section and convert it to a Helm template. There is a starter template in the `Templates/HelmChart` folder. In a helm chart, the folder structure goes something like this:

 - Chart.yaml: A definition file for the chart overall, defining the name of the chart and its associated version.
 - values.yaml: A variable file that is fed into the templates to generate the final templates. Variables can be submitted either by a values file (you can also combine multiple values files together) or as a command line parameter at the time of deployment.
 - templates/: The folder that contains the templatized version of the manifests to be generated. New files can be added here as necessary to create new manifests.

In the starter template, there is a list of applications with a single nginx app in it. The example will iterate over the list of applications, creating the appropriate manifests. To deploy the example as is, you can navigate to that folder from a terminal and then run the following command:

``` bash
helm upgrade --install -f values.yaml testservice . # testservice is the name of the Helm release. This is how Helm knows which resources to update when performing operations.

helm upgrade --install testservice --set apps.webserver.replicaCount=3 -f values.yaml .

kubectl get deployments

kubectl port-forward deployment/webserver 8080:80

helm uninstall testservice
```

The exercise for the remainder of this section is to take the voting app that was built in the last exercise, and create the appropriate Helm templates. The values file will need to be updated with each of the new apps that need to be deployed, and then a new template will need to be added to expose the deployments with a service.

### Helm Syntax
There are two main pieces of Helm syntax that you will need to know for this exercise. The first is referencing a variable from the values file. For that, you will use the below example, where myVarName is the name of the variable being referenced in the values file:

``` yaml
# Values file
myVarName: Hello World

# Template file
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.myVarName }}
```

The second important piece of syntax is that of range. Range allows for iteration over an array or dictionary in helm. For example:

``` yaml
# Values file
apps:
  myApp:
    port: 80

# Template file
{{- range $appName, $config := .Values.apps }} # $appName is the name of the key in a dictionary, and $config is the value in the dictionary.
apiVersion: v1
kind: Service
metadata:
  name: {{ $appName }}
spec:
  type: ClusterIP
  ports:
    - port: {{ $config.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app: {{ $appName }}
---
{{- end }} # This end syntax is necessary when using multi-line Helm tags.
```

For testing purposes, it can often be useful to see the rendered manifests before actually deploying them. For that, you can use the `--dry-run` flag with Helm, and the manifests will be generated without actually deploying anything to the cluster.

``` bash
helm upgrade --install -f values.yaml testservice --dry-run .
```

## WordPress with Helm (Optional)
This is an optional exercise that walks through pulling a chart from a public source to run WordPress on Kubernetes.

``` bash
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install wp-release --set wordpressUsername=admin --set wordpressPassword=password --set mariadb.mariadbRootPassword=secretpassword bitnami/wordpress

kubectl port-forward service/wp-release-wordpress 8080:80
```

## Additional Resources
https://kubernetes.io/docs/tutorials/