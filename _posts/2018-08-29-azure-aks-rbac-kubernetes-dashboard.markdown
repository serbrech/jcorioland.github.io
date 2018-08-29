---
layout: post
title:  "How to access Kubernetes dasbhoard on an Azure Kubernetes Service cluster with RBAC enabled"
date:   2018-08-29 11:00:00 +0200
categories: Azure, AKS, Kubernetes
author: 'Julien Corioland'
identifier: '5cd46126-f5c0-492a-9ec6-f30d3359925e'
---

RBAC (Role Based Access Control) is enabled by default when you deploy a new Azure Kubernetes Service cluster, which is great. But if you are not use to that, you may have some trouble to access the Kubernetes dashboard using `kubectl proxy` or `az aks browse` command line tools (remember to never expose the dashboard over the Internet, even if RBAC is enabled!).

In this post, I will explain how you can simply configure RBAC on your cluster to solve authorization access issues.

<!--more-->

So, you've deployed your Azure Kubernetes Service cluster, everything went well, you may even have deployed your first workloads on it. Now it's time to launch the dashboard and you got something like that:

![Kubernetes Dashboard - No Access](/images/kubernetes-rbac-dashboard/dashboard-rbac-no-rights.png)

Don't panic. This is the normal behavior. As your cluster is RBAC-enabled, by default the pod that runs the dashboard has a minimal role bound to its service account:

```bash
kubectl describe role kubernetes-dashboard-minimal -n kube-system
```

```tsv
Name:         kubernetes-dashboard-minimal
Labels:       addonmanager.kubernetes.io/mode=Reconcile
              k8s-app=kubernetes-dashboard
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"rbac.authorization.k8s.io/v1beta1","kind":"Role","metadata":{"annotations":{},"labels":{"addonmanager.kubernetes.io/mode":"Reconcile","k...
PolicyRule:
  Resources       Non-Resource URLs  Resource Names                   Verbs
  ---------       -----------------  --------------                   -----
  secrets         []                 []                               [get update create delete]
  configmaps      []                 [kubernetes-dashboard-settings]  [get update]
  services/proxy  []                 [heapster]                       [get]
  services/proxy  []                 [http:heapster:]                 [get]
  services/proxy  []                 [http:metrics-server:]           [get]
  services/proxy  []                 [https:heapster:]                [get]
  services/proxy  []                 [https:metrics-server:]          [get]
  services/proxy  []                 [metrics-server]                 [get]
  services        []                 [heapster]                       [proxy]
  services        []                 [metrics-server]                 [proxy]
```

If you want to make sure the Kubernetes dashboard can access all the resources in the cluster, you can simply create a `ClusterRoleBinding` object to bind the `cluster-admin` role to the service account that runs the Kubernetes dashboard pod, using the following command:

```bash
kubectl create clusterrolebinding kubernetes-dashboard \
    --clusterrole=cluster-admin \
    --serviceaccount=kube-system:kubernetes-dashboard
```

Once this command applied, just hit refresh in your browser and you should have a Kubernetes dashboard up and running with no access error messages anymore:

![Kubernetes Dashboard - Cluster Admin](/images/kubernetes-rbac-dashboard/dashboard-rbac-cluster-admin.png)

OK, this is great. But now, you should know that the Kubernetes dashboard pod can do anything a cluster administrator can do. This can be fine with your strategy. But you may also want to control a little bit more what happens here.

In that case, you can start from the minimal role definition [here](https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/dashboard/dashboard-rbac.yaml) and add the rules that you want add to the dashboard.
While it's done, just apply the yaml file again:

```bash
kubectl apply -f dashboard-rbac.yaml
```

Hope this helps!

Julien