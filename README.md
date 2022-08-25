# Couchbase Ansible Playbooks for Kubernetes

## Contributing

If you're interested in contributing content, please issue a PR(Pull Request) and your changes will be reviewed and considered for merging.

## Explanation

It is assumed that you already have Ansible installed. For more information on Ansible, review their [installation instructions](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).


## Setup
It is recommended that you use a stand-alone host that has Ansible installed on it and use the host to promote changes to your newly provisioned Couchbase nodes.

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
To use the default ```group_vars/eks.yml``` configurations, you may leave the file in tact and execute the playbooks without making any additional changes. If you want/need specific configuration, you may change those at anytime.

To provision a new EKS cluster in your AWS VPC, execute the _RunAllPlaybooks.yml file.

NOTE: You must have the latests [aws cli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) tool and the [eksctl cli](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html) tools installed. The playbooks will check for these tools and will fail if they are not properly installed.

## Example Global Variables(eks.yml)
```
# this file holds all global variables that are accessible by all playbooks in the eks_playbooks directory
---
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
    cao: default
    dac: default
    crd: default
  couchbase_cluster:
    install_version: 7.1.1

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
