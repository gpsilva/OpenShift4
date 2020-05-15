# 2 Review Application Components

Several of the the most important application components of OpenShift.

- Application code runs in containers. Pods are comprised of one or more containers. Usually you have one container per Pod. You can have more than one container in a Pod if you want them to share network and storage space—for example, to have access to the same volume. Each Pod has an IP address. Usually containers do not have individual IP addresses.

- Pods are not persistent. They can start, stop, crash, and restart. Each time a Pod restarts, it is assigned a new IP address. For this reason, you cannot rely on the Pod’s IP address remaining the same.

- A Pod can have labels, and these labels are applied to all Pods throughout this particular Deployment. Labels can be used to identify and select Pods. Labels are key-value pairs. Examples of labels include app: web, class: frontend, and zone: east.

- Pods can crash. When they crash, you usually want them to be automatically restarted. The ReplicaSet object (or ReplicationController) is responsible for doing this. It can watch a single Pod or a set of Pods. It selects them by selectors and labels on the Pods. Its only job is to make sure that the requested number of Pods is up and running. For example, you may have a ReplicaSet called rs-web that specifies app: web as a selector, and you have Pods with a matching label. In this case, the ReplicaSet manages the number of replicas of those particular Pods and makes sure that the specified number of Pods exist.

- A ReplicaSet (or ReplicationController) provides only a very basic function. What if you want to update the application code? Or start a new ReplicaSet and switch to it? Or gradually migrate Pods from an old version to a new one? Or roll back to the old version if the new one is not working properly? Deployments and DeploymentConfigs can do this for you. Furthermore, they can do this automatically when they detect changes in your application configuration.

- IP addresses of Pods are not stable. Each time a Pod restarts, it is assigned a new IP address. To access a Pod, you may want the Pod to have a stable IP address. Or you may want to run several identical Pods and load balance between them, in which case you must have a stable IP address for the load balancer. In both cases, you use a Service.

- Services, like ReplicaSets, use selectors to find the Pods associated with them. Selectors select Pods with specific labels. For example, a Service called service-web provides a stable IP address (called a ClusterIP) for the Pods with the app: web label.

- A ClusterIP provided by a Service is internal to the cluster. To access the application from outside the cluster requires a Route. The Route object gets information about associated Pods from the back-end Service and acts as an external load balancer for those Pods. Routes are usually implemented using HAProxy. Routes provide domain name addresses, which can be used to access the application.

# 3. Explore Deployment Components

### 1. Create a new project:

`oc new-project $GUID-deployments --display-name="Openshift Deployments"`

### 2. Create manifest "ocp-probe-deployment.yaml" which describes a REST API service deployment

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ocp-probe
spec:
  selector:
    matchLabels:
      app: ocp-probe
  replicas: 1
  template:
    metadata:
      labels:
        app: ocp-probe
    spec:
      containers:
      - name: ocp-probe
        image: quay.io/gpte-devops-automation/ocp-probe:v0.3
        ports:
        - containerPort: 8080

~~~

### 3. Examine the Deployment manifest closely, and note in particular the following:

* It is formatted as YAML, which uses indentation to mark different levels of its hierarchical structure.

* At the first level (not indented), there are four components: apiVersion, kind, metadata, and spec. The first three are self-explanatory: this is a Deployment (kind), its name is ocp-probe (metadata.name), and it uses the apps/v1 version of Kubernetes API. In the YAML Manifest Anatomy lab, you review this in more detail.

* a lot of information in the fourth component, spec.

    - It specifies the selector that it uses to find the Pods to manage—in this case, app: ocp-probe.

    - It specifies the number of replicas to run (only one in the beginning).

    - It defines the template to use for this Deployment’s Pods. Under template you see a very similar structure that includes metadata and spec. In this case, these components are applied to the Pods that comprise the application.

    - Note that you define the app: ocp-probe label. This label is applied to all Pods using this template, which is how you define that they all belong to this Deployment (as in the selector defined previously).

    - Under spec, you specify the containers that are to be run in each Pod. You give them a name (ocp-probe) and specify the container image you want to use-- in this case, quay.io/gpte-devops-automation/ocp-probe:v0.3. At this level, you can also specify network ports used by this container as well as other parameters. (The lab includes more about that later).

### 4. Create this Deployment:
`oc create -f ocp-probe-deployment.yaml`

### 5. Status immediately after `oc create`command to watch the rollout process:
`oc rollout status deployment ocp-probe`
Waiting for deployment "ocp-probe" rollout to finish: 0 of 1 updated replicas are available...
deployment "ocp-probe" successfully rolled out

# 4. Scale Deployment

In some cases, we want to run serveral copies of the application, either for scalability or for high availability purposes. For stateless applications simply start more instances of the same Pod.

### 1. Scale application:
`oc scale deployment ocp-probe --replicas=3`
deployment.extensions/ocp-probe scaled

### 2. Check status of the Deployment rollout to watch the progress of the rollout:
`oc rollout status deployment ocp-probe`
Waiting for deployment "ocp-probe" rollout to finish: 1 of 3 updated replicas are available...
Waiting for deployment "ocp-probe" rollout to finish: 2 of 3 updated replicas are available...
deployment "ocp-probe" successfully rolled out

`oc get pods`

NAME                        READY   STATUS    RESTARTS   AGE
ocp-probe-87b4cfcdd-hvr25   1/1     Running   0          20s
ocp-probe-87b4cfcdd-vck6j   1/1     Running   0          20s
ocp-probe-87b4cfcdd-xjzs8   1/1     Running   0          8m3s

