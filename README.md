# kubernetes-GKE

This tutorial has a intention to deploy a Keycloak application on GKE(google Kubernetes)

================================================================================================

# Deploy Keycloak on GKE AutoPilot with Cert-Manager

First of all, we need to install an **Ingress-Controller** for GKE (Kubernetes). Click on the Nginx documentation and search for GCE-GKE

https://kubernetes.github.io/ingress-nginx/deploy/#environment-specific-instructions

After find it, you we should run the commands bellow:

Run te coommands bellow:

```kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole cluster-admin \
  --user $(gcloud config get-value account)


kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.0/deploy/static/provider/cloud/deploy.yaml```
