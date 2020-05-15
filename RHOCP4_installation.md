# Installer Overview

On supported platforms, installer can provision underlying infrastructure for cluster

Currently supported platforms: AWS, Libvirt, VMware

Coming soon: Red Hat® OpenStack® Platform

Recommended option

If needed, installer can stop short of creating infrastructure

Use cases:

Unsupported platforms

Scenarios where installer-created infrastructure would be incompatible

In these cases, installer allows user to provision infrastructure using installer-generated cluster assets

### OpenShift Installer

Overview
RHOCP 4 installed using simple installer binary

Available from GitHub openshift/installer

Installer configures entire cloud infrastructure:

* VMs
* Load balancers
* Storage
* Networking
* Other resources

RHOCP 4 beta deployed on AWS only
* More platforms being added for version 4.1+

### Prerequisites

* Installer can run from Linux® or macOS
* Does not need to be run as root
* Optionally use AWS CLI to list created resources and check connectivity
    * https://s3.amazonaws.com/aws-cli/awscli-bundle.zip

* Need AWS admin credentials
    Create specific Access Key and Secret Access Key
* Also need pull secret
  - Get at Developer Preview - OpenShift 4
  - Log in with subscription account or Red Hat Developer account

Additional Prerequisites:

- Public Route53 domain
  * Does not need to be top-level domain
  * Installer creates objects under domain
  * Needs to exist before installation starts

- Private/public key pair
  * Create using ssh-keygen -f ~/.ssh/cluster-key
  * Do not use passphrase for key

### Quick installation
When all prerequisites are met, run installer:

`openshift-install create cluster --dir=$HOME/mycluster`

- Installer prompts for
    * SSH public key
    * Platform: Always aws
    * Region: Default in AWS is us-east-1
    * Base domain: Public route 53 domain, needs to exist prior to installation
    * Cluster name: Must be unique within AWS account
    * Pull secret: From try.openshift.com as single-line JSON

After the installer begins running, it sets up the entire OpenShift cluster, including the infrastructure necessary to run the cluster. This takes about half an hour. While the installer installs the cluster, it prints some progress statements.

When the installation has finished, the installer prints the following information:
- Access credentials for an administrative user called kubeadmin.
- The URL of the web console. The console may need a few more minutes to be ready after the installation has completed.
- The location of the kubeconfig file containing the certificate for the system:admin user.
It is recommended to save this information somewhere.

