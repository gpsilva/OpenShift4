## 2 Explore Blue/Green Strategy

The Rolling strategy discussed previously uses a Canary deployment approach. It starts a new Pod and waits until it is ready. If something is wrong with the new version and the first Pod never gets ready, the DeploymentConfig cancels the upgrade and rolls back the previous working version.

In some situations a simple readiness test may not be sufficient to determine whether the new version is working properly. Sometimes you may want to start the new version and run it for a while, and be ready to revert to the previous version if something goes wrong.

This approach is called blue/green deployment and it uses two versions of the application running simultaneously. First, you use the Route object to point to the current version. Then you start the new version and switch the Route to the new pods. After you test the new version and are satisfied with it, you can scale down the previous one and delete that Deployment.

In this lab, you use the same ocp-probe application you used in the previous labs. This current version is v0.4 and referred to as the green version.

The application supports several REST API calls. A simple curl command specifying /version as the path returns the application version.

### 2.1 Implement Blue/Green Strategy

#### 1. Create a new project
`oc new-project $GUID-bluegreen --display-name="Blue-Green Deployments" `

#### 2. Create the application:
`oc new-app --docker-image=quay.io/gpte-devops-automation/ocp-probe:v0.4 --name=green`

* Note the `--name` parameter at the end.

#### 3. Wait until see that the application's Pods are in the `Running` state:
`oc get pods`
NAME             READY   STATUS      RESTARTS   AGE
green-1-b9k7r    1/1     Running     0          51s
green-1-deploy   0/1     Completed   0          59s

#### 4. Determine whether the Service was created for this application:
`oc get svc`
NAME    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
green   ClusterIP   172.30.111.250   <none>        8080/TCP   3m32s

* By default it has the same name as the application.

### 2.2 Make Service Visible Outside Cluster

This Service is visible only for clients inside the cluster. In this section, you create a Route using the oc expose command to make it visible from outside the cluster. You use the --name parameter to specify the Route’s name. This Route serves both the blue and green Deployments, so call it bluegreen.

#### 1. Create a Route specifying the `--name` parameter `bluegreen:`
`oc expose svc green --name=bluegreen`
route.route.openshift.io/bluegreen exposed

#### 2. Discover the URL for the application from outside the cluster:
`oc get route`
AME        HOST/PORT                                                       PATH   SERVICES   PORT       TERMINATION   WILDCARD
bluegreen   bluegreen-4e41-bluegreen.apps.shared.na.openshift.opentlc.com          green      8080-tcp                 None

#### 3. Use `curl` to check the version of the deployed application, using the URL of application's Route from the previous command's output. Create an environment variable for the Route:
`export ROUTE=$(oc get route bluegreen -o jsonpath='{.spec.host}')`

`curl $ROUTE/version`
0.4

### 2.3 Create Blue Deployment

#### 1. Create a Deployment for the application's new version, named `blue`
`oc new-app --docker-image=quay.io/gpte-devops-automation/ocp-probe:v0.5 --name=blue`

#### 2. Wait for the application Pod to enter the `Running` state and verify that the Service is Available:
`oc get pods`
NAME             READY   STATUS      RESTARTS   AGE
blue-1-c2mr5     1/1     Running     0          32s
blue-1-deploy    0/1     Completed   0          40s
green-1-b9k7r    1/1     Running     0          30m
green-1-deploy   0/1     Completed   0          30m

`oc get svc`
NAME    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
blue    ClusterIP   172.30.244.38    <none>        8080/TCP   52s
green   ClusterIP   172.30.111.250   <none>        8080/TCP   30m

### 2.4 Switch Route to Blue

This time you do not need to run oc expose to expose the Service because you already have an external Route pointing to the green Service.

In this section, you switch the Route to the blue Service. There are a couple ways to do this. First, you use the oc edit command as it is the easiest way and helps you to find the right place in the Route’s manifest. Later, you use the oc patch command to perform the switch. Using oc patch is useful if you want to automate switching between blue and green deployments.

#### 1. Open the Route's manifest for editing with `oc edit route bluegreen` and find the `spec` section:
~~~yaml
spec:
  host: bluegreen-55bb-bluegreen.apps.shared.na.openshift.opentlc.com
  port:
    targetPort: 8080-tcp
  subdomain: ""
  to:
    kind: Service
    name: green
    weight: 100
  wildcardPolicy: None
  ~~~

#### 2. In the `name:` line replace `green` with `blue`:
~~~yaml
spec:
  host: bluegreen-55bb-bluegreen.apps.shared.na.openshift.opentlc.com
  port:
    targetPort: 8080-tcp
  subdomain: ""
  to:
    kind: Service
    name: blue
    weight: 100
  wildcardPolicy: None
  ~~~

#### 3. Rerun `curl` to verify that the new version is deployed. Replace the previous route with actual route:
`curl $ROUTE/version`
0.5

### 2.5 Use command line to switch deployments

The oc patch command is used to switch between deployments. This is useful if you want to automate the process or if you want to include this step in a CI/CD pipeline.

#### 1. Switch it back to `green` using the command line:
`oc patch route/bluegreen -p '{"spec":{"to":{"name":"green"}}}'`
route.route.openshift.io/bluegreen patched

#### 2. Verify that the version reverted to `0.4`
`curl $ROUTE/version`
0.4

## 3. Use Weights for A/B Testing and Canary Deployments

About the weight: parameter in the Route’s manifest. In blue/green scenarios, you switch the Route from one backing Service to another with a weight equal to 100%. The weight parameter is particularly suited to canary deployments because it controls the amount of traffic, such as five percent, to send to either of the two active deployments.

It is also used in A/B testing to split traffic equally so that you can compare the response rate or some other metric.

### 3.1 Use A/B Testing

#### 1. Configure the deployments in A/B testing mode by editing the Route to add `alternateBackends` to the `spec:` section:

`oc edit route bluegreen`
~~~yaml
spec:
  host: bluegreen-55bb-bluegreen.apps.shared.na.openshift.opentlc.com
  port:
    targetPort: 8080-tcp
  subdomain: ""
  to:
    kind: Service
    name: green
    weight: 50
  alternateBackends:
  - kind: Service
    name: blue
    weight: 50
~~~

#### 2. Check the version of the service every second to test the deployment:

`while true; do curl $ROUTE/version ; echo ""; sleep 1; done`
0.5
0.4
0.4
0.5
0.5
0.4

#### 3. On the YAML text `alternateBackends` is a YAML list whose elements start with `-` (hypen).

* This means that can have morte than two Service back ends in case want to test three or more versions of the application.
  
### 3.2 Implement Canary Deployment Strategy

In this section, we change the weights to 9 for green and 1 for blue to implement a canary deployment strategy.

* The weights many not add up to 100.
* When you open the Route with oc edit, you may see the elements in a slightly different order. This is because the Route manifest is not kept as a YAML file but rather stored in the etcd database. Each time you retrieve it from the database for editing, the sub-elements in each section are ordered alphabetically.

#### 1. Edit the route:
~~~yaml
spec:
  alternateBackends:
  - kind: Service
    name: blue
    weight: 1
  host: bluegreen-55bb-bluegreen.apps.shared.na.openshift.opentlc.com
  port:
    targetPort: 8080-tcp
  subdomain: ""
  to:
    kind: Service
    name: green
    weight: 9
~~~

#### 2. Save it and run the `while` loop again:
`while true; do curl $ROUTE/version ; echo ""; sleep 1; done`    
0.4
0.4
0.4
0.4
0.4
0.4
0.4
0.4
0.4
0.4
0.4
0.4
0.4
0.4
0.4
0.5
0.4
0.5
0.4
0.4

* Because it was set the probability for the blue deployment to be chosen to 10 percent, it takes a while for version 0.5 to appear.

### 3.3 Run Canary Deployment from command line

It's possible to achieve the same results using the command line. After completed tested and want to switch 90 percent of the traffic to the 0.5 version but leave 10 percent on the 0.4 version just in case.

#### 1. Set the Route back-end weights:
`oc set route-backends bluegreen blue=9 green=1`
route.route.openshift.io/bluegreen backends updated

#### 2. The results:
`while true; do curl $ROUTE/version ; echo ""; sleep 1; done`
0.5
0.5
0.5
0.5
0.5
0.5
0.5
0.4
0.5
0.5
0.5
0.5
0.5
0.5


