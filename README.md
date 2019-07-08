# DevOps multi-container setup using kubernetes
Deploy static assets and the war file containing dynamic part of application separately.

## Prerequisites
Your system needs the `kubectl` ,`kubernetes cluster` &`helm`:
- [Kubernetes cluster on gcp](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster)
- [Install and Set Up Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [helm installation](https://helm.sh/docs/using_helm/)
- [Docker hub account](https://hub.docker.com/)


## Proposed solution
I'm building separate container for both static and dynamic part as per the requirement and will be deploying it over kubernetes Cluster.</br>
Using `Nginx` Ingress controller to tie both the static and dynamic containers.

### 1. Building & pushing the docker images
Clone the repository
```bash
    #Building the static docker image & prior to build we need to unzip the static files and copy it to web folder present under docker-static.
    docker login #Login/Authentication to Dockerhub account
    cd docker-static;
    docker build -t bobbyraj007/docker-static:thoughtworks-v3 .
    docker push bobbyraj007/docker-static:thoughtworks-v3
    
    #Building the dynamic docker image 
    #Copy the provided war file to docker-dynamic directory before building the docker image.
    cd docker-dynamic
    docker build -t bobbyraj007/tomcat-war:thoughtworks-v1 .
    docker push bobbyraj007/tomcat-war:thoughtworks-v1
```

> Note: I'm using the `tomcat base image` for deploying the war file, we can also use the `alpine` version of `tomcat` to make the container light-weight.

### 2. Deploying the above build docker images using kubernetes
Make sure `kubectl` is installed and `kubeconfig` file is configured for the targeted kubernetes cluster.
```bash
    #Create a namespace
    kubectl create ns java-app
    
    #Deploying the static container
    cd k8s
    kubectl apply -f docker-static.yaml -n java-app
    
    #Deploying the Dynamic Container
    kubectl apply -f tomcat-dynamic.yaml -n java-app
```
> Note:  we can create a helm chart for above kubernetes configurations.

 ### 3. install nginx ingress controller on the cluster
 
 we can leverage helm for installing Nginx ingress controller and it only takes couple of commands to setup `nginx ingress controller` .
 
 If your Kubernetes cluster has RBAC enabled, from the Cloud Shell, deploy an NGINX controller Deployment and Service by running the following command:
 ```bash
  helm install --name nginx-ingress stable/nginx-ingress --set rbac.create=true --set controller.publishService.enabled=true
  ```
 
 Deploy NGINX Ingress Controller with RBAC disabled.
 If your Kubernetes cluster has RBAC disabled, from the Cloud Shell, deploy an NGINX controller Deployment and Service by running the following command:
  ```bash
   helm install --name nginx-ingress stable/nginx-ingress
  ```

 In the output under RESOURCES, you should see the following:
 
 ``` bash
  ==> v1/Service
  NAME                           TYPE          CLUSTER-IP    EXTERNAL-IP  PORT(S)                     AGE
  nginx-ingress-controller       LoadBalancer  10.7.248.226  pending      80:30890/TCP,443:30258/TCP  1s
  nginx-ingress-default-backend  ClusterIP     10.7.245.75   none         80/TCP                      1s
  
 ```
 
### 4. Setup Nginx for static and dynamic container
```bash
    cd k8s
    kubectl apply -f ingress.yaml 
```

After setting up the ingress, it will provide the `loadbalancer url` which can be used for smoke testing and also we can configure ingress with the `tls` certificate to make it accessible over https. </br>
> Note: we can club the two ingress rules i.e. `styles` & `images` in one by keeping all the static content under one folder & updating the path in the codebase for war.