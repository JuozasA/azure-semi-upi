# Openshift Container Platform 4.2 deployment on Azure Public Cloud using User provisioned Infrastructure

Process described here (even though this is for AWS, openshift-install commands are the same):
https://docs.openshift.com/container-platform/4.1/installing/installing_aws_user_infra/installing-aws-user-infra.html#installation-creating-aws-worker_installing-aws-user-infra

1. generate install config files and specify the directory where to save the generated files localy:<br>
 `./openshift-install create install-config --dir=<directory name>`
  
Edit the install-config.yaml file to set the number of compute, or worker, replicas, vm size, disk size and networking, as shown in the following compute stanza:

```
apiVersion: v1
baseDomain: <example.com>
compute:
- hyperthreading: Enabled
  name: worker
  platform:
    azure:
      type: Standard_D2s_v3
      osDisk:
        diskSizeGB: 64
        managedDisk:
          storageAccountType: Premium_LRS  
  replicas: 3
controlPlane:
  hyperthreading: Enabled
  name: master
  platform:
    azure:
      type: Standard_D4s_v3
      osDisk:
        diskSizeGB: 64
        managedDisk:
          storageAccountType: Premium_LRS
  replicas: 3
metadata:
<...>
```

2. generate kubernetes manifests:<br>
`./openshift-install create manifests --dir=<directory name>`

3. Obtain the Ignition config files:<br>
`./openshift-install create ignition-configs --dir=<directory name>`

4. Extract the infrastructure name from the Ignition config file metadata, run the following command:<br>
`jq -r .infraID /<installation_directory>/metadata.json`

Place the name to terraform vars file as "cluster_id".<br>

Fill in other information in terraform vars:
```
  "azure_subscription_id": ""
  "azure_client_id": ""
  "azure_client_secret": ""
  "azure_tenant_id": ""
  "azure_base_domain_resource_group_name": ""
  "cluster_id": "", <- infrastructure name form metadata.json
  "cluster_domain": "", <- cluster name from install-config.yaml (e.g. openshiftpoc)
  "base_domain": "" <- base domain
```

5. Copy ignition config files to the directory with terraform scripts.<br>

6. Run terraform scripts:<br>
`terraform apply -var-file="terraform.tfvars.json"`

6. Wait for Cluster to complete installation:<br>
`./openshift-install wait-for install-complete --dir=<directory name>`
