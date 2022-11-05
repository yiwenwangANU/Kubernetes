# Storage
## Questions
- Create the volume for a pod (`volume`)
- Create persistent volume (by storage class)+ persistent volume claim, update the pod to use the persistent volume (`persistent volume, storage class`)
##  Volume on Host ([volume](https://kubernetes.io/docs/concepts/storage/volumes/) )

Specify the volume of a pod for persistent storage, data will not lose when pod deleted
```sh
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: Directory
```

##  Persistent Volumes ([persistent volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) )

In order to centrally manage the volume of the pod, create Persistent Volume as EBS volume and Persistent Volume Claim as I want EBS volume.

Create the persistent volume
```sh
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

Create the persistent volume claim
```sh
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]
```

Volume claim will automitically find the most matched persistent volume.

Finally use the claim as the volume in the pod difinition file
```sh
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

##  Storage Class ([storage class](https://kubernetes.io/docs/concepts/storage/storage-classes/) )

I don't want to provision the volume on cloud or create the PV, so I use storage class to provision volume automatically for the PVC. Command varied by different cloud providers.
```sh
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```

Use the storage class to provision from the PVC by specify ` storageClassName: `