# Service Accounts

* Service accounts are non-person identities for application integrations:

    - Each pod runs as service account
    - Used by external agents to access cluster API
    - Access managed with role bindings and cluster role bindings

- Examples:
  - DeploymentConfigs use deployer pod that runs with deployer service account
  - Applications in pod containers make API calls for discovery
  - Jenkins servers use API to create agents running in containers
  - Operators use cluster API to watch custom resources and API to manage ConfigMaps, deployments, etc.

## Default Service Accounts

Every project namespace starts with three standard service accounts: builder, deployer, and default.

- builder is used for builds and has access to push images to image streams in the namespace.
- deployer is used by DeploymentConfigs, particularly for deployer pods.
- The deployer pods are responsible for managing application deployment by creating and scaling replication controllers.
- The default service account has no special access and is used by normal application pods.

### Service Account Management

- Service management commands:
  - Create service accounts: `oc create serviceaccount NAME -n NAMESPACE`
  - List service accounts: `oc get serviceaccount NAME -n NAMESPACE`
  - Describe service accounts: `oc describe serviceaccount NAME -n NAMESPACE`
  - Delete service accounts: `oc delete serviceaccount NAME -n NAMESPACE`

### Service Account Tokens

- Service accounts use tokens to authenticate to cluster API
- Within running container, find service account token in /run/secrets/kubernetes.io/serviceaccount/token file
- Tokens used by external agents to act as service account with OpenShift® API
  - Allow external agent to access integrated container image registry
  - Allow existing Jenkins server to run agents as pods within cluster

### Accessing Service Account Tokens

* Service account tokens are stored as secrets within project namespaces.
  - List service account tokens:
`oc get secret --field-selector=type=kubernetes.io/service-account-token -n NAMESPACE`

  - Get active token for service account:
`oc serviceaccount get-token SERVICE_ACCOUNT -n NAMESPACE`

  - Get specific token from secret:
`oc describe secret SECRET -n NAMESPACE`

### Service Account Token Rotation

* By default, service account tokens are not rotated, but it is common security policy to rotate all credentials.
* To rotate a service account token, simply delete it.
* The deleted token is immediately disabled and a replacement token is automatically generated.
  * Pods using deleted secret need to be restarted
  * External services need updated credentials
* Replacement token automatically created
  * OpenShift maintains at least two token secrets per service account

### RBAC for Service Accounts

* Service account username: 
  - system:serviceaccount:NAMESPACE:SERVICE_ACCOUNT
   
* Service account group: system:serviceaccounts:NAMESPACE
  * Represents all service accounts in project namespace
  * serviceaccounts is plural in group name
   
* Cluster role bindings, role bindings managed in same way as users, groups
  * If service account in same namespace as role binding, oc policy can use option:
`--serviceaccount=SERVICE_ACCOUNT` Or `-z`

#### Examples
1. Add role binding in monitored-project namespace to grant view cluster role to monitor-agent service account in monitor namespace:
`oc policy add-role-to-user view -n monitored-project \  system:serviceaccount:monitor:monitor-agent`

2. Add role binding in myapp-build namespace to allow pods in myapp-dev to pull images from image streams:
`oc policy add-role-to-group system:image-puller -n myapp-build      system:serviceaccounts:myapp-dev`

### Image Pull Secrets

* Image pull secrets provide credentials used to pull images for pod containers
* Create image pull secret:
`oc create secret docker-registry SECRET_NAME -n NAMESPACE \`
`--docker-username=USERNAME \`
`--docker-password=PASSWORD \`
`--docker-email=EMAIL`

Link image pull secret to service account:
`oc secrets link --for=pull SERVICE_ACCOUNT SECRET_NAME -n NAMESPACE`

### Security Context Constraints

* Role-based access control controls what users can do
* Security context constraints (SCCs), in contrast, control:
  * Actions pod can perform
  * What pod can access
* SCCs define conditions pod must run with to be accepted into system
  
* SCCs let administrator control:
  * Capabilities container can request to be added
  * Use of host directories as volumes
  * SELinux context of container
  * User ID
  * Use of host namespaces and networking
  * Allocation of FSGroup that owns pod’s volumes
  * Configuration of allowable supplemental groups
  * Requirement for use of read-only root file system
  * Usage of volume types
  * Configuration of allowable secure computing mode (seccomp) profiles

#### SCC Management

* Grant access to SCCs by adding service account username to SCC
* Managing SCC access within SCC definition:
  
`oc adm policy add-scc-to-user SCC_NAME USER_NAME`

`oc adm policy add-scc-to-group SCC_NAME GROUP_NAME`

`oc adm policy remove-scc-from-user SCC_NAME USER_NAME`

`oc adm policy remove-scc-from-group SCC_NAME USER_NAME`

* If service account listed in multiple SCC entries, SCC priority applied

#### Management of SCC Access Through RBAC

* An alternative to directly managing SCC membership is to use role-based access control to add security context constraints:

1. Create a cluster role with use verb on SCC:

~~~yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: scc-SCC_NAME
rules:
- apiGroups:
  - security.openshift.io
  resourceNames:
  - nonroot
  resources:
  - SCC_NAME
  verbs:
  - use
~~~

2. Add access to SCC with role binding:
`oc adm policy add-role-to-user scc-SCC_NAME \`
`system:serviceaccount:NAMESPACE:SERVICEACCOUNT`

### Troubleshooting Service Accounts

1.  Service Accounts in Pods
  * Discover which service account pod uses:
    * Found in pod *spec.serviceAccount* but deprecated
  * List pods with service accounts:
`oc adm policy add-role-to-user scc-SCC_NAME \`
    `system:serviceaccount:NAMESPACE:SERVICEACCOUNT`

2.  Service Account Impersonation
  * Service accounts can be impersonated by project administrators:
`oc --as=system:serviceaccount:NAMESPACE:SERVICE_ACCOUNT \`
`     --as-group=system:serviceaccounts:NAMESPACE ...`

* Example: check if monitor agent can list pods in monitored project:
`oc get pods -n monitored-project \`
`    --as=system:serviceaccount:monitor:monitor-agent \`
`    --as-group=system:serviceaccounts:monitor`

3. Accessing cluster API with ServiceAccount Token
   * Fetch service account token and use it to authenticate to cluster API:
`TOKEN=$(oc serviceaccounts get-token SERVICE_ACCOUNT -n NAMESPACE)`
`oc --token=$TOKEN ...`

    * Example: Test if operator can list deployments in namespace:
`TOKEN=$(oc serviceaccounts get-token operator -n app-operator)`
`oc --token=$TOKEN get deployment -n app-dev`


