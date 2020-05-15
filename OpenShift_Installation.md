# OpenShift Container Platform Installation on AWS

:// Course Properties :course_name: Red Hat OpenShift Container Platform 4 Installation :need_client: false

OpenShift Container Platform Installation on AWS Lab
In this lab, you deploy and configure Red Hat® OpenShift® Container Platform 4 using the fully automated installer on Amazon Web Services.

Goals
Download and configure the OpenShift installer

Deploy a highly available OpenShift Container Platform cluster

Configure the OpenShift Container Platform cluster

Provisioned Environment Hosts
Bastion host: bastion.$GUID.$SANDBOX.opentlc.com, clientvm.$GUID.internal

1. Provision Lab Environment
If you previously set up an environment for this class, you can skip this section and go directly to the Configure Bastion VM to Run OpenShift Installer section.
In this section, you provision the lab environment to provide access to all of the components required to perform the labs. The lab environment is a shared cloud-based environment so that you can access it over the Internet from anywhere. However, do not expect performance to match a dedicated environment.

Go to the OPENTLC lab portal and use your OPENTLC credentials to log in.

If you do not remember your credentials, go to the OPENTLC Account Management page for assistance.
Navigate to Services → Catalogs → All Services → OPENTLC OpenShift 4 Labs.

On the left, select OpenShift 4 Installation Lab.

On the right, click Order.

If you are not in North America, click Lab Parameters and select your appropriate region.

On the bottom right, click Submit.

Do not select App Control → Start after ordering the lab. Ordering the lab automatically begins the build process. Selecting Start may corrupt the lab environment or cause other complications.
After a few minutes, expect to receive three emails:

The first email lets you know provisioning has started.

The second email indicates that the VMs were requested successfully and that provisioning is continuing.

The final email states that provisioning is complete and contains instructions for connecting to the environment.

Wait until you receive the final email before trying to connect to your environment.
Save the final email and make a note of the following items:

GUID: A unique four-character ID for your environment.

Top level domain: A custom URL—for example, .sandboxNNN.opentlc.com, where NNN is a unique three-digit number, or SANDBOXID.

AWS access credentials: An ID and secret key provided for the duration of this lab.

SSH command: A custom command provided for accessing your bastion host via SSH.

SSH password: A password that is specific to your user ID.

This password is different than your OPENTLC password!
1.1. Start Environment After Shutdown
To conserve resources, the lab environment shuts down automatically after eight hours. This section provides directions for restarting the lab environment after it has shut down automatically.

Go to the OPENTLC lab portal and use your OPENTLC credentials to log in.

Expect to see a list of your active services.

If you do not see the Active Services screen, navigate to Services → My Services.
In the list of your services, select the lab environment for this course.

Select App Control → Start to start your lab environment.

Select Yes at the Are you sure? prompt.

On the bottom right, click Submit.

After a few minutes, expect to receive an email letting you know that the lab environment has started.

1.2. Test Server Connections
The bastion administration host serves as an access point into the environment and is not part of the Red Hat® OpenShift® Container Platform environment.

Connect to your administration host and make sure you can access each of your provisioned hosts:

ssh <OPENTLC User Name>@bastion.<GUID>.sandbox<SANDBOXID>.opentlc.com
Validate that the GUID variable is set correctly for your environment:

echo $GUID
Sample Output
c3po
2. Configure Bastion VM to Run OpenShift Installer
In this section, you prepare the bastion VM to install OpenShift Container Platform. This includes installing the AWS Command Line Interface, the OpenShift Installer, and the OpenShift CLI.

Simple instructions and access credentials are available at https://cloud.openshift.com/clusters/install.
2.1. Set Up Installation Prerequisites
Connect to your bastion instance using the information provided in the final provisioning email.

The bastion host is also called workstation.
Switch to root using the sudo command and double check your GUID:

sudo -i
echo ${GUID}
Install the AWS CLI tools:

# Download the latest AWS Command Line Interface
curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
unzip awscli-bundle.zip

# Install the AWS CLI into /bin/aws
./awscli-bundle/install -i /usr/local/aws -b /bin/aws

# Validate that the AWS CLI works
aws --version

# Clean up downloaded files
rm -rf /root/awscli-bundle /root/awscli-bundle.zip
While the OpenShift installer does not need these tools, you use them later to verify your credentials and list created AWS resources.
Get the OpenShift installer binary:

OCP_VERSION=4.1.0
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-install-linux-${OCP_VERSION}.tar.gz
tar zxvf openshift-install-linux-${OCP_VERSION}.tar.gz -C /usr/bin
rm -f openshift-install-linux-${OCP_VERSION}.tar.gz /usr/bin/README.md
chmod +x /usr/bin/openshift-install
Get the oc CLI tool:

wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-client-linux-${OCP_VERSION}.tar.gz
tar zxvf openshift-client-linux-${OCP_VERSION}.tar.gz -C /usr/bin
rm -f openshift-client-linux-${OCP_VERSION}.tar.gz /usr/bin/README.md
chmod +x /usr/bin/oc
Check that the OpenShift installer and CLI are in /usr/bin:

ls -l /usr/bin/{oc,openshift-install}
Sample Output
-rwxr-xr-x. 2 root root  76593328 May 19 23:53 /usr/bin/oc*
-rwxr-xr-x. 1 root root 216764576 May 22 03:22 /usr/bin/openshift-install*
Set up bash completion for the OpenShift CLI:

oc completion bash >/etc/bash_completion.d/openshift
Press Ctrl+D or type exit to log out of your root shell.

Save your provided AWS credentials to the $HOME/.aws/credentials file, making sure to replace YOURACCESSKEY and YOURSECRETACCESSKEY with your individual credentials:

Get your AWS credentials from the final lab provisioning email.

Optionally, select the AWS region that is closest to your location:

Do NOT select us-east-1.
us-east-2 (Ohio)

us-west-1 (California)

eu-central-1 (Frankfurt)

ap-southeast-1 (Singapore)

export AWSKEY=<YOURACCESSKEY>
export AWSSECRETKEY=<YOURSECRETKEY>
export REGION=us-east-2

mkdir $HOME/.aws
cat << EOF >>  $HOME/.aws/credentials
[default]
aws_access_key_id = ${AWSKEY}
aws_secret_access_key = ${AWSSECRETKEY}
region = $REGION
EOF
Check to see that your credentials work:

aws sts get-caller-identity
Sample Output
{
    "Account": "709118467217",
    "UserId": "AIDA2KGWBDSIWVQXVL7II",
    "Arn": "arn:aws:iam::709118467217:user/wkulhane@redhat.com-05a8"
}
Using your browser, go to https://cloud.openshift.com/clusters/install, and log in with your Red Hat login.

You need a Red Hat Subscription or Developer Account to access the OpenShift page.
For Infrastructure Provider, click AWS.

Click Installer-Provisioned-Infrastructure.

Ignore all other instructions, find the section Pull Secret, click Copy Pull Secret to place the secret in your clipboard, and then save it to a file.

You paste the pull secret into the prompt of the installer later.

Example Pull Secret
{"auths":{"cloud.openshift.com":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K3JocGRzYWRtaW5zcmVkaGF0Y29tMWZyZ3NpZHV6cTJkem5zajNpdzBhdG1samg3OjJMSTFEVTM1MFVCQks1ODRCTFVBODBFTTUABDDD13RDI0Qko2Q0I5VzNFSFIzS0pSSFhOSFgyVllNMlFFMVQ=","email":"youremail@redhat.com"},"quay.io":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K3JocGRzYWRtaW5zcmVkaGF0Y29tMWZyZ3NpZHV6cTJkem5zajNpdzBhdG1samg3OjJMSTFEVTM1MFVCQks1ODRCTFVBODBFTTUABDDD13RDI0Qko2Q0I5VzNFSFIzS0pSSFhOSFgyVllNMlFFMVQ=","email":"youremail@redhat.com"},"registry.connect.redhat.com":{"auth":"NTE1NDg0ODB8dWhjLTFGckdzSURVWlEyRHpuU0ozaVcwQXRtTEpoNzpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSXpPR1prTVdJNFpqYzJOamcwTmpKbVlXTTRaVFpsWVRnd09EUTJOMkkzTnlKOS5YUmQ5LS1LQ3kzVlpVbF9ldTc0THpQMFEzOVYwRUVfeWRZOE5pVGRScUlyd2hVRHYtcFF2ZEtLV1ZpVmlaQWF0QkhEUVdmVDB1Z2pfTWIzYmNPUktqSXdBNldQTXYxWTc1RmhYQUg1S2Myc3lnSHVxWTRfZlhSOXJnbW42N0l0MmhiUXJyb3BBNXlaYXpXSzhPeTBJb29VWFAteDBPUjZ2VDJTVGktbm5sblBLbEFSWTBEZkxJYmk3OHZlZXFadUpyUDl4SzlXdnRaOEZOREpzQnlUc2VmeFRoVmtLMDVwVDlhTk9nTkxITGJMeU5sdEc1RE9xU1JiZ1hLMDJ6RXNaU3BwYmZLdVAwNVJYQWljQy14WEZiamtLaFpkYTgwV3lnZDJKcTZXWVF3WW83ZXgtLUh1MEpKeXBTczRINVY0Nm50dTNVRlNVUERBZEJ5VmVDU2RxckpzUWZoSmlpLVdJbXdjWnp6LUNwTlRfNVo0ei1WUkc0aV9hVF9TWnVkQzVySmFLdFpHS1RQWlg0SDlNLWxDeFlHZDJNYzhuWlc4NWVUeTJPYnBVOHA2S19sU3A3Wm15RzhEbWh6bFAtYTQzb0J1V3hJTHg3Y283U3BkOFRyYVNRbjVnaFpvc0VKZGp6X2ljTlFhVktNazFHQjEwbU1uOXJBeGdUcm5qU09aSEZvcXdmX2Y2dnZFWi0ySUp2Qk91UUZRQThsZDlzRDVDb1ZWNEdwTWx1Rl8zZGJqcXhuVTE0WXdHT2RhSldSOEtMTlFwbU9RV0JrWFJIcVpwN01UT0ZDX0dMVDRWeGNTMXhva0p6RUFxN1c4NzBSQVo4VnAtUGdscEJCc2RDT2tfdGNCNEY5T2hkZ0NPb3JMNHJkZmp6cEJobUZuMEhzVkFFNGJkaWhfRjNGSQ==","email":"youremail@redhat.com"},"registry.redhat.io":{"auth":"NTE1NDg0ODB8dWhjLTFGckdzSURVWlEyRHpuU0ozaVcwQXRtTEpoNzpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSXpPR1prTVdJNFpqYzJOamcwTmpKbVlXTTRaVFpsWVRnd09EUTJOMkkzTnlKOS5YUmQ5LS1LQ3kzVlpVbF9ldTc0THpQMFEzOVYwRUVfeWRZOE5pVGRScUlyd2hVRHYtcFF2ZEtLV1ZpVmlaQWF0QkhEUVdmVDB1Z2pfTWIzYmNPUktqSXdBNldQTXYxWTc1RmhYQUg1S2Myc3lnSHVxWTRfZlhSOXJnbW42N0l0MmhiUXJyb3BBNXlaYXpXSzhPeTBJb29VWFAteDBPUjZ2VDJTVGktbm5sblBLbEFSWTBEZkxJYmk3OHZlZXFadUpyUDl4SzszrefOEZOREpzQnlUc2VmeFRoVmtLMDVwVDlhTk9nTkxITGJMeU5sdEc1RE9xU1JiZ1hLMDJ6RXNaU3BwYmZLdVAwNVJYQWljQy14WEZiamtLaFpkYTgwV3lnZDJKcTZXWVF3WW83ZXgtLUh1MEpKeXBTczRINVY0Nm50dTNVRlNVUERBZEJ5VmVDU2RxckpzUWZoSmlpLVdJbXdjWnp6LUNwTlRfNVo0ei1WUkc0aV9hVF9TWnVkQzVySmFLdFpHS1RQWlg0SDlNLWxDeFlHZDJNYzhuWlc4NWVUeTJPYnBVOHA2S19sU3A3Wm15RzhEbWh6bFAtYTQzb0J1V3hJTHg3Y283U3BkOFRyYVNRbjVnaFpvc0VKZGp6X2ljTlFhVktNazFHQjEwbU1uOXJBeGdUcm5qU09aSEZvcXdmX2Y2dnZFWi0ySUp2Qk91UUZRQThsZDlzRDVDb1ZWNEdwTWx1Rl8zZGJqcXhuVTE0WXdHT2RhSldSOEtMTlFwbU9RV0JrWFJIcVpwN01UT0ZDX0dMVDRWeGNTMXhva0p6RUFxN1c4NzBSQVo4VnAtUGdscEJCc2RDT2tfdGNCNEY5T2hkZ0NPb3JMNHJkZmp6cEJobUZuMEhzVkFFNGJkaWhfRjNGSQ==","email":"youremail@redhat.com"}}}
Double check that your pull secret contains credentials for all three container registries: quay.io, registry.connect.redhat.com, and registry.redhat.io.

You can now close or minimize the browser window and return to your installer VM.

Create an SSH keypair to be used for your OpenShift environment:

ssh-keygen -f ~/.ssh/cluster-${GUID}-key -N ''
3. Install OpenShift Container Platform
In this section you use the OpenShift installer to deploy a highly available OpenShift Container Platform cluster on AWS.

Documentation for the installer can be found at https://https://docs.openshift.com/container-platform/4.1/installing/installing_aws/installing-aws-default.html and https://github.com/openshift/installer/blob/master/docs/user/overview.md

The main assets generated by the installer are the Ignition configs for the bootstrap, master, and worker machines. Given these three configs (and correctly configured infrastructure), it is possible to start an OpenShift cluster. The process for bootstrapping a cluster looks like the following:

The bootstrap machine boots and starts hosting the remote resources required for the master machines to boot.

The master machines fetch the remote resources from the bootstrap machine and finish booting.

The master machines use the bootstrap node to form an etcd cluster.

The bootstrap node starts a temporary Kubernetes control plane using the newly created etcd cluster.

The temporary control plane schedules the production control plane to the master machines.

The temporary control plane shuts down, yielding to the production control plane.

The bootstrap node injects OpenShift-specific components into the newly formed control plane.

The installer then tears down the bootstrap node.

The result of this bootstrapping process is a fully running OpenShift cluster. The cluster will then download and configure the remaining components needed for day-to-day operation, including the creation of worker machines on supported platforms.

The installer uses a wizard approach, asking a few questions about the environment before executing the installation. If you want to avoid the wizard or run the installer from a shell script, it is useful to set up environment variables for a particular cloud platform.

3.1. Run OpenShift Installer
Run the OpenShift installer and answer the prompts:

Select your Public Key, replacing GUID with your assigned GUID.

Select aws as the Platform.

Select any Region near you, except us-east-1.

The default is us-east-2.

Select sandboxNNN.opentlc.com as the Base Domain, where NNN is your SANDBOXID.

For the Cluster Name, type cluster-GUID, replacing GUID with your GUID.

This gives the cluster a unique name.

When prompted, paste the contents of your Pull Secret in JSON format.

Do not include any spaces or white characters and make sure it is in one line.

openshift-install create cluster --dir $HOME/cluster-${GUID}

? SSH Public Key /home/<OpenTLC User>/.ssh/cluster-<GUID>-key.pub
? Platform aws
? Region us-east-2 (Ohio)
? Base Domain sandboxNNN.opentlc.com
? Cluster Name cluster-<GUID>
? Pull Secret ***************************************************************************************************************************************************************
Sample Output
INFO Creating infrastructure resources...
INFO Waiting up to 30m0s for the Kubernetes API at https://api.cluster-05a8.sandbox129.opentlc.com:6443...
INFO API v1.13.4+838b4fa up
INFO Waiting up to 30m0s for bootstrapping to complete...
INFO Destroying the bootstrap resources...
INFO Waiting up to 30m0s for the cluster at https://api.cluster-05a8.sandbox129.opentlc.com:6443 to initialize...
INFO Waiting up to 10m0s for the openshift-console route to be created...
INFO Install complete!
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/home/wkulhane-redhat.com/cluster-05a8/auth/kubeconfig'
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.cluster-05a8.sandbox129.opentlc.com
INFO Login to the console with user: kubeadmin, password: bFfSt-tNNwo-ST6ih-2HcpS
If you want to follow along with the install, you can tail the ${HOME}/cluster-${GUID}/.openshift_install.log file from another terminal window. This file is also useful for troubleshooting.
Make note of the following items from the output of the install command:

The location of the kubeconfig file, which is required for setting the KUBECONFIG environment variable and, as suggested, sets the OpenShift user ID to system:admin.

The kubeadmin user ID and associated password (bFfSt-tNNwo-ST6ih-2HcpS in the example).

The password for the kubeadmin user is also written into the auth/kubeadmin-password file.