### Pull secret
`
{"auths":{"cloud.openshift.com":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K2dpbHZhbnNpbHZhMXg3dHhydjRrdGNzOWp4bXE1ZXNjcXlkc2RnOlgzT00wRkJBTzdMRzBaSEJKTFIzRFAwRzBESk5SR1RXQjNPNzYyR0M0UzhYNFFaUEJJT0U3TThXODdWOFNDUVg=","email":"gilvan.silva@certsys.com.br"},"quay.io":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K2dpbHZhbnNpbHZhMXg3dHhydjRrdGNzOWp4bXE1ZXNjcXlkc2RnOlgzT00wRkJBTzdMRzBaSEJKTFIzRFAwRzBESk5SR1RXQjNPNzYyR0M0UzhYNFFaUEJJT0U3TThXODdWOFNDUVg=","email":"gilvan.silva@certsys.com.br"},"registry.connect.redhat.com":{"auth":"NTI4ODYxODV8dWhjLTFYN3R4UnY0a3RDczlKWE1xNUVzY3FZZFNEZzpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSTFZV05pWkRrMk1UQmlOMlkwWXpBMllqRmtPRFUzTUdWbVl6RTNPR0l3TnlKOS5mM3QtZHdic01OQkhlV2IyUjJjSUhQX2hObXNKM1F4T3pVN0ptaWs1YmdkSEJVelJRV3dpVkRmRklIV2VIZ1JMR0RlMlc5eGVDRHFkbFZCTEdabG9UOXY0TUJIWFRpTjdnZElTVEZvMUJ3dUh2ck5sVE1ZVGNTWnEtVjhlaDdjTUVPRDNud0hNVmw1aGZhSWZyWWctTlVNeTJoV21FZTZEdXJ2dGZ4SGlRQ2V4NDF1QUc0a2VERVJwQ3VRcUhJUnFRYndtSDlFMkdtMmQyRGRCbzdPYklKYnhBVFRnWncyM25CNXk3ZDlkZU5aVWVUc0hpTFI0UV9WUTNWVlI5ZmlNVTZQR3B5VDhwdzE5OThUV3AwZFpNQkh5LS02cGZwTkhLUTNuZkQwOVNGZ3FGRWI1YWRZUEJpNUpQWlFSeDF0REd4R3RyMm4yWjYtQXFBazB4T01KdTRMR05Xd0xOUUdNbDBlNTdfaFF1Yjd3ZlJBa2VZWUNBNGxoNDk0enZLT280UDZ2bXQ0d01uZGtLbE9DQ04wQkpLNWVqUW9PaHJZUVBob0VPN3Q1X0F1djBIcGhqanhHVThFNGU3NmduQVpyQ1JHLTl0WVd3dUNmRlMtcXpwSlRBU1ZtNkF1WEVCY0xPdFN0eVk3TkNaa0ZPd0UxUHROS1c5LThZdUpKNkNwdUhfSkt3THhYS0VNZGNCV0RoWVRrMWhfQVFTOXZoZ2w3WTVBUHMwT3dNemdNNWpqdjZudC02c1ZvQldQbU96bG1wYTZ3c3N6SHVBMVAtajRxd1hnd2w4MHd3ck1TOHRCbGZ3cFlyYWJpWHlJcDJxVFNIX1NNZHpmOXNmZlJTQm5ueXQ2N2d1R0RhNk5wNWZuNVpGd0FwUG5VaGN4NG9ueEtsV185ZVEyWmVPaw==","email":"gilvan.silva@certsys.com.br"},"registry.redhat.io":{"auth":"NTI4ODYxODV8dWhjLTFYN3R4UnY0a3RDczlKWE1xNUVzY3FZZFNEZzpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSTFZV05pWkRrMk1UQmlOMlkwWXpBMllqRmtPRFUzTUdWbVl6RTNPR0l3TnlKOS5mM3QtZHdic01OQkhlV2IyUjJjSUhQX2hObXNKM1F4T3pVN0ptaWs1YmdkSEJVelJRV3dpVkRmRklIV2VIZ1JMR0RlMlc5eGVDRHFkbFZCTEdabG9UOXY0TUJIWFRpTjdnZElTVEZvMUJ3dUh2ck5sVE1ZVGNTWnEtVjhlaDdjTUVPRDNud0hNVmw1aGZhSWZyWWctTlVNeTJoV21FZTZEdXJ2dGZ4SGlRQ2V4NDF1QUc0a2VERVJwQ3VRcUhJUnFRYndtSDlFMkdtMmQyRGRCbzdPYklKYnhBVFRnWncyM25CNXk3ZDlkZU5aVWVUc0hpTFI0UV9WUTNWVlI5ZmlNVTZQR3B5VDhwdzE5OThUV3AwZFpNQkh5LS02cGZwTkhLUTNuZkQwOVNGZ3FGRWI1YWRZUEJpNUpQWlFSeDF0REd4R3RyMm4yWjYtQXFBazB4T01KdTRMR05Xd0xOUUdNbDBlNTdfaFF1Yjd3ZlJBa2VZWUNBNGxoNDk0enZLT280UDZ2bXQ0d01uZGtLbE9DQ04wQkpLNWVqUW9PaHJZUVBob0VPN3Q1X0F1djBIcGhqanhHVThFNGU3NmduQVpyQ1JHLTl0WVd3dUNmRlMtcXpwSlRBU1ZtNkF1WEVCY0xPdFN0eVk3TkNaa0ZPd0UxUHROS1c5LThZdUpKNkNwdUhfSkt3THhYS0VNZGNCV0RoWVRrMWhfQVFTOXZoZ2w3WTVBUHMwT3dNemdNNWpqdjZudC02c1ZvQldQbU96bG1wYTZ3c3N6SHVBMVAtajRxd1hnd2w4MHd3ck1TOHRCbGZ3cFlyYWJpWHlJcDJxVFNIX1NNZHpmOXNmZlJTQm5ueXQ2N2d1R0RhNk5wNWZuNVpGd0FwUG5VaGN4NG9ueEtsV185ZVEyWmVPaw==","email":"gilvan.silva@certsys.com.br"}}}
`

INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/home/gilvan.silva-certsys.com.br/cluster-48f0/auth/kubeconfig' 
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.cluster-48f0.sandbox254.opentlc.com 
INFO Login to the console with user: kubeadmin, password: EXJa3-QCr6R-mvIpT-QA6AT 