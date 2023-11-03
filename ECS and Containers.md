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