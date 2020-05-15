Supported Infrastructures
OpenShift supported on RHEL and RHCOS

OpenShift 4.1 supported on AWS, VMware, and bare metal

Future releases to support more infrastructures

OpenShift 3.11 supported anywhere RHEL runs:

On bare-metal physical machines or virtualized infrastructure, and in private or certified public clouds

Includes all supported virtualization platforms: Red Hat Virtualization, vSphere, Hyper-V

On Red Hat OpenStack® Platform and certified public cloud providers like Amazon, Google, Azure

x86 and IBM Power server architectures supported

Multi-cluster hybrid approach supports deploying OpenShift clusters on all of these infrastructures and federating them

Node Host
OpenShift runs on RHEL and RHCOS
OpenShift has two types of nodes:
    Workers
    Masters

Nodes are:
    Instances of RHEL or RHCOS with OpenShift installed
    Where end-user applications run
    Orchestrated by master nodes
    node daemon and other software run on all nodes

Containers
Application instances and components run in OCI-compliant containers
OpenShift worker node can run many containers
Node capacity is related to memory and CPU capabilities of underlying resources
    Hardware or virtualized

Pod
One or more containers deployed together on one host
    Consists of colocated group of containers with shared resources such as volumes and IP addresses
    Smallest compute unit that can be defined, deployed, managed

May contain one or more tightly coupled, colocated applications, ones that run with shared context
Example: Web server and application to pull and sync files

    Models application-specific logical host in containerized environment
    In pre-container world, applications executed on same physical or virtual host
    In OpenShift, pods replace individual application containers as smallest deployable unit

Pods are Orchestrated unit in OpenShift
    OpenShift schedules and runs all containers in pod on same node
Complex applications made up of many pods, each with own containers
    Interact externally and also with one another inside OpenShift environment

OpenShift runs container images in containers wrapped by meta object called "pod"
Possible to have multiple containers in single pod
    Example: To support cluster features as sidecar containers
Most applications benefit from flexibility of single-container pod:
    Different components such as application server and database generally not placed in single pod
    Allows for individual application components to be easily scaled horizontally
Application components are wired together by services.

Service
Defines logical set of pods and access policy
    Provides permanent internal IP address and host name for other applications to use as pods are created and destroyed
Service layer connects application components together
    Example: Front-end web service connects to database instance by communicating with its service
Services allow simple internal load balancing across application components
OpenShift automatically injects service information into running containers for ease of discovery

Labels
Used to organize, group, or select API objects
    Example: Tag pods with labels, services use label selectors to identify pods they proxy to
    Makes it possible for services to reference groups of pods
    Able to treat pods with potentially different containers as related entities
Most objects able to include labels in metadata
Group arbitrarily related objects with labels
    Example: All pods, services, replication controllers, and deployment configurations of application

Master Nodes
Instances of RHCOS
Primary functions:
    Orchestrate all activities on worker nodes
    Know and maintain state within OpenShift environment
Use multiple masters for high availability

Master - API and Authentication
Masters provide single API that all tooling and systems interact with
    All administration requests goes through this API
All API requests SSL-encrypted and authenticated
Authorizations handled via fine-grained role-based access control (RBAC)
Masters can be tied into external identity management systems
    Examples: LDAP, Active Directory, OAuth providers like GitHub and Google

Access
All users access OpenShift through same standard interfaces
Web UI, CLI, IDEs all go through authenticated, RBAC-controlled API
Users do not need system-level access to OpenShift nodes
CI/CD systems integrate with OpenShift through these interfaces
When OpenShift is deployed on RHCOS enables use of next-generation systems management and monitoring tools
When OpenShift is deployed on RHEL enables use of existing systems management and monitoring tools

etcd
etcd is another critical component of the OpenShift architecture. 
It is the distributed key-value data store for state and other information within the OpenShift environment.
etcd also holds things like RBAC rules, application environment information, and non-application user data.

The OpenShift masters are capable of monitoring application health via user-defined pod probes. 
Users configure pod probes for liveness and readiness. 
The masters scale out pods based on CPU utilization metrics.

Remediating Pod Failures
Masters automatically restart pods that fail probes or exit due to container crash
Pods that fail too often marked as bad and temporarily not restarted
Service layer sends traffic only to healthy pods
    Masters automatically orchestrate this to maintain component availability

Scheduler
Component responsible for determining pod placement
Accounts for current memory, CPU, and other environment utilization when placing pods on worker nodes
For application high availability, spreads pod replicas between worker nodes

Integrated Container Registry
OpenShift Container Platform includes integrated container registry to store and manage container images

