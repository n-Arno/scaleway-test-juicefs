juicefs@scaleway
================

To leverage JuiceFS as a mean to leverage Scaleway Object Storage as a POSIX filesystem for RWM volumes, follow these steps.

Pre-requisite
-------------

- A Scaleway Kapsule cluster and the associated `kubeconfig.yaml`.
- kubectl and helm CLI installed.
- A Scaleway Object Storage bucket with associated API keys (access key and secret key).
- A Scaleway Managed PostgreSQL connected to the same Private Network as the Kapsule cluster.

Create the credentials
----------------------

Edit the `credentials.yaml` file and add the needed information:

```
---
apiVersion: v1
kind: Secret
metadata:
  name: juicefs-secret
  namespace: default
  labels:
    juicefs.com/validate-secret: "true"
type: Opaque
stringData:
  name: juicefs
  metaurl: "postgres://<user>:<password>@<private endpoint ip of the db>:5432/<name of the database created>"
  storage: scw
  bucket: https://<bucket name>.s3.<region>.scw.cloud
  access-key: SCWXXXXXXXXXXXXXXXXXXX
  secret-key: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Apply the manifest to create the secret.

```
kubectl apply -f credentials.yaml
```

Install the JuiceFS CSI driver
------------------------------

Install the CSI driver with helm using either `./install.sh` or the following commands.

```
helm repo add juicefs https://juicedata.github.io/charts/
helm repo update
helm upgrade --install juicefs-csi-driver juicefs/juicefs-csi-driver -n kube-system
```

Create the storage class
------------------------

Create the storage class leveraging the previously created secret.

```
kubectl apply -f storage-class.yaml
```

Test
----

You can test the creation of a PVC and its association to a pod using this manifest.

```
kubectl apply -f test.yaml
```

You can validate data is written using this command:

```
kubectl exec -ti juicefs-app -- tail -f /data/out.txt
```