The URL of the web console (https://console-openshift-console.apps.cluster-<GUID>.sandbox<SANDBOXID>.opentlc.com in the example) and the credentials (again) to log into the web console.

Optionally, save this information into a text file on your bastion host.

It is possible to run a multi-step installation.

Create the installation configuration: openshift-install create install-config --dir $HOME/cluster-${GUID}.

Update the generated install-config.yaml file—for example, change the AWS EC2 instance types.

Create the YAML manifest templates: openshift-install create manifest-templates --dir $HOME/cluster-${GUID}.

Changing the manifest templates is unsupported.

Create the YAML manifests: openshift-install create manifests --dir $HOME/cluster-${GUID}

Changing the manifests is unsupported.

Create the Ignition configuration files: openshift-install create ignition-configs --dir $HOME/cluster-${GUID}.

Changing the Ignition configuration files is unsupported.

Install the cluster: openshift-install create cluster --dir $HOME/cluster-${GUID}.

To delete the cluster, use: openshift-install destroy cluster --dir $HOME/cluster-${GUID}.

3.1.1. Clean Up Cluster (Reference)
Perform these steps only if installation fails.
If the installer fails and does not provide login information, use these steps before retrying the install. You can use oc commands to explore the failure.

To clean up the environment, do the following:

Delete the cluster:

openshift-install destroy cluster --dir $HOME/cluster-${GUID}
Delete all of the files created by the OpenShift installer:

rm -rf $HOME/.kube
rm -rf $HOME/cluster-${GUID}
3.2. Validate Cluster
Set up the CLI:

export KUBECONFIG=$HOME/cluster-${GUID}/auth/kubeconfig
echo "export KUBECONFIG=$HOME/cluster-${GUID}/auth/kubeconfig" >>$HOME/.bashrc
This makes you system:admin on the cluster.

Validate that you are in fact the cluster administrator:

oc whoami
Sample Output
system:admin
Validate that all nodes have a status of Ready:

oc get nodes
Sample Output
NAME                                         STATUS   ROLES    AGE   VERSION
ip-10-0-131-156.us-east-2.compute.internal   Ready    master   17m   v1.13.4+cb455d664
ip-10-0-143-133.us-east-2.compute.internal   Ready    worker   12m   v1.13.4+cb455d664
ip-10-0-147-42.us-east-2.compute.internal    Ready    worker   12m   v1.13.4+cb455d664
ip-10-0-155-71.us-east-2.compute.internal    Ready    master   17m   v1.13.4+cb455d664
ip-10-0-165-103.us-east-2.compute.internal   Ready    worker   12m   v1.13.4+cb455d664
ip-10-0-175-54.us-east-2.compute.internal    Ready    master   17m   v1.13.4+cb455d664
Validate that all of the pods are running, and that none of them is in Error or CrashLoopBackoff states:

Expect to see about 184 pods in total and about 133 running pods (for the 4.1.0 installer).

oc get pod -A
Sample Output
NAMESPACE                                               NAME                                                                  READY   STATUS      RESTARTS   AGE
openshift-apiserver-operator                            openshift-apiserver-operator-9fc88746f-6d69p                          1/1     Running     1          22m
openshift-apiserver                                     apiserver-blfmg                                                       1/1     Running     0          16m
openshift-apiserver                                     apiserver-jmjhf                                                       1/1     Running     0          17m
openshift-apiserver                                     apiserver-swqj5                                                       1/1     Running     0          15m

[...]

openshift-service-ca                                    apiservice-cabundle-injector-659d59ffd-hvhkd                          1/1     Running     0          21m
openshift-service-ca                                    configmap-cabundle-injector-5f55854946-5ddv8                          1/1     Running     0          21m
openshift-service-ca                                    service-serving-cert-signer-f44b5b4c4-6vxsq                           1/1     Running     0          21m
openshift-service-catalog-apiserver-operator            openshift-service-catalog-apiserver-operator-fd77d9bb-mcfgc           1/1     Running     0          18m
openshift-service-catalog-controller-manager-operator   openshift-service-catalog-controller-manager-operator-6d866rgxk       1/1     Running     0          18m
Open your web browser and navigate to your web console URL, https://console-openshift-console.apps.cluster-<GUID>.sandbox<SANDBOXID>.opentlc.com, replacing <GUID> and <SANDBOXID>.

If you ever forget the route to the web console, you can easily retrieve it with this command: oc get route console -n openshift-console.
When prompted, accept the self-signed (insecure) certificates twice—once for the console application and once for the authentication endpoint.

Log in to the OpenShift cluster with the kubeadmin credentials.

Red Hat has observed that some web browsers have issues with the console. If the page hangs, try a different browser.
Explore the various areas of the web console.

3.3. Verify AWS Instances
You can use the AWS Command Line Interface to examine the created instances.

List all of the EC2 instances that were created, making sure to specify the region that you used during the install:

aws ec2 describe-instances --region=us-east-2 --output table
List all created resources:

openshift-install graph
Sample Output
digraph G {
	rankdir=LR;
	"installconfig.InstallConfig"->"Target Install Config";
	"installconfig.sshPublicKey"->"installconfig.InstallConfig";
	"installconfig.baseDomain"->"installconfig.InstallConfig";
	"installconfig.platform"->"installconfig.baseDomain";
	"installconfig.clusterName"->"installconfig.InstallConfig";
	"installconfig.baseDomain"->"installconfig.clusterName";

[...]

	"tls.KubeletServingCABundle";
	"tls.MCSCertKey";
	"tls.RootCA";
	"tls.ServiceAccountKeyPair";

}
;

}
4. Validation
Next, you validate that this lab was completed successfully.

Run the following command:

grade_lab ocp4_installation 02_1
Sample Output
Starting grade_lab Process
Finished grade_lab Process

Course name: ocp4_installation
Lab number: 02_1
Student id: wkulhane-redhat.com

  PASS: Check that the Web Console route exists


Grading Report:        /tmp/grading_report.txt
Grading Log (ansible): /tmp/opentlc_ansible.log
This concludes this lab.

