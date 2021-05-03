# Minio With External Secrets

## External Secrets

### Thanos Resource

#### Object Store Secret

There have to be defined an object store secret, where object bucket is defined.

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

In thanos kind of custom resources, external secret can be referenced:

```
   existingObjstoreSecret: <SECRET NAME>
```
 
#### Minio Secret

When minio is enabled, access and secret keys should be defined in external secret:

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
