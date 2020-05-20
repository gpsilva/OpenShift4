## Authentication

Overview
* Most OpenShift® clusters: Multi-user environments accessed by variety of audiences

    - Application developers developing and deploying applications
    - Operations teams monitoring system health and responding to incidents
    - Security staff auditing access and application security
    - Red Hat® OpenShift Container Platform administrators
  
* Authentication: Process by which users prove identity to cluster

### OAuth Authentication and Verification

* Authentication process:

  - User directed to OAuth server by API or web console

  - OAuth server negotiates authentication based on identity provider configuration
  
  - User authenticates to OAuth server and receives OAuth token
  
  - OAuth token included with subsequent API requests to identify user
  
  - OAuth token’s default lifespan is 86,400 seconds (24 hours) after which user must reauthenticate

* OAuth Tokens

    - OAuth token verification:
      - OpenShift API and other components check token validity
      - Active OAuth access tokens retrieved via oc get oauthaccesstokens
      - Deleting access token terminates OAuth session

* Command Line Authentication
    - Log in with password: oc login -u USER -p PASSWORD API_URL
        - API URL distinct from console URL and usually found at https://api.CLUSTERDOMAIN:6443
        - Authentication session information written to ~/.kube/config
        - Alternate kubeconfig file location may be set
            - KUBECONFIG environment variable
            - --config option
* Command Line Authentication with Token
    - Log in with OAuth token: oc login --token=TOKEN API_URL

* Command Line Session Inspection
    - Inspect current session:
        - oc whoami shows current user
        - oc whoami --show-console shows cluster web console URL
        - oc whoami --show-server shows cluster API URL
        - oc whoami --show-token shows current OAuth token
  
* Authentication Process Inspection
    - Inspect authentication flow with verbose logging:
        - oc login --loglevel=9 -u USER -p PASSWORD API_URL

    - With cluster domain ocp4.example.com:
        - Client contacts https://api.ocp4.example.com:6443/, directed to authenticate with OAuth
  
        - Client contacts https://oauth-openshift.apps.ocp4.example.com/ to authenticate and receive OAuth token
  
        - Subsequent calls to cluster API include Authorization: Bearer TOKEN HTTP header

* Authentication Configuration
    - Managed by cluster-authentication-operator:
        - Configured with oauth.config.openshift.io custom resource named "cluster"
        - spec.identityProviders lists additional methods for user authentication
        - spec.tokenConfig.accessTokenMaxAgeSeconds configures lifespan of OAuth sessions

* kubeadmin User
    - kubeadmin (kube:admin) user provided for initial cluster configuration
        - Disable kubeadmin user after initial cluster configuration
        - kubeadmin user password stored in kubeadmin secret in kube-system namespace
        - To remove kubeadmin password after configuring other cluster-admin users:
             `oc delete secret kubeadmin -n kube-`
            
### Identity Providers

Supported identity providers:

*HTPasswd:* Validates usernames, passwords against htpasswd password database stored within cluster as secret

*LDAP:* Validates usernames, passwords against LDAPv3 server using simple bind authentication

*Basic authentication (remote):* Validates usernames, passwords against remote server using server-to-server basic authentication request

*GitHub:* Authenticate with GitHub or GitHub Enterprise OAuth authentication server

*GitLab:* Authenticate with GitLab or any GitLab instance

*Google:* Authenticate using Google’s OpenID Connect integration

*Keystone:* Authenticate with OpenStack® Keystone v3 server

*OpenID Connect:* Authenticate with any server that supports OpenID authorization code flow

*Request Header:* Authenticate with authenticating proxy using X-Remote-User header

### HTPasswd Authentication

* HTPasswd Identity Provider Overview

    - HTPasswd supports authentication with passwords stored in cluster
    - Password hashes stored within cluster as secret
        - Secret configured in openshift-config namespace
        - Passwords stored in htpasswd format

* htpasswd Secret Creation
1. Create empty htpasswd file:
`touch htpasswd`

2. Use htpasswd command to add passwords for each user in htpasswd file:
`htpasswd -Bb htpasswd USER PASSWORD`

3. Create htpasswd secret from htpasswd file in openshift-config namespace:
`oc create secret generic htpasswd --from-file=htpasswd -n openshift-config`

* Updating Passwords in htpasswd Secret

1. Dump current htpasswd secret content to htpasswd file:
`oc get secret htpasswd -n openshift-config -o jsonpath={.data.htpasswd} base64 -d >htpasswd `

2. Add or update user passwords:
`htpasswd -Bb htpasswd USER PASSWORD`

3. Patch htpasswd secret data with content from file:
`oc patch secret htpasswd -n openshift-config -p '{"data":{"htpasswd":"'$(base64 -w0 htpasswd)'"}}'`

### LDAP Authentication

* LDAP Identity Provider Overview

* LDAP is a common Identity and Access Management (IAM) component
    - Often used as central authentication component with user identities and passwords

* LDAP authentication steps:
    - Query LDAP for user identity by searching for username provided
        - If search does not return exactly one entry, deny access

    - Attempt to authenticate using found user identity and user-provided password
        - If authentication successful, build identity using configured attributes

* LDAP Binding
  
    - LDAP authentication is called "binding"
    - LDAP "bind" action authenticates to LDAP server with user identity (DN) and password
    - LDAP authentication requires both LDAP search for user and bind to check user password
        - Separate account often required to initially bind to LDAP to perform first user search

* LDAP TLS Security
    - Best practie to always use secure communication for LDAP authentication
        - Unless specifically disabled, TLS used whether ldap:// or ldaps:// connections
        - LDAP TLS security requires TLS certificate authority (CA) certificate
        - Disabling LDAP TLS security possible but inadvisable

* LDAP TLS Certificate Authority ConfigMap
    - LDAP CA certificate must be stored in ConfigMap for use by LDAP identity provider:
        - CA ConfigMap must be in openshift-config namespace
        - Data key in ConfigMap must be ca.crt
    - To create ConfigMap in openshift-config project namespace with ca.crt data key:
`oc create configmap ldap-tls-ca -n openshift-config --from-file=ca.crt=PATH_TO_LDAP_CA_FILE`

* LDAP Bind Secret
    - If LDAP server requires binding for search, bind password must be provided as secret
        - Bind secret must be in openshift-config namespace
        - LDAP bind secret must use bindPassword data key
        - Base secret name on name for LDAP identity provider

    - To create secret in openshift-config project namespace with bindPassword data key:
`oc create secret generic ldap-bind-password -n openshift-config --from-literal=bindPassword=PASSWORD`

* LDAP Connection Configuration

*ldap.url:* LDAP user search URL in format ldap://host:port/basedn?attribute?scope?filter

*ldap.ca.name:* OpenShift ConfigMap name containing ca.crt data key with TLS certificate authority to validate LDAP server certificate

*ldap.insecure:* If = true, no TLS security used for LDAP connection. Set to false when ldap.url uses ldaps:// URL, as these URLs always attempt to connect using TLS

*ldap.bindDN:* LDAP bind user DN to perform authentication for LDAP user search bind

*ldap.bindPassword.name:* OpenShift secret name with bindPassword data key for LDAP user search bind

* LDAP user search URL
  - LDAP user search URL format: ldap://host:port/basedn?attribute?scope?filter

*ldap:* LDAP protocol; ldap or ldaps

*host:* Hostname for LDAP server

*port:* LDAP port; default is 389 for ldap:// URLs, 636 for ldaps://

*basedn:* DN of directory branch where all searches start—either top of directory tree or subtree in directory

*attribute:* Attribute to search for. If list, only first attribute used. If no attributes provided, defaults to uid.

*scope:* Scope of search, either one or sub. If not provided, sub used by default.

*filter:* LDAP search filter often used to restrict access to users belonging to particular LDAP group. Defaults to (objectClass=*).

* LDAP user attribute configuration

*ldap.attributes.email:* List of attributes to use as email address; first non-empty attribute used

*ldap.atrributes.id:* List of attributes to use as identity; first non-empty attribute used. At least one attribute required. If none of listed attributes have value, authentication fails.

*ldap.attributes.name:* List of attributes to use as display name; first non-empty attribute used

*ldap.attributes.preferredUsername:* List of attributes to use as preferred user name when provisioning user for this identity; first non-empty attribute used.

* Troubleshooting

*system: admin user*

  - system:admin user bypasses OAuth authentication

      - system:admin cannot use console login

      - Authentication performed with TLS certificates

      - TLS credentials stored in kubeconfig file created during initial installation

      - Save copy of initial kubeconfig file in secure location for recovery

* Authentication logs
  * Openshift OAuth pod logs
    * List pods in openshift-authentication namespace:
        - `oc get pods -n openshift-authentication`
NAME                               READY   STATUS    RESTARTS   AGE
oauth-openshift-6b845c48fd-5tml7   1/1     Running   0          123m
oauth-openshift-6b845c48fd-fq7tm   1/1     Running   0          123m

* Check pods logs of each oauth-openshift pod:
      - `oc logs -n openshift-authentication oauth-openshift-6b845c48fd-fq7tm`
      - `oc logs -n openshift-authentication oauth-openshift-6b845c48fd-5tml7`

* Openshift authentication operator logs

    - Operator is pod found in openshift-authentication-operator project namespace
      - `oc logs -n openshift-authentication-operator deployment/authentication-operator`


* User and Identity Resources
    - User and identity resources created on first login
        - Get user resources: oc get users
        - Get identity resources: oc get identities

* Dynamically created user and identity records produce error on username conflicts:

    - oauth-openshift pod logs indicate AuthenticationError: user "USERNAME" cannot be claimed by identity "IDENTITY" because it is already mapped to [CURRENT_IDENTITY]” Local Passwords:jkupfere]
  
    - If error encountered after OAuth reconfiguration, user and identity may need to be deleted
    - RBAC preserved when user and identity resources deleted and recreated on next login

* LDAP email, name, and preferred_username attributes stored in identity record

