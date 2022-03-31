## Deploying Spring Boot Application in Production Kubernetes

# Introduction
Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can “just run”. Recently the fantastic team at Spring have been adding useful features so that repetitive typical features that are retaliated to packaging and running the application are streamlined.

Creating an image which is compatible with Open Container Initiative is as simple as running mvn spring-boot:build-image, and the image will be built using a cloud native buildpack.

As we want to run our images in a production kubernetes cluster, we need to a few more files to our code and do some preparations on the cluster so that we can run the images as deployments in the k8s cluster.

# Adding manifest files
To run application in k8s we need a deployment, service and an ingress file.

We can create a new folder in the root of the project, at the level of src folder, and call it as manifests. Then inside this folder we can make all of our files.

In my case the application is called monitoring.
```
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: monitoring
  template:
    metadata:
      labels:
        app: monitoring
    spec:
      containers:
        - image: ghcr.io/manitaggarwal/monitoring:latest
          imagePullPolicy: Always
          name: monitoring
          ports:
            - containerPort: 8080
      imagePullSecrets:
        - name: regcred
```
```
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: monitoring
spec:
  type: ClusterIP
  selector:
    app: monitoring
  ports:
    - port: 8080
      targetPort: 8080
```
```
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: monitoring
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  namespace: default
spec:
  tls:
    - hosts:
        - subdomain.example.com
      secretName: monitoring-tls
  rules:
    - host: subdomain.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: monitoring
                port:
                  number: 8080
```
# Preparing k8s cluster
As we want to reach our service and use a domain name to access it we will need an ingress controller. I have another post where I have detailed how to install ingress-nginx, in more detail.

To install ingress-nginx Ingress Nginx in itself will not give us the ability to generate SSL certificates, for that we need cert manager.
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx
```
To install cert manager
```
kubectl create namespace cert-manager
helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.0.4 --set installCRDs=true
```
To add Cert Issuer
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
  namespace: cert-manager
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: manit@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```
# Preparing pipeline
We could manually apply the manifests to achieve our goal, but I am using Github Actions so that when I push to git repo the runner picks up the changes and builds and deploys the images to the desired k8s cluster.
```
name: Java CI with Maven

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  IMAGE_NAME: ghcr.io/manitaggarwal/monitoring:${{ github.sha }}

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'
    - name: Login to GitHub Container Registry
      uses: docker/login-action@v1
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    - name: Build with Maven
      run: | 
        mvn spring-boot:build-image
        docker tag monitoring:0.0.1-SNAPSHOT ${{ env.IMAGE_NAME }}
    - name: Push to Registry
      run: |
        docker push ${{ env.IMAGE_NAME }}
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: Azure/k8s-set-context@v1
      with:
        kubeconfig: ${{ secrets.KUBE_CONFIG }}
    - name: Run on Cluster
      run: |
        kubectl apply -f manifests/
        kubectl set image deployment/monitoring monitoring=${{ env.IMAGE_NAME }}
```
## Sources
[Pulling from private registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

[Cert manager](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes)