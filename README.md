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

tomspencerlondon@cloudshell:~$ kubectl describe pod hello-world-rest-api-77b5fbc7-b5ksf
Name:             hello-world-rest-api-77b5fbc7-b5ksf
Namespace:        default
Priority:         0
Service Account:  default
Node:             gk3-in28minutes-cluster-pool-1-f9bc7297-bw9c/10.128.0.17
Start Time:       Sun, 21 May 2023 08:47:04 +0000
Labels:           app=hello-world-rest-api
                  pod-template-hash=77b5fbc7
Annotations:      <none>
Status:           Pending
SeccompProfile:   RuntimeDefault
IP:               10.50.0.197
IPs:
  IP:           10.50.0.197
Controlled By:  ReplicaSet/hello-world-rest-api-77b5fbc7
Containers:
  hello-world-rest-api:
    Container ID:   
    Image:          DUMMY_IMAGE:TEST
    Image ID:       
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       InvalidImageName
    Ready:          False
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
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-w9f8s (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-w9f8s:
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
  Type     Reason         Age                   From                                   Message
  ----     ------         ----                  ----                                   -------
  Normal   Scheduled      2m39s                 gke.io/optimize-utilization-scheduler  Successfully assigned default/hello-world-rest-api-77b5fbc7-b5ksf to gk3-in28minutes-cluster-pool-1-f9bc7297-bw9c
  Warning  Failed         26s (x12 over 2m37s)  kubelet                                Error: InvalidImageName
  Warning  InspectFailed  12s (x13 over 2m37s)  kubelet                                Failed to apply default image tag "DUMMY_IMAGE:TEST": couldn't parse image reference "DUMMY_IMAGE:TEST": invalid reference format: repository name must be lowercase
```

The Deployment creates a new replicaset, but it failed to add one instance which means it is in failed set.
A valid update succeeds:
```bash
tomspencerlondon@cloudshell:~$ kubectl set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.2.RELEASE
deployment.apps/hello-world-rest-api image updated
tomspencerlondon@cloudshell:~$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-world-rest-api-8674b858bf-44gf7   1/1     Running   0          20s
hello-world-rest-api-8674b858bf-l9857   1/1     Running   0          7s
hello-world-rest-api-8674b858bf-n9r98   1/1     Running   0          14s
tomspencerlondon@cloudshell:~$ kubectl get rs
NAME                              DESIRED   CURRENT   READY   AGE
hello-world-rest-api-5b4c66787d   0         0         0       10h
hello-world-rest-api-77b5fbc7     0         0         0       6m5s
hello-world-rest-api-8674b858bf   3         3         3       37s
```
We have now deployed v2:
![image](https://github.com/TomSpencerLondon/kubernetes-for-beginners/assets/27693622/728206d7-df35-43ae-8ef6-378580229037)

The events show a successful update:
```bash
tomspencerlondon@cloudshell:~$ kubectl get events --sort-by=.metadata.creationTimestamp
LAST SEEN   TYPE      REASON              OBJECT                                       MESSAGE
8m9s        Normal    ScalingReplicaSet   deployment/hello-world-rest-api              Scaled up replica set hello-world-rest-api-77b5fbc7 to 1
8m9s        Normal    Scheduled           pod/hello-world-rest-api-77b5fbc7-b5ksf      Successfully assigned default/hello-world-rest-api-77b5fbc7-b5ksf to gk3-in28minutes-cluster-pool-1-f9bc7297-bw9c
8m9s        Normal    SuccessfulCreate    replicaset/hello-world-rest-api-77b5fbc7     Created pod: hello-world-rest-api-77b5fbc7-b5ksf
3m5s        Warning   InspectFailed       pod/hello-world-rest-api-77b5fbc7-b5ksf      Failed to apply default image tag "DUMMY_IMAGE:TEST": couldn't parse image reference "DUMMY_IMAGE:TEST": invalid reference format: repository name must be lowercase
5m56s       Warning   Failed              pod/hello-world-rest-api-77b5fbc7-b5ksf      Error: InvalidImageName
2m41s       Warning   FailedScheduling    pod/hello-world-rest-api-8674b858bf-44gf7    0/4 nodes are available: 2 Insufficient cpu, 4 Insufficient memory. preemption: 0/4 nodes are available: 4 No preemption victims found for incoming pod.
2m41s       Normal    SuccessfulCreate    replicaset/hello-world-rest-api-8674b858bf   Created pod: hello-world-rest-api-8674b858bf-44gf7
2m41s       Normal    ScalingReplicaSet   deployment/hello-world-rest-api              Scaled up replica set hello-world-rest-api-8674b858bf to 1 from 0
2m41s       Normal    ScalingReplicaSet   deployment/hello-world-rest-api              Scaled down replica set hello-world-rest-api-77b5fbc7 to 0 from 1
2m41s       Normal    SuccessfulDelete    replicaset/hello-world-rest-api-77b5fbc7     Deleted pod: hello-world-rest-api-77b5fbc7-b5ksf
2m39s       Normal    Scheduled           pod/hello-world-rest-api-8674b858bf-44gf7    Successfully assigned default/hello-world-rest-api-8674b858bf-44gf7 to gk3-in28minutes-cluster-pool-1-f9bc7297-bw9c
2m37s       Normal    Pulling             pod/hello-world-rest-api-8674b858bf-44gf7    Pulling image "in28min/hello-world-rest-api:0.0.2.RELEASE"
2m36s       Normal    Pulled              pod/hello-world-rest-api-8674b858bf-44gf7    Successfully pulled image "in28min/hello-world-rest-api:0.0.2.RELEASE" in 1.608462088s (1.60853043s including waiting)
2m36s       Normal    Created             pod/hello-world-rest-api-8674b858bf-44gf7    Created container hello-world-rest-api
2m35s       Normal    ScalingReplicaSet   deployment/hello-world-rest-api              Scaled down replica set hello-world-rest-api-5b4c66787d to 2 from 3
2m33s       Normal    Killing             pod/hello-world-rest-api-5b4c66787d-865sb    Stopping container hello-world-rest-api
2m35s       Normal    Started             pod/hello-world-rest-api-8674b858bf-44gf7    Started container hello-world-rest-api
2m35s       Normal    ScalingReplicaSet   deployment/hello-world-rest-api              Scaled up replica set hello-world-rest-api-8674b858bf to 2 from 1
2m35s       Normal    SuccessfulCreate    replicaset/hello-world-rest-api-8674b858bf   Created pod: hello-world-rest-api-8674b858bf-n9r98
2m35s       Normal    SuccessfulDelete    replicaset/hello-world-rest-api-5b4c66787d   Deleted pod: hello-world-rest-api-5b4c66787d-865sb
2m35s       Warning   FailedScheduling    pod/hello-world-rest-api-8674b858bf-n9r98    0/4 nodes are available: 2 Insufficient cpu, 4 Insufficient memory. preemption: 0/4 nodes are available: 4 No preemption victims found for incoming pod.
2m34s       Normal    Scheduled           pod/hello-world-rest-api-8674b858bf-n9r98    Successfully assigned default/hello-world-rest-api-8674b858bf-n9r98 to gk3-in28minutes-cluster-pool-1-bdc4a129-khq2
2m32s       Normal    Pulling             pod/hello-world-rest-api-8674b858bf-n9r98    Pulling image "in28min/hello-world-rest-api:0.0.2.RELEASE"
2m30s       Normal    Pulled              pod/hello-world-rest-api-8674b858bf-n9r98    Successfully pulled image "in28min/hello-world-rest-api:0.0.2.RELEASE" in 1.672359762s (1.67244215s including waiting)
2m30s       Normal    Created             pod/hello-world-rest-api-8674b858bf-n9r98    Created container hello-world-rest-api
2m29s       Normal    Started             pod/hello-world-rest-api-8674b858bf-n9r98    Started container hello-world-rest-api
2m28s       Normal    Killing             pod/hello-world-rest-api-5b4c66787d-gdznr    Stopping container hello-world-rest-api
2m28s       Normal    SuccessfulCreate    replicaset/hello-world-rest-api-8674b858bf   Created pod: hello-world-rest-api-8674b858bf-l9857
2m28s       Warning   FailedScheduling    pod/hello-world-rest-api-8674b858bf-l9857    0/4 nodes are available: 2 Insufficient cpu, 4 Insufficient memory. preemption: 0/4 nodes are available: 4 No preemption victims found for incoming pod.
2m28s       Normal    SuccessfulDelete    replicaset/hello-world-rest-api-5b4c66787d   Deleted pod: hello-world-rest-api-5b4c66787d-gdznr
2m28s       Normal    ScalingReplicaSet   deployment/hello-world-rest-api              Scaled down replica set hello-world-rest-api-5b4c66787d to 1 from 2
2m28s       Normal    ScalingReplicaSet   deployment/hello-world-rest-api              Scaled up replica set hello-world-rest-api-8674b858bf to 3 from 2
2m27s       Normal    Scheduled           pod/hello-world-rest-api-8674b858bf-l9857    Successfully assigned default/hello-world-rest-api-8674b858bf-l9857 to gk3-in28minutes-cluster-pool-1-bdc4a129-khq2
2m25s       Normal    Created             pod/hello-world-rest-api-8674b858bf-l9857    Created container hello-world-rest-api
2m25s       Normal    Pulled              pod/hello-world-rest-api-8674b858bf-l9857    Container image "in28min/hello-world-rest-api:0.0.2.RELEASE" already present on machine
2m24s       Normal    Started             pod/hello-world-rest-api-8674b858bf-l9857    Started container hello-world-rest-api
2m22s       Normal    Killing             pod/hello-world-rest-api-5b4c66787d-4hldr    Stopping container hello-world-rest-api
2m22s       Normal    SuccessfulDelete    replicaset/hello-world-rest-api-5b4c66787d   Deleted pod: hello-world-rest-api-5b4c66787d-4hldr
2m22s       Normal    ScalingReplicaSet   deployment/hello-world-rest-api              Scaled down replica set hello-world-rest-api-5b4c66787d to 0 from 1
```
The deployment uses rolling updates. It adds one pod at a time and increases and decreases pods as the operations are successful. Replica Sets ensure that the desired number of pods are always running.
Replica Sets are tied to specific versions. 

#### Understanding Services in Kubernetes
To understand the usefulness of services we can look at our pods:
```bash
tomspencerlondon@cloudshell:~$ kubectl get pods -o wide
NAME                                    READY   STATUS    RESTARTS   AGE   IP            NODE                                           NOMINATED NODE   READINESS GATES
hello-world-rest-api-8674b858bf-44gf7   1/1     Running   0          32m   10.50.0.198   gk3-in28minutes-cluster-pool-1-f9bc7297-bw9c   <none>           <none>
hello-world-rest-api-8674b858bf-l9857   1/1     Running   0          32m   10.50.0.138   gk3-in28minutes-cluster-pool-1-bdc4a129-khq2   <none>           <none>
hello-world-rest-api-8674b858bf-n9r98   1/1     Running   0          32m   10.50.0.137   gk3-in28minutes-cluster-pool-1-bdc4a129-khq2   <none>           <none>
```
We can delete a pod:
```bash
tomspencerlondon@cloudshell:~$ kubectl delete pod hello-world-rest-api-8674b858bf-n9r98
pod "hello-world-rest-api-8674b858bf-n9r98" deleted
```

The pod is deleted. We don't want the url to change. The service allows us to delete and create pods without changing the url.
The pod is throwaway but the service remains irrespective of the changes to the pods. The service was created when we exposed the
deployment. We are using a loadbalancer service.

```bash
tomspencerlondon@cloudshell:~$ kubectl get services
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
hello-world-rest-api   LoadBalancer   10.50.128.184   104.197.69.62   8080:31534/TCP   18h
kubernetes             ClusterIP      10.50.128.1     <none>          443/TCP          21h
```
The cluster IP service can only be accessed from within the cluster. 


### Understanding Kubernetes Architecture - Master Node and Nodes
The distributed database for the master node is kept in etcd. The API server
is kept in kube-apiserver. The kube-scheduler and the kube-controller-manager are also part of the master node.
The kubernetes cluster state is kept in the etcd distributed database. Kubectl speaks to the cluster through the API
server. The scheduler is responsible for scheduling the nodes on the cluster. The controller manager manages the overall
health of the cluster. The kube-controller-manager executes the desired state of the cluster: 

![image](https://github.com/TomSpencerLondon/kubernetes-for-beginners/assets/27693622/52bf2b05-a0c3-4c44-8bfc-12dbe47e37d7)

The worker nodes:
![image](https://github.com/TomSpencerLondon/kubernetes-for-beginners/assets/27693622/0d1db462-5d8a-4e35-9d06-9e4ebed6559c)

In a single node we might have several pods. The node agent reports the state of a pod to the controller manager.
The netwroking component helps with exposing services for the nodes. The container runtime provides the environment for running
the pods.

Both the master nodes and the worker nodes are kept in the cluster. If a master node goes down the applications will continue working.
The master node is not in charge of access to the url. We wouldn't be able to make changes.

```bash
tomspencerlondon@cloudshell:~$ kubectl get componentstatuses
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                  
scheduler            Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   
etcd-1               Healthy   {"health":"true"}   
```

#### Understanding google cloud regions and zones
This link shows the available regions and zones for google cloud:
https://cloud.google.com/about/locations

### Using Kubernetes and Docker with Spring Boot Hello World Rest API

This is the dockerfile for our hello-world-rest-api:
````dockerfile
FROM openjdk:8-jdk-alpine
VOLUME /tmp
EXPOSE 8080
ADD target/*.jar app.jar
ENTRYPOINT [ "sh", "-c", "java -jar /app.jar" ]
````

We build our application with:
```bash
mvn clean install
```
This also builds the image with this configuration:
```xml
	<build>
		<finalName>hello-world-rest-api</finalName>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			<!-- Docker -->
			<plugin>
				<groupId>com.spotify</groupId>
				<artifactId>dockerfile-maven-plugin</artifactId>
				<version>1.4.8</version>
				<executions>
					<execution>
						<id>default</id>
						<goals>
							<goal>build</goal>
							<!-- <goal>push</goal> -->
						</goals>
					</execution>
				</executions>
				<configuration>
					<repository>tomspencerlondon/${project.name}</repository>
					<tag>${project.version}</tag>
<!--					<skipDockerInfo>true</skipDockerInfo>-->
				</configuration>
			</plugin>

		</plugins>

	</build>
```

Our image is built:
```bash
tom@tom-ubuntu:~/Projects/kubernetes-for-beginners$ docker images
REPOSITORY                     TAG              IMAGE ID       CREATED          SIZE
in28min/hello-world-rest-api   0.0.4-SNAPSHOT   a8a4573385cd   45 seconds ago   122MB
```
We can run the docker image with:
```bash
docker run -p 8080:8080 in28min/hello-world-rest-api:0.0.4-SNAPSHOT 
```

![image](https://github.com/TomSpencerLondon/kubernetes-for-beginners/assets/27693622/347d2b9e-a149-473c-8e7f-dd4372f80141)

We need to build the image with our repository name and then we can push the image to docker hub.
We add our repository name to the repository section of the build docker image step above and then we can push the image to dockerhub.

```bash
tom@tom-ubuntu:~/Projects/kubernetes-for-beginners$ docker push tomspencerlondon/hello-world-rest-api:0.0.4-SNAPSHOT
```
![image](https://github.com/TomSpencerLondon/kubernetes-for-beginners/assets/27693622/6ece2257-bbfc-4723-8ea9-bd569f6fee22)

### Using gcloud
We need gcloud and kubectl to deploy our application from the command line on our machine.
To login to gcloud we run:
```bash
tom@tom-ubuntu:~$ gcloud auth login
```
We can now connect to the cluster we created earlier:
```bash
tom@tom-ubuntu:~$ gcloud container clusters get-credentials in28minutes-cluster --region us-central1 --project devops-rnd-383110
Fetching cluster endpoint and auth data.
kubeconfig entry generated for in28minutes-cluster.
```





### Deploy 01 Spring Boot Hello World Rest API to Kubernetes

We can create our deployment:

We can create our cluster and then connect to the cluster. We can then create our deployment and get the external IP address
which we set with our loadbalancer:

```bash
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl create deployment hello-world-rest-api --image=tomspencerlondon/hello-world-rest-api:0.0.4-SNAPSHOT
Warning: Autopilot set default resource requests for Deployment default/hello-world-rest-api, as resource requests were not specified. See http://g.co/gke/autopilot-defaults
deployment.apps/hello-world-rest-api created
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080 
service/hello-world-rest-api exposed
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl get services
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-world-rest-api   LoadBalancer   10.50.128.237   <pending>     8080:30476/TCP   16s
kubernetes             ClusterIP      10.50.128.1     <none>        443/TCP          26m
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl get services --watch
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
hello-world-rest-api   LoadBalancer   10.50.128.237   34.70.221.222   8080:30476/TCP   65s
kubernetes             ClusterIP      10.50.128.1     <none>          443/TCP          27m


```
We can then access the IP:
![image](https://github.com/TomSpencerLondon/kubernetes-for-beginners/assets/27693622/ae429055-712c-45da-874b-4a896613ba8c)

We can scale the amount of pods with:
```bash
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl scale deployment hello-world-rest-api --replicas=3
deployment.apps/hello-world-rest-api scaled
```
We can record the changes for our rollouts with --record:

```bash
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl rollout history deployment hello-world-rest-api
deployment.apps/hello-world-rest-api 
REVISION  CHANGE-CAUSE
1         <none>

tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl rollout status deployment hello-world-rest-api
deployment "hello-world-rest-api" successfully rolled out
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl set image deployment hello-world-rest-api hello-world-rest-api=tomspencerlondon/hello-world-rest-api:0.0.4-SNAPSHOT --record
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/hello-world-rest-api image updated
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl rollout history deployment hello-world-rest-api
deployment.apps/hello-world-rest-api 
REVISION  CHANGE-CAUSE
1         kubectl set image deployment hello-world-rest-api hello-world-rest-api=tomspencerlondon/hello-world-rest-api:0.0.4-SNAPSHOT --record=true

```
We can undo deployments:
```bash
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl rollout undo deployment hello-world-rest-api --to-revision=1
deployment.apps/hello-world-rest-api skipped rollback (current template already matches revision 1)
```

We can review the logs of the application:
```bash
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-world-rest-api-6649f4796d-m46xn   1/1     Running   0          5m58s
hello-world-rest-api-6649f4796d-n67fx   1/1     Running   0          5m58s
hello-world-rest-api-6649f4796d-pwbcf   1/1     Running   0          5h54m
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl logs hello-world-rest-api-6649f4796d-m46xn
```
We can watch the url with:
```bash
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ watch curl http://34.70.221.222:8080/hello-world
```

We can get a yaml file with the details of the deployment:
```bash
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl get deployment hello-world-rest-api -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    autopilot.gke.io/resource-adjustment: '{"input":{"containers":[{"name":"hello-world-rest-api"}]},"output":{"containers":[{"limits":{"cpu":"500m","ephemeral-storage":"1Gi","memory":"2Gi"},"requests":{"cpu":"500m","ephemeral-storage":"1Gi","memory":"2Gi"},"name":"hello-world-rest-api"}]},"modified":true}'
    deployment.kubernetes.io/revision: "1"
    kubernetes.io/change-cause: kubectl set image deployment hello-world-rest-api
      hello-world-rest-api=tomspencerlondon/hello-world-rest-api:0.0.4-SNAPSHOT --record=true
  creationTimestamp: "2023-05-21T17:37:34Z"
  generation: 3
  labels:
    app: hello-world-rest-api
  name: hello-world-rest-api
  namespace: default
  resourceVersion: "266091"
  uid: f9d60db0-7d06-4175-b4b1-0a829770037b
spec:
  progressDeadlineSeconds: 600
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: hello-world-rest-api
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: hello-world-rest-api
    spec:
      containers:
      - image: tomspencerlondon/hello-world-rest-api:0.0.4-SNAPSHOT
        imagePullPolicy: IfNotPresent
        name: hello-world-rest-api
        resources:
          limits:
            cpu: 500m
            ephemeral-storage: 1Gi
            memory: 2Gi
          requests:
            cpu: 500m
            ephemeral-storage: 1Gi
            memory: 2Gi
        securityContext:
          capabilities:
            drop:
            - NET_RAW
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        seccompProfile:
          type: RuntimeDefault
      terminationGracePeriodSeconds: 30
      tolerations:
      - effect: NoSchedule
        key: kubernetes.io/arch
        operator: Equal
        value: amd64
status:
  availableReplicas: 3
  conditions:
  - lastTransitionTime: "2023-05-21T17:37:34Z"
    lastUpdateTime: "2023-05-21T17:39:27Z"
    message: ReplicaSet "hello-world-rest-api-6649f4796d" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  - lastTransitionTime: "2023-05-21T23:28:28Z"
    lastUpdateTime: "2023-05-21T23:28:28Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  observedGeneration: 3
  readyReplicas: 3
  replicas: 3
  updatedReplicas: 3

```
We can also save the information in a yaml file:

```bash
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl get deployment hello-world-rest-api -o yaml > deployment.yaml
```
We can also save information about the service:
```bash
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl get service hello-world-rest-api -o yaml > service.yaml
```

We can then change the replicas in the yaml file and apply the changes:
```bash
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl apply -f deployment.yaml
Warning: resource deployments/hello-world-rest-api is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/hello-world-rest-api configured
tom@tom-ubuntu:~/Projects/kubernetes-crash-course/01-hello-world-rest-api$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-world-rest-api-6649f4796d-n67fx   1/1     Running   0          14m
hello-world-rest-api-6649f4796d-pwbcf   1/1     Running   0          6h3m
```

