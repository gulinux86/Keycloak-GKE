## This tutorial has a intention to deploy a Keycloak application on GKE(Kubernetes) with Cert-Manager

* First of all, we need to install an **Ingress-Controller** for GKE (Kubernetes). Click on the Nginx documentation and search for GCE-GKE

### https://kubernetes.github.io/ingress-nginx/deploy/#environment-specific-instructions

After find it, you we should run the commands bellow:

Run te coommands bellow:

```
kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user $(gcloud config get-value account)

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.0/deploy/static/provider/cloud/deploy.yaml
```

After installation run the command to check the **EXTERNAL-IP** and and Ingress-Controller on the ingress-nginx namespaces:

```
kubectl get all -n ingress-nginx
```
```
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
```

### Our Cluster will use Cert-manager to support SSL on Keycloak application. Let’s install the Cert-Manager via Helm.

```
helm install --create-namespace --namespace cert-manager --set installCRDs=true --set global.leaderElection.namespace=cert-manager cert-manager jetstack/cert-manager
```

### Checking Cert-Manager’s namespace:

```
kubectl get all -n cert-manager
```
```
NAME                                         READY   STATUS    RESTARTS   AGE
pod/cert-manager-66d588bfc5-cch5q            1/1     Running   0          24h
pod/cert-manager-cainjector-c89c4c45-4wm4k   1/1     Running   0          24h
pod/cert-manager-webhook-6c4bd6b66b-m9nbj    1/1     Running   0          24h

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/cert-manager           ClusterIP   10.32.129.211   <none>        9402/TCP   24h
service/cert-manager-webhook   ClusterIP   10.32.128.239   <none>        443/TCP    24h

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cert-manager              1/1     1            1           24h
deployment.apps/cert-manager-cainjector   1/1     1            1           24h
deployment.apps/cert-manager-webhook      1/1     1            1           24h

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/cert-manager-66d588bfc5            1         1         1       24h
replicaset.apps/cert-manager-cainjector-c89c4c45   1         1         1       24h
replicaset.apps/cert-manager-webhook-6c4bd6b66b    1         1         1       24h
```

### Now, we need a Cluster-Issuer that will make available the certificates to the whole server and including all namespaces:

### https://cert-manager.io/docs/configuration/acme/
```
vim clusterissuer.yaml
```
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: YOUR-EMAIL-HERE
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: letsencrypt-staging
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: nginx
---

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: YOUR-EMAIL-HERE
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: letsencrypt-prod
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: nginx
```

* the file above, we have two options. **Staginhg** and **Prod** and if you need to deploy it on test enrironment you should use **Staginhg**, but fell free to chose the best option for your case.

* Checking cluster issuer:
```
kubectl get clusterissuer
```
```
NAME                  READY   AGE
letsencrypt-prod      True    24h
letsencrypt-staging   True    24h
```
```
kubectl describe clusterissuer letsencrypt-prod
```
```
NAME                  READY   AGE
letsencrypt-prod      True    24h
letsencrypt-staging   True    24h
```

```
kubectl describe clusterissuer letsencrypt-prod
```
```
Name:         letsencrypt-prod
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  cert-manager.io/v1
Kind:         ClusterIssuer
Metadata:
  Creation Timestamp:  2023-04-26T14:51:55Z
  Generation:          1
  Managed Fields:
    API Version:  cert-manager.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        .:
        f:acme:
          .:
          f:email:
          f:privateKeySecretRef:
            .:
            f:name:
          f:server:
          f:solvers:
    Manager:      kubectl-client-side-apply
    Operation:    Update
    Time:         2023-04-26T14:51:55Z
    API Version:  cert-manager.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        .:
        f:acme:
          .:
          f:lastRegisteredEmail:
          f:uri:
        f:conditions:
          .:
          k:{"type":"Ready"}:
            .:
            f:lastTransitionTime:
            f:message:
            f:observedGeneration:
            f:reason:
            f:status:
            f:type:
    Manager:         cert-manager-clusterissuers
    Operation:       Update
    Subresource:     status
    Time:            2023-04-26T14:51:56Z
  Resource Version:  32884
  UID:               174e0495-37bf-4b9b-85ea-49d7e5329fc4
