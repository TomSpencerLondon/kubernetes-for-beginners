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

### Cleaning up
In the cloud, it is best practice to delete the resources when we are not using them.

If you do not want to delete and create a new cluster every time, we can reduce the cluster size to zero.

After finishing our work for the day we can reduce cluster node size to zero.
```bash
gcloud container clusters resize --zone <name_of_zone> <name_of_your_cluster> --num-nodes=0
gcloud container clusters resize --zone us-central1 in28minutes-cluster --num-nodes=0
```
When we are ready to start again, increase the number of nodes:
```bash
gcloud container clusters resize --zone <name_of_zone> <name_of_your_cluster> --num-nodes=3
```

![image](https://github.com/TomSpencerLondon/kubernetes-for-beginners/assets/27693622/e099536d-7d47-4a6b-96ed-79e85580c197)

We can get events with:
```bash
kubectl get events
```

We can also get information about the cluster with the following:
```bash
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-world-rest-api-5b4c66787d-gdznr   1/1     Running   0          14m
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl get replicaset
NAME                              DESIRED   CURRENT   READY   AGE
hello-world-rest-api-5b4c66787d   1         1         1       14m
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl get service
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
hello-world-rest-api   LoadBalancer   10.50.128.184   104.197.69.62   8080:31534/TCP   13m
kubernetes             ClusterIP      10.50.128.1     <none>          443/TCP          3h43m
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl get deployment
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
hello-world-rest-api   1/1     1            1           15m
```
Each of the above commands have specific responsibilities.
This link is useful for kubernetes commands:
https://github.com/in28minutes/kubernetes-crash-course#commands-executed-during-the-course

### Understanding Pods in Kubernetes

Pods are the most important concept in Kubernetes. It is the smallest deployment unit in Kubernetes.
If I want to create a container in Kubernetes we have to use a Pod. The container lives inside a Pod.

```bash
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl get pods -o wide
NAME                                    READY   STATUS    RESTARTS   AGE   IP            NODE                                           NOMINATED NODE   READINESS GATES
hello-world-rest-api-5b4c66787d-gdznr   1/1     Running   0          19m   10.50.0.130   gk3-in28minutes-cluster-pool-1-bdc4a129-khq2   <none>           <none>
```
Each Pod has an IP address. Each Pod has a description of the number of containers in the Pod. All the containers in the Pod share resources.
Within the Pod containers can talk to each other on localhost. 

```bash
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl explain pods
KIND:       Pod
VERSION:    v1

DESCRIPTION:
    Pod is a collection of containers that can run on a host. This resource is
    created by clients and scheduled onto hosts.
    
FIELDS:
  apiVersion    <string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind  <string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata      <ObjectMeta>
    Standard object's metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec  <PodSpec>
    Specification of the desired behavior of the pod. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

  status        <PodStatus>
    Most recently observed status of the pod. This data may not be up to date.
    Populated by the system. Read-only. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
```
A Kubernetes node contains multiple Pods.

```bash
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl describe pod hello-world-rest-api-5b4c66787d-gdznr
Name:             hello-world-rest-api-5b4c66787d-gdznr
Namespace:        default
Priority:         0
Service Account:  default
Node:             gk3-in28minutes-cluster-pool-1-bdc4a129-khq2/10.128.0.16
Start Time:       Sat, 20 May 2023 21:56:39 +0000
Labels:           app=hello-world-rest-api
                  pod-template-hash=5b4c66787d
Annotations:      <none>
Status:           Running
SeccompProfile:   RuntimeDefault
IP:               10.50.0.130
IPs:
  IP:           10.50.0.130
Controlled By:  ReplicaSet/hello-world-rest-api-5b4c66787d
Containers:
  hello-world-rest-api:
    Container ID:   containerd://8b126d0874914162ce1525a5722badf3f7af688b42cfe6e34abe633c8ff38c6d
    Image:          in28min/hello-world-rest-api:0.0.1.RELEASE
    Image ID:       docker.io/in28min/hello-world-rest-api@sha256:00469c343814aabe56ad1034427f546d43bafaaa11208a1eb0720993743f72be
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 20 May 2023 21:57:31 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Requests:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Environment:          <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pqw2s (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-pqw2s:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              <none>
Tolerations:                 kubernetes.io/arch=amd64:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From                                   Message
  ----     ------            ----  ----                                   -------
  Warning  FailedScheduling  22m   gke.io/optimize-utilization-scheduler  0/2 nodes are available: 2 Insufficient cpu, 2 Insufficient memory. preemption: 0/2 nodes are available: 2 No preemption victims found for incoming pod.
  Normal   TriggeredScaleUp  22m   cluster-autoscaler                     pod triggered scale-up: [{https://www.googleapis.com/compute/v1/projects/devops-rnd-383110/zones/us-central1-b/instanceGroups/gk3-in28minutes-cluster-pool-1-bdc4a129-grp 0->1 (max: 1000)}]
  Normal   Scheduled         21m   gke.io/optimize-utilization-scheduler  Successfully assigned default/hello-world-rest-api-5b4c66787d-gdznr to gk3-in28minutes-cluster-pool-1-bdc4a129-khq2
  Normal   Pulling           20m   kubelet                                Pulling image "in28min/hello-world-rest-api:0.0.1.RELEASE"
  Normal   Pulled            20m   kubelet                                Successfully pulled image "in28min/hello-world-rest-api:0.0.1.RELEASE" in 11.129463978s (11.129792404s including waiting)
  Normal   Created           20m   kubelet                                Created container hello-world-rest-api
  Normal   Started           20m   kubelet                                Started container hello-world-rest-api
```

Pods provide a way to put containers together with labels and an IP address.

### Understanding Replicasets

Replicasets ensure a specific number of Pods are running:

```bash
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl get replicasets
NAME                              DESIRED   CURRENT   READY   AGE
hello-world-rest-api-5b4c66787d   1         1         1       25m

tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl get pods -o wide
NAME                                    READY   STATUS    RESTARTS   AGE   IP            NODE                                           NOMINATED NODE   READINESS GATES
hello-world-rest-api-5b4c66787d-gdznr   1/1     Running   0          27m   10.50.0.130   gk3-in28minutes-cluster-pool-1-bdc4a129-khq2   <none>           <none>
```

If we delete a pod a new pod will be automatically created. The Replicaset creates Pods to match the desired state.

To scale our deployment we can run:
```bash
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl scale deployment hello-world-rest-api --replicas=3
deployment.apps/hello-world-rest-api scaled
```

We can get information on a kubernetes cluser with the following:
```bash
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl get replicaset
NAME                              DESIRED   CURRENT   READY   AGE
hello-world-rest-api-5b4c66787d   3         3         3       31m
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl get replicaset
NAME                              DESIRED   CURRENT   READY   AGE
hello-world-rest-api-5b4c66787d   3         3         3       32m
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-world-rest-api-5b4c66787d-4hldr   1/1     Running   0          3m8s
hello-world-rest-api-5b4c66787d-865sb   1/1     Running   0          3m8s
hello-world-rest-api-5b4c66787d-gdznr   1/1     Running   0          32m
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl get events --sort-by=.metadata.creationTimestamp
```

We can ask kubernetes to explain replicasets:
```bash
tomspencerlondon@cloudshell:~ (devops-rnd-383110)$ kubectl explain replicaset
GROUP:      apps
KIND:       ReplicaSet
VERSION:    v1

DESCRIPTION:
    ReplicaSet ensures that a specified number of pod replicas are running at
    any given time.
```
Kubernetes protects us from errors in image upload:
```bash
tomspencerlondon@cloudshell:~$ kubectl set image deployment hello-world-rest-api hello-world-rest-api=DUMMY_IMAGE:TEST
deployment.apps/hello-world-rest-api image updated
```

The replicaset protects us because we have a desired state:
```bash
tomspencerlondon@cloudshell:~$ kubectl get rs -o wide
NAME                              DESIRED   CURRENT   READY   AGE   CONTAINERS             IMAGES                                       SELECTOR
hello-world-rest-api-5b4c66787d   3         3         3       10h   hello-world-rest-api   in28min/hello-world-rest-api:0.0.1.RELEASE   app=hello-world-rest-api,pod-template-hash=5b4c66787d
hello-world-rest-api-77b5fbc7     1         1         0       58s   hello-world-rest-api   DUMMY_IMAGE:TEST                             app=hello-world-rest-api,pod-template-hash=77b5fbc7

tomspencerlondon@cloudshell:~$ kubectl get pods
NAME                                    READY   STATUS             RESTARTS   AGE
hello-world-rest-api-5b4c66787d-4hldr   1/1     Running            0          10h
hello-world-rest-api-5b4c66787d-865sb   1/1     Running            0          10h
hello-world-rest-api-5b4c66787d-gdznr   1/1     Running            0          10h
hello-world-rest-api-77b5fbc7-b5ksf     0/1     InvalidImageName   0          103s
```

