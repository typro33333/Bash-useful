This file will save all command will use in k8s (kubernet).

## 1. Introduction kubernet (k8s)

Kubernetes is an open source container orchestration engine for automating deployment, scaling, and management of containerized applications. The open source project is hosted by the Cloud Native Computing Foundation (CNCF).

![image](https://user-images.githubusercontent.com/62330937/179439305-f2aaeda5-cfad-47ea-85a7-7b15c771c6e1.png)

- `Master Server` is the main machine of the cluster, where it controls the whole cluster.

- `etct` is the basic component required for Kubernetes, it stores the common configuration for the whole cluster, etct runs at the master machine. etct is an open source project (see etcd) that provides key/value pair storage

- `kube-apiserver` runs on the master machine, providing Restful APIs for clients (like kubectl) to interact with Kubernetes

- `kube-scheduler` runs at master, this component helps to choose which Node to run applications based on resources and other components so that the system is stable.

- `kube-controller` runs at master, it controls cluster state, interacts to perform tasks of creating, deleting, updating ... resources

- `Kubelet service` runs on all machines (Node), it takes care of running, stopping, maintaining applications running on its node.

- `Kube-proxy`: provides a proxy network for applications to receive traffic from outside the network into the cluster.

## 2. Kubectl in kubenets (k8s)

Kubectl as kubenet-controller to manage and minitoring k8s runtime. To more information you can call it by:

- `kubectl --help`

But you run k8s by another resource like: `minikube`, `kubeadm`, `kubekey` you will typing so along e.g: `minikube kubectl --`, `kubeadm `... To compact it make your life is better
command this:

- `alias k="(minikube| kubeadm| kubekey) kubectl --"`

Now you can run with k --help instead above command.

### Kubectl work with pod



