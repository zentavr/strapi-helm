# TODO
- [ ] PV. We must honor additional CSI parameters. EFS on Fargate supports only static volume provisioning.
  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: efs-pv
  spec:
    capacity:
      storage: 5Gi
    volumeMode: Filesystem
    accessModes:
      - ReadWriteMany
    persistentVolumeReclaimPolicy: Retain
    storageClassName: efs-sc
    csi:
      driver: efs.csi.aws.com
      volumeHandle: fs-4af69aab
  ```
- [x] PVC
  - [ ] In case of AWS we need to use more features, i.e.:
    * `volumeName`
    * `selector`. PV should have additional labels!!!
    Maybe the solution could be:
    1. Define `.persistence.enabled` as `true`
    2. Define `.persistence.existingClaim` to some value
    3. Define `.extraDeploy` with `PersistentVolumeClaim` and `PersistentVolume` extra objects. 
- [ ] Secrets. Can we use Amazon Secret Manager?
  * [ ] Secrets for DB
  * [ ] Secrets for something else?
- [ ] Service Account: Do we need it? Probably yes.
- [ ] Refactor Ingress
- [ ] refactor Service
- [ ] refactor deployment
- [ ] Do we need Extralist? In case we want to extra deploy something.
- [ ] TLS Secrets
- [ ] Network Policies
  * [ ] Backend Ingress
  * [ ] Egress
  * [ ] Ingress

Multiply pods spec:
```yaml
---
# storageclass.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
---
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-claim
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
---
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: efs-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: efs-sc
  csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-4af69aab
---
# pod1.yml
apiVersion: v1
kind: Pod
metadata:
  name: app1
spec:
  containers:
  - name: app1
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(date -u) >> /data/out1.txt; sleep 5; done"]
    volumeMounts:
    - name: persistent-storage
      mountPath: /data
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: efs-claim
```


---
[Bitnami Chart Template]: https://github.com/bitnami/charts/tree/main/bitnami/ghost/templates
[PersistentVolumeClaim API Ref]: https://docs.openshift.com/container-platform/3.11/rest_api/core/persistentvolumeclaim-core-v1.html
[PV and PVCs Examples]: https://rtfm.co.ua/ru/kubernetes-persistentvolume-i-persistentvolumeclaim-obzor-i-primery/
