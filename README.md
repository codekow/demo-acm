# General Notes for ACM demo

## quickstart minio s3
```
# deploy minio
oc new-project minio
oc apply -f minio/ -n minio

# setup a random user/pass
export MINIO_ROOT_USER=$(openssl rand -base64 18)
export MINIO_ROOT_PASSWORD=$(openssl rand -base64 33)

echo ${MINIO_ROOT_USER} / ${MINIO_ROOT_PASSWORD}

oc set env deployment/minio MINIO_ROOT_USER="${MINIO_ROOT_USER}" -n minio
oc set env deployment/minio MINIO_ROOT_PASSWORD="${MINIO_ROOT_PASSWORD}" -n minio
```

## quickstart observability
- [ACM 2.4 docs](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.4/html/observability/observing-environments-intro#enabling-observability)
```
# setup pull secret
#DOCKER_CONFIG_JSON=`oc extract secret/multiclusterhub-operator-pull-secret -n open-cluster-management --to=-`
DOCKER_CONFIG_JSON=`oc extract secret/pull-secret -n openshift-config --to=-`
oc create secret generic multiclusterhub-operator-pull-secret \
    -n open-cluster-management-observability \
    --from-literal=.dockerconfigjson="$DOCKER_CONFIG_JSON" \
    --type=kubernetes.io/dockerconfigjson
```
```
# s3 thanos-storage / cr
oc apply -f acm/thanos-object-storage-s3.yaml -n open-cluster-management-observability
oc apply -f acm/multiclusterobservability_cr.yaml
```

## azure storage principal setup
See ["Create an Azure service principal"](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli) and ["Assign an Azure role for access to blob data"](https://docs.microsoft.com/en-us/azure/storage/common/storage-auth-aad-rbac-cli) pages for more details.

```
az ad sp create-for-rbac --name "<name>" \
      --role "Storage Blob Data Owner" \
      --scopes "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/blobServices/default/containers/<container>" \
      > azure-principal.json
```

## hack cloud creds (azure)
```
oc extract -n kube-system secret/azure-credentials --to=-
az login --service-principal -u [client_id] -p [auzre_client_secret] --tenant [tenant_id]
```

