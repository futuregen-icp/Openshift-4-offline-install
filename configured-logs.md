+ Configure the Local Registry
```
oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'

oc edit configs.imageregistry.operator.openshift.io

storage:
  pvc:
    claim:


# nfs를 이용한 pv 생성 

vi pvc-image-registry.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: "image-registry-storage"
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 100Gi
  nfs:
    path: /nfs1/registry
    server: 192.168.5.10
  persistentVolumeReclaimPolicy: Retain

apiVersion: v1
kind: PersistentVolume
metadata:
  name: image-registry-storage
  annotations:
    volume.beta.kubernetes.io/mount-options: rw,nfsvers=4,noexec 
spec:
  capacity:
    storage: 100Gi
  accessModes:
  - ReadWriteMany
  nfs:
    path: /nfs1/registry
    server: 192.168.5.10
  persistentVolumeReclaimPolicy: Retain
  claimRef:
    name: image-registry-storage
    namespace: openshift-image-registry

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: image-registry-storage 
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi 


$ oc create -f storage/pv-image-registry.yaml

$ oc edit configs.imageregistry.operator.openshift.io/cluster
...
managementState: Managed (이전값 : Removed )
...
storage:
    pvc:
      claim:
...
storage:
    pvc:
      claim:
```

+ User management
```

/opt/ocp4.6/ocp-install-20210223/oauth/hapasswd
oc create secret generic htpass-secret --from-file=htpasswd=/opt/install/oauth/htpasswd -n openshift-config

oc create secret generic htpass-secret --from-file=/ocp/ocp4.6/ocp-install-20210223/oauth/htpasswd -n openshift-config

oc adm policy add-cluster-role-to-user cluster-admin admin admin
oc adm policy add-role-to-user admin admin
```

+ Custom Image : Create container image as openjdk-11-rhel7 
```
podman pull registry.access.redhat.com/openjdk/openjdk-11-rhel7:latest
podman tag registry.access.redhat.com/openjdk/openjdk-11-rhel7:latest registry.oss2.fu.igotit.co.kr:5000/openjdk/openjdk:8
podman tag registry.access.redhat.com/openjdk/openjdk-11-rhel7:latest registry.oss2.fu.igotit.co.kr:5000/openjdk/openjdk:latest
podman push registry.oss2.fu.igotit.co.kr:5000/openjdk/openjdk:8
podman push registry.oss2.fu.igotit.co.kr:5000/openjdk/openjdk:latest

```