### 3. Examine the application's resources:
`oc get all`

NAME                            READY   STATUS    RESTARTS   AGE
pod/ocp-probe-87b4cfcdd-hvr25   1/1     Running   0          4m1s
pod/ocp-probe-87b4cfcdd-vck6j   1/1     Running   0          4m1s
pod/ocp-probe-87b4cfcdd-xjzs8   1/1     Running   0          11m

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ocp-probe   3/3     3            3           11m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ocp-probe-87b4cfcdd   3         3         3       11m

* The Deployment created a ReplicaSet to keep track of the Pods belonging to it.

### 4. Examine the details of the ReplicaSet:
`oc describe rs ocp-probe-87b4cfcdd`

Name:           ocp-probe-87b4cfcdd
Namespace:      4e41-deployments
Selector:       app=ocp-probe,pod-template-hash=87b4cfcdd
Labels:         app=ocp-probe
                pod-template-hash=87b4cfcdd
Annotations:    deployment.kubernetes.io/desired-replicas: 3
                deployment.kubernetes.io/max-replicas: 4
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/ocp-probe
Replicas:       3 current / 3 desired
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=ocp-probe
           pod-template-hash=87b4cfcdd
  Containers:
   ocp-probe:
    Image:        quay.io/gpte-devops-automation/ocp-probe:v0.3
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  16m    replicaset-controller  Created pod: ocp-probe-87b4cfcdd-xjzs8
  Normal  SuccessfulCreate  8m19s  replicaset-controller  Created pod: ocp-probe-87b4cfcdd-hvr25
  Normal  SuccessfulCreate  8m19s  replicaset-controller  Created pod: ocp-probe-87b4cfcdd-vck6j

* Similarly can scale the Deployment down if the application traffic is low. Simply choose a lower number in the --replicas parameter. There are ways to scale the Deployment up and down automatically. To get more information about that, run the oc autoscale -h command for help on autoscale.
  
# 5. Upgrade Application

### 1. Find the line with `image: ` change the tag `v0.3` to `v0.4` :
`oc edit deployment ocp-probe`
deployment.extensions/ocp-probe edited

### 2. Immediately after saving the manifest, run status command to watch Deployment updated:
Waiting for deployment "ocp-probe" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "ocp-probe" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "ocp-probe" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "ocp-probe" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "ocp-probe" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "ocp-probe" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "ocp-probe" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "ocp-probe" rollout to finish: 1 old replicas are pending termination...
deployment "ocp-probe" successfully rolled out

### 3. Check the method use by the Deployment to upgrade the application:
`oc get all`
NAME                            READY   STATUS    RESTARTS   AGE
pod/ocp-probe-b899d558c-c2sr9   1/1     Running   0          2m58s
pod/ocp-probe-b899d558c-nkh6g   1/1     Running   0          2m40s
pod/ocp-probe-b899d558c-t9zpq   1/1     Running   0          2m49s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ocp-probe   3/3     3            3           26m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ocp-probe-87b4cfcdd   0         0         0       26m
replicaset.apps/ocp-probe-b899d558c   3         3         3       3m

* Note that the Deployment created a new ReplicaSet and started three Pods with the new image there.
* The Deployment scaled the old ReplicaSet down to 0 replicas.

### 4. Examine the Deployment events:
`oc describe deployment ocp-probe`
Name:                   ocp-probe
Namespace:              4e41-deployments
CreationTimestamp:      Wed, 06 May 2020 22:14:28 -0300
Labels:                 app=ocp-probe
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=ocp-probe
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=ocp-probe
  Containers:
   ocp-probe:
    Image:        quay.io/gpte-devops-automation/ocp-probe:v0.4
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   ocp-probe-b899d558c (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  27m    deployment-controller  Scaled up replica set ocp-probe-87b4cfcdd to 1
  Normal  ScalingReplicaSet  20m    deployment-controller  Scaled up replica set ocp-probe-87b4cfcdd to 3
  Normal  ScalingReplicaSet  4m40s  deployment-controller  Scaled up replica set ocp-probe-b899d558c to 1
  Normal  ScalingReplicaSet  4m31s  deployment-controller  Scaled down replica set ocp-probe-87b4cfcdd to 2
  Normal  ScalingReplicaSet  4m31s  deployment-controller  Scaled up replica set ocp-probe-b899d558c to 2
  Normal  ScalingReplicaSet  4m22s  deployment-controller  Scaled down replica set ocp-probe-87b4cfcdd to 1
  Normal  ScalingReplicaSet  4m22s  deployment-controller  Scaled up replica set ocp-probe-b899d558c to 3
  Normal  ScalingReplicaSet  4m13s  deployment-controller  Scaled down replica set ocp-probe-87b4cfcdd to 0

* Note how the Deployment scaled up first to 3 replicas and then, due to the upgrade, created a new ReplicaSet—in this case, ocp-probe-87b4cfcdd.
* The Deployment then scaled the old ReplicaSet down to 2, scaled the new one up to 2, and so on until it reached 3 replicas in the new ReplicaSet and 0 in the old one.

# 6. Conclusion
  
* Describe the role of Deployments
* Describe the main components of a Deployment
* Describe different application deployment strategies
* Describe rollout triggers
* Edit a Deployment to change the deployment strategy and trigger parameters


