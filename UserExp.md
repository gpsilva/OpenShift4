Project
    Allows groups of users or developers to work together
    Unit of isolation and collaboration
    Defines scope of resources
    Allows project administrators and collaborators to manage resources
    Restricts and tracks use of resources with quotas and limits

Project
Kubernetes namespace with additional annotations
    Central vehicle for managing resource access for regular users
    Lets community of users organize and manage content in isolation from other communities
Users:
    Receive access to projects from administrators
    Have access to own projects if allowed to create them

Each project has own:
    Objects: Pods, services, replication controllers, etc.
    Policies: Rules that specify which users can or cannot perform actions on objects
    Constraints: Quotas for objects that can be limited
    Service accounts: Users that act automatically with access to project objects

Users and User Types
Interactions with OpenShift® always associated with user
    System permissions granted by adding roles to users or groups

User types:
    Regular Users
        How most interactive OpenShift users are represented
        Created automatically in system upon first login, or via API
        Represented with user object

    System Users:
        Many created automatically when infrastructure defined
        Let infrastructure interact with API securely
        Include: cluster administrator, per-node user, service accounts

Login and Authentication
    Every user must authenticate to access OpenShift
    API requests lacking valid authentication are authenticated as anonymous user
    Policy determines what user is authorized to do
Web Console Authentication
    Access web console at URL provided by your administrator
    Provide login credentials to obtain token to make API calls
    Use web console to navigate projects

CLI Login
    Use oc tool to log in to same address as web console
    Provide login credentials to obtain token to make API calls
        oc login -u <my-user-name> --server="<master-api-public-addr>:<master-public-port>"
    Administrators can have key generated by cluster for password-less authentication


Resource Quotas
OpenShift can limit:
    Number of objects created in project
    Amount of compute/memory/storage resources requested across objects in project
    Based on specified label
        Examples: To limit to department of developers or environment such as test

Multiple teams can share single OpenShift cluster
    Each team in own project or projects
    Resource quotas prevent teams from depriving each other of cluster resources
    
ResourceQuota object enumerates hard resource usage limits per project
ClusterResourceQuota object enumerates hard resource usage limits for users across the cluster


LimitRanges
LimitRanges express CPU and memory requirements of pods' containers
    Set the request and limit of CPU and memory particular pods' container may consume
    Aid OpenShift scheduler in assigning pods to nodes

LimitRanges express quality of service tiers:
    Best Effort, Burstable, Guaranteed

Default LimitRange for all pods/containers can be set for each project

Compute Resources Managed by Quota Across Pods in Non-Terminal State
Resource Name                   Description
cpu
requests.cpu                    Sum of CPU requests cannot exceed the value set

memory
requests.memory                 Sum of memory requests cannot exceed this value

limits.cpu                      Sum of CPU limits cannot exceed this value

limits.memory                   Sum of memory limits cannot exceed this value.


Object Counts Managed by Quota
Resource Name                   Description
pods                            Total number of pods in non-terminal state
                                that can exist in project (pod is in terminal
                                state if status.pahse in (Failed, Succeeded))

replicationcontrollers          Total number of replication controllers that
                                can exist in project

resourcequotas                  Total number of  of resource quotas that can 
                                exist in a project

services                        Total number of  of services that can 
                                exist in a project

secrets                         Total number of  of secrets that can 
                                exist in a project

configmaps                      Total number of  of ConfigMap objects that can 
                                exist in a project

persistentvolumeclaims          Total number of  of persistent volume claims  
                                that can exist in a project

Quota Enforcement
After quota created in project:
    Project restricts ability to create resources that may violate quota constraint
    Usage statistics are calculated every few seconds (configurable)

If project modification exceeds quota:
    Server denies action
    Returns error message
Error message includes:
    Quota constraint violated
    Current system usage statistics

Container Log Aggregation
Three components make up EFK logging stack:
    Elasticsearch: Object store where all logs stored
    Fluentd: Gathers logs from nodes, feeds them to Elasticsearch
    Kibana: Web UI for Elasticsearch

After EFK deployed, stack aggregates logs from all nodes and projects into Elasticsearch
    Provides Kibana UI to view them
Cluster administrators can view all logs
Developers can view only logs for projects for which they have permission