When new image pushed to registry, registry notifies OpenShift and passes along image information including:
    Namespace
    Name
    Image metadata

Various OpenShift components react to new image by creating new builds and deployments


Service Broker and Service Catalog

When developing microservices-based applications to run on cloud-native platforms, 
there are many ways to provision different resources and share their coordinates, 
credentials, and configuration, depending on the service provider and the platform.
To give developers a more seamless experience, OpenShift Container Platform includes
a service catalog, an implementation of the Open Service Broker API (OSB API) for Kubernetes. 
This allows users to connect any of their applications deployed in OpenShift Container Platform
to a wide variety of service brokers.
The service catalog allows cluster administrators to integrate multiple platforms using
a single API specification. The OpenShift Container Platform web console displays the cluster
service classes offered by service brokers in the service catalog, allowing users to discover 
and instantiate those services for use with their applications.
As a result, service users benefit from ease and consistency of use across different types
of services from different providers, while service providers benefit from having one integration
point that gives them access to multiple platforms.

Service Broker and Service Catalog
Many ways to provision resources and share coordinates, credentials, and configuration,
depending on service provider and platform
OpenShift Container Platform includes service catalog
    Based on Open Service Broker API (OSB API) for Kubernetes
    Connect any application deployed in OpenShift Container Platform to variety of service brokers
Service catalog allows cluster administrators to integrate multiple platforms using single API specification
Web console displays cluster service classes offered by service brokers in service catalog
    Easily discover and instantiate service for use with applications

Application Data
Containers natively ephemeral
    Data not saved when containers restarted or created
OpenShift provides persistent storage subsystem that automatically connects real-world storage to correct pods
    Allows use of stateful applications

OpenShift Container Platform provides wide array of persistent storage types including:
    Raw devices: iSCSI, Fibre Channel
    Enterprise storage: NFS
    Cloud-type options: Gluster®/Ceph®, AWS EBS, pDisk

Routing Layer
Provides external clients access to applications running inside OpenShift
Close partner to service layer
Runs in pods inside OpenShift
Provides:
    Automated load balancing to pods for external clients
    Load balancing and auto-routing around unhealthy pods
Routing layer pluggable and extensible
    Options include hardware or non-OpenShift software routers

Replication Controller
The job of a replication controller is to ensure that a specified number of pod replicas are running at all times. If pods exit or are deleted, the replication controller acts to instantiate more pods up to the specified number. If there are more pods running than needed, the replication controller deletes as many pods as necessary to match the specified number of replicas.

Definition includes:
    Number of replicas desired (adjustable at any time)
    Pod definition for creating replicated pod
    Selector for identifying managed pods

Selector is set of labels assigned to all pods managed by replication controller
    Included in pod definition that replication controller instantiates
    Used by replication controller to determine how many pod instances are running, to adjust as needed

Does not include:
    Performing auto-scaling based on load or traffic
    Tracking load or traffic

Kubernetes Operators
A Kubernetes native application
    Puts all operational knowledge into Kubernetes primitives
    Administrators, shell scripts and automation software (eg. Ansible®) now in Kubernetes pods
    Integrates natively with Kubernetes concepts and APIs

Design
Operators:
    Are pods with operator code that interact with Kubernetes API server
    Run "reconciliation loops" to check on application service
        Make sure user-specified state of objects is achieved
    Manage all resources deployed and your application
    Act as application-specific controllers
    Extend Kubernetes API with Custom Resource Definition (CRD)

Operator Framework
Operator SDK
    Developers build, package and test an operator
    No knowledge of Kubernetes API complexities required
Operator Lifecycle Manager
    Helps install, update, manage the lifecycle of all of the operators in cluster
Operator Metering
    Usage reporting for operators and resources within Kubernetes

OpenShift Networking
    Container networking based on integrated Open vSwitch
    Platform-wide routing tier
    Ability to plug in third-party software-defined network (SDN) solutions
    Integrated with DNS and existing routing and load balancing

Route
Exposes service by giving it externally reachable host name
    Consists of route name, service selector, and (optional) security configuration
    Router can consume defined route and endpoints identified by service
        Provides named connectivity
        Lets external clients reach OpenShift-hosted applications

While it is easy to deploy a complete multi-tier application, traffic from anywhere outside the OpenShift environment cannot reach the application without the routing layer.

The router container can run on any node host in the environment. Administrators create wildcard DNS entries—either CNAME or A record—on the DNS server, and the DNS entry resolves to the node host that hosts the router container.

The router is the ingress point for all traffic destined for OpenShift services.

It is important to note that the router is different from the route resource. The router is the load balancer and the route is the specific configuration.

