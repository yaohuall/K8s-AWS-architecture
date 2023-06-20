# K8s
## Control plane
A Kubernetes control plane node runs a collection of system services that make up the control plane of the cluster. For production environments, multiple control plane nodes configured for high availability (HA) are vital. Generally speaking, 3 or 5 is recommended, and you should spread them across availability zones.

⚡ **Control plane** contains **the API server**, **the cluster store**, **the controller manager and controllers**, **the scheduler**.
### The API server
All communication, between all components, must go through the API server. It exposes a RESTful API posting YAML configuration files to over HTTPS. The API server is the front-end into the control plane and all instructions and communication pass through it. By default, it exposes a RESTful endpoint on port 443.
### The cluster store
The cluster store is the only stateful part of the control plane and persistently stores the entire configuration and state of the cluster. The cluster store is currently based on etcd, a distributed db. Basically, you should run between 3-5 etcd replicas for high abailability.
### The controller manager and controllers
Controller manager is a controller of controllers, meaning it spawns allthe core controllers and monitors them. Controllers for example are Deployment controller, the StatefulSet controller or the ReplicaSet controller. Controller runs as a background watch-loop constantly watching the API Server for changes and ensure the observed state of the cluster matches the desired state.
### The scheduler
The scheduler watches the API server for new work tasks and assigns them to appropriate healthy worker nodes. The scheduler is only responsible for picking the nodes to run tasks, it isn’t responsible for running them. A task is normally a Pod/container.

## Worker node
1. Watch the API server for new work assignments
2. Execute work assignments
3. Report back to the control plane (via the API server) 

⚡ **Worker node** contains **Kubelet**, **Container runtime**, **Network proxy(kube-proxy)**.

Watch the API server for new work tasks and executes the task and maintains a reporting channel back to the control plane.
### Container runtime
The kubelet needs a container runtime to perform container-related tasks. This includes pulling images and starting and stopping containers. kubelet has a plugin model called the **Container Runtime Interface(CRI)** to mask the internal machinery of Kubernetes and exposes a clean documented interface for 3rd-party container runtimes to plug into.
### Kube-proxy
Runs on every node and is responsible for local cluster networking. It ensures each node gets its own unique IP address, and it implements local iptables or IPVS rules to handle routing and loadbalancing of traffic.

## Kubernetes DNS
Kubernetes cluster has an internal DNS service that is vital to service discovery. It has a static IP address that is hard-coded into every Pod on the cluster. Ensures each app can locate it and use it for discovery.

### Pod
A Pod is just a wrapper that allows a container to run on Kubernetes. The preferred model is to deploy all Pods via higher-level Deployment controller.<br>
All containers in the same Pod will share the same IP address (the Pod’s IP).
The deployment of a Pod is an atomic operation. This means a Pod is only ready for service when all its containers are up and running.
A single Pod can only be scheduled to a single node - Kubernetes cannot schedule a single Pod across multiple nodes.

### Declarative model
1. Declare the desired state of an application microservice in a manifest file
2. Post it to the API server
3. Kubernetes stores it in the cluster store as the application’s desired state
4. Kubernetes implements the desired state on the cluster
5. A controller makes sure the observed state of the application doesn’t vary from the desired state

## Service account
When Pods contact the API server, Pods authenticate as a particular ServiceAccount (for example, default). There is always at least one ServiceAccount in each namespace.

Every Kubernetes namespace contains at least one ServiceAccount: the default ServiceAccount for that namespace, named default. If you do not specify a ServiceAccount when you create a Pod, Kubernetes automatically assigns the ServiceAccount named default in that namespace.

You can fetch the details for a Pod you have created. For example:

    kubectl get pods/<podname> -o yaml
    
In the output, you see a field spec.serviceAccountName.

To list service accounts:

    kubectl get serviceaccounts

### API token for service account
You can get a time-limited API token for that ServiceAccount using kubectl:

    kubectl create token build-robot
The output from that command is a token that you can use to authenticate as that ServiceAccount.
# AWS
### SSL/TLS certificate
AWS can't manage third-party certificate renewal automatically. You can send a notification to renew the 3rd party certificate.
### AWS Textract
Amazon Textract 是一項符合 HIPAA 各項規定的服務。HIPAA 資格適用於提供 Amazon Textract 的所有 AWS 區域。這表示您可以使用 Amazon Textract 協助處理從影像中提取的受保護醫療資訊 (PHI)，為您的醫療保健應用程式提供技術支援。
### SQS
#### Visibility timeout
During this time, the consumer processes and deletes the message. However, if the consumer fails before the message and system doesn't call the **DeleteMessage** action forthat message before the visibility timeout expires, the message becomes visible to other consumers and the message is received again.
### RDS Proxy
Serverless, autoscaling, high availability <br>
Reduce the stress on db res(CPU, RAM) & failover time
#### reference
**Nigel Poulton - The Kubernetes Book, 2023 Edition** <br>
https://aws.amazon.com/tw/about-aws/whats-new/2019/10/amazon-textract-is-now-a-hipaa-eligible-service/ <br>
https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
