# Kubernetes Deployment Instructions

## What is Kubernetes?

The [website](https://kubernetes.io/) puts it best: "Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications."

You might see Kubernetes referred to as "k8s", pronounced "kates".

What does this mean for Realm and our customers? Since Realm Object Server is technically collection of microservices, it can be difficult, if not impossible, to make assumptions about deployment environments that may influence design decisions. This leads to situations where we need to implement features and/or workarounds for single customers. This can lead to an ever-growing list of test paths, which leads to instability, which leads to unhappy customers.

With the primitives and patterns that Kubernetes provides, we can make assumptions that simplify our software, enable end-to-end testing for all server team members and lessen the knowledge burden for support.

### Basic Kubernetes Primitives

All of these concepts are defined in Kubernetes by what's known as a "manifest." A manifest is simply a YAML file that follows a particular schema. When a manifest is deployed to Kubernetes, a resource is persisted using the contents of the YAML file.

It is important to note that the existence of these resources in Kubernetes does _not_ mean that the behavior defined by the manifest is realized, but that Kubernetes will continuously try to ensure that this is the case. Kubernetes will update the resource with additional status information.

All resources in Kubernetes can also contain metadata defined by the user, in the form of labels and annotations. This is consistent across all resource types.

#### Nodes

* A Node is a physical \(or virtual\) machine that participates in a Kubernetes Cluster.

#### Pods

* A pod is a collection of one or more running containers.
* Each pod has its own IP address that is routable from any other pod in the cluster.
* Each container in a pod shares the same network stack, so they can access each other over `localhost`.
* Pods are immutable -- a change to the arguments, environment variables, or any other runtime attribute requires the creation of a new pod.

#### ReplicaSets

* A ReplicaSet is a collection of zero or more pods of the same configuration.
* The pod configuration is defined by a pod template
* ReplicaSets are also immutable, with the exception of the number of pods it should maintain.
* ReplicaSets are not typically managed by end users, but by Deployments \(below\).

#### Deployments

* A Deployment is a means to define a workload consisting zero or more ReplicaSets.
* Consists of a Pod template, which is used to define the attributes of the resulting pods \(e.g., docker image, CLI args, environment variables, etc\).
* When a user modifies a Deployments pod template, Kubernetes will facilitate the deployment of new pods by creating a new ReplicaSet and then removing the old one.

#### Services

* A Service is a collection of zero or more pods that can be addressed by a single IP address.
* Service IP addresses can be local to the cluster \(`ClusterIP`\), local to the network that contains the cluster \(`NodePort`\), or bound to a cloud-provided LoadBalancer such as Amazon ELB \(`LoadBalancer`\).
* Kubernetes will continuously manage the list of participating pods based on the its label selector.

#### Ingresses

* An Ingress is a means to define Layer 7 \(HTTP\) routing of traffic to zero or more Services.
* Ingresses are typically used to expose a Service or group of Services to the internet.
* Ingresses require the cluster to have a working Ingress Controller, which implements the behavior that the user has defined.

#### ConfigMaps

* A ConfigMap is a list of data blobs that is defined by the user.
* Data blobs are typically the contents of configuration files that pods \(or more specifically, containers\) might require to run.
* ConfigMaps can be mounted in Pods as a directory, allowing applications in the pod to use them as if they were on a typical linux system.

#### Secrets

* Secrets are similar to ConfigMaps, but the data blobs are always Base64 encoded.

#### Namespaces

* A namespace is a means to partition a cluster into different organizational groups.
* Access to namespaces can be controlled by role-based access control \(RBAC\)
* Many resource types must be namespaced, including all resource types describe above \(except Nodes\).
* There is a default namespace, named `default`. This namespace is used by utilities unless otherwise specified.

### Basic Management

The most basic and powerful tool for managing applications in Kubernetes is `kubectl`. You may hear it pronounced "Kyoob cuddle" or "Kyoob control." Its capabilities are immense, as shown in this [cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).

Let's use the utility to run a Hello World application:

```text
kubectl run hello-world --image tutum/hello-world
```

You have just deployed an application to your cluster! Let's inspect what has been done:

```text
kubectl get deployments                            # List all Deployments
kubectl get deployment hello-world -oyaml          # Get details of the deployment in YAML form
kubectl rollout status deployment hello-world -w   # Ensure the deployment has completed
kubectl get pods                                   # List resulting Pods
```

Now that the application is running, we can expose it so that it's accessible:

```text
kubectl expose deployment hello-world --port=80 --type=NodePort
```

The above command creates a Service:

```text
kubectl get service hello-world -oyaml
```

Now we can access our application:

```text
export NODE_PORT=$(kubectl get -o jsonpath="{.spec.ports[0].nodePort}" service hello-world)
curl http://localhost:$NODE_PORT/   # Access it from CLI
open http://localhost:$NODE_PORT/   # Access it in the browser
```

## Kubernetes on MacOS

The easiest way to get started with Kubernetes is by using Docker for Mac. Start by downloading Docker for Mac [here](https://store.docker.com/editions/community/docker-ce-desktop-mac). Be sure to use the download link for "Get Docker CE for Mac \(edge\)" \(the default button gets you "stable", make sure to click the button further down the page marked "edge"\). You will need to register or sign in to the Docker Store in order to download. Then, follow the provided install instructions.

If you have not already done so, launch the Docker application that you downloaded. Then, open the preferences and enable Kubernetes in the "Kubernetes" tab. Click "Apply" and wait a few minutes while Kubernetes starts.

After installation, verify that you have the latest version that was tested with this documenation:

```text
$ docker version
Client:
 Version:      18.05.0-ce
 API version:  1.37
 Go version:   go1.9.5
 Git commit:   f150324
 Built:        Wed May  9 22:12:05 2018
 OS/Arch:      darwin/amd64
 Experimental: false
 Orchestrator: swarm

Server:
 Engine:
  Version:      18.05.0-ce
  API version:  1.37 (minimum version 1.12)
  Go version:   go1.10.1
  Git commit:   f150324
  Built:        Wed May  9 22:20:16 2018
  OS/Arch:      linux/amd64
  Experimental: true
```

#### Helm CLI and tiller

[Helm](https://helm.sh/) is required to use our preferred method of installing Realm Object Server, so you will need to install the commandline utility using Homebrew:

```text
brew install kubernetes-helm
```

Then, prepare your cluster for using helm:

```text
helm init --upgrade --wait
```

#### Install the Kubernetes Dashboard using Helm

You can use Helm to easily install the Kubernetes Dashboard in your cluster:

```text
helm install --kube-context=docker-for-desktop --wait \
            --name kubernetes-dashboard --namespace kube-system \
            --set service.type=NodePort,service.nodePort=30443 \
            stable/kubernetes-dashboard
```

You can now access a UI for your cluster at the URL: [https://127.0.0.1:30443/](https://127.0.0.1:30443/). Bypass the insecure certificate warning and choose "Skip" when presented with an authentication dialog.

## ROS Helm Chart Documentation

"Helm helps you manage Kubernetes applications — Helm Charts helps you define, install, and upgrade even the most complex Kubernetes application." - [helm.sh](https://helm.sh/)

Helm uses a "chart," which is simply a set of templates that define what Kubernetes resources should be present in order to provide the service it defines. Typically, a simple chart will include a template for:

* ConfigMap
* Secret
* Deployment
* Service
* Ingress

These are all basic building blocks for running apps in Kubernetes, and you should be familiar with them before proceeding. To find out more, refer to [Kubernetes Basics](kubernetes-deployment-instructions.md#what-is-kubernetes).

### Prerequisites

To get started using Docker for Mac, please refer to [Kubernetes on macOS](kubernetes-deployment-instructions.md#kubernetes-on-macos).

Ensure you've installed [Helm](https://docs.helm.sh/using_helm/#installing-helm).

### Deploying the ROS using the Helm Chart

You will need a local copy of the ROS repository in order to perform this installation. If you have not already cloned it, you can do so by running:

```text
git clone https://github.com/realm/realm-server-side-samples.git
```

All commands after this assume that you are in a subdirectory of this repository:

```text
cd realm-server-side-samples/helm
```

First, let's try to deploy ROS using default configuration values:

```text
helm upgrade --install my-ros ./realm-object-server
```

This command could take a few minutes. You can check the status of the deployment by using the Kubernetes dashboard or by listing the current Helm releases in your cluster:

```text
$ helm list
NAME                	REVISION	UPDATED                 	STATUS  	CHART                     	NAMESPACE  
kubernetes-dashboard	1       	Thu Jun  7 16:00:46 2018	DEPLOYED	kubernetes-dashboard-0.7.0	kube-system
my-ros              	3       	Thu Jun  7 14:52:36 2018	DEPLOYED	realm-object-server-0.1.0 	default    
```

For future reference, you an uninstall ROS using the following command. If you get stuck, this might be the best course of action after which you can start over:

```text
helm delete --purge my-ros
```

In order to obtain logs from ROS, we must first identify the pods:

```text
$ kubectl get pods
NAME                                          READY     STATUS    RESTARTS   AGE
my-ros-realm-object-server-5f4fcbfbfc-gp7l2   1/1       Running   0          3d
my-ros-sync-default-0                         1/1       Running   2          3d
```

The first pod listed is ROS without the sync server. This is also known as "core services". The second pod listed is the sync worker. Use the pod names to obtain logs:

```text
kubectl logs my-ros-realm-object-server-5f4fcbfbfc-gp7l2
kubectl logs my-ros-sync-default-0
```

### Overriding Defaults

Now that we've verified that ROS is running, we can override the chart defaults in order to expose it. Create an overrides file named `overrides.yaml` using your favorite editor:

```text
service:
  type: NodePort
  nodePort: 30080
```

This file is the same structure and format of the charts [`values.yaml`](https://github.com/realm/realm-object-server-private/blob/master/helm/realm-object-server/values.yaml). You can refer to that file for all configurable attributes.

Now we can update our release with these modifications:

```text
helm upgrade --install my-ros ./realm-object-server -f overrides.yaml
```

Verify that our service is exposed with the supplied values:

```text
kubectl describe service my-ros-realm-object-server
```

... and look for these attributes:

```text
Type:                     NodePort
NodePort:                 http  30080/TCP
```

Great! Let's try to access the service with our browser. You should be presented with the ROS splash screen.

```text
open http://localhost:30080
```

We can also access ROS using Realm Studio. The Helm chart contains a default private key, so you can use the following admin token to authenticate to the above URL:

```text
eyJhcHBfaWQiOiJpby5yZWFsbS5hdXRoIiwiaWRlbnRpdHkiOiJfX2FkbWluIiwiYWNjZXNzIjpbImRvd25sb2FkIiwidXBsb2FkIiwibWFuYWdlIl0sInNhbHQiOiIyOGMwNDgzMCJ9:c2S8hMDUua/zfizq3AqZFOB07Adow6JOUuSucyTyvhTtVdJBN3tGjxD/7FKL9CJ77JI8DqoNB/1grR9iXZlkGXU7aiPxttA+lYtoEU9Rbo85IyKN2Yf5C28U6X8gUrI6hGeTSCm1DPCInrW8ZcBKOfTb67IY9PLlAU/9gGap4LyguvejD/TEpsLSWgTSiS/UME5IzZa4Y5YjQ1f8G5bhFSDaIIN3yrS8O8VXHbZ/qpBXdmPku6Jn7q+L7W4usvgPxLf57Te3TfM5eqAvKtD/vx+SJAiAJifPdig0Xt1Zy2ZsoV5zrG4q+GP0E4sDQ/AYP4HVeeuoMkNgi2q58jmJuQ==
```

It is important to mention that this is only one method of exposing ROS so that it is accessible. This method is suitable for local installations using Docker for Mac.

### Exposing ROS via LoadBalancer

When using a cloud provider such as AWS or GKE, you can use the `LoadBalancer` service type to expose ROS to the internet. Here is a basic example of the overrides required to do so:

```text
service:
  type: LoadBalancer
```

This override works in GKE, however AWS requires that we set a couple of options in order to properly handle websocket connections:

```text
service:
  type: LoadBalancer
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "tcp"
    service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "3600"
```

### Enabling Stats

The `realm-object-server` Helm chart also includes stats collection as an option. To enable it, add the following to your `overrides.yaml` file \(remember, these options are shown and documented in the charts `values.yaml` file\).

```text
prometheus:
  enabled: true
grafana:
  enabled: true
  # You can also configure the service and/or ingress, just as with
  # core services:
  service:
    type: NodePort
    port: 30081
```

This will ensure that Prometheus is installed alongside Realm Object Server. It will be configured to scrape ROS periodically for stats, which can be viewed in Grafana. The Grafana installation includes a few canned dashboards that are preconfigured to view these metrics.

