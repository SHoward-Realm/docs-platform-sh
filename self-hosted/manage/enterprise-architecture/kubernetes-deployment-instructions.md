# Kubernetes Deployment Instructions

## Prerequisites

In order to deploy Realm Object Server using Kubernetes, you must have a Kubernetes Cluster available to you. Fortunately, you can easily run Kubernetes on your workstation if you're looking for a quick start.

### Preparing a Kubernetes Cluster

Our cluster deployment utilizes Kubernetes. Before you get started, you'll need to create a cluster if you do not already have one.  

{% tabs %}
{% tab title="macOS" %}
The easiest way to get started with Kubernetes is by using Docker for Mac. Start by downloading Docker for Mac [here](https://store.docker.com/editions/community/docker-ce-desktop-mac). Be sure to use the download link for "Get Docker CE for Mac \(edge\)." You will need to register or sign in to the Docker Store in order to download. Then, follow the provided install instructions.

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
{% endtab %}

{% tab title="Azure" %}
Most major cloud cloud providers offer some kind of managed Kubernetes service.  Microsoft Azure provides with via [Azure Kubernetes Service \(AKS\)](https://docs.microsoft.com/en-us/azure/aks/)

Creating a cluster is easy with AKS.  You can use their [quickstart guide as a reference](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal). After you create the cluster, we recommend ensuring that you can connect via the `kubectl` CLI.  This is also explained in the quickstart guide. At the time of writing, you can do this from an Azure cloud shell with the following command: 

```bash
##input your cluster information for connection 
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

##verify connection by getting cluster information
kubectl get nodes
```
{% endtab %}
{% endtabs %}

### Helm CLI and tiller

[Helm](https://helm.sh/) is required to use our preferred method of installing Realm Object Server, so you will need to install the commandline utillity. 

{% tabs %}
{% tab title="macOS" %}
If you use macOS, you can use Homebrew to install it. Otherwise, refer to [Helm's documentation](https://docs.helm.sh/using_helm/#quickstart-guide).



```text
brew install kubernetes-helm
```

If you have a new cluster, or one that has not been prepared with Tiller, you can do so using the CLI tool that you installed:

```text
helm init --upgrade --wait
```

{% hint style="info" %}
For production clusters, make sure you understand the security implcations of running Tiller in your cluster. You can read more [here](https://docs.helm.sh/using_helm/#securing-your-helm-installation).
{% endhint %}
{% endtab %}

{% tab title="Azure" %}
Using Helm and Tiller in Azure is easy, and you will [find the process documented here](https://docs.microsoft.com/en-us/azure/aks/kubernetes-helm). We recommend using the Azure Cloud Shell which comes with the Helm CLI already installed.  After this, you will need to configure Helm which is detailed in the article linked above.

{% hint style="info" %}
If you are using an RBAC-enabled cluster, you will also need to create a service account to run tiller properly.  
{% endhint %}
{% endtab %}
{% endtabs %}

### Optional: Install the Kubernetes Dashboard

If you're running a cluster using Docker for Mac, you can use Helm to easily install the Kubernetes Dashboard in your cluster:

```text
helm install --kube-context=docker-for-desktop --wait \
            --name kubernetes-dashboard --namespace kube-system \
            --set service.type=NodePort,service.nodePort=30443 \
            stable/kubernetes-dashboard
```

You can now access a UI for your cluster at the URL: [https://127.0.0.1:30443/](https://127.0.0.1:30443/). Bypass the insecure certificate warning and choose "Skip" when presented with an authentication dialog.

{% hint style="info" %}
Do NOT use this installation method for Kubernetes Dashboard on a cluster that is exposed to the internet! Proper deployment considerations are outlined at the projects Github Repository: [https://github.com/kubernetes/dashboard](https://github.com/kubernetes/dashboard)
{% endhint %}

## Using the Realm Object Server Helm Chart

Realm maintains a [Helm Chart](https://docs.helm.sh/developing_charts/#charts) that can be used to install and maintain your deployed Realm Object Server. Think of it as APT or RPM, but for distributed applications.

### Add the repository

Our charts are hosted at Github, and can be added to Helm with the following command:

```text
helm repo add realm https://realm.github.io/charts
```

### Prepare overrides file

As an Enterprise customer, you should have access to a Feature Token. This token must be passed to our Helm chart in order for Realm Object Server to operate. You can do so with an overrides file:

{% code-tabs %}
{% code-tabs-item title="overrides.yaml" %}
```yaml
sync:
  featureToken: YOUR_FEATURE_TOKEN_HERE
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Deploy Realm Object Server

Once you have created an overrides file, you can use it when deploying:

```bash
helm install realm/realm-object-server -f overrides.yaml
```

When successful, you should see some output similar to this:

```text
NAME:   foolish-octopus
LAST DEPLOYED: Thu Jun 21 20:11:02 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Role
NAME                          AGE
foolish-octopus-sync-default  1s

==> v1/StatefulSet
NAME                          DESIRED  CURRENT  AGE
foolish-octopus-sync-default  1        1        1s

==> v1/RoleBinding
NAME                          AGE
foolish-octopus-sync-default  1s

==> v1/Service
NAME                                 TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)            AGE
foolish-octopus-realm-object-server  NodePort   10.103.219.36   <none>       80:30080/TCP       1s
foolish-octopus-metrics              ClusterIP  10.110.66.250   <none>       9081/TCP           1s
foolish-octopus-sync-default         ClusterIP  10.105.177.213  <none>       7800/TCP,9080/TCP  1s

==> v1/Secret
NAME             TYPE    DATA  AGE
foolish-octopus  Opaque  1     1s

==> v1/ConfigMap
NAME                                 DATA  AGE
foolish-octopus                      3     1s

==> v1/ServiceAccount
NAME                          SECRETS  AGE
foolish-octopus-sync-default  1        1s

==> v1/Deployment
NAME                                 DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
foolish-octopus-realm-object-server  1        1        1           0          1s

==> v1/Endpoints
NAME                          ENDPOINTS  AGE
foolish-octopus-sync-default  <none>     1s

==> v1/Pod(related)
NAME                                                  READY  STATUS             RESTARTS  AGE
foolish-octopus-realm-object-server-547948d998-zgwbq  0/1    ContainerCreating  0         1s
foolish-octopus-sync-default-0                        0/1    ContainerCreating  0         1s
```

As you can see, Helm has named our deployment `foolish-octopus`. If you like, you can provide your own name for the deployment upon installation by using the `--name` parameter. Now check the status of your deployment:

```text
$ helm list
NAME                    REVISION    UPDATED                     STATUS      CHART                         NAMESPACE  
kubernetes-dashboard    1           Thu Jun  7 16:00:46 2018    DEPLOYED    kubernetes-dashboard-0.7.0    kube-system
foolish-octopus         3           Thu Jun 21 20:11:02 2018    DEPLOYED    realm-object-server-0.1.0     default
```

For future reference, you an uninstall Realm Object Server using the following command. If you get stuck, this might be the best course of action after which you can start over:

```text
helm delete --purge foolish-octopus
```

In order to obtain logs from Realm Object Server, we must first identify the pods:

```text
$ kubectl get pods
NAME                                                   READY     STATUS    RESTARTS   AGE
foolish-octopus-realm-object-server-5f4fcbfbfc-gp7l2   1/1       Running   0          1h
foolish-octopus-sync-default-0                         1/1       Running   2          1h
```

The first pod listed is Realm Object Server without the sync server. This is also known as "core services". The second pod listed is the sync worker. Use the pod names to obtain logs:

```text
kubectl logs foolish-octopus-realm-object-server-5f4fcbfbfc-gp7l2
kubectl logs foolish-octopus-sync-default-0
```

### Overriding Defaults

We've already demonstrated how to override the chart's default configuration when we provided the feature token. If, for example, you would like to expose Realm Object Server by NodePort \(suitable when using Docker for Mac\), you can simply add to the overrides file:

Now that we've verified that Realm Object Server is running, we can override the chart defaults in order to expose it. Create an overrides file named `overrides.yaml` using your favorite editor:

{% code-tabs %}
{% code-tabs-item title="overrides.yaml" %}
```yaml
sync:
  featureToken: YOUR_FEATURE_TOKEN_HERE
service:
  type: NodePort
  nodePort: 30080
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This file is the same structure and format of the chart's [`values.yaml`](https://github.com/realm/realm-object-server-private/blob/master/helm/realm-object-server/values.yaml). You can refer to that file for all configurable attributes: [https://github.com/realm/charts/blob/master/realm-object-server/values.yaml](https://github.com/realm/charts/blob/master/realm-object-server/values.yaml)

Now we can update our release with these modifications:

```text
helm upgrade foolish-octopus realm-object-server -f overrides.yaml
```

Verify that our service is exposed with the supplied values:

```text
kubectl describe service foolish-octopus-realm-object-server
```

... and look for these attributes:

```text
Type:                     NodePort
NodePort:                 http  30080/TCP
```

Great! Let's try to access the service with our browser. You should be presented with the Realm Object Server splash screen.

```text
open http://localhost:30080
```

We can also access Realm Object Server using Realm Studio. The Helm chart contains a default private key, so you can use the following admin token to authenticate to the above URL:

```text
eyJhcHBfaWQiOiJpby5yZWFsbS5hdXRoIiwiaWRlbnRpdHkiOiJfX2FkbWluIiwiYWNjZXNzIjpbImRvd25sb2FkIiwidXBsb2FkIiwibWFuYWdlIl0sInNhbHQiOiIyOGMwNDgzMCJ9:c2S8hMDUua/zfizq3AqZFOB07Adow6JOUuSucyTyvhTtVdJBN3tGjxD/7FKL9CJ77JI8DqoNB/1grR9iXZlkGXU7aiPxttA+lYtoEU9Rbo85IyKN2Yf5C28U6X8gUrI6hGeTSCm1DPCInrW8ZcBKOfTb67IY9PLlAU/9gGap4LyguvejD/TEpsLSWgTSiS/UME5IzZa4Y5YjQ1f8G5bhFSDaIIN3yrS8O8VXHbZ/qpBXdmPku6Jn7q+L7W4usvgPxLf57Te3TfM5eqAvKtD/vx+SJAiAJifPdig0Xt1Zy2ZsoV5zrG4q+GP0E4sDQ/AYP4HVeeuoMkNgi2q58jmJuQ==
```

{% hint style="info" %}
IMPORTANT: This method of exposing Realm Object Server is suitable for local installations using Docker for Mac.
{% endhint %}

### Exposing Realm Object Server via LoadBalancer

When using a cloud provider such as AWS or GKE, you can use the `LoadBalancer` service type to expose Realm Obbject Server to the internet. Here is a basic example of the overrides required to do so:

{% code-tabs %}
{% code-tabs-item title="overrides.yaml" %}
```yaml
service:
  type: LoadBalancer
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This override works in GKE, however AWS requires that we set a couple of options in order to properly handle websocket connections:

{% code-tabs %}
{% code-tabs-item title="overrides.yaml" %}
```yaml
service:
  type: LoadBalancer
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "tcp"
    service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "3600"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Enabling Stats

The `realm-object-server` Helm chart also includes stats collection as an option. To enable it, add the following to your `overrides.yaml` file \(remember, these options are shown and documented in the charts `values.yaml` file\).

{% code-tabs %}
{% code-tabs-item title="overrides.yaml" %}
```yaml
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
{% endcode-tabs-item %}
{% endcode-tabs %}

This will ensure that Prometheus is installed alongside Realm Object Server. It will be configured to scrape Realm Object Server periodically for stats, which can be viewed in Grafana. The Grafana installation includes a few canned dashboards that are preconfigured to view these metrics.

{% hint style="info" %}
IMPORTANT: Grafana and Prometheus are only provided here as an example of how this could work. It is not properly secured for use in production clusters.
{% endhint %}

