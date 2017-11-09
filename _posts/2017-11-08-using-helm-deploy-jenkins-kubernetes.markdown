---
layout: post
title:  "Using Helm to deploy Jenkins in Kubernetes"
date:   2017-11-08 11:13:58 +0100
categories: Kubernetes, Azure, Jenkins, DevOps
---

[Helm](https://github.com/kubernetes/helm) is a package manager for Kubernetes that allows to deploy applications in a smooth way. In this blog post, I will describe how you can use Helm to deploy all the services you need to have a Jenkins server up and running in your Kubernetes cluster.
I assume that you already have a Kubernetes cluster running with Helm installed.

## Install Kube Lego and Kubernetes Ingress with Nginx

[kube-lego](https://github.com/jetstack/kube-lego) automatically requests certificates for Kubernetes Ingress resources from Let's Encrypt to enable SSL on Kubernetes ingress.

To install kube-log, update the following command with your valid email and execute it:

`helm install stable/kube-lego --set config.LEGO_EMAIL=<valid-email>,config.LEGO_URL=https://acme-v01.api.letsencrypt.org/directory`

Results:

``` yaml
NAME:   quiet-eagle
LAST DEPLOYED: Mon Oct 16 16:10:51 2017
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/Deployment
NAME                   DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
quiet-eagle-kube-lego  1        1        1           0          0s


NOTES:
This chart installs kube-lego to generate TLS certs for Ingresses.

EXAMPLE INGRESS YAML:

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example
  namespace: foo
  annotations:
    kubernetes.io/ingress.class: nginx
    # Add to generate certificates for this ingress
    kubernetes.io/tls-acme: 'true'
spec:
  rules:
    - host: www.example.com
      http:
        paths:
          - backend:
              serviceName: exampleService
              servicePort: 80
            path: /
  tls:
    # With this configuration kube-lego will generate a secret in namespace foo called `example-tls`
    # for the URL `www.example.com`
    - hosts:
        - "www.example.com"
      secretName: example-tls
```

Now install the Nginx ingress component for Kubernetes:

`helm install stable/nginx-ingress`

Wait for the component to be installed and follow the instructions printed by Helm to retrieve the public IP address of the nginx-ingress service.

Results:

``` yaml
NAME:   giddy-bobcat
LAST DEPLOYED: Mon Oct 16 16:11:23 2017
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/Deployment
NAME                                        DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
giddy-bobcat-nginx-ingress-controller       1        1        1           0          5s
giddy-bobcat-nginx-ingress-default-backend  1        1        1           1          5s

==> v1/ConfigMap
NAME                                   DATA  AGE
giddy-bobcat-nginx-ingress-controller  1     5s

==> v1/Service
NAME                                        CLUSTER-IP    EXTERNAL-IP  PORT(S)                     AGE
giddy-bobcat-nginx-ingress-controller       10.0.202.208  <pending>    80:31295/TCP,443:32203/TCP  5s
giddy-bobcat-nginx-ingress-default-backend  10.0.134.188  <none>       80/TCP                      5s


NOTES:
The nginx-ingress controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w giddy-bobcat-nginx-ingress-controller'

An example Ingress that makes use of the controller:

  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

Update your DNS provider to add a DNS record (A) that points to the external IP address of the nginx-ingress service

```
*.k8s.your-domain.com in A <nginx-ingress service external IP address>
```

Wait for the DNS propagation to be completed.

## Installing Jenkins

To install Jenkins you can also use an helm chart. But you need to provide some parameters values to helm to configure it properly to be able to use the ingress component, deploy Jenkins with some preinstalled modules, like the Kubernetes plugins that allows to run Jenkins agents into containers within the cluster, for example.

Create a file `jenkins-values.yaml` and be careful to replace `jenkins.k8s.your-domain.com` with the domain you configured for the ingress component: 

``` yaml
Master:
  Memory: "512Mi"
  HostName: jenkins.k8s.your-domain.com
  ServiceType: ClusterIP 
  InstallPlugins:
      - kubernetes:0.12
      - workflow-aggregator:2.5
      - credentials-binding:1.13
      - git:3.5.1
      - pipeline-github-lib:1.0
      - ghprb:1.39.0
      - blueocean:1.1.7

ScriptApproval:
    - "method groovy.json.JsonSlurperClassic parseText java.lang.String"
    - "new groovy.json.JsonSlurperClassic"
    - "staticMethod org.codehaus.groovy.runtime.DefaultGroovyMethods leftShift java.util.Map java.util.Map"
    - "staticMethod org.codehaus.groovy.runtime.DefaultGroovyMethods split java.lang.String"
    - "method java.util.Collection toArray"
    - "staticMethod org.kohsuke.groovy.sandbox.impl.Checker checkedCall java.lang.Object boolean boolean java.lang.String java.lang.Object[]"
    - "staticMethod org.kohsuke.groovy.sandbox.impl.Checker checkedGetProperty java.lang.Object boolean boolean java.lang.Object"

Ingress:
  Annotations:
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"

  TLS:
    - secretName: jenkins.k8s.your-domain.com
      hosts:
        - jjenkins.k8s.your-domain.com

Agent:
  Enabled: false
``` 

Then, run the following command to install Jenkins:

`helm install --namespace jenkins --name jenkins -f jenkins-values.yaml stable/jenkins`

Results:

``` yaml
NAME:   jenkins
LAST DEPLOYED: Mon Oct 16 17:19:38 2017
NAMESPACE: jenkins
STATUS: DEPLOYED

RESOURCES:
==> v1/Secret
NAME             TYPE    DATA  AGE
jenkins-jenkins  Opaque  2     7s

==> v1/ConfigMap
NAME                   DATA  AGE
jenkins-jenkins        3     7s
jenkins-jenkins-tests  1     7s

==> v1/PersistentVolumeClaim
NAME             STATUS  VOLUME                                    CAPACITY  ACCESSMODES  STORAGECLASS  AGE
jenkins-jenkins  Bound   pvc-7512c815-b285-11e7-a09f-000d3a26f11a  8Gi       RWO          default       7s

==> v1/Service
NAME                   CLUSTER-IP    EXTERNAL-IP  PORT(S)    AGE
jenkins-jenkins-agent  10.0.167.233  <none>       50000/TCP  7s
jenkins-jenkins        10.0.86.1     <none>       8080/TCP   7s

==> v1beta1/Deployment
NAME             DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
jenkins-jenkins  1        1        1           0          7s

==> v1beta1/Ingress
NAME             HOSTS                                ADDRESS  PORTS  AGE
jenkins-jenkins  jenkins.k8s.your-domain.com  80       7s


NOTES:
1. Get your 'admin' user password by running:
  printf $(kubectl get secret --namespace jenkins jenkins-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo

2. Visit http://jenkins.k8s.your-domain.com

3. Login with the password from step 1 and the username: admin

For more information on running Jenkins on Kubernetes, visit:
https://cloud.google.com/solutions/jenkins-on-container-engine
```

Confirm the pod has started:

`watch kubectl get pods --namespace jenkins`

Results:

``` sh
Every 2.0s: kubectl get pods --namespace jenkins Mon Oct 16 17:06:30 2017

NAME                               READY     STATUS    RESTARTS   AGE
jenkins-jenkins-4019532517-bglg9   1/1       Running   0          53s
```

## Login to Jenkins

In the output of the install there will be a script to retrieve the user password from the Jenkins instance (NOTES #1). It will look something like the below, pay special attention to the name (in this case `jenkins-jenkins`):

`printf $(kubectl get secret --namespace jenkins jenkins-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo`

Make a note of the password an log in to the Jenkins cluster by visiting the service using the hostname you have configured in the `jenkins-values.yaml` file.

Log in with the username `admin` and the password you retreived earlier. You may want to change your admin password and/or create a new user at this point.