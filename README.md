# General Notes for ACM demo

## quickstart minio s3
# deploy minio
oc new-project minio
oc apply -f minio/ -n minio

# setup a random user/pass
export MINIO_ROOT_USER=$(openssl rand -base64 18)
export MINIO_ROOT_PASSWORD=$(openssl rand -base64 33)

echo ${MINIO_ROOT_USER} / ${MINIO_ROOT_PASSWORD}

oc set env deployment/minio MINIO_ROOT_USER="${MINIO_ROOT_USER}" -n minio
oc set env deployment/minio MINIO_ROOT_PASSWORD="${MINIO_ROOT_PASSWORD}" -n minio

## quickstart observability
- [ACM 2.4 docs]()
# setup pull secret
# DOCKER_CONFIG_JSON=`oc extract secret/multiclusterhub-operator-pull-secret -n open-cluster-management --to=-`
DOCKER_CONFIG_JSON=`oc extract secret/pull-secret -n openshift-config --to=-`
oc create secret generic multiclusterhub-operator-pull-secret \\
-n open-cluster-management-observability \\
--from-literal=.dockerconfigjson="$DOCKER_CONFIG_JSON" \\
--type=kubernetes.io/dockerconfigjson
# s3 thanos-storage / cr
oc apply -f acm/thanos-object-storage-s3.yaml -n open-cluster-management-observability
oc apply -f acm/multiclusterobservability_cr.yaml


## hack cloud creds (azure)
oc extract -n kube-system secret/azure-credentials --to=-
az login --service-principal -u [client_id]() -p [auzre_client_secret]() --tenant [tenant_id]()

## Login to Azure with the `az` command line tool using the cluster credentials revealed in the preceding command:
az login --service-principal -u 892ff158b-abc5-43fe-99a2-d85963cca6af -p rw7q6f/]TwraMm6.?6n0C_lU[:7ci8HI --tenant 1ce7852f-dcf3-31bc-afe6-3bf81ab721fb
]()

## List your Azure resources and locate an Azure resource group that is appropriate for the storage account
`az resource list`

## Create the storage account
`az storage account create --name acmstorageobserv --resource-group DefaultResourceGroup-CUS --location centralus --sku Standard_LRS`

## List the newly created storage account’s keys
`az storage account keys list   --resource-group DefaultResourceGroup-CUS   --account-name acmstorageobserv`

## Create an Azure storage container
`az storage container create -n prometheus --account-name key1 --account-key 8Ebx646mnzGFjJcNtyn1017oE8y46WJayXTVs0MyA4nlJ6olrQOKqNJ2BMG0THLkfyEZG8VrS/i5Vm2sHD48Ig== --public-access container`

## Make sure you can upload to the storage
`    az storage blob upload --account-name acmstorageobserv --account-key 8Ebx646mnzHKjJcFtyn1017oE8y24WJayXTVs0MyA4nlJ6olrQOKqNJ2BMG0THLkfyEZG8VrS/i5Vm2sHD48Ig== --container-name prometheus --file /Desktop/screenshot.png --name myblob`

## azure storage principal setup
See ["Create an Azure service principal"]() and ["Assign an Azure role for access to blob data"]() pages for more details.

az ad sp create-for-rbac --name "\<name\>" \\
  --role "Storage Blob Data Owner" \\
  --scopes "/subscriptions/\<subscription\>/resourceGroups/\<resource-group\>/providers/Microsoft.Storage/storageAccounts/\<storage-account\>/blobServices/default/containers/\<container\>" \\
> azure-principal.json

## Create storage inside of ACM:
```
	apiVersion: v1
	kind: Secret
	metadata:
	  name: thanos-object-storage
	  namespace: open-cluster-management-observability
	type: Opaque
	stringData:
	  thanos.yaml: |
	    type: AZURE
	    config:
	      storage_account: acmstorageobserv
	      storage_account_key: 8Ebx646mnzHKjJcFtyn1017oE8y24WJayXTVs0MyA4nlJ6olrQOKqNJ2BMG0THLkfyEZG8VrS/i5Vm2sHD48Ig==
	      container: prometheus
	      endpoint: acmstorageobserv.blob.core.windows.net
	      max_retries: 0

```