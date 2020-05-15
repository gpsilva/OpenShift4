## 2. Explore DeploymentConfigs

Another feature of DeploymentConfigs is that, if the deployment of a new version results in an error, the DeploymentConfig automatically rolls back to the previous working version. It waits for 10 minutes by default, and if the application still is not ready, the failing revision is deleted and the previous DeploymentConfig revision is rolled out.

There are other differences between Deployments and DeploymentConfigs that are described in the Comparing Deployments and DeploymentConfigs section of the OpenShift Container Platform documentation.

The documentation describes the difference as follows:

One important difference between Deployments and DeploymentConfigs is the properties of the CAP theorem that each design has chosen for the rollout process. DeploymentConfigs prefer consistency, whereas Deployments have a preference for availability over consistency.

For DeploymentConfigs, if a node running a deployer Pod goes down, it does not get replaced. The process waits until the node comes back online or is manually deleted. Manually deleting the node also deletes the corresponding Pod. This means that you cannot delete the Pod to unstick the rollout, as the kubelet is responsible for deleting the associated Pod.

However, Deployment rollouts are driven by a controller manager. The controller manager runs in high availability mode on masters and uses leader election algorithms to value availability over consistency. During a failure, it is possible for other masters to act on the same Deployment at the same time, but this issue is reconciled shortly after the failure occurs.

Deployments and ReplicaSets are considered the preferred way of deploying applications. On the other hand, when you use the oc new-app command, it creates DeploymentConfigs and ReplicationControllers. It is a good idea for you to be familiar with both methods. This lab provides you with a quick overview of their similarities and differences.

### 2.1 Create DeploymentConfig

#### 1. Create a new project to explore DeploymentConfig:

`oc new-project $GUID-deploymentconfigs --display-name="Openshift DeploymentConfigs`

#### 2. Copy the Deployment manifest from previous lab to `ocp-probe-deploymentconfig.yaml`

#### 3. Edit `apiVersion`, the `kind`of the manifest, and remove `matchLabels:` line to convert the Deployment manifest to a DeploymentConfig onde.

~~~yaml
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: ocp-probe
spec:
  selector:
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

#### 4. Create the DeploymentConfig:
`oc create -f ocp-probe-deploymentconfig.yaml`
deploymentconfig.apps.openshift.io/ocp-probe created

#### 5. Run the `oc rollout status` immediately after `create` command:
`oc rollout status dc ocp-probe`
Waiting for rollout to finish: 0 of 1 updated replicas are available...
Waiting for latest deployment config spec to be observed by the controller loop...
replication controller "ocp-probe-1" successfully rolled out

#### 6. Determine which Openshift resources were created:
`oc get all`
NAME                     READY   STATUS      RESTARTS   AGE
pod/ocp-probe-1-deploy   0/1     Completed   0          70s
pod/ocp-probe-1-znlw5    1/1     Running     0          61s

NAME                                DESIRED   CURRENT   READY   AGE
replicationcontroller/ocp-probe-1   1         1         1       70s

NAME                                           REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfig.apps.openshift.io/ocp-probe   1          1         1         config

#### 7.  The differences between the Deployment manifest and what is created using the DeploymentConfig manifest:

* A ReplicationController is created instead of a ReplicaSet.

* There is a new Pod called ocp-probe-1-deploy. This Pod is used by the DeploymentConfig to perform the deployment.

* In the case of Deployments, the process of Pod creation is managed by the cluster’s controller manager. But in the case of DeploymentConfigs, a separate Pod deploys the application Pods.

### 2.2 Scale Up and Upgrade DeploymentConfig

#### 1. Scale up the DeploymentConfig, roll out the ReplicationController, and list the Pods:
`oc scale dc/ocp-probe --replicas=3`
deploymentconfig.apps.openshift.io/ocp-probe scaled

`oc rollout status dc ocp-probe`
replication controller "ocp-probe-1" successfully rolled out

`oc get pods`
NAME                 READY   STATUS      RESTARTS   AGE
ocp-probe-1-deploy   0/1     Completed   0          22m
ocp-probe-1-grk9p    1/1     Running     0          25s
ocp-probe-1-rmzxk    1/1     Running     0          25s
ocp-probe-1-sw5qf    1/1     Running     0          26s
ocp-probe-1-znlw5    1/1     Running     0          22m

* Note that the Pods and ReplicationController initially have a -1 added to their names.

* Note that the output of the oc rollout status command is less verbose, but the result is the same.


#### 2. Change the version tag from `v0.3` to `v0.4`
`oc edit dc/ocp-probe`
deploymentconfig.apps.openshift.io/ocp-probe edited

`oc rollout status dc ocp-probe`
Waiting for rollout to finish: 0 out of 3 new replicas have been updated...
Waiting for rollout to finish: 0 out of 3 new replicas have been updated...
Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for rollout to finish: 1 old replicas are pending termination...
Waiting for rollout to finish: 1 old replicas are pending termination...
Waiting for latest deployment config spec to be observed by the controller loop...
replication controller "ocp-probe-2" successfully rolled out

* After the upgrade, a new ReplicationController appears with -2 added to its name.

#### 3. Current Openshift resources in project:

`oc get all`
NAME                     READY   STATUS      RESTARTS   AGE
pod/ocp-probe-1-deploy   0/1     Completed   0          5h35m
pod/ocp-probe-2-6wkx6    1/1     Running     0          4m39s
pod/ocp-probe-2-cxg8h    1/1     Running     0          4m29s
pod/ocp-probe-2-deploy   0/1     Completed   0          4m59s
pod/ocp-probe-2-zdhzt    1/1     Running     0          4m50s