Fluentd
    Pulls logs from container file system and OpenShift services on host
    Sends them to respective Elasticsearch clusters that store aggregated log data
    Users and platform administrators access their respective Kibana interfaces to see application’s or platform’s aggregated logs

Metrics Collection and Alerting with Prometheus - Features
    Pre-configured and self-updating monitoring stack based on Prometheus open source project
    Provides monitoring of cluster components
    Ships with set of alerts
    Creates Grafana dashboards

Metrics Collection and Alerting with Prometheus - Design
Cluster Monitoring Operator (CMO)
    Watches deployed monitoring components and resources, ensures they are up to date

Prometheus Operator (PO)
    Manages Prometheus and Alertmanager
    Automatically generates monitoring targets based on Kubernetes label queries

node-exporter: Agent deployed on every node to collect metrics about it
kube-state-metrics: Exporter agent converts Kubernetes objects to metrics consumable by Prometheus

Metrics Collection and Alerting with Prometheus - HPA
Configure horizontal pod autoscaling (HPA) based on any metric from Prometheus
    Autoscale based on any cluster-level metrics from OpenShift
    Autoscale based on any application metrics

Autoscales pods and machines
    Prometheus Adapter connects to single Prometheus instance (or via Kubernetes service)
    Manual deployment and configuration of adapter
    Cluster administrator needs to whitelist cluster metrics for HPA

Templates and Operators
Templates
    Templates can only create new resources
        Cannot manage or delete resources
    Templates not associated with pod

Operators
    Operators create, manage, and delete resources
    Operators implemented by operator pods

Templates and Operators
What Is a Template?
    Describes set of objects that can be parameterized and processed to produce list of objects for OpenShift to create
    Process templates to create anything you have permission to create within project
        Examples: Service, build configuration, deployment configuration
    Can also define set of labels to apply to every object defined in template

What Are Templates For?
    Create instantly deployable applications for developers or customers
    Provide option to use preset variables or randomize values (like passwords)

Labels
    Used to manage generated resources
    Apply to every resource generated from template
    Organize, group, or select objects and resources
    Resources and pods are "tagged" with labels
    Labels allow services and replication controllers to:
        Determine pods they relate to
        Reference groups of pods
        Treat pods with different containers as similar entities

Parameters
    Share configuration values between different objects in template
        Examples: Database username, password, and port needed by front-end pods 
        to communicate to back-end database pods

    Values can be static or generated by template
    Templates let you define parameters that take on values
        Value substituted wherever parameter is referenced
        Can define references in any text field in object definition
    Example:
        Set generate to expression to specify generated values
        from specifies pattern for generating value using pseudo-regular expression syntax

Creating Template from Existing Objects
    Can export existing objects from project in template form
    Modify exported template by adding parameters and customizations
    To export objects from project in template form:
        $ oc export all --as-template=<template_name>

OpenShift Container Platform Web Console - Operator Hub
    Deploy and manage application in cluster with Operator
    Discover Operators from Kubernetes community and Red Hat® partners
    Install Operators on clusters to provide optional add-ons and shared services to developers
    Capabilities provided by operator appear in Developer Catalog, providing self-service experience
    Add shared apps, services, or source-to-image builders to your project
    Cluster administrators can install additional applications that show up automatically

Helm Charts and Operators
    Helm charts similar to templates
        Collection of files that describe related set of Kubernetes resources
    Very popular in Kubernetes community
    Choose from many available Helm charts
    Use charts with Helm Operator to deploy application easily

Helm charts are much like templates. A Helm chart is a collection of files that describe a related set of Kubernetes resources. They are very popular in the Kubernetes community. They can be choosed from the many Helm charts available and used with the Helm Operator to deploy applications easily. Helm charts are not as robust as true operators.

Operator Interaction with OpenShift
    Operators are pods that take advantage of custom resource definitions (CRDs)
    CRDs allow extension of Kubernetes/OpenShift API
        API then knows new resources
    CRDs allow creation of custom resources (CRs)
    Operator watches for creation of CR, reacts by creating application
    CRs managed in same way as stock OpenShift objects
        create, get, describe, delete, etc.
        Example:
            oc get tomcats

Custom Resource (Application) Creation Using an Operator
    Custom Resources are simple definitions of resource you want Operator to create
    Running Operator watches for CR to be created in either:
        Entire OpenShift cluster
        Project that Operator is running in
    When CR created, operator receives event
    Operator then creates all OpenShift resources that make up application
