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

