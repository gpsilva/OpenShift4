## Cluster Management Overview

OpenShift Container Platform 4 clusters are fully managed using Operators. 
The OpenShift installation process creates these Operators. 
Operator activities are managed through custom resources, which are extensions of the Kubernetes API.

Using Operators means that the cluster effectively manages itself—even the operating system is managed by OpenShift Container Platform. 
Cluster upgrades are handled by the cluster after it is deployed.

Changing the cluster configuration are made by changing specific custom resources—for example, changing an IngressController object to configure routers or changing the OAuth object to configure authentication for the cluster.

### Kubernetes Machine API

The Machine API is one of the most important custom resources for managing the cluster. 
It manages Machines, which map to Nodes in OpenShift terminology.

It is possible to create, update, and delete machines. 
Creating a machine also configures the OpenShift node and even the underlying infrastructure, or VM, for the node.

MachineSets are used to control sets of machines. 
In OpenShift Container Platform 4, the installer creates a machineset for each Availability Zone in the selected Region of the cloud provider.

In upcoming releases, MachineDeployments is planned to allow for more robust control of machinesets and machines—for example, by making it possible to do rolling upgrades of nodes upon a change to the machinedeployment.

#### Machines

Machines and Machinesets managed in openshift-machine-api namespace

Example:

`oc get machines -n openshift-machine-api`

NAME                                         INSTANCE              STATE     TYPE        REGION      ZONE         AGE
cluster-48f0-bgxkf-master-0                  i-0b90dd5b0e502bb15   running   m4.xlarge   us-east-2   us-east-2a   177m
cluster-48f0-bgxkf-master-1                  i-0f05b3b9e48a3a128   running   m4.xlarge   us-east-2   us-east-2b   177m
cluster-48f0-bgxkf-master-2                  i-063955efc3c8ad25b   running   m4.xlarge   us-east-2   us-east-2c   177m
cluster-48f0-bgxkf-worker-us-east-2a-fjr7d   i-08755d99da20c0a0a   running   m4.large    us-east-2   us-east-2a   177m
cluster-48f0-bgxkf-worker-us-east-2b-j8kxh   i-060dbb738b0ac579e   running   m4.large    us-east-2   us-east-2b   177m
cluster-48f0-bgxkf-worker-us-east-2c-swzk8   i-02d731455f7617f61   running   m4.large    us-east-2   us-east-2c   177m

#### Machinesets

Machineset represents:
- Set of machines
- Abstraction of underlying infrastructure

In RHOCP 4, AWS abstraction maps to AWS Availability Zone
- Installer creates machineset for each Availability Zone in selected Region

Machinesets include templates for new machines. 
In addition to other information, the template contains the following important properties:
- Instance types
- Disk types, size, and input/output operations per second, or IOPS, characteristics
- Labels and taints

It is possible to edit machinesets to change the defaults for new machines. 
It is also possible to create new machinesets—for example, to create machines that use a graphics processing unit, or GPU.

It is possible easily add or remove machines to or from the cluster by scaling the machinesets.

`oc get machinesets -n openshift-machine-api`
NAME                                   DESIRED   CURRENT   READY   AVAILABLE   AGE
cluster-48f0-bgxkf-worker-us-east-2a   1         1         1       1           3h6m
cluster-48f0-bgxkf-worker-us-east-2b   1         1         1       1           3h6m
cluster-48f0-bgxkf-worker-us-east-2c   1         1         1       1           3h6m