Spec:
  Acme:
    Email:            YOUR-EMAIL-HERE
    Preferred Chain:
    Private Key Secret Ref:
      Name:  letsencrypt-prod
    Server:  https://acme-v02.api.letsencrypt.org/directory
    Solvers:
      http01:
        Ingress:
          Class:  nginx
Status:
  Acme:
    Last Registered Email:  YOUR-EMAIL-HERE
    Uri:                    https://acme-v02.api.letsencrypt.org/acme/acct/1080703067
  Conditions:
    Last Transition Time:  2023-04-26T14:51:56Z
    Message:               The ACME account was registered with the ACME server
    Observed Generation:   1
    Reason:                ACMEAccountRegistered
    Status:                True
    Type:                  Ready
Events:                    <none>
```

* Checking for certificates:
```
kubectl get certificates.cert-manager.io
```
```
NAME                       READY   SECRET                     AGE
tls   True    tls   17h
gustavopereiranogueira@cloudshell:~$ kubectl get certificaterequests.cert-manager.io
NAME                             APPROVED   DENIED   READY   ISSUER             REQUESTOR                                         AGE
tls-qfxmn   True                True    letsencrypt-prod   system:serviceaccount:cert-manager:cert-manager   17h
```


* Consulting Secrets:

```
kubectl get secrets
```
```
NAME                       TYPE                DATA   AGE
tls   kubernetes.io/tls   2      17h

kubectl describe secrets
Name:         tls
Namespace:    default
Labels:       controller.cert-manager.io/fao=true
Annotations:  cert-manager.io/alt-names: YOUR-DOMAIN
              cert-manager.io/certificate-name: tls-YOUR-DOMAIN
              cert-manager.io/common-name: cloud.contaja.com.br
              cert-manager.io/ip-sans:
              cert-manager.io/issuer-group: cert-manager.io
              cert-manager.io/issuer-kind: ClusterIssuer
              cert-manager.io/issuer-name: letsencrypt-prod
              cert-manager.io/uri-sans:

Type:  kubernetes.io/tls

Data
====
tls.crt:  5603 bytes
tls.key:  1679 bytes
```


### We should install an Ingress as well, to read the rules prepared to redirect the requests to the right port. So, the Ingress will be  ready to use TLS protocol:

```
vim ingress.keycloak.yaml
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: keycloak
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
    - hosts:
        - YOUR-DOMAIN
      secretName: tls-YOUR-DOMAIN
  ingressClassName: nginx
  rules:
  - host: YOUR-DOMAIN
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: keycloak
            port:
              number: 8080
```

* After configuring it, let’s deploy it running the command below:

```
kubectl apply -f ingress.keycloak.yaml
```

* Checking the ingress:

```
kubectl get ingress
```

```
NAME       CLASS   HOSTS                  ADDRESS         PORTS     AGE
keycloak   nginx   your-domain   34.29.129.164   80, 443   17h
```

* Like above, we can see the same IP address from the **EXTERNAL-IP** from the Ingress-Controller.

* Now we definitely will deploy the Keycloak Application, running the manifesto below:

```
vim keycloak.yaml
```

```
---
apiVersion: v1
kind: Service
metadata:
  name: keycloak
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: keycloak
    app: keycloak
  ports:
    - protocol: TCP
      port: 8080
      name: http

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloak
  labels:
    app: keycloak
    app.kubernetes.io/name: keycloak
spec:
  replicas: 1
  selector:
    matchLabels:
      app: keycloak
  template:
    metadata:
      labels:
        app: keycloak
        app.kubernetes.io/name: keycloak
    spec:
      containers:
      - name: keycloak
        image: quay.io/keycloak/keycloak:18.0.2
        args: ["start-dev","--health-enabled=true", "--proxy=edge"]
        env:
        - name: KEYCLOAK_ADMIN
          value: "admin"
        - name: KEYCLOAK_ADMIN_PASSWORD
          value: "admin"
        - name: KEYCLOAK_LOGLEVEL
          value: "DEBUG"
        ports:
        - name: http
          containerPort: 8080
        readinessProbe:
          httpGet:
            path: /realms/master
            port: 8080
```       
