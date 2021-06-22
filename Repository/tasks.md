- Servers
```
l2-10-base1.lab2.dslee.lab 172.10.20.10 (200G, dns,haproxy)
l2-11-nfs1.lab2.dslee.lab 172.10.20.11 (400G, registry/nfs/repos)
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

- Definitions
```
repos download_path=/var/ftp/pub/repos
openshift version : 4.6.35
role : registry
```

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

## Bootstrap
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


## bootstrap

sudo nmcli con mod "Wired Connection" ipv4.method manual  ipv4.addresses 172.10.20.30/24 ipv4.gateway 172.10.20.10 ipv4.dns 172.10.20.10
sudo nmcli con down "Wired Connection" 
sudo nmcli con up   "Wired Connection" 
sudo nmcli general hostname l2-30-bootstrap.lab2.dslee.lab
sudo coreos-installer install /dev/sda \ 
  --ignition-url=http://172.10.20.10:8080/bootstrap.ign \ 
  --insecure-ignition \ 
  --append-karg "ip=172.10.20.30::172.10.20.10:255.255.255.0:l2-30-bootstrap.lab2.dslee.lab:eth0:none nameserver=172.10.20.10"

## Validations
GODEBUG=x509ignoreCN=0 podman pull mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d
podman login redhat.io

## Issues
[root@l2-10-base1 ~]# podman pull mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d
Trying to pull mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d...
  Get "https://mirrorregistry.lab2.dslee.lab:5000/v2/": x509: certificate relies on legacy Common Name field, use SANs or temporarily enable Common Name matching with GODEBUG=x509ignoreCN=0
Error: Error initializing source docker://mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d: error pinging docker registry mirrorregistry.lab2.dslee.lab:5000: Get "https://mirrorregistry.lab2.dslee.lab:5000/v2/": x509: certificate relies on legacy Common Name field, use SANs or temporarily enable Common Name matching with GODEBUG=x509ignoreCN=0


[root@l2-10-base1 ~]# podman login mirrorregistry.lab2.dslee.lab:5000
Authenticating with existing credentials...
Existing credentials are invalid, please enter valid username and password
Username (admin): admin
Password: 
Error: error authenticating creds for "mirrorregistry.lab2.dslee.lab:5000": error pinging docker registry mirrorregistry.lab2.dslee.lab:5000: Get "https://mirrorregistry.lab2.dslee.lab:5000/v2/": x509: certificate relies on legacy Common Name field, use SANs or temporarily enable Common Name matching with GODEBUG=x509ignoreCN=0
[root@l2-10-base1 ~]# 
```
