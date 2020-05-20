## Manage OpenShift Nodes

OpenShift uses the machine API types introduced by the Kubernetes Cluster API.

This enables administrators to introspect and manage machines in the cluster using a Kubernetes-native approach.

The Machine object represents a "Kubernetes machine" that can run workloads in the cluster.

## 1. View the machines in the cluster:

`oc get machines -n openshift-machine-api`
NAME                                         INSTANCE              STATE     TYPE        REGION      ZONE         AGE
cluster-48f0-bgxkf-master-0                  i-0b90dd5b0e502bb15   running   m4.xlarge   us-east-2   us-east-2a   3h11m
cluster-48f0-bgxkf-master-1                  i-0f05b3b9e48a3a128   running   m4.xlarge   us-east-2   us-east-2b   3h11m
cluster-48f0-bgxkf-master-2                  i-063955efc3c8ad25b   running   m4.xlarge   us-east-2   us-east-2c   3h11m
cluster-48f0-bgxkf-worker-us-east-2a-fjr7d   i-08755d99da20c0a0a   running   m4.large    us-east-2   us-east-2a   3h10m
cluster-48f0-bgxkf-worker-us-east-2b-j8kxh   i-060dbb738b0ac579e   running   m4.large    us-east-2   us-east-2b   3h10m
cluster-48f0-bgxkf-worker-us-east-2c-swzk8   i-02d731455f7617f61   running   m4.large    us-east-2   us-east-2c   3h10m

- The specification for each machine is specific to each target cloud platform. Because you are using the machine API on Amazon Web Services, the .spec.providerConfig section of a Machine resource describes how a machine should be provisioned on Amazon. Note the TYPE of machine as well as the REGION and ZONE. These values are cloud-specific.

- Also note that there are three masters and three worker nodes, one in each availability zone: us-east-2a, us-east-2b, and us-east-2c.

## 2. Examine the machine definition for one of your worker nodes
`oc describe machine cluster-48f0-bgxkf-worker-us-east-2a-fjr7d -n openshift-machine-api`

Name:         cluster-48f0-bgxkf-worker-us-east-2a-fjr7d
Namespace:    openshift-machine-api
Labels:       machine.openshift.io/cluster-api-cluster=cluster-48f0-bgxkf
              machine.openshift.io/cluster-api-machine-role=worker
              machine.openshift.io/cluster-api-machine-type=worker
              machine.openshift.io/cluster-api-machineset=cluster-48f0-bgxkf-worker-us-east-2a
Annotations:  <none>
API Version:  machine.openshift.io/v1beta1
Kind:         Machine
Metadata:
  Creation Timestamp:  2020-05-14T19:54:08Z
  Finalizers:
    machine.machine.openshift.io
  Generate Name:  cluster-48f0-bgxkf-worker-us-east-2a-
  Generation:     1
  Owner References:
    API Version:           machine.openshift.io/v1beta1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  MachineSet
    Name:                  cluster-48f0-bgxkf-worker-us-east-2a
    UID:                   8bd8c827-961c-11ea-8294-0215f3ab7a5c
  Resource Version:        10461
  Self Link:               /apis/machine.openshift.io/v1beta1/namespaces/openshift-machine-api/machines/cluster-48f0-bgxkf-worker-us-east-2a-fjr7d
  UID:                     aca94ae6-961c-11ea-8294-0215f3ab7a5c
Spec:
  Metadata:
    Creation Timestamp:  <nil>
  Provider Spec:
    Value:
      Ami:
        Id:         ami-0649fd5d42859bdfc
      API Version:  awsproviderconfig.openshift.io/v1beta1
      Block Devices:
        Ebs:
          Iops:         0
          Volume Size:  120
          Volume Type:  gp2
      Credentials Secret:
        Name:        aws-cloud-credentials
      Device Index:  0
      Iam Instance Profile:
        Id:           cluster-48f0-bgxkf-worker-profile
      Instance Type:  m4.large
      Kind:           AWSMachineProviderConfig
      Metadata:
        Creation Timestamp:  <nil>
      Placement:
        Availability Zone:  us-east-2a
        Region:             us-east-2
      Public Ip:            <nil>
      Security Groups:
        Filters:
          Name:  tag:Name
          Values:
            cluster-48f0-bgxkf-worker-sg
      Subnet:
        Filters:
          Name:  tag:Name
          Values:
            cluster-48f0-bgxkf-private-us-east-2a
      Tags:
        Name:   kubernetes.io/cluster/cluster-48f0-bgxkf
        Value:  owned
      User Data Secret:
        Name:  worker-user-data
Status:
  Addresses:
    Address:     10.0.143.127
    Type:        InternalIP
    Address:     
    Type:        ExternalDNS
    Address:     ip-10-0-143-127.us-east-2.compute.internal
    Type:        InternalDNS
  Last Updated:  2020-05-14T19:59:22Z
  Node Ref:
    Kind:  Node
    Name:  ip-10-0-143-127.us-east-2.compute.internal
    UID:   4f968249-961d-11ea-987d-06c362ef2b08
  Provider Status:
    API Version:  awsproviderconfig.openshift.io/v1beta1
    Conditions:
      Last Probe Time:       2020-05-14T19:55:32Z
      Last Transition Time:  2020-05-14T19:55:32Z
      Message:               machine successfully created
      Reason:                MachineCreationSucceeded
      Status:                True
      Type:                  MachineCreation
    Instance Id:             i-08755d99da20c0a0a
    Instance State:          running
    Kind:                    AWSMachineProviderStatus
Events:
  Type    Reason   Age                     From            Message
  ----    ------   ----                    ----            -------
  Normal  Updated  2m40s (x25 over 3h12m)  aws-controller  Updated machine cluster-48f0-bgxkf-worker-us-east-2a-fjr7d


## 3. List all machines and their associated instances types:

`oc get machines -n openshift-machine-api -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{"\t"}{.spec.providerSpec.value.instanceType}{end}{"\n"}'`

cluster-48f0-bgxkf-master-0	m4.xlarge
cluster-48f0-bgxkf-master-1	m4.xlarge
cluster-48f0-bgxkf-master-2	m4.xlarge
cluster-48f0-bgxkf-worker-us-east-2a-fjr7d	m4.large
cluster-48f0-bgxkf-worker-us-east-2b-j8kxh	m4.large
cluster-48f0-bgxkf-worker-us-east-2c-swzk8	m4.large

- The OpenShift installer provisions the required compute infrastructure to run the control plane.

- Once the API server is running, the OpenShift installer creates a Machine resource for each control plane machine using the machine API. The machine API adopts the existing machines. Once the machines are adopted, the machine API is responsible for ensuring the continued existence of master machines.

## 4. List the nodes that make up the OpenShift Control Plane. These machines have a machine.openshift.io/cluster-api-machine-type=master label:

`oc get machines -l machine.openshift.io/cluster-api-machine-type=master -n openshift-machine-api`

NAME                          INSTANCE              STATE     TYPE        REGION      ZONE         AGE
cluster-48f0-bgxkf-master-0   i-0b90dd5b0e502bb15   running   m4.xlarge   us-east-2   us-east-2a   3h24m
cluster-48f0-bgxkf-master-1   i-0f05b3b9e48a3a128   running   m4.xlarge   us-east-2   us-east-2b   3h24m
cluster-48f0-bgxkf-master-2   i-063955efc3c8ad25b   running   m4.xlarge   us-east-2   us-east-2c   3h24m

- The OpenShift installer creates a highly available control plane by default for the target platform. In AWS each cluster is associated with a region.
- Each master is placed in a separate availability zone in that region.

## 5. View the region for each control plane machine:

` oc get machines -l machine.openshift.io/cluster-api-machine-type=master -n openshift-machine-api -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.providerSpec.value.placement.region}{"\n"}{end}' `

cluster-48f0-bgxkf-master-0	us-east-2
cluster-48f0-bgxkf-master-1	us-east-2
cluster-48f0-bgxkf-master-2	us-east-2

## 6. View the availability zone for each control plane machine:

`oc get machines -l machine.openshift.io/cluster-api-machine-type=master -n openshift-machine-api -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.providerSpec.value.placement.availabilityZone}{"\n"}{end}' `

cluster-48f0-bgxkf-master-0	us-east-2a
cluster-48f0-bgxkf-master-1	us-east-2b
cluster-48f0-bgxkf-master-2	us-east-2c

- Once the cluster control plane is bootstrapped, all worker machines are provisioned by the machine API.
  
- The OpenShift installer creates a machineset with the desired number of replicas for the cluster. The worker nodes run end-user workloads on the cluster and execute on machines that have the *machine.openshift.io/cluster-api-machine-type=worker* label.
  
## 7. Find all of the worker machines:

`oc get machines -l machine.openshift.io/cluster-api-machine-type=worker -n openshift-machine-api`

NAME                                         INSTANCE              STATE     TYPE       REGION      ZONE         AGE
cluster-48f0-bgxkf-worker-us-east-2a-fjr7d   i-08755d99da20c0a0a   running   m4.large   us-east-2   us-east-2a   3h30m
cluster-48f0-bgxkf-worker-us-east-2b-j8kxh   i-060dbb738b0ac579e   running   m4.large   us-east-2   us-east-2b   3h30m
cluster-48f0-bgxkf-worker-us-east-2c-swzk8   i-02d731455f7617f61   running   m4.large   us-east-2   us-east-2c   3h30m

### 1.1 Explore Machinesets

The OpenShift installer creates a highly available pool of worker machines by default for the target platform. On AWS, each cluster is associated with a region. The install creates a machineset for each availability zone in that region.

If the region supports three availability zones, and the user requests three machines, then a machineset is created for each availability zone with a replica size of one.

The MachineSet object allows an administrator to request a preferred number of machines replicated from a common Machine template.

#### 1. View the machineset in the cluster

`oc get machinesets -n openshift-machine-api`

NAME                                   DESIRED   CURRENT   READY   AVAILABLE   AGE
cluster-48f0-bgxkf-worker-us-east-2a   1         1         1       1           3h34m
cluster-48f0-bgxkf-worker-us-east-2b   1         1         1       1           3h34m
cluster-48f0-bgxkf-worker-us-east-2c   1         1         1       1           3h34m

* Note that there is a machineset for each availability zone in AWS.

#### 2. Find out the number of machine replicas that each machineset controls:

`oc get machinesets -n openshift-machine-api -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{"\t"}{.spec.replicas}{end}{"\n"}'`

cluster-48f0-bgxkf-worker-us-east-2a	1
cluster-48f0-bgxkf-worker-us-east-2b	1
cluster-48f0-bgxkf-worker-us-east-2c	1

### 1.2 Change Instance type of machineset

To provision a new machine, you can create a new Machine resource. An easy way to create a new machine is to scale up a machineset.

Machinesets hold the configuration for the types of machines they provision. The default instance type for an AWS machine is m4.large (as of openshift-install 0.16.1). If you want to use a different instance type, modify the instance type in the machineset before adding machines to it. If you want to replace an existing node at the same time, you also need to scale the machineset down to zero to delete the existing node and then scale it up to the number of desired replicas.

In the example above, there were three machines in availability zones us-east-2a, us-east-2b, and us-east-2c. In this section, you change what kind of instances are created in availability zone us-east-2c.

#### 1. Mofify the *machineset* configuration and change *instanceType* to *m5.2xlarge* :

`oc patch machineset cluster-48f0-bgxkf-worker-us-east-2c --type='merge' --patch='{"spec": { "template": { "spec": { "providerSpec": { "value": { "instanceType": "m5.2xlarge"}}}}}}' -n openshift-machine-api`
machineset.machine.openshift.io/cluster-48f0-bgxkf-worker-us-east-2c patched

* This change does not replace a node with a new node.

#### 2. Use `oc scale` to scale the machineset for cluster-48f0-bgxkf-worker-us-east-2c to zero replicas:

`oc scale machineset cluster-48f0-bgxkf-worker-us-east-2c --replicas=0 -n openshift-machine-api`
machineset.machine.openshift.io/cluster-48f0-bgxkf-worker-us-east-2c scaled

#### 3. Check the number of machine replicas that each machineset controls

`oc get machinesets -n openshift-machine-api`
NAME                                   DESIRED   CURRENT   READY   AVAILABLE   AGE
cluster-48f0-bgxkf-worker-us-east-2a   1         1         1       1           3h57m
cluster-48f0-bgxkf-worker-us-east-2b   1         1         1       1           3h56m
cluster-48f0-bgxkf-worker-us-east-2c   0         0                             3h56m

#### 4. Check the machines in the cluster

`oc get machines -n openshift-machine-api`
NAME                                         INSTANCE              STATE     TYPE        REGION      ZONE         AGE
cluster-48f0-bgxkf-master-0                  i-0b90dd5b0e502bb15   running   m4.xlarge   us-east-2   us-east-2a   3h58m
cluster-48f0-bgxkf-master-1                  i-0f05b3b9e48a3a128   running   m4.xlarge   us-east-2   us-east-2b   3h58m
cluster-48f0-bgxkf-master-2                  i-063955efc3c8ad25b   running   m4.xlarge   us-east-2   us-east-2c   3h58m
cluster-48f0-bgxkf-worker-us-east-2a-fjr7d   i-08755d99da20c0a0a   running   m4.large    us-east-2   us-east-2a   3h57m
cluster-48f0-bgxkf-worker-us-east-2b-j8kxh   i-060dbb738b0ac579e   running   m4.large    us-east-2   us-east-2b   3h57m

* Note that there are now only two worker machines.

#### 5. Check the OpenShift nodes:

`oc get nodes`
NAME                                         STATUS   ROLES    AGE     VERSION
ip-10-0-142-196.us-east-2.compute.internal   Ready    master   3h59m   v1.13.4+cb455d664
ip-10-0-143-127.us-east-2.compute.internal   Ready    worker   3h54m   v1.13.4+cb455d664
ip-10-0-157-195.us-east-2.compute.internal   Ready    master   4h      v1.13.4+cb455d664
ip-10-0-157-67.us-east-2.compute.internal    Ready    worker   3h55m   v1.13.4+cb455d664
ip-10-0-170-252.us-east-2.compute.internal   Ready    master   4h      v1.13.4+cb455d664

* You may need to run this command a few times to see results similar to the sample because it takes a little while for the machine controller to properly remove the node from the cluster. Removing the node includes marking it as NonSchedulable and then moving all of the pods off the node to other nodes in the cluster in an orderly fashion. Once this process is complete, the machine controller removes the node and then deletes the AWS EC2 instance.

#### 6. Now scale the machieset back up to one replica:

`oc scale machineset cluster-48f0-bgxkf-worker-us-east-2c --replicas=1 -n openshift-machine-api`
machineset.machine.openshift.io/cluster-48f0-bgxkf-worker-us-east-2c scaled

* This new machine is created using the new instance type, m5.2xlarge.

#### 7. Check the number of machine replicas tat each machineset controls:
`oc get machinesets -n openshift-machine-api`
NAME                                   DESIRED   CURRENT   READY   AVAILABLE   AGE
cluster-48f0-bgxkf-worker-us-east-2a   1         1         1       1           4h15m
cluster-48f0-bgxkf-worker-us-east-2b   1         1         1       1           4h15m
cluster-48f0-bgxkf-worker-us-east-2c   1         1                             4h15m

* Note that the third machineset shows that one is desired and current, but none are yet ready and available.

#### 8. Check the machines in the cluster:

`oc get machines -n openshift-machine-api`
NAME                                         INSTANCE              STATE     TYPE         REGION      ZONE         AGE
cluster-48f0-bgxkf-master-0                  i-0b90dd5b0e502bb15   running   m4.xlarge    us-east-2   us-east-2a   4h17m
cluster-48f0-bgxkf-master-1                  i-0f05b3b9e48a3a128   running   m4.xlarge    us-east-2   us-east-2b   4h17m
cluster-48f0-bgxkf-master-2                  i-063955efc3c8ad25b   running   m4.xlarge    us-east-2   us-east-2c   4h17m
cluster-48f0-bgxkf-worker-us-east-2a-fjr7d   i-08755d99da20c0a0a   running   m4.large     us-east-2   us-east-2a   4h16m
cluster-48f0-bgxkf-worker-us-east-2b-j8kxh   i-060dbb738b0ac579e   running   m4.large     us-east-2   us-east-2b   4h16m
cluster-48f0-bgxkf-worker-us-east-2c-tcvvq   i-02e282a2e8a52119f   running   m5.2xlarge   us-east-2   us-east-2c   4m17s

* Note that are three worker machines again, two of type m4.large and one of type m5.2xlarge.

#### 9. Check the OpenShift nodes:

`oc get nodes`
NAME                                         STATUS   ROLES    AGE     VERSION
ip-10-0-142-196.us-east-2.compute.internal   Ready    master   4h19m   v1.13.4+cb455d664
ip-10-0-143-127.us-east-2.compute.internal   Ready    worker   4h14m   v1.13.4+cb455d664
ip-10-0-157-195.us-east-2.compute.internal   Ready    master   4h20m   v1.13.4+cb455d664
ip-10-0-157-67.us-east-2.compute.internal    Ready    worker   4h14m   v1.13.4+cb455d664
ip-10-0-164-208.us-east-2.compute.internal   Ready    worker   2m29s   v1.13.4+cb455d664
ip-10-0-170-252.us-east-2.compute.internal   Ready    master   4h20m   v1.13.4+cb455d664

* You may need to run this command a few times because it takes a few minutes for the new AWS EC2 instance to be provisioned and added to the cluster. If you do not see the node at all or see the new node as NotReady, run the command again a minute or so later and see if it has changed to Ready and available to accept pods.

#### 10. Double-check the machineset to verify that the machine is now registered as both ready and available:

