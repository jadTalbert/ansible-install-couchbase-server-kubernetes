# Couchbase Ansible Playbooks for Kubernetes

## Contributing

If you're interested in contributing content, please issue a PR(Pull Request) and your changes will be reviewed and considered for merging.

## Explanation

It is assumed that you already have Ansible installed. For more information on Ansible, review their [installation instructions](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).


## Setup
It is recommended that you use a stand-alone host that has Ansible installed on it and use the host to promote changes to your newly provisioned EKS cluster.

## Step1:

- The two files below are located in the root directory of this repository. Modify them to suit your needs, but follow the base syntax.

##### Configure the Default Inventory File
```
#this is the default inventory file
#this contains one group [eks] with an associated group_vars/eks.yml file to store all global variables for an EKS deployment. For more details, see the variable descriptions below.

[eks]
localhost ansible_connection=local

```

## Step 2:
```
# ansible.cfg file to point to your inventory file
[defaults]
inventory = /Users/jadtalbert/ansible-install-couchbase-server-kubernetes/inventory

```


## Step 3:
To use the default ```group_vars/eks.yml``` configurations, you may leave the file in tact and execute the playbooks without making any additional changes if you are using a Mac. If you are using another operating system, then you can [download](https://www.couchbase.com/downloads) the Couchbase Autonomous Operator for your specific needs.

To provision a new EKS cluster in your AWS VPC, execute the following: ansible-playbooks/eks_playbooks/_RunAllPlaybooks.yml file.

NOTE: You must have the latest [aws cli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) tool and the [eksctl cli](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html) tools installed. The playbooks will check for these tools and will fail if they are not properly installed.

## Example Global Variables(eks.yml)
```
# this file holds all global variables that are accessible by all playbooks in the eks_playbooks directory

  kubectl_context: docker-desktop
  platform_cmd: kubectl
  eks_cluster:
    name: ansiblePlaybookEKSCluster
    region: us-east-1
    profile: default
    version: 1.21
    logging:
      file_name: eks.log
      file_path: /tmp/
  couchbase_operator:
    download_url: https://packages.couchbase.com/couchbase-operator/2.3.2/couchbase-autonomous-operator_2.3.2-kubernetes-macos-amd64.zip
    file_name: couchbase-autonomous-operator_2.3.2-kubernetes-macos-amd64.zip
    download_dest_dir: /tmp/
    unzip_dest_dir: /tmp/
  install_name_spaces:
    cao: dev
    dac: default
    crd: dev
  couchbase_cluster:
    pod_name_prefix: cb-ansible-tesla
    install_version: 7.1.1
    tls:
      tls_ca_secret_name: couchbase-server-ca
      tls_server_secret_name: couchbase-server-tls
      easy_rsa:
        vars_path: tls/vars.j2
        vars:
          cn: Easy RSA Generated Common Name
          dn_mode: org
          digest: sha256
          country: US
          province: North Carolina
          city: Charlotte
          orginization: Couchbase
          email: jad.talbert@couchbase.com
          org_unit: Professional Services


```

## What you can expect:
###### You need to have the right tools installed, so make sure you have the aws cli and eksctl tools available and properly configured for your AWS VPC.

 The playbooks will do all of the heavy lifting for you, so sit back and let them do the work. However, keep in mind that this is not a fast process, as AWS generally take 15-20 minutes to completely provision an EKS cluster successfully.

If the playbooks fail due to an EKS capacity limit, then you will need to review your CloudFormation templates and clean them up manually. This is NOT a problem with the playbooks, but rather the behavior of AWS.

 IMPORTANT EBS NOTE: These playbooks use persistent volumes and the default is to retain them. If you do not want to retain your EBS volumes, you can set them to be deleted when the cluster is deleted. See the example below for details.

If you want to implement TLS, you should review the [Couchbase TLS](https://docs.couchbase.com/operator/current/concept-tls.html) documentation. These playbooks use easy-rsa by default, and the requisite files are automatically created for you under the ```couchbase/tls/pki``` directory. To customize your certificate(s), easy-rsa uses a ```vars``` file, which has been customized using a Jinja2 template located here -> ```couchbase/tls/vars.j2```. You can modify this file to meet your needs. After making changes, you will need to re-run the ```couchbase/tls/configure_tls.yml``` using ```ansible-playbook eks_playbooks/couchbase/tls/configure_tls.yml```. This will regenerate your certificate files in the ```pki``` directory.

#### vars.j2 sample:
variables are located in the ```group_vars/eks.yml``` file.
```
set_var EASYRSA_REQ_COUNTRY	{{couchbase_cluster.tls.easy_rsa.vars.country}}
set_var EASYRSA_REQ_PROVINCE	{{couchbase_cluster.tls.easy_rsa.vars.province}}
set_var EASYRSA_REQ_CITY	{{couchbase_cluster.tls.easy_rsa.vars.city}}
set_var EASYRSA_REQ_ORG	{{couchbase_cluster.tls.easy_rsa.vars.orginization}}
set_var EASYRSA_REQ_EMAIL	{{couchbase_cluster.tls.easy_rsa.vars.email}}
set_var EASYRSA_REQ_OU		{{couchbase_cluster.tls.easy_rsa.vars.org_unit}}

```

#### eks_playbooks/couchbase/couchbase_cluster.yml
```
#create the storage class that will be used for persisent volumens
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: cb-default-sc
  namespace: default
parameters:
  type: gp2
  fsType: ext4
provisioner: kubernetes.io/aws-ebs
reclaimPolicy: Retain #change this to your desired strategy
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

## Global Variable Definitions for eks.yml:
group_vars                | Description
----------------------------|-----------------------------
kubectl_context        | kubectl context
platform_cmd | kubectl
eks_cluster  | object to hold EKS cluster specifics
eks_cluster.name | name of your EKS cluster
eks_cluster.region |  deploy EKS cluster to the specified region
eks_cluster.profile  |  aws cli profile to use e.g. [default] generally located in ~/.aws/credentials
eks_cluster.version  |  supported EKS version to deploy(e.g. 1.21)
eks_cluster.logging | child objedt to hold the log path for the eksctl create cluster command output. This allows you to tail -f /tmp/eks.log to see the output in real-time. Ansible is not tail friendly
eks_cluster.logging.file_name  |  name of the log file to create and write to while eksctl command is executing
eks_cluster.logging.file_path  |  custom path to write the log file when eksctl is executed
couchbase_operator  | object to hold operator specfic configurations
couchbase_operator.download_url  |  URI to download the operator. You can get this from the couchbase.com web site
couchbase_operator.file_name  |  the zip file name of the operator
couchbase_operator.download_dest_dir  | path where you want to put the downloaded operator .zip files
couchbase_operator.unzip_dest_dir  |  path where you want to put the unzipped/uncompressed operator files
install_name_spaces.cao  |  kubernetes namespace for the operator
install_name_spaces.dac  | kubernetes namespace for the DAC
install_name_spaces.crd  | kubernetes namespace for the CRD(Custom Resource Definitions)
couchbase_cluster  |  object to hold version information for your couchbase nodes
couchbase_cluster.install_version  | version of Couchbase you want to install
couchbase_cluster.enable_tls  | to enable TLS for the Kubernetes cluster. Documentation can be found [here](https://docs.couchbase.com/operator/current/tutorial-tls.html)
couchbase_cluster.pod_name_prefix  |  custom pod name
couchbase_cluster.tls  | easy-rsa object to hold TLS related variables for certificate creation  
couchbase_cluster.tls.tls_ca_secret_name  | kubernetes tls ca secret   
couchbase_cluster.tls.tls_server_secret_name  |  kubernetes tls server secret
couchbase_cluster.tls.easy_rsa  |  object to hold all easy-rsa variables.
couchbase_cluster.tls.easy_rsa.vars_path  |  Jinja2 template for easy-rsa file that contains variables for building certificates
couchbase_cluster.tls.easy_rsa.cn | certificate Common Name(CN)  
couchbase_cluster.tls.easy_rsa.dn_mode  |  certificate distinguished name(DN)
couchbase_cluster.tls.easy_rsa.digest  | default sha256  
couchbase_cluster.tls.easy_rsa.country  |  orginazation Country
couchbase_cluster.tls.easy_rsa.province  | orginazation province/state   
couchbase_cluster.tls.easy_rsa.city   |  orginization city
couchbase_cluster.tls.easy_rsa.orginazation  |  orginization name
couchbase_cluster.tls.easy_rsa.email  |  orginization email
couchbase_cluster.tls.easy_rsa.unit  |  orginization business unit

## How To:

##### Create Couchbase Users & Groups(Role Based Access Control or RBAC)
When creating a Couchbase local user, follow the below steps:

- open the couchbase/secrets/create_secrets.yml file and add your applicable secrets
- open the users/couchbase_users.yml and add your users
- open the groups/create_groups.yml and add your groups and respective roles or you can use the one already provided.
- open the role_bindings/create_role_bindings.yml file and add your user to bind the role.
- execute kubectl apply -f on each of the files listed above and the operator will create your user and attach it to the proper role and group.


##### Change the default EKS namespace for your cluster
- Open the group_vars/eks.yml file and modify the ```install_name_spaces``` object with your namespace(s). Note, the Couchbase Operator is a per-namespace operator, and the operator will be deployed to the specified namespace.
