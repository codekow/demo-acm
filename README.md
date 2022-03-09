# General Notes for ACM demo

## deploy minio s3
```
# quick deploy minio

oc new-project minio
oc apply -f minio/ -n minio

# setup a random user/pass
export MINIO_ROOT_USER=$(openssl rand -base64 18)
export MINIO_ROOT_PASSWORD=$(openssl rand -base64 33)
echo ${MINIO_ROOT_USER} / ${MINIO_ROOT_PASSWORD}
oc set env deployment/minio MINIO_ROOT_USER=${MINIO_ROOT_USER}
oc set env deployment/minio MINIO_ROOT_PASSWORD=$${MINIO_ROOT_PASSWORD}
```

## hack cloud creds (azure)
```
oc extract -n kube-system secret/azure-credentials --to=-
az login --service-principal -u [client_id] -p [auzre_client_secret] --tenant [tenant_id]
```

