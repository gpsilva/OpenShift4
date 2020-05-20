# Role-Based Access Control

* RBAC objects determine whether user allowed to perform specific action with regard to type of resource
  - OpenShift® RBAC controls access—if RBAC does not allow access, access denied by default

* Roles: Scoped to project namespaces, map allowed actions (verbs) to resource types in namespace

* ClusterRoles: Cluster-wide, map allowed actions (verbs) to cluster-scoped resource types or resource types in any project namespace

* RoleBindings: Grant access by associating Roles or ClusterRoles to users or groups for access within project namespace

* ClusterRoleBindings: Grant access by associating ClusterRoles to users or groups for access to cluster-scoped resources or resources in any project namespace

## Roles and Rules

* Roles, cluster roles = collections of rules

* Rule = list of verbs permitted on listed subjects
    - Examples: create, delete, get, list, patch, update

* Rule subjects are generally resources accessed through API groups
    - List available resources and api-groups with oc api-resources
    - Subject may be constrained to specific resource names

## Standard Resource Management Verbs

* create            Create resource
* delete            Delete resource
* deletecolletion   Delete colletion of resources
* get               Get resource
* list              Get multiple resources
* patch             Apply patch to change resource
* update            Update resource
* watch             Watch for changes on websocket

## Describing Cluster Roles

*   Use oc describe clusterrole to visualize roles in cluster RBAC
    - Includes matrix of verbs and resources associated with role
    - Lists additional system roles used for OpenShift operations
    - For full details use oc get clusterrole -o yaml

## Describing Roles

* Use oc describe role -n NAMESPACE to visualize roles in project namespace
    - Custom role definitions can be added to project namespaces
    - Custom role can only add access that user creating it possesses
    - For full details use oc get role -n NAMESPACE -o yaml

## Important Cluster Roles

* admin: 
  * Project namespace administrator
  * Rights to manage most resource types in project namespace
  * Can manage RoleBindings within namespace
  * Does not include access to manage ResourceQuotas, LimitRanges, custom resource types

* basic-user: 
  * Can get basic information about projects and users

* cluster-admin:
  * Can perform any action on any resource type
  * Not intended for use with RoleBindings on namespaces as this permits override of OpenShift security features such as project namespace node restrictions

* edit:
  * Can modify most objects in project
  * Can use oc exec and oc rsh to execute arbitrary commands in containers
  * Cannot view or modify roles or role bindings

* self-provisioner:
  * Can create own projects
  * Automatic administrator of self-provisioned projects
  * Default for all authenticated users

* sudoer:
  * Access to impersonate system:admin user for full access
  * Used with oc --as=system:admin ...

* system:image-puller:
  * Ability to pull container images from image streams in project namespace
  * Used when build and deployment project namespaces separated
  * Used when container images need to be pulled remotely from cluster’s integrated registry

* system:image-pusher
  * Ability to push container images into image streams in project namespace
  * Used when container images need to be pushed remotely into cluster’s integrated registry

* view:
  * Can view most objects in project
  * Cannot make any modifications
  * Cannot view or modify roles, role bindings, or secrets

## Describing Cluster Roles Example
`oc describe clusterrole basic-user`
Name:         basic-user
Labels:       <none>
Annotations:  openshift.io/description: A user that can get basic information about projects.
              rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources                                           Non-Resource URLs  Resource Names  Verbs
  ---------                                           -----------------  --------------  -----
  selfsubjectrulesreviews                             []                 []              [create]
  selfsubjectaccessreviews.authorization.k8s.io       []                 []              [create]
  selfsubjectrulesreviews.authorization.openshift.io  []                 []              [create]
  clusterroles.rbac.authorization.k8s.io              []                 []              [get list watch]
  clusterroles                                        []                 []              [get list]
  clusterroles.authorization.openshift.io             []                 []              [get list]
  storageclasses.storage.k8s.io                       []                 []              [get list]
  users                                               []                 [~]             [get]
  users.user.openshift.io                             []                 [~]             [get]
  projects                                            []                 []              [list watch]
  projects.project.openshift.io                       []                 []              [list watch]
  projectrequests                                     []                 []              [list]
  projectrequests.project.openshift.io                []                 []              [list]


## Describing Role Bindings
* Example: View cluster role bindings

Use oc describe clusterrolebinding and oc describe rolebinding -n NAMESPACE
`oc describe clusterrolebinding cluster-admin cluster-admins`
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind   Name            Namespace
  ----   ----            ---------
  Group  system:masters  


Name:         cluster-admins
Labels:       <none>
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind   Name                   Namespace
  ----   ----                   ---------
  Group  system:cluster-admins  
  User   system:admin    

## Addition of Role Bindings in Namespaces

* Add cluster role to user to manage resources in namespace:
`oc policy add-role-to-user CLUSTER_ROLE USER -n NAMESPACE`

* Add namespace role to user to manage resources in namespace:
`oc policy add-role-to-user ROLE USER -n NAMESPACE --role-namespace=NAMESPACE`

* Add cluster role to group to manage resources in namespace:
`oc policy add-role-to-group CLUSTER_ROLE GROUP -n NAMESPACE`

* Add namespace role to group to manage resources in namespace:
`oc policy add-role-to-group ROLE GROUP -n NAMESPACE --role-namespace=NAMESPACE`

Create role bindings using oc apply, oc create or modify to add subjects using oc apply, oc patch, oc replace

## Removal of User Role Bindings from Namespaces

* Remove cluster role from user in namespace:
`oc policy remove-role-from-user CLUSTER_ROLE USER -n NAMESPACE`

* Remove namespace role from user in namespace:
`oc policy remove-role-from-user ROLE USER -n NAMESPACE --role-namespace=NAMESPACE`

* Remove all role bindings for user in namespace:
`oc policy remove-user USER -n NAMESPACE`

Remove role bindings using oc delete or modify to remove subjects using oc apply, oc patch, oc replace

## Removal of Group Role Bindings from Namespaces
Remove cluster role from group in namespace:
`oc policy remove-role-from-group CLUSTER_ROLE GROUP -n NAMESPACE`

* Remove namespace role from group in namespace:
`oc policy remove-role-from-group ROLE GROUP -n NAMESPACE --role-namespace=NAMESPACE`

* Remove all role bindings for group in namespace:
`oc policy remove-user GROUP -n NAMESPACE`

Remove role bindings using oc delete or modify to remove subjects using oc apply, oc patch, oc replace

## Cluster Role Binding Management

* Add cluster role to user:
`oc adm policy add-cluster-role-to-user CLUSTER_ROLE USER`

* Add cluster role to group:
`oc adm policy add-cluster-role-to-group CLUSTER_ROLE GROUP`

* Remove cluster role from user:
`oc adm policy remove-cluster-role-from-user CLUSTER_ROLE USER`

* Remove cluster role from group:
`oc adm policy remove-cluster-role-from-group CLUSTER_ROLE GROUP`

Manage cluster role bindings using oc apply, oc create, oc delete, oc patch, oc replace

## Project Self-Provisioning

* Normal users create project namespaces through project requests
  - oc new-project creates project request (projectrequests.project.openshift.io)
  - Project request contains description, display name
  - OpenShift creates project namespace for user with annotation openshift.io/requester: USER
  - Role binding for admin cluster role created for project requester
  - No access for requester to update project namespace after creation

* Cluster administrators often disable self-provisioning to more tightly control project management

### Implementation

  * self-provisioners cluster role allows users to create project requests
  * By default, all OAuth-authenticated users have self-provisioner access:
  `oc describe clusterrolebinding self-provisioners`
  Name:         self-provisioners
Labels:       <none>
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
Role:
  Kind:  ClusterRole
  Name:  self-provisioner
Subjects:
  Kind   Name                        Namespace
  ----   ----                        ---------
  Group  system:authenticated:oauth  

### Disabling Self-Provisioning

1. Set autoupdate annotation to false:
`oc annotate clusterrolebinding self-provisioners rbac.authorization.kubernetes.io/autoupdate=false --overwrite`

2. Then remove cluster role self-provisioner from system group system:authenticated:oauth:
`oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth`

## Custom Cluster Roles

* Create when no standard role provides needed functionality
* Avoid editing standard cluster roles
* Core cluster roles updated with Red Hat® OpenShift Container Platform version
* Editing standard role makes tracking platform updates difficult
* Create based on standard role plus or minus some aspects
* Create to aggregate privileges to standard role

### Creation

* Base custom cluster roles on standard roles
  - Get role definition with oc get clusterrole ROLE -o yaml >PATH_TO_ROLE_YAML
  - Set name metadata and openshift.io/description annotation
  - Update or remove aggregate-to settings
  - Add and remove rules, verbs, resources
  - Save definition as YAML file in version control
  - Create/update role with oc auth reconcile -f PATH_TO_ROLE_YAML

* The most difficult aspect of managing custom roles is often identifying what the role is supposed to do.
    - The openshift.io/description annotation is the recommended method of adding documentation to your roles.
    - Be sure to describe how the role was designed, including its purpose, which cluster role was used as reference, and how it differs from the standard role.

* Custom Role to Add Privileges to Standard Cluster Role
    - Use aggregate-to-* labels to add rules to standard cluster roles:
      - rbac.authorization.k8s.io/aggregate-to-admin
      - rbac.authorization.k8s.io/aggregate-to-edit
      - rbac.authorization.k8s.io/aggregate-to-view

Example: Granting Access to Manage Custom Resource Kind

~~~yaml
apiVersion: authorization.openshift.io/v1
kind: ClusterRole
metadata:
  name: gogs-admin-rules
  labels:
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
rules:
- apiGroups:
  - example.com
  resources:
  - widgets
  - widgets/status
  verbs:
  - create
  - delete
  - deletecollection
  - get
  - list
  - patch
  - update
  - watch
~~~

## Management

* Custom role creation follows similar process to cluster role creation except:
  - Project administrators can create custom roles
  - Roles scoped in namespace
  - Can be associated only with role bindings in namespace
  - Roles often based on cluster roles but with resource kind changed to Role
  
* When project administrator creates role, it may not include access that exceeds administrator’s own access
  - Cannot use wildcards (*) for apiGroups, resources, or verbs

## Troubleshooting

* Access Checks
  - To determine if you can perform specific verb on kind of resource:
`oc auth can-i VERB KIND [-n NAMESPACE]`

Examples:
    - Check access to patch namespaces:
`oc auth can-i patch namespaces`

    - Check access to list pods in openshift-authentication namespace:
`oc auth can-i get pods -n openshift-authentication`

From within OpenShift project, determine which verbs you can perform against all namespace-scoped resources:
`oc policy can-i --list`

- Discover which users can perform action on resource:
oc policy who-can VERB KIND
`oc policy who-can get imagestreams -n openshift`
resourceaccessreviewresponse.authorization.openshift.io/<unknown> 

Namespace: openshift
Verb:      get
Resource:  imagestreams.image.openshift.io

Users:  admin
        system:admin
        system:serviceaccount:kube-system:clusterrole-aggregation-controller
        system:serviceaccount:kube-system:generic-garbage-collector
        system:serviceaccount:kube-system:namespace-controller
....
Groups: system:authenticated
        system:cluster-admins
        system:cluster-readers
        system:masters

* User Impersonation
  - Cluster administrators can impersonate other users with oc --as=USER
    - Use to validate RBAC configuration
    - Examples:
      -  `oc --as=andrew can-i list pods -n openshift-authentication`
      -  `oc --as=andrew get pods -n example-app-dev`


* Group Impersonation
    - Test group RBAC with oc --as=: --as-group=GROUP
      - --as-group=GROUP may be repeated for multiple groups
      - --as-group=GROUP requires --as=... option, but need not reference valid user
      - --as=: useful convention for specifying arbitrary user string

      - Example: Attempt to get pods in namespace as group:
        `oc get pods -n example-app-db --as=: --as-group=example-app-dev

      - Example: Check if project self-provisioning is available for users authenticated with oauth:
        `oc auth can-i create projectrequests --as=: --as-group=system:authenticated --as-group=system:authenticated:oauth`
