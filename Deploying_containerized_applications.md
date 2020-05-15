# What are Containers?

Containers are a set of one or more processes taht are isolated from the rest of the system.
They provide many of the same benefits as a Virtual Machine.

### Low Hardware Footprint

Use Operatiing Systems internal features, such as a cgroups and namespaces to create an isolated environment, minimizing CPU and memory overhead.

### Quick and reusable deployment

Containers deploy quickly because there is no need to install the entire undelying operating system.
Using an image allows for creating multiple containers from the same blueprint.

### Environment isolation

Containers work in a closed environment where changes made to the host OS or other applications do not affect the container.

### Multiple Environment Deployment

In additional deployment, any environment differences could break the application. With containers, all application dependencies and environment settings are encapsulated in the container image.

### Podman

Podman is a an open source tool for managing containers and container images and interacting with image registries.
Podman uses image format specified by Open Container Initiative (OCI). Those specifications define an standard, community-driven, non-proprietary image format.
Podman stores local images in local file-system. Doing so avoids unnecessary client/server architecutre or having daemons running on local machine.
Podman follows the same command patterns as the Docker CLI.
Podman is compatible with Kubernetes.

### Kubernete features

- Service discovery and load balancing
- Horizontal Scaling
- Self-healing with user defined healt checks
- Automated rollout and rollback
- Secrets and configuration management
- Operators
  
# Red Hat OpenShift Container Platform (RHOCP)
Is a set of modular components and services build on top of Kubernetes container infrastructure.
RHOCP adds the capabilities to provide a production PaaS platform such as remote management, multitenancy, increased security, monitoring and auditing, application life-cycle management, and self-service interface for developers.

### RHOCP Features:

- Integrated developer workflow
- Routes
- Metrics and logging
- Unified UI

### Running Containers

The container image specifies a process that starts inside the container known as the <entry point>.
The `podman run` command uses all parameters after the image name as the entry point command for the container.

### CMD and ENTRYPOINT

Defining an ENTRYPOINT n the Dockerfile creates containers that are executables. The ENTRYPOINT can be a script that is added to the container with an ADD instruction.
The default ENTRYPOINT instructions is /bin/sh -c
Use CMD to pass in arguments to the ENTRYPOINT instruction.
The CMD argument can be overwritten when running a container docker run.

### Image Layering

Each instruction in a Dockerfile creates a new layer.
Having too many instructions in a Dockerfile causes too many layers, resulting in a large images.


### Kubernetes Architecture

#### Pod

The smalles unit manageable in Kubernetes is a *pod*
A *pod*  consists of one or more containers with its storage resources and IP.

#### Master node

A *master node* provides basic cluster services such as *APIs* and *controllers*.

#### Worker node
A *worker node* performs work in a Kubernetes cluster. Application *pods* are scheduled onto *worker nodes*

#### Controller

A *controller* is a Kubernetes process that watches resources  and makes changes based on that state.

#### Services

*Services* define a single, persistent IP/port combination that provides access to a pool of *pods*.

#### Replication Controller

A kubernetes resource that defines how pods are replicated (horizontally scaled) into differente nodes. Replication controllers are a basic Kubernetes service to provide high availability for pods and containers.

#### Persistent volumes

Define storage areas to be used by Kubernetes pods.

#### Persistent Volume Claims

Represent a request for storage by a pod. *PVCs* links a PV to a *pod* so its containers can make use of it.

#### ConfigMaps and Secrets

Contains a set of keys and values that can be used by other resources.

### OpenShift Resource Types

#### Deployment config (dc)

Represents the set of containers included in a pod, and the deployment strategies to be used. A *dc* also provides a basic but extensible continuous delivery workflow.

#### Build Config (bc)

Defines a process to be executed in the OpenShift project. Used by the OpenShift Source-to-Image (S2I) feature to build a container image from application source code stored in a Git repository.

#### Routes

Represents a DNS host name recognized by the OpenShift router as an ingress point for applications and microservices.

### Whatś new in OpenShift 4?

#### CoreOS
OpenShift Container Platform 4 uses Red Hat Enterprise Linux CoreOS, a container-oriented operating system. This OS is specifically designed to run containerized applications from OpenShift and provide simpler installation.

#### Operators

A convenient way to deploy applications and software components for applications to use that leverage the container platform.


## Working with Routes

Routes are implemented by a shared router service that run as *pods* inside the OCP instance and can be scaled and replicated like any other *pod*.

### Source-to-Image Benefits

#### User efficiency

Developers do not need to understand Dockerfiles and operating system commands such as yum install. They work using their starndard programming language tools.

#### Patching

S2I allows for rebuilding all the applications consistently if a base image needs a patch due to a security issue.

#### Speed

With S2I, the assembly process can perform a large number of complex operations without creating a new layer at each step, resulting in faster builds.

#### Ecosystem

S2I encourages a shared ecosystem of images where base images and scripts can be customized and reused across multiple types of applications.

### Image Streams

* An Image Stream is a resource that defines related container images by tag. For example, a MySQL image stream containing a list of MySQL container image versions.

* OpenShift can be configured to "watch" an image stream and automatically perform an action when new images are created, such as triggering a new build when an updated parent image is available.

