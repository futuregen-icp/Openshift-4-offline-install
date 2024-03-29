## Definitions

- Openshift verision : 4.6.35
- Servers
```
l2-10-base1.lab2.dslee.lab 172.10.20.10 (200G, dns,haproxy)
l2-11-nfs1.lab2.dslee.lab 172.10.20.11 (400G, registry/nfs/repos)
l2-31-master1.lab2.dslee.lab 172.10.20.31
l2-32-master2.lab2.dslee.lab 172.10.20.32
l2-33-master3.lab2.dslee.lab 172.10.20.33
l2-41-infra1.lab2.dslee.lab 172.10.20.41
l2-42-infra2.lab2.dslee.lab 172.10.20.42
```

## Configure Repository
- Repository path
```
repos download_path=/var/ftp/pub/repos
```


-  Configure subscriptions
```
subscription-manager repos --disable="*"

subscription-manager repos     --enable="rhel-7-server-rpms"     \
	                           --enable="rhel-7-server-extras-rpms"     \
	                           --enable="rhel-7-server-ose-4.6-rpms"     \
	                           --enable="rhel-7-server-ansible-2.9-rpms"

sudo yum -y install yum-utils createrepo docker git vsftpd

yum -y install httpd
```

- Download repogitory (Duration time : It will probably take about 5 hours.)
```
for repo in rhel-7-server-rpms rhel-7-server-extras-rpms rhel-7-server-ansible-2.9-rpms rhel-7-server-ose-4.6-rpms
do
	reposync --gpgcheck -lm --repoid=${repo} --download_path=/var/ftp/pub/repos
	createrepo -v /var/ftp/pub/repos/${repo} -o /var/ftp/pub/repos${repo}
done
```

- FTP deamon start 
```
systemctl enable vsftpd
systemctl start vsftpd
```

## Openshift installation
- Download fils
```
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.6.35/openshift-install-linux.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.6.35/openshift-client-linux.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.6/4.6.1/rhcos-installer.x86_64.iso
wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.6/4.6.8/rhcos-metal.x86_64.raw.gz
```

- Install package 
```
yum install -y vim jq httpd-tools podman skopeo
```

- Configure mirror registry 
```
yum install -y vim jq httpd-tools podman skopeo

mkdir -p /opt/registry/data 
mkdir -p /opt/registry/auth 
mkdir -p /opt/registry/certs

cd /opt/registry/certs
openssl req -newkey rsa:4096 -nodes -sha256 -keyout mirrorregistry.lab2.dslee.lab.key -x509 -days 3650 -out mirrorregistry.lab2.dslee.lab.crt
cp /opt/registry/certs/mirrorregistry.lab2.dslee.lab.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust

htpasswd -bBc /opt/registry/auth/htpasswd admin admin 
vi /etc/containers/registries.conf 
[registries.search]
registries = ['mirrorregistry.lab2.dslee.lab:5000', 'registry.access.redhat.com', 'registry.redhat.io']

podman run -d --name mirror-registry -p 5000:5000 --restart=always \
   -v /opt/registry/data:/var/lib/registry \
   -v /opt/registry/auth:/auth \
   -e "REGISTRY_AUTH=htpasswd" \
   -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
   -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
   -v /opt/registry/certs:/certs \
   -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/mirrorregistry.lab2.dslee.lab.crt \
   -e REGISTRY_HTTP_TLS_KEY=/certs/mirrorregistry.lab2.dslee.lab.key \
   docker.io/library/registry:2

cat pull-secret.text | jq . > pull-secret.json

echo -n 'admin:admin' | base64 -w0
YWRtaW46YWRtaW4=

    "mirrorregistry.lab2.dslee.lab": { 
      "auth": "YWRtaW46YWRtaW4=", 
      "email": "you@example.com"
  },

## vsersion check mirror registry
[root@l2-10-base1 registry]# openshift-install version
openshift-install 4.6.35
built from commit 68ab13d26311a3e03854a00fd7cf5b1583ae9b69
release image quay.io/openshift-release-dev/ocp-release@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d

## settting environments
OCP_RELEASE=4.6.35
LOCAL_REGISTRY='mirrorregistry.lab2.dslee.lab:5000'
LOCAL_REPOSITORY='ocp4/openshift4'
PRODUCT_REPO='openshift-release-dev'
LOCAL_SECRET_JSON='/opt/registry/pull-secret.json'
RELEASE_NAME='ocp-release'
ARCHITECTURE='x86_64'
REMOVABLE_MEDIA_PATH='/data'

## validation check mirror registry
oc adm release mirror -a ${LOCAL_SECRET_JSON}  \
     --from=quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-${ARCHITECTURE} \
     --to=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} \
     --to-release-image=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}-${ARCHITECTURE} --dry-run

## download mirror images to local path
oc adm release mirror -a ${LOCAL_SECRET_JSON} --to-dir=${REMOVABLE_MEDIA_PATH}/mirror quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-${ARCHITECTURE}

## disalbe selinux
sestatus
setenforce 0

## Take the media to the restricted network environment and upload the images to the local container registry.
GODEBUG=x509ignoreCN=0  oc image mirror -a ${LOCAL_SECRET_JSON} --from-dir=${REMOVABLE_MEDIA_PATH}/mirror "file://openshift/release:${OCP_RELEASE}*" ${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} --registry-config=/opt/registry/pull-secret.json
GODEBUG=x509ignoreCN=0 oc adm release extract -a ${LOCAL_SECRET_JSON} --command=openshift-install "${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}" --image



GODEBUG=x509ignoreCN=0 podman login mirrorregistry.lab2.dslee.lab:5000

## Definitions
mirrorregistry.lab2.dslee.lab
```

