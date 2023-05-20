### Kubernetes for Beginners

#### Docker
There are some advantages for Docker:
- Standardized application packaging
  - same packaging for all types of applications
- Features
  - Language neutral
  - cloud neutral
  - enables standardization
There are also some challenges with Docker:
- 1000 microservices
- 1000 instances

#### Creating Kubernetes cluster iwth Google Kubernetes Engine (GKE)
Kubernetes' architecture includes:
- Master Nodes (manages cluster)
- Worker Nodes (run your applications)
- inside a cluster

A cluster is the combination of Worker Nodes and the Master Node. The Master node ensures that the nodes are available
and are doing work.


### Kubernetes - Fun Facts
- K8S stands for Kubernetes
- Kubernetes means helmsman of a ship
- Kubernetes on Cloud
  - AKS - Azure Kubernetes Service
  - EKS - Elastic Kubernetes Service (AWS)
  - GKE - Google Kubernetes Engine

### Kubernetes cluster
To deploy a pod in my cluster I do the following commands after creating the cluster:
```bash
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to devops-rnd-383110.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ gcloud container clusters get-credentials in28minutes-cluster --region us-central1 --project devops-rnd-383110
Fetching cluster endpoint and auth data.
kubeconfig entry generated for in28minutes-cluster.
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.1", GitCommit:"4c9411232e10168d7b050c49a1b59f6df9d7ea4b", GitTreeState:"clean", BuildDate:"2023-04-14T13:21:19Z", GoVersion:"go1.20.3", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v5.0.1
Server Version: version.Info{Major:"1", Minor:"25", GitVersion:"v1.25.8-gke.500", GitCommit:"f117e29cb87cfb7e1de32ab4e163fb01ac5d0af9", GitTreeState:"clean", BuildDate:"2023-03-23T10:22:38Z", GoVersion:"go1.19.7 X:boringcrypto", Compiler:"gc", Platform:"linux/amd64"}
WARNING: version difference between client (1.27) and server (1.25) exceeds the supported minor version skew of +/-1
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.1.RELEASE
```

We then expose the deployment with:
```bash
kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080 
```
We can now access the application on the ingress port:
![image](https://github.com/TomSpencerLondon/LeetCode/assets/27693622/2c137020-c875-44f9-b7ca-1f762e225248)


