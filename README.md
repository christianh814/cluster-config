# ARCHIVED

This repo is archived. Please seee [https://github.com/christianh814/openshift-cluster-config](https://github.com/christianh814/openshift-cluster-config) for the latest and greatest




# OpenShift Cluster Config
Setting up OpenShift clusters using Kustomize and a central ArgoCD instance

These are highlevel instructions until I can fill out the README some more.

# Install ArgoCD

> :warning: This is based on the argocd operator v0.0.12 using an "Automatic" update strategy on OpenShift 4.5.2

In order to install ArgoCD into your Hub/Management (whatever you want to call it) cluster, you to have to have admin access to the cluster.

```shell
$ oc whoami
system:admin
```

Once you've verified you're a cluster admin, install ArgoCD using the operator using this repo:

```
until oc apply -k https://github.com/christianh814/cluster-config/argocd/install; do sleep 2; done
```

This will start the installation of ArgoCD. You can monitor the install with a `watch` on the following command:

```
oc get pods -n argocd
```

Once all the required pods are up and running, you can login to the ArgoCD instance using the route generated. To get your argocd route:

> :heavy_exclamation_mark: This installation uses the OpenShift oAuth. You login using your OpenShift credentials.

```
oc get route argocd-server -n argocd -o jsonpath='{.spec.host}{"\n"}'
```

# Add Clusters

After the install is done you need to add your clusters. Before you can add your clusters, you need to login to the Hub/Management server using the `argocd` cli. You do this with the following command.

```
argocd login --sso --insecure $(oc get route argocd-server -n argocd -o jsonpath='{.spec.host}')
```

This will open your default browser and ask you to login using the OpenShift oAuth.

## Your First Cluster

Make sure you login to your cluster **FIRST**! Verify you're logged into your first cluster with the following:

```shell
$ oc cluster-info 
Kubernetes master is running at https://api.cluster1.chx.osecloud.com:6443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
$ oc whoami
kube:admin
```

Now you can add your cluster to ArgoCD with the following

```
argocd cluster add default/api-cluster1-chx-osecloud-com:6443/kube:admin
```

> :heavy_exclamation_mark: **NOTE** You get the cluster name by running `argocd cluster add` (without any other options) after you've logged in.

Verify the cluster has been added

```shell
$ argocd cluster list 
SERVER                                      NAME                                                   VERSION  STATUS      MESSAGE
https://api.cluster1.chx.osecloud.com:6443  default/api-cluster1-chx-osecloud-com:6443/kube:admin  1.18+    Successful  
https://kubernetes.default.svc                                                                              Successful 
```
## Your Second Cluster

Follow the same procedure to add your second cluster.

Verify you're logged into the second cluster as an administrator

```shell
$ oc cluster-info 
Kubernetes master is running at https://api.cluster2.chx.osecloud.com:6443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
$ oc whoami
kube:admin
```

Add your second cluster, once verified.

```
argocd cluster add  default/api-cluster2-chx-osecloud-com:6443/kube:admin
```

Verify that you now have two clusters

> :rotating_light: Note that `https://kubernetes.default.svc` is the server that ArgoCD is running on. AKA the Hub/Management server

```shell
$ argocd cluster list
SERVER                                      NAME                                                   VERSION  STATUS      MESSAGE
https://api.cluster2.chx.osecloud.com:6443  default/api-cluster2-chx-osecloud-com:6443/kube:admin  1.18+    Successful  
https://api.cluster1.chx.osecloud.com:6443  default/api-cluster1-chx-osecloud-com:6443/kube:admin  1.18+    Successful  
https://kubernetes.default.svc                                                                              Successful
```

# Deploying cluster configurations

To deploy the cluster configurations, make sure you're back on the management cluster as a cluster admin.

```shell
$ oc cluster-info 
Kubernetes master is running at https://api.management.chx.osecloud.com:6443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
$ oc whoami
system:admin
```

## Deploy First cluster configurations.

Use this repo to deploy `cluster1` configurations

```
oc apply -k https://github.com/christianh814/cluster-config/clusters/cluster1
```

## Deploy Second cluster configurations.

In the same manner...deploy the second cluster config

```
oc apply -k https://github.com/christianh814/cluster-config/clusters/cluster2
```