`oc get machinesets -n openshift-machine-api`
NAME                                   DESIRED   CURRENT   READY   AVAILABLE   AGE
cluster-48f0-bgxkf-worker-us-east-2a   1         1         1       1           4h22m
cluster-48f0-bgxkf-worker-us-east-2b   1         1         1       1           4h22m
cluster-48f0-bgxkf-worker-us-east-2c   1         1         1       1           4h22m

### 1.3 Remove node from cluster

To deprovision a machine, you can delete a Machine resource. But because the machines are monitored by machinesets, the machine simply gets reprovisioned. It is possible to execute this command intentionally to replace a "broken" node—but to actually remove a node, the better way is to scale down a MachineSet resource.

Prior to deleting the machine, the node is automatically cordoned and drained of its pods.

For this exercise, assume that the node in availability zone 2a is no longer functioning as expected.

#### 1. Remove the node from the cluster:
`oc scale machineset cluster-48f0-bgxkf-worker-us-east-2a --replicas=0 -n openshift-machine-api`
machineset.machine.openshift.io/cluster-48f0-bgxkf-worker-us-east-2a scaled

#### 2. Validate the number of replicas for each machineset
`oc get machineset -n openshift-machine-api`
NAME                                   DESIRED   CURRENT   READY   AVAILABLE   AGE
cluster-48f0-bgxkf-worker-us-east-2a   0         0                             4h31m
cluster-48f0-bgxkf-worker-us-east-2b   1         1         1       1           4h31m
cluster-48f0-bgxkf-worker-us-east-2c   1         1         1       1           4h31m

* Expected to see that availability zone us-east-2a has zero replicas.

#### 3. Once the worker node has disappeared, check the OpenShift nodes using the `-o wide`parameter to display more information about each node:

`oc get nodes -o wide`
NAME                                         STATUS   ROLES    AGE     VERSION             INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                                                   KERNEL-VERSION               CONTAINER-RUNTIME
ip-10-0-142-196.us-east-2.compute.internal   Ready    master   4h33m   v1.13.4+cb455d664   10.0.142.196   <none>        Red Hat Enterprise Linux CoreOS 410.8.20190520.0 (Ootpa)   4.18.0-80.1.2.el8_0.x86_64   cri-o://1.13.9-1.rhaos4.1.gitd70609a.el8
ip-10-0-157-195.us-east-2.compute.internal   Ready    master   4h33m   v1.13.4+cb455d664   10.0.157.195   <none>        Red Hat Enterprise Linux CoreOS 410.8.20190520.0 (Ootpa)   4.18.0-80.1.2.el8_0.x86_64   cri-o://1.13.9-1.rhaos4.1.gitd70609a.el8
ip-10-0-157-67.us-east-2.compute.internal    Ready    worker   4h28m   v1.13.4+cb455d664   10.0.157.67    <none>        Red Hat Enterprise Linux CoreOS 410.8.20190520.0 (Ootpa)   4.18.0-80.1.2.el8_0.x86_64   cri-o://1.13.9-1.rhaos4.1.gitd70609a.el8
ip-10-0-164-208.us-east-2.compute.internal   Ready    worker   16m     v1.13.4+cb455d664   10.0.164.208   <none>        Red Hat Enterprise Linux CoreOS 410.8.20190520.0 (Ootpa)   4.18.0-80.1.2.el8_0.x86_64   cri-o://1.13.9-1.rhaos4.1.gitd70609a.el8
ip-10-0-170-252.us-east-2.compute.internal   Ready    master   4h33m   v1.13.4+cb455d664   10.0.170.252   <none>        Red Hat Enterprise Linux CoreOS 410.8.20190520.0 (Ootpa)   4.18.0-80.1.2.el8_0.x86_64   cri-o://1.13.9-1.rhaos4.1.gitd70609a.el8

* Expect to see that one of the worker nodes is now newer than the other one. This is the node that you added to the cluster.
* Also note the internal and external IP addresses, and that none of the nodes has an external IP address.

### Explore on Your own

Here are a few ideas to get you started:

How would you create NetworkPolicy objects?

Can you provision and deploy an operator from the web console? Where would you do this?

Where is the router running, and what controls the router?

Which API object holds the configuration for the router? Which project is that object in?

Which storage options are available on a freshly created cluster?

There is a custom resource default of type Tuned in project openshift-cluster-node-tuning-operator. What can you deduce by describing this object?

How is the cluster DNS configured?

Hint: There is an API object of type ClusterDNS.

Which project is that in? What controls it?
