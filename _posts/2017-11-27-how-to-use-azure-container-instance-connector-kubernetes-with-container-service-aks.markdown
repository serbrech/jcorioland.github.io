---
layout: post
title:  "How to use the Azure Container Instance connector for Kubernetes with Azure Container Service (AKS)"
date:   2017-11-27 10:00:00 +0100
categories: Kubernetes, Azure, AKS, ACI
author: 'Julien Corioland'
---

[Azure Container Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/) is a new service (currently in preview) that allows to deploy a managed Kubernetes cluster into Azure. Basically, you only have to pay for the nodes (virtual machines) that run in your cluster and you do not have to deal with Kubernetes masters. Actually, you do not even see Kubernetes masters that are totally managed by the AKS service.
[Azure Container Instance (ACI)](https://docs.microsoft.com/en-us/azure/container-instances/) is a serverless service that allows to spin up both Linux and Windows Containers, without having to deal with complex infrastructure or orchestration system. Machines that run your containers are not visible and you do not pay for them. You only pay for your containers, on a per-minute billing base.
In this blog post I will explain how it is possible to use the ACI-Connector for Kubernetes, that allows to ask Kubernetes to schedule workloads into Azure Container Instance.

## Deploy a Kubernetes cluster using AKS

Deploying a Kubernetes cluster using Azure Container Service (AKS) is just super easy. First, you need to install [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) on your system or you can also use the [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview).

### Create a resource group for the cluster

```bash
az group create --name aks-cluster-rg --location westeurope
```

*Note: while I am writing this blog post, AKS is in preview and only available from few Azure regions (at least eastus and westeurope)*

### Create the AKS cluster

```bash
az aks create --name aks-cluster --resource-group aks-cluster-rg --location westeurope --node-count 2
```

*Note: if you do not have a SSH key, you can also use the `--generate-ssh-keys` parameters to create a new one during the deployment process and if you do not want to use your default SSH key ($HOME/.ssh/id_rsa.pub) you can override this value using the `--ssh-key-value` parameter.*

Wait while the cluster is being created.

### Connect to the cluster

Now that your cluster is up and running, you can connect to it using the `kubectl` command line tool. If you do not have it installed, use the following command to install it:

```bash
az aks install-cli
```

Then you can get the configuration to connect your new cluster using:

```bash
az aks get-credentials --name aks-cluster --resource-group aks-cluster-rg
```

It will automatically download the Kubernetes configuration and merge it with the other configurations you may already have on your machine.

To ensure that everything is OK, try to list the nodes using:

```bash
kubectl get nodes
```

Result:

```bash
jcorioland@DESKTOP-1NB5G25:~$ kubectl get nodes
NAME                       STATUS    ROLES     AGE       VERSION
aks-nodepool1-15523320-0   Ready     agent     3d        v1.8.1
aks-nodepool1-15523320-1   Ready     agent     3d        v1.8.1
```

# Install the Azure Container Instance connector for Kubernetes

The Azure Container Instance connector for Kubernetes is an open source project hosted on [GitHub](https://github.com/Azure/aci-connector-k8s).

*Note: At the time I am writing this blog post, it is currently in preview and you should not use it for production.*

First, create a resource group for Azure Container Instance:

```bash
az group create --name aci-rg --location westeurope
```

Then, you need to create a service principal that is contributor on your subscription:

```bash
az ad sp create-for-rbac --role=Contributor --scopes="/subscriptions/YOUR_SUBSCRIPTION_ID/"
```

*Note: you can get your subscription id using the `az account show` command.*

In the result, you need to save the following values:

- the `appId` field that represents the client id
- the `password` field that represents the client secret
- the `tenant` field

Now, create a YAML file using the following content and replace environment variables with your values:

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: aci-connector
  namespace: default
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: aci-connector
    spec:
      containers:
      - name: aci-connector
        image: microsoft/aci-connector-k8s:latest
        imagePullPolicy: Always
        env:
        - name: AZURE_CLIENT_ID
          value: <CLIENT_ID>
        - name: AZURE_CLIENT_KEY
          value: <CLIENT_KEY>
        - name: AZURE_TENANT_ID
          value: <TENANT_ID>
        - name: AZURE_SUBSCRIPTION_ID
          value: <SUBSCRIPTION_ID>
        - name: ACI_RESOURCE_GROUP
          value: <RESOURCE_GROUP>
```

You can now deploy the Azure Container Instance connector for Kubernetes using the following command:

```bash
kubectl create -f aci-connector.yaml
```

That's it! After few minutes you should see that a new node has appeared in your cluster (using `kubectl get nodes`):

```bash
jcorioland@DESKTOP-1NB5G25:~$ kubectl get nodes
NAME                       STATUS    ROLES     AGE       VERSION
aci-connector              Ready     <none>    3d        v1.6.6
aks-nodepool1-15523320-0   Ready     agent     3d        v1.8.1
aks-nodepool1-15523320-1   Ready     agent     3d        v1.8.1
```

This will allow Kubernetes to schedule containers using Azure Container Instance instead of running on your classical nodes (the virtual machines running in your subscriptin).

To ask Kubernetes to deploy on this new node, you can use the `nodeName` property in the Kubernetes manifest file. For example, to deploy a new nginx pod on ACI:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx
  dnsPolicy: ClusterFirst
  nodeName: aci-connector
```

