# OpenShift Command Line Tool
 `code`   oc
  Download from:
        https://cloud.redhat.com/openshift/install → any provider → Download Command-line Tools
        Your cluster: https://downloads-openshift-console.apps.${CLUSTER_DOMAIN_NAME}/amd64/linux/oc (for Linux®)
    
    Copy to your $PATH
        Tab completion
            $ oc completion bash > oc_bash_completion
            $ sudo cp oc_bash_completion /etc/bash_completion.d/

## Cluster Login
`oc login -u USERNAME https://api.${CLUSTER_DOMAIN_NAME}:6443`

Options:

* -p: Provide password (not recommended)

* --certificate-authority='': Path to cert file (to avoid warnings)

* --insecure-skip-tls-verify=false: Removes HTTPS protection

* --token: Bearer token for authentication

## Session Information
`oc whoami`

`oc whoami --show-server`

`oc whoami --show-console`

`oc whoami --show-token`

`oc version or oc version --short`


https://docs.openshift.com/container-platform/4.1/cli_reference/getting-started-cli.html
https://docs.openshift.com/container-platform/4.1/cli_reference/developer-cli-commands.html
https://docs.openshift.com/container-platform/4.1/cli_reference/administrator-cli-commands.html
https://blog.openshift.com/oc-command-newbies/
https://kubernetes.io/docs/reference/kubectl/cheatsheet/

## 1- create a new application:
`oc new-app cakephp-mysql-example`

## 2 - Follow the suggestions in the command output and use the oc status command:
`oc status`

## 3 - Watch the build process:
`oc logs -f pod/cakephp-mysql-example-1-build`

## 4 - watch the deployments process:
`oc logs -f pod/cakephp-mysql-example-1-deploy`

## 5 - Wait until the deployment step is complete, then explore the resources created for you in this project:
`oc get all`

NAME                                   READY   STATUS      RESTARTS   AGE
pod/cakephp-mysql-example-1-build      0/1     Completed   0          78m
pod/cakephp-mysql-example-1-deploy     0/1     Completed   0          75m
pod/cakephp-mysql-example-1-hook-pre   0/1     Completed   0          75m
pod/cakephp-mysql-example-1-zfkdk      1/1     Running     0          75m
pod/mysql-1-deploy                     0/1     Completed   0          78m
pod/mysql-1-qbsvb                      1/1     Running     0          78m

NAME                                            DESIRED   CURRENT   READY   AGE
replicationcontroller/cakephp-mysql-example-1   1         1         1       75m
replicationcontroller/mysql-1                   1         1         1       78m

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/cakephp-mysql-example   ClusterIP   172.30.116.45   <none>        8080/TCP   78m
service/mysql                   ClusterIP   172.30.89.114   <none>        3306/TCP   78m

NAME                                                       REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfig.apps.openshift.io/cakephp-mysql-example   1          1         1         config,image(cakephp-mysql-example:latest)
deploymentconfig.apps.openshift.io/mysql                   1          1         1         config,image(mysql:5.7)

NAME                                                   TYPE     FROM   LATEST
buildconfig.build.openshift.io/cakephp-mysql-example   Source   Git    1

NAME                                               TYPE     FROM          STATUS     STARTED             DURATION
build.build.openshift.io/cakephp-mysql-example-1   Source   Git@133cb8b   Complete   About an hour ago   3m26s

NAME                                                   IMAGE REPOSITORY                                                                                                                     TAGS     UPDATED
imagestream.image.openshift.io/cakephp-mysql-example   default-route-openshift-image-registry.apps.shared.na.openshift.opentlc.com/gpsilva-fedora31-openshift-tools/cakephp-mysql-example   latest   About an hour ago

NAME                                             HOST/PORT                                                                                     PATH   SERVICES                PORT    TERMINATION   WILDCARD
route.route.openshift.io/cakephp-mysql-example   cakephp-mysql-example-gpsilva-fedora31-openshift-tools.apps.shared.na.openshift.opentlc.com          cakephp-mysql-example   <all>                 None

### Note the following:

Two Pods are running — one with a MySQL database and one with the application itself. The other Pods build and deploy container images. They performed their functions and show a Completed status.

Two ReplicationControllers ensure that a predefined number of Pods are running all the time. In this case, there is one database Pod and one application Pod.

Two Services--again, one for the database and one for the application. Their role is load-balancing and providing a permanent address for the Pods behind them.

Two DeploymentConfigs, each responsible for deploying application components and redeploying them when necessary.

