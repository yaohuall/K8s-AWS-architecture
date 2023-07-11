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

### Namespaces
**Namespaces** partition a Kubernetes cluster and are designed as an easy way to apply quotas and policies to groups of objects. They’re not designed for strong workload isolation.

Each Namespace can have its own users and RBAC rules, as well as resource quotas.

### Service
#### EndpointSlice objects
These hold the list of Pods that match the Service’s label selector and are dynamically updated as matching Pods come and go.
# AWS
### SSL/TLS certificate
AWS can't manage third-party certificate renewal automatically. You can send a notification to renew the 3rd party certificate. 

**AWS Config** has a managed rule named acm-certificate-expiration-check to check for expiring certs.
### AWS Textract
Amazon Textract 是一項符合 HIPAA 各項規定的服務。HIPAA 資格適用於提供 Amazon Textract 的所有 AWS 區域。這表示您可以使用 Amazon Textract 協助處理從影像中提取的受保護醫療資訊 (PHI)，為您的醫療保健應用程式提供技術支援。
### SQS
#### SQS with ASG
![image](https://github.com/yaohuall/K8s-AWS-playground/assets/109536074/b50567fa-1cf3-4d7e-833a-9cde4971bf8e)

### SQS vs SNS vs Kinesis
| Feature | SQS                                   | SNS                                       | Kinesis                                                                |
|---------|---------------------------------------|-------------------------------------------|------------------------------------------------------------------------|
|         | Consumer pull data                    | Push data to many subscribers             | Standard: pull data from Kinesis Push data, 2mb per shard per consumer |
|         | Data is deleted after being consumed  | Up to 12500000 subscribers, 100000 topics | Can replay data                                                        |
|         | Can have as many as consumers as want | Data lost if not delivered                | real-time big tat, analytics and ETL                                   |
|         | No need to provision throughput       | No need to provision throughput           | Data expires after X days                                              |
|         | Can FIFO                              | SNS FIFO with SQS FIFO                    | Ordering at the shard level                                            |
|         | Individual message delay capability   | Integrates with SQS for fan-out arch      | Provisioned mode or on-demand capacity mode                            |
#### Visibility timeout
During this time, the consumer processes and deletes the message. However, if the consumer fails before the message and system doesn't call the **DeleteMessage** action forthat message before the visibility timeout expires, the message becomes visible to other consumers and the message is received again.
### RDS
#### RDS Proxy
Serverless, autoscaling, high availability <br>
Reduce the stress on db res(CPU, RAM) & failover time
#### cloud & on-premise db restore options
1. Restoring a RDS/Aurora backup or a snapshot creates a new database
2. Restoring MySQL RDS database from S3
3. Restoring MySQL Aurora cluster from S3
4. Aurora database cloning: faster than snapshot & restore, the new db cluster uses the same cluster volume and data as the original but will change when data updates are made, fast & cost-effective. **Useful to create a staging database from a prod db without impacting the production db.
#### DB security
1. DB master & replicas encryption using AWS KMS - must be defined as lauch time. <br>
2. If the master is not encrypted, the read replicas cannot be encrypted. <br>
3. To encrypt an un-encrypted db, use DB snapshot & restore as encrypted.

#### Scheduling start and stop of RDS
![image](https://github.com/yaohuall/K8s-AWS-playground/assets/109536074/6b4ce992-2648-4720-9951-05f11f3808d1)

### DynamoDB
Point-in-time recovery: continuous backups, can restore that table to any point in time during the last 35 days. DynamoDB maintains incremental backups of your table.
### S3
With a gateway endpoint, you can access Amazon S3 from your VPC, without requiring an internet gateway or NAT device for your VPC, and **with no additional cost**. However, **gateway endpoints do not allow access from on-premises networks, from peered VPCs in other AWS Regions, or through a transit gateway. For those scenarios, you must use an interface endpoint, which is available for an additional cost**.
#### Non-IAM access
Create a VPC endpoint for S3 and add a bucket policy that allows access from the VPC endpoint. **VPC gateway endpoint** supports **S3 and DynamoDB** and is free.
#### Legal hold
The Object Lock governance mode: legal hold operation enables you to place a legal hold on an object version. Like setting a retention period, a legal hold prevents an object version from being overwritten or deleted. However, a legal hold doesn't have an associated retention period and remains in effect until removed.

Object Lock compliance mode: no users include **AWS root account** can modify objects.
#### Requester Pays
Share data but not incur charges associated with others accessing the data.
#### Prevent accidental delete
**MFA delete** force users to generate a code on a device before doing important operations(eg., permanently deleting) on S3.
#### S3 Event Notificaitons with Amazon EventBridge
1. Advanced filtering
2. Multiple destinations
3. EventBridge Capabilities
   
### Snowball family
**snowcone**: 14TB storage

### Cloudwatch
You can configure a **CloudWatch Logs** log group to stream data it receives to your AWS OpenSearch Service cluster in near real-time through a **CloudWatch Logs subscription**.

### AWS Global Accelerator vs CloudFront
Both use the AWS global network and its edge locations around the world. <br>
Both services integrate with AWS Shield for DDoS protection.

#### CloudFront
Improves performance for both cacheable content(imgs, videos) <br>
Content is served at the edge.<br>
Dynamic content(API acceleration and dynamic site delivery) <br>

#### Global Accelerator
Improves performance for apps over TCP or UDP <br>
Proxying packets at the edge to apps running in more AWS Regions. <br>
Good for non-HTTP use cases, such as gaming(UDP), IoT(MQTT). Good for HTTP use cases require static IP, fast regional failover

### Service control policy
AWS control tower guardrails & AWS organizations SCPS.

### AWS Direct Connection
Lead times are often longer than 1 month to establish a new connection. <br>
Real-time data transfering.



#### reference
**Nigel Poulton - The Kubernetes Book, 2023 Edition** <br>
https://aws.amazon.com <br>
https://aws.amazon.com/tw/blogs/database/schedule-amazon-rds-stop-and-start-using-aws-lambda/ <br>
https://aws.amazon.com/tw/about-aws/whats-new/2019/10/amazon-textract-is-now-a-hipaa-eligible-service/ <br>
https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