- Bootstrap
```
## install config
imageContentSources:
- mirrors:
  - mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4
  source: quay.io/openshift-release-dev/ocp-release
- mirrors:
  - mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4
  source: quay.io/openshift-release-dev/ocp-v4.0-art-dev

imageContentSources:
- mirrors:
  - mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4
  source: quay.io/openshift-release-dev/ocp-release
- mirrors:
  - mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4
  source: quay.io/openshift-release-dev/ocp-v4.0-art-dev


## Openshift install command

manifest dir define : /opt/ocp4/install-20210622
cd /opt/ocp4/install-20210622
openshift-install create manifests --dir=/opt/ocp4/install-20210622
openshift-install create ignition-configs --dir=/opt/ocp4/install-20210622

manifest dir define : /opt/ocp4/install-20210622-1
cd /opt/ocp4/install-20210622-1
openshift-install create manifests --dir=/opt/ocp4/install-20210622-1
openshift-install create ignition-configs --dir=/opt/ocp4/install-20210622-1

manifest dir define : /opt/ocp4/install-20210622-2
cd /opt/ocp4/install-20210622-2
openshift-install create manifests --dir=/opt/ocp4/install-20210622-2
openshift-install create ignition-configs --dir=/opt/ocp4/install-20210622-2

manifest dir define : /opt/ocp4/install-20210622-3
cd /opt/ocp4/install-20210622-3
openshift-install create manifests --dir=/opt/ocp4/install-20210622-3
openshift-install create ignition-configs --dir=/opt/ocp4/install-20210622-3

manifest dir define : /opt/ocp4/install-20210623
cd /opt/ocp4/install-20210623
openshift-install create manifests --dir=/opt/ocp4/install-20210623
openshift-install create ignition-configs --dir=/opt/ocp4/install-20210623

openshift-install --dir=/opt/ocp4/install-20210623 wait-for bootstrap-complete --log-level=info

## bootstrap

sudo nmcli con mod "Wired Connection" ipv4.method manual  ipv4.addresses 172.10.20.30/24 ipv4.gateway 172.10.20.10 ipv4.dns 172.10.20.10
sudo nmcli con down "Wired Connection" 
sudo nmcli con up   "Wired Connection" 
sudo nmcli general hostname l2-30-bootstrap.lab2.dslee.lab
sudo coreos-installer install /dev/sda \ 
  --ignition-url=http://172.10.20.10:8080/bootstrap.ign \ 
  --insecure-ignition \ 
  --append-karg "ip=172.10.20.30::172.10.20.10:255.255.255.0:l2-30-bootstrap.lab2.dslee.lab:eth0:none nameserver=172.10.20.10"

```

- Bootstrap complates
```
## Check bootstraper 
[root@l2-10-base1 .kube]# openshift-install --dir=/opt/ocp4/install-20210623 wait-for install-complete --log-level debug
DEBUG OpenShift Installer 4.6.35                   
DEBUG Built from commit 68ab13d26311a3e03854a00fd7cf5b1583ae9b69 
DEBUG Loading Install Config...                    
DEBUG   Loading SSH Key...                         
DEBUG   Loading Base Domain...                     
DEBUG     Loading Platform...                      
DEBUG   Loading Cluster Name...                    
DEBUG     Loading Base Domain...                   
DEBUG     Loading Platform...                      
DEBUG   Loading Pull Secret...                     
DEBUG   Loading Platform...                        
DEBUG Using Install Config loaded from state file  
INFO Waiting up to 40m0s for the cluster at https://api.lab2.dslee.lab:6443 to initialize... 
DEBUG Cluster is initialized                       
INFO Waiting up to 10m0s for the openshift-console route to be created... 
DEBUG Route found in openshift-console namespace: console 
DEBUG Route found in openshift-console namespace: downloads 
DEBUG OpenShift console route is created           
INFO Install complete!                            
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/opt/ocp4/install-20210623/auth/kubeconfig' 
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.lab2.dslee.lab 
INFO Login to the console with user: "kubeadmin", and password: "6W7fF-dLv2i-TaxdM-fwjR6" 
INFO Time elapsed: 0s   

## Check cluster operator
[root@l2-10-base1 .kube]# oc get co
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
authentication                             4.6.35    True        False         False      87m
cloud-credential                           4.6.35    True        False         False      5h21m
cluster-autoscaler                         4.6.35    True        False         False      5h6m
config-operator                            4.6.35    True        False         False      5h7m
console                                    4.6.35    True        False         False      87m
csi-snapshot-controller                    4.6.35    True        False         False      5h7m
dns                                        4.6.35    True        False         False      5h6m
etcd                                       4.6.35    True        False         False      4h54m
image-registry                             4.6.35    True        False         False      4h48m
ingress                                    4.6.35    True        False         False      3h56m
insights                                   4.6.35    True        False         False      5h7m
kube-apiserver                             4.6.35    True        False         False      4h53m
kube-controller-manager                    4.6.35    True        False         False      5h4m
kube-scheduler                             4.6.35    True        False         False      5h5m
kube-storage-version-migrator              4.6.35    True        False         False      3h56m
machine-api                                4.6.35    True        False         False      5h7m
machine-approver                           4.6.35    True        False         False      5h6m
machine-config                             4.6.35    True        False         False      5h6m
marketplace                                4.6.35    True        False         False      5h6m
monitoring                                 4.6.35    True        False         False      3h55m
network                                    4.6.35    True        False         False      5h7m
node-tuning                                4.6.35    True        False         False      5h7m
openshift-apiserver                        4.6.35    True        False         False      4h48m
openshift-controller-manager               4.6.35    True        False         False      5h6m
openshift-samples                          4.6.35    True        False         False      4h47m
operator-lifecycle-manager                 4.6.35    True        False         False      5h7m
operator-lifecycle-manager-catalog         4.6.35    True        False         False      5h7m
operator-lifecycle-manager-packageserver   4.6.35    True        False         False      4h48m
service-ca                                 4.6.35    True        False         False      5h7m
storage                                    4.6.35    True        False         False      5h7m
****
```

