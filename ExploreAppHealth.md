## Build and Deploy Application

`oc new-project $GUID-app-management`
`oc new-app cakephp-mysql-example`

## Examine Quotas

`oc get appliedclusterresourcequotas`

NAME                                            LABEL SELECTOR   ANNOTATION SELECTOR
clusterquota-gilvan.silva-certsys.com.br-4e41   <none>           map[openshift.io/requester:gilvan.silva-certsys.com.br]

### Examine assigned quotas:

`oc describe appliedclusterresourcequota clusterquota-gilvan.silva-certsys.com.br-4e41`

Name:		clusterquota-gilvan.silva-certsys.com.br-4e41
Created:	8 days ago
Labels:		<none>
Annotations:	<none>
Namespace Selector: ["4e41-app-management"]
Label Selector: 
AnnotationSelector: map[openshift.io/requester:gilvan.silva-certsys.com.br]
Resource		Used	Hard
--------		----	----
configmaps		2	100
limits.cpu		1	10
limits.memory		1Gi	20Gi
persistentvolumeclaims	0	20
pods			2	20
requests.cpu		100m	5
requests.memory		1Gi	6Gi
requests.storage	0	50Gi
secrets			10	150
services		2	150

* As shown, there is used 1 CPU core (virtual CPU) and 1Gi of memory.

#### Determine which Pods requested those resources using some new techniques:

`oc get pods --field-selector=status.phase=Running -o json  | jq '.items[] | {name: .metadata.name, res: .spec.containers[].resources}'`

* The --field-selector option causes the selected Pods to include only those that are running.

* The output is piped to the jq command to display only the fields that you are interested in, such as the Pod’s name and its resources (both requests and limits, both CPU and memory).

## Explore Memory Limits and Quotas

### Edit the cakephp-mysql-example DeploymentConfig:

- Change the memory limits to 50Gi so that the resources section of the YAML manifest

- Save your edits and exit from the editor.

- Immediately invoke the watch oc get pods command:
  
    - `watcg ic get pods`
- Expect the attempt to deploy a new version of the cakephp-mysql-example DeploymentConfig (cakephp-mysql-example-2-deploy) to display an error.
- As a result of the failed deployment, the running good container cakephp-mysql-example-1-dpnp9 was not terminated and continues to run.

#### Examine the log to determine why cakephp-mysql-example-2 did not start:

`$ oc logs -f dc/cakephp-mysql-example`
error: pre hook failed: the pre hook failed: couldn't create lifecycle pod for cakephp-mysql-example-2: pods "cakephp-mysql-example-2-hook-pre" is forbidden: [maximum memory usage per Container is 6Gi, but limit is 50Gi., maximum memory usage per Pod is 12Gi, but limit is 53687091200.], aborting rollout of 4e41-app-management/cakephp-mysql-example-2

#### Use the command line to change the container’s memory limit resources to 1Gi:
`oc set resources dc cakephp-mysql-example --limits=memory=1Gi`
deploymentconfig.apps.openshift.io/cakephp-mysql-example resource requirements updated

#### Watch the output of oc get pods and note that the updated deployment succeeds:
Every 2,0s: oc get pods                                                                                         gpsilva-fedora: Wed May  6 20:21:31 2020

NAME                               READY   STATUS      RESTARTS   AGE
cakephp-mysql-example-1-build	   0/1     Completed   0          118m
cakephp-mysql-example-1-deploy     0/1     Completed   0          115m
cakephp-mysql-example-1-hook-pre   0/1     Completed   0          115m
cakephp-mysql-example-2-deploy     0/1     Error       0          21m
cakephp-mysql-example-3-deploy     0/1     Completed   0          16m
cakephp-mysql-example-3-hook-pre   0/1     Completed   0          16m
cakephp-mysql-example-3-kfkrf	   1/1     Running     0          16m
mysql-1-deploy                     0/1     Completed   0          118m
mysql-1-fjqm8                      1/1     Running     0          80m
mysql-2-deploy                     0/1     Error       0          91m

* Revision 3 of the DeploymentConfig starts and completes the deployment.

* Note that the new cakephp-mysql-example-3-kfkrf pod is up and running.
  
### Conclusion

* Describe what health checks are and why they are useful

* Change health checks in the DeploymentConfig

* Describe a container’s resource constraints

* Change resource constraints both in a YAML manifest and using the command line

* Find and debug deployment issues

