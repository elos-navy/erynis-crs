# Custom Resources For Erynis Operator

## Generate CRs From Templates

First edit config variables to match target infrastructure:
```
vim config
```

Then generate target CRs from templates:
```
source config

envsubst < grafana.template.yaml > grafana.yaml
envsubst < pushgateway.template.yaml > pushgateway.yaml
envsubst < thanosreceive.template.yaml > thanosreceive.yaml
envsubst < thanos.template.yaml > thanos.yaml
```

And check/change generated CRs to match target infrastructure:
```
vim grafana.yaml
vim pushgateway.yaml
vim thanosreceive.yaml
vim thanos.yaml
```

## Thanos Resource

### Object Store

Object store parameters (where all metrics will be stored) can be defined in plaintext directly in Thanos CR, or in separate secret and CR will contain name of the secret (this secret is not managed by erynis operator and should be created separatly, for example via argo).

Object store can have following syntax:

Example of minio/s3 bucket:

```
  objstoreConfig: |-
    type: s3
    config:
      bucket: thanos
      endpoint: minio.erynis-monitoring.svc.cluster.local:9000
      access_key: minio
      secret_key: password123
      insecure: true
```

Example of azure bucket:

```
  objstoreConfig: |-
    type: AZURE
    config:
      storage_account: "..."
      storage_account_key: "..."
      container: "erynis-azure-blob"
      endpoint: "blob.core.windows.net"
      max_retries: 0
```

#### External Secret

Secret should be created with following syntax:

```
 ...
 data:
   objstore.yml: |-
     type: s3
     config:
       bucket: thanos
       endpoint: thanos-minio.erynis-monitoring.svc.cluster.local:9000
       access_key: MINIO_USERNAME
       secret_key: MINIO_PASSWORD
       insecure: true
```

In Thanos kind CR, external secret can be referenced:

```
   existingObjstoreSecret: <SECRET NAME>
```

### Minio

If there is no object storage available, minio can be used as a storage emulation of s3. It can be enabled and setup in Thanos CR. Here is an example of minio with persistent volume, route and one bucket is created with the name `thanos`.

```
  minio:
    enabled: true
    openshiftRoute:
      enabled: true
      host: thanos-minio.apps.openshift.elostech.cz
    accessKey:
      password: "minio"
    secretKey:
      password: "password123"
    defaultBuckets: "thanos"
    persistence:
      enabled: true
      size: 50Gi
```

#### External Secret

Because minio definition holds sensitive access and secret keys, those data can be stored in external secret (not managed by erynis operator! It has to be created separatly). Access and secret keys should be defined in external secret like this:

```
 data:
   access-key: MINIO_USERNAME
   secret-key: MINIO_PASSWORD
```

Referencing to external secret is done in Thanos kind of custom resource like this:

```
   minio:
     existingSecret: <SECRET NAME>
```