The BuildConfig, Build, and ImageStream objects create and manage your application’s container images. When you used the new-app command you gave OpenShift the address of the source code repository, OpenShift built a container image out of that source code and placed it in the internal image registry. The deployment watches for changes in the image stream to determine whether to redeploy your application.

The Route is used to access your application from outside the cluster. Similar to the Service, it provides a permanent path to your application, even when the Pods behind it are restarted and assigned new IP addresses.

# Explore Application Resources

## Explore Pods
`oc get pods`

NAME                                READY   STATUS      RESTARTS   AGE
cakephp-mysql-example-1-86vf5       1/1     Running     0          9m38s
cakephp-mysql-example-1-build       0/1     Completed   0          13m
cakephp-mysql-example-1-deploy      0/1     Completed   0          9m56s
cakephp-mysql-example-1-hook-pre    0/1     Completed   0          9m48s
httpd-deployment-777449dd8b-2qrj2   1/1     Running     0          44h
httpd-deployment-777449dd8b-b5scz   1/1     Running     0          44h
mysql-1-deploy                      0/1     Completed   0          13m
mysql-1-mz8mb                       1/1     Running     0          12m


`oc get pods --field-selector status.phase=Running`

NAME                            READY   STATUS    RESTARTS   AGE
NAME                                READY   STATUS    RESTARTS   AGE
cakephp-mysql-example-1-86vf5       1/1     Running   0          10m
httpd-deployment-777449dd8b-2qrj2   1/1     Running   0          44h
httpd-deployment-777449dd8b-b5scz   1/1     Running   0          44h
mysql-1-mz8mb                       1/1     Running   0          13m


`oc get pods ccakephp-mysql-example-1-86vf5`
AME                            READY   STATUS    RESTARTS   AGE
cakephp-mysql-example-1-86vf5   1/1     Running   0          12m

`oc get pods cakephp-mysql-example-1-zfkdk -o yaml` */ file saved on the same path (cakephp-mysql-example-1-86vf5.yaml)
### Explore the YAML output—in particular, look for the following fields (the dot notation indicates that the YAML structure is nested):

* metadata.labels: OpenShift uses labels to identify and select components.

* metadata.name: The name of the Pod.

* spec.containers.image: The image used to deploy this container.

* spec.containers.ports: The network ports used by this container.

* spec.containers.resources: The resources requested by this container.

* spec.volumes: Storage volumes used by this Pod. In this case, a Secret is mounted as a volume in the application container.

* status.phase: The status of the Pod. In this case, it is running. You used this field to filter the Pods. You can use other fields for that purpose as well.

## Explore ReplicationController

`oc get rc cakephp-mysql-example-1 -o yaml` */ file saved on the same path (cakephp-mysql-example-1.yaml)

### In the YAML output, examine the following fields:

* metadata.ownerReferences: Note that the owner of this ReplicationController is a DeploymentConfig called cakephp-mysql-example. Using this information you can build a tree of relationships in this application.

* spec.replicas: Specifies the number of replicas to run. Even if you do not need more than one copy of a Pod, you still use a ReplicationController to make sure the Pod is restarted if it goes down for some reason.

* spec.template.spec.containers: Specifies the containers managed by this ReplicationController. You can find the environment variables passed to each container, the image to use, the ports, the health check probes, and other parameters.
  
### Determining the owner of this ReplicationController

`oc get dc cakephp-mysql-example -o yaml` */ file saved at the same path (cakephp-mysql-example.yaml)

#### Examine the output and in particular the following fields:

* spec.strategy: This section describes the rollout strategy chosen for this deployment. In this case, its type is Recreate (the other strategy is called Rolling).

    - In the same section, locate the recreateParams subsection. The application developers defined a pre hook for this deployment. It creates a Pod that runs a database migration script. You may have noticed a Completed Pod with the name cakephp-mysql-example-1-hook-pre. This is the Pod that performed the step.

    - You can check to see what happened in that Pod by running oc logs cakephp-mysql-example-1-hook-pre.

* spec.triggers: Specifies the triggers that cause a new rollout of this DeploymentConfig.

  - One of the triggers has a type of ConfigChange. This trigger causes the entire DeploymentConfig to be redeployed each time you change the DeploymentConfig—for example, a resource request or health check parameters.

  - The other trigger is ImageChange. This causes redeployment when the image used for containers is changed.

* status.conditions: Specifies when this DeploymentConfig was rolled out previously and whether it was successful. This information can be very helpful in troubleshooting the deployment.

## Explore Database Service

### Retrieve the mysql Service details:

`oc get svc mysql -o yaml` */ file saved at the same path (mysql.yaml)

#### Examine the following fields of the database Service:

* metadata.labels.app:  This label is used to group all of the OpenShift objects that belong to the same application. You may recall that the same app label is in the Pod and DeploymentConfig YAML manifests, and its value (cakephp-mysql-example) is the same.

* spec.clusterIP: This is the IP address to find this database service in the cluster. When Pods are restarted, they get a new IP address. Giving them a permanent address makes it easier to locate them. OpenShift Service objects provide such addresses.

* spec.ports.port: You use the default MySQL port, 3306.

* spec.selector.name: This is how the Service finds its Pods. Pods with a label name: mysql are selected and included in this Service. Your MySQL pod must have a metadata.labels.name set to the same value of mysql. (Be sure to use metadata.labels.name, not metadata.name, when configuring the Pod.)
  
### Explore Route
Display the Route object:
`oc get route cakephp-mysql-example -o yaml`

* If Service objects are used to access application Pods inside the cluster, this object creates an external route to access the application from the outside.

Examine its fields:

* spec.host: This is the external domain name of the application. You can enter it as a URL in your browser and find the application’s front page.
  
* spec.to.kind and spec.to.name: Define that this route is to use the cakephp-mysql-example Service to find the Pods to send traffic to. Note the weight option. It is used if you want to send part of the traffic to another set of Pods, such as in blue/green or canary deployment strategies. You learn about that later.

## Create Application from YAML Files

### Create and Configure Application

`oc new-project $GUID-yaml-app`

Wrote and saved a yaml file named httpd-deployment-gps.yaml at this same path

### Create this Deployment:

`oc create -f httpd-deployment-gps.yaml`

### Resutls:

`oc get all`

NAME                                    READY   STATUS    RESTARTS   AGE
pod/httpd-deployment-777449dd8b-lztvl   1/1     Running   0          11s
pod/httpd-deployment-777449dd8b-pnmbq   1/1     Running   0          11s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/httpd-deployment   2/2     2            2           13s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/httpd-deployment-777449dd8b   2         2         2       13s

- As expected, there is a Deployment and a ReplicaSet with two Pod replicas.

- The Pods are running, but cannot be accessed. Must have a Service in front of the Pods. A Service acts as a load balancer in front of Pods. The Service knows which pods to use as a back end by using labels and selectors.
  
#### Examine the lables associated with the Pods:

`oc get pods --show-labels`

NAME                                READY   STATUS    RESTARTS   AGE     LABELS
httpd-deployment-777449dd8b-lztvl   1/1     Running   0          3m52s   app=httpd,pod-template-hash=777449dd8b
httpd-deployment-777449dd8b-pnmbq   1/1     Running   0          3m52s   app=httpd,pod-template-hash=777449dd8b

*  both Pods have the app=httpd label. You configured the Pods to use these labels.

### Configure Service Selector

`oc create -f httpd-service.yaml` */ saved at the same path

`oc get all`

NAME                                    READY   STATUS    RESTARTS   AGE
pod/httpd-deployment-777449dd8b-lztvl   1/1     Running   0          88m
pod/httpd-deployment-777449dd8b-pnmbq   1/1     Running   0          88m

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/httpd-deployment   ClusterIP   172.30.203.132   <none>        8080/TCP   15s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/httpd-deployment   2/2     2            2           88m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/httpd-deployment-777449dd8b   2         2         2       88m

### Create a Route (from manifest httpd-route.yaml)

`oc create -f httpd-route.yaml`

### Route created:

`oc get all`

NAME                                    READY   STATUS    RESTARTS   AGE
pod/httpd-deployment-777449dd8b-lztvl   1/1     Running   0          94m
pod/httpd-deployment-777449dd8b-pnmbq   1/1     Running   0          94m

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/httpd-deployment   ClusterIP   172.30.203.132   <none>        8080/TCP   5m56s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/httpd-deployment   2/2     2            2           94m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/httpd-deployment-777449dd8b   2         2         2       94m

NAME                                        HOST/PORT                                                            PATH   SERVICES           PORT   TERMINATION   WILDCARD
route.route.openshift.io/httpd-deployment   httpd-deployment-gps-yaml-app.apps.shared.na.openshift.opentlc.com          httpd-deployment   8080                 None

### Conclusion

After completing this lab you should be able to:

* Create projects (a.k.a. namespaces) in OpenShift

* Create a simple multilayered application with oc new-app

* Use oc new-app parameters and options to customize an application

* Create a multilayered application with a YAML manifest

* Explore an application’s components and resources

* Use labels, selectors, and annotations

* Use oc get to display information about an application’s resources

* Delete an application

