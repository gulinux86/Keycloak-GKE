# kubernetes-GKE

# This tutorial has a intention to deploy a Keycloak application on GKE(Kubernetes) with Cert-Manager

 First of all, we need to install an **Ingress-Controller** for GKE (Kubernetes). Click on the Nginx documentation and search for GCE-GKE

# https://kubernetes.github.io/ingress-nginx/deploy/#environment-specific-instructions

After find it, you we should run the commands bellow:

Run te coommands bellow:

```kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user $(gcloud config get-value account)```

```kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.0/deploy/static/provider/cloud/deploy.yaml```


After installation run the command to check the EXTERNAL-IP and and Ingress-Controller on the ingress-nginx namespaces:

kubectl get all -n ingress-nginx

NAME                                            READY   STATUS    RESTARTS   AGE
pod/ingress-nginx-controller-5cdcb78546-bqvdp   1/1     Running   0          24h

NAME                                         TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.32.129.234   34.29.129.164   80:30378/TCP,443:30381/TCP   24h
service/ingress-nginx-controller-admission   ClusterIP      10.32.128.146   <none>          443/TCP                      24h

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           24h

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-5cdcb78546   1         1         1       24h

NAME                                       COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   1/1           2m9s       24h
job.batch/ingress-nginx-admission-patch    1/1           2m17s      24h