- Operator hub configuration
```
## Operator hub
oc get operatorhubs.config.openshift.io cluster -o yaml

oc patch operatorhubs.config.openshift.io cluster --type json -p '[{"op": "add", "path":
"/spec/disableAllDefaultSources", "value": true}]'

[root@l2-10-base1 .kube]# oc get operatorhubs.config.openshift.io cluster -o yaml
apiVersion: config.openshift.io/v1
kind: OperatorHub
metadata:
  annotations:
    release.openshift.io/create-only: "true"
  creationTimestamp: "2021-06-23T00:16:12Z"
  generation: 2
  managedFields:
  - apiVersion: config.openshift.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:release.openshift.io/create-only: {}
      f:spec: {}
    manager: cluster-version-operator
    operation: Update
    time: "2021-06-23T00:16:12Z"
  - apiVersion: config.openshift.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:spec:
        f:disableAllDefaultSources: {}
    manager: kubectl-patch
    operation: Update
    time: "2021-06-23T04:40:47Z"
  - apiVersion: config.openshift.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:status:
        .: {}
        f:sources: {}
    manager: marketplace-operator
    operation: Update
    time: "2021-06-23T04:40:47Z"
  name: cluster
  resourceVersion: "96520"
  selfLink: /apis/config.openshift.io/v1/operatorhubs/cluster
  uid: 1203860d-8949-452e-a81a-1f3fea9768ff
spec:
  disableAllDefaultSources: true
status:
  sources:
  - disabled: true
    message: CatalogSource.operators.coreos.com "redhat-operators" not found
    name: redhat-operators
    status: Error
  - disabled: true
    message: CatalogSource.operators.coreos.com "certified-operators" not found
    name: certified-operators
    status: Error
  - disabled: true
    message: CatalogSource.operators.coreos.com "community-operators" not found
    name: community-operators
    status: Error
  - disabled: true
    message: CatalogSource.operators.coreos.com "redhat-marketplace" not found
    name: redhat-marketplace
    status: Error

## Operatorhub example

cat catalogsource.yml

apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: my-operator-catalog
  namespace: openshift-marketplace
spec:
  sourceType: grpc
  image: bastion.ocp-dc.redfaceh.com:5000/mirror-olm/redhat-operator-index:v4.6
  displayName: My Operator Catalog
  publisher: RedFace
  updateStrategy:
    registryPoll:
      interval: 30m

oc create -f catalogsource.yml
oc get catalogsources.operators.coreos.com -n openshift-marketplace
```
- Tech sprint status
```
오픈시프트설치 - Done
코어os의 노드추가 - Done
rhel7의 노드 추가 - To do
클러스터 오퍼레이터 인증 이슈 해결 - Done
기본 카탈로그소스 비활성화 - Done
카탈로그소스 정의  - To do
NFS 설정 이미지레지스트 설정 - To do
스토리지 클래스 설정 - To do
사용자 관리 - To do
  - Htpasswd 
  - Active Directory - long term plan
인프라노드의 선택 - To do
모니터링 환경 구성 - To do
로깅 환경 구성 - To do
4.6.15 to 4.6.35 오프라인 버전 업그레이드  - To do
hostpath 구성 하고, pv 생성하고, pvc 생성하고, vm에 마운트 > 여러 파드에서 사용할 때에의 디렉토리 구조 확인 - To do
```