NAME                                DESIRED   CURRENT   READY   AGE
replicationcontroller/ocp-probe-1   0         0         0       5h35m
replicationcontroller/ocp-probe-2   3         3         3       5m

NAME                                           REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfig.apps.openshift.io/ocp-probe   2          3         3         config

#### 4. Other changes:

* There is a new `-deploy` pod with index `-2` and a set of application Pods with index `-2`, which reflect the deployment revison.
* There is a `REVISION` number in the DeploymentConfig line. This is a good way to keep track deployments and see which revision is currently running.

## 3. Explore Deployment Strategies

### 3.1 Rolling and Recreate Strategies

When upgraded the DeploymentConfig in the previous exercise, you saw one deployment strategy in action. Using this strategy, a new ReplicationController is created and new Pods are started in it one by one. At the same time the old Pods are scaled down in parallel. This deployment strategy is called a Rolling strategy and its main goal is to prevent any downtime for the running applications. This is the default.

A Rolling deployment strategy is the preferred one because you do not lose application availability during the upgrade. However, there are some requirements for using a Rolling strategy. Your application must support running old and new versions of code simultaneously. Not all applications can do this.

Some applications require additional steps such as data migration when upgrading to the new version. In that case, you must use the Recreate strategy. It stops all of the running Pods first and then starts the new version Pods. You can configure additional steps before (pre) stopping the old Pods, after stopping them but before starting the new ones (mid), or after starting the new Pods (post).

### 3.2 Implement Recreate Deployment Strategy

#### 1. Run the `oc get dc ocp-probe -o yaml` command and find the `Rolling` setting in `spec.strategy.type`

#### 2. Open the DeploymentConfig and make the following changes:
* a. In the `strategy`section fin the line with `type: ` and change it from `Rolling` to `Recreate`
* b. Delete the line with `rollingParams:` and everything idented after that line in that section (lines with `timeSeconds`, `intervalSecons`, etc).
* c. Change the image version from `v0.4` to `v0.5`.
  
#### 3. Save the edit and return to the command line.

#### 4. Use the `oc get pods -w` command to watch the Pods being stopped and started:

`oc get pods -w`
NAME                 READY   STATUS              RESTARTS   AGE
ocp-probe-1-deploy   0/1     Completed           0          6h53m
ocp-probe-2-6wkx6    1/1     Running             0          82m
ocp-probe-2-cxg8h    1/1     Running             0          82m
ocp-probe-2-deploy   0/1     Completed           0          82m
ocp-probe-2-zdhzt    1/1     Running             0          82m
ocp-probe-3-deploy   0/1     ContainerCreating   0          7s
ocp-probe-3-deploy   0/1   ContainerCreating   0     8s
ocp-probe-2-6wkx6   1/1   Terminating   0     82m
ocp-probe-2-cxg8h   1/1   Terminating   0     82m
ocp-probe-2-zdhzt   1/1   Terminating   0     82m
ocp-probe-3-deploy   1/1   Running   0     9s
ocp-probe-2-cxg8h   1/1   Terminating   0     83m
ocp-probe-2-zdhzt   1/1   Terminating   0     83m
ocp-probe-2-6wkx6   1/1   Terminating   0     83m
ocp-probe-2-zdhzt   0/1   Terminating   0     83m
ocp-probe-2-6wkx6   0/1   Terminating   0     83m
ocp-probe-2-cxg8h   0/1   Terminating   0     83m
ocp-probe-2-zdhzt   0/1   Terminating   0     83m
ocp-probe-2-zdhzt   0/1   Terminating   0     83m
ocp-probe-2-cxg8h   0/1   Terminating   0     83m
ocp-probe-2-cxg8h   0/1   Terminating   0     83m
ocp-probe-2-6wkx6   0/1   Terminating   0     83m
ocp-probe-2-6wkx6   0/1   Terminating   0     83m
ocp-probe-3-vrnp4   0/1   Pending   0     0s
ocp-probe-3-vrnp4   0/1   Pending   0     0s
ocp-probe-3-vrnp4   0/1   ContainerCreating   0     0s
ocp-probe-3-vrnp4   0/1   ContainerCreating   0     8s
ocp-probe-3-vrnp4   1/1   Running   0     9s
ocp-probe-3-smnk7   0/1   Pending   0     0s
ocp-probe-3-smnk7   0/1   Pending   0     0s
ocp-probe-3-kswbk   0/1   Pending   0     0s
ocp-probe-3-kswbk   0/1   Pending   0     0s
ocp-probe-3-smnk7   0/1   ContainerCreating   0     0s
ocp-probe-3-kswbk   0/1   ContainerCreating   0     0s
ocp-probe-3-deploy   0/1   Completed   0     60s
ocp-probe-3-deploy   0/1   Completed   0     60s
ocp-probe-3-kswbk   0/1   ContainerCreating   0     7s
ocp-probe-3-kswbk   1/1   Running   0     8s
ocp-probe-3-smnk7   0/1   ContainerCreating   0     8s
ocp-probe-3-smnk7   1/1   Running   0     9s

* Note that all of the Pods from revision `2` were terminated first and only afterwards the new Pods (revision `3`) were started one by one.
  
#### 5. Add a section with recreation parameters and specify `pre`, `mid` and `post` steps:
~~~yaml
strategy:
  type: Recreate
  recreateParams:
    pre: {}
    mid: {}
    post: {}
~~~

* These steps are called lifecycle hooks.

