# Key Notes

## Intro to Containers

* Container images are made of layers. They are all read-only. Modifications create new layers.
* Running containers have the same layers as the corresponding container image PLUS a RW layer, where runtime data is localized.
* Dockerfiles are used to build images.
* Containers are portable and self-contained. Always run as expected.
* They are lightweight. The OS is the parent OS, they are only processes and fs layers are shared.
* Ports are exposed to the host and beyond
* Provide the isolation and only contains what the application needs.

## ECS Concepts

* After you first defined a cluster you have to define the following (bottom to top):
    - A container definition: contains container image and exposed ports.
    - A task definition (similar to pods): contains container(s) definition(s) to use, resources to apply (CPU, MEM) and has a task role.
    - The task role specifies what can resources in the task (containers) can do within AWS.
    - A Service wraps around a task and specifies the number of copies for a task, the HA options, restart policy, etc.

## ECS Cluster Modes

- In both modes, the ECS service is responsible for SchedulingAndOrchoestration, ClusterManager and PlacementEngine.
- EC2 Mode:
    - Deploy EC2 hosts as an ECS cluster in your VPC. Deploys them as an ASG where you control the specs.
    - Services / Tasks are then deployed in your EC2 instances and images pulled from ECR.
    - You manage the EC2 hosts and it comes with wasted resource if you don't use it.
- Fargate Mode:
    - Containers deployed in a shared Fargate Infrastructure that you don't see or manage.
    - For each resource scheduled, an ENI is deployed in your VPC. So resources are deployed outside of your infra but injected in yours via ENI.
    - You don't manage the infra and only pay for the running tasks and their resources.
- If you have large workload and is price conscious: use EC2 mode (can better control EC2 usage, use reserved instance or spot, etc.)
- If you have large workload and is overhead conscious: use Fargate.
- If you have small/burst load: Fargate (no need to schedule empty EC2 80% of the time).
- If you have batch/periodic load: Fargate.

## Elastic Container Registry (ECR)

- Managed container registry service. Each AWS account has a public and private registry.
- A registry can have many repositories and a repository can have many images which can have many tags.
- Public registry is R/O for everyone. R/W requires permissions.
- Private registry requires permissions for both R/O and R/W.
- Benefits of ECR include:
    - Integrated with IAM wrt permissions
    - Image scanning (basic and enhanced modes) for vulnerabilities
    - Near RT metrics to cloudwatch (push/pull/auth), API actions (cloudtrail) and events (eventsbridge)
    - Can enable replication (cross region and cross account)

## Kubernetes 101 Summary

- Cluster: a deployment of Kubernetes, management, orchestration, ...
- Node: Resources; pods are placed on nodes to run.
- Pod: 1+ containers; smallest unit in K8S, often 1 container, 1 pod.
- Service: Abstraction, serving running on 1 or more pods.
- Job: Ad-hoc, creates one or more pods until completion.
- Ingress: exposes a way into a service (Ingress=>routing=>service=>1+pods)
- Ingress Controller: used to provide ingress (ex. AWS LB Controller uses ALN/NLB)
- Persistent Storage (PV): volume whose lifecycle lives beyond any 1 pod using it

## EKS 101

- AWS Managed K8S: open source & cloud agnostic
- Can be run on AWS, Outposts, EKS anywhere, EKS Distro
- The Control plane is entirely managed by AWS and scales on multiple AZ.
- Integration with AWS services: ELB, ECR, IAM, VPC,...
- EKS Cluster: EKS Control pane + nodes
- etcd managed and distributed accross mutliple AZs.
- Nodes are either self-managed, managed node groups or fargate pods: Need to check the workload requirements as disponibilities vary.