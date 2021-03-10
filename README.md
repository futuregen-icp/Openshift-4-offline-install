# Openshift 4.6 offline installation

```
## OS Version
Red Hat Enterprise Linux Server release 7.9 (Maipo)

## Install packages

yum install -y vim jq httpd-tools podman skokepo

## Definitions

Domain : oss2.fu.igotit.co.kr
Mirror registry domain name : registry.oss2.fu.igotit.co.kr

## DNS recode

registry 	IN	A	192.168.5.10
api		IN	A	192.168.5.101
api-int		IN	A	192.168.5.101
apps		IN	A	192.168.5.111
bootstrap	IN	A	192.168.5.100
master1		IN	A	192.168.5.101
master2		IN	A	192.168.5.102
master3		IN	A	192.168.5.103
worker1		IN	A	192.168.5.111
worker2		IN	A	192.168.5.112
etcd-0		IN	A	192.168.5.101
etcd-1		IN	A	192.168.5.102
etcd-2		IN 	A	192.168.5.103
_etcd-server-ssl._tcp   IN SRV 0 10 2380 etcd-0
_etcd-server-ssl._tcp   IN SRV 0 10 2380 etcd-1
_etcd-server-ssl._tcp   IN SRV 0 10 2380 etcd-2

## Download file

wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.6.15/openshift-install-linux.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.6.15/openshift-client-linux.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.6/4.6.1/rhcos-installer.x86_64.iso
wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.6/4.6.3/rhcos-metal.x86_64.raw.gz

## SSH key 생성
ssh-keygen -t rsa -b 4096 -N '' -f ~/.ssh/id_rsa
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa

## 인증서 정보 업데이트
cd /opt/registry/certs
openssl req -newkey rsa:4096 -nodes -sha256 -keyout registry.oss2.fu.igotit.co.kr.key -x509 -days 3650 -out registry.oss2.fu.igotit.co.kr.crt
cp /opt/registry/certs/registry.oss2.fu.igotit.co.kr.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust

## ing

podman run -d --name mirror-registry -p 5000:5000 --restart=always \
   -v /opt/registry/data:/var/lib/registry \
   -v /opt/registry/auth:/auth \
   -e "REGISTRY_AUTH=htpasswd" \
   -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
   -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
   -v /opt/registry/certs:/certs \
   -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.oss2.fu.igotit.co.kr.crt \
   -e REGISTRY_HTTP_TLS_KEY=/certs/registry.oss2.fu.igotit.co.kr.key \
   docker.io/library/registry:2
```


## registry pull secret

```
## copy for reference

"registry.connect.redhat.com": { 
      "auth": "fHVoYy1wb29sLWI1N2EwOTA4LWViZTktNGVkYi1iMzMyLWI5N2UyNjA2NzlmNzpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSXlZVE5tTWpSa09UTTRNRGswTVdOaVlqZGpOemsxTURabU1qZG1OVGsxTVNKOS5FMnRkcWYyZ2kybllONjM1NXZ4QVhOSjZWckZCS1JDX21fVkd6dDEwNnFrbm54VzBWMVVGYXMzbTNNXzUwM3RtSmVtSWtJd1RZbXQxSUIwaW82R19ETWxPd1VTcnlyUk1DYXNlYzZuMHI5NmlRMzdkRlo0dDVVMFpCTWZYVTBSbVQ2blg0dkI2N3JDd0R1a0xCaFZjZUxWYTlabDFEWG4waVJJU2Rtcm9Iak11OFhzeE1YQXU0d3c1em5zcVd6cXE3NEZjT2JDQW5SeWpKdS1lLVIyc1I5ZWVzNUJhN0oxRkJySzBBWDI1dG04c2lELWVKU2xrSURfMEc1ZnNRazVZdkpRLXo3YW03MnNYakRyWGpiRmc5QjNrb3hGT2t0UTFmNm41X050YVJpR2lZVDdEQTNNQi1oS1VxNkFlam92UkJLNlp6YTREUkR6U3B2MUdvRUM2WUFxTlpnV1U3Wkx6bXZpdzdfV2pTY2tvaFNTOWV1R1k2TGVUQnF0VE1qRjVLWllMSVg3UUxuSkgzTnpzRW5BbFdjbHdQcTNJSE5KUDBwWTdxU1NPcFY2S01iX2s5VnFrcGVfRm9qd0o5RGZTN0dNdEhnOWFQZXNYQXFsUzlNTm0xNUZDYnUzQVl1ejNDMmE5cThqNjkwcUFLSTV1eHhtai1NVUVfMEZaQ010T0xyMnhId3hKc1VsY0ZfQWpmalhzcjRWZWJFcXR2ZFItOUdvV2tsM0xxMnJUbGIxVHR6ZFZYaGg3SF9WQW50UUJ0Y1FxQlJva0RpNl82NFBFR3NmVThlTEx6aktwVkFiTktMY1NXWmZja3NWa09JNy1CTUhKNXJKei0zZmZyMHFvQkZfaUc0OEljY0VpLVVBT00wM0NEZ1o1Y005M3plWkktc1ZJd3EwbFowOA==", 
      "email": "dsleecom@futuregen.co.kr" 
    },

## Mirror registry
registry.oss2.fu.igotit.co.kr
admin:$2y$05$PE46rV6j/LJPYMyxrCYkjO/lJbnPfGJ51xpJ.G1MuIjJEBDd88IIu

## Paste

"registry.oss2.fu.igotit.co.kr": { 
      "auth": "admin:$2y$05$PE46rV6j/LJPYMyxrCYkjO/lJbnPfGJ51xpJ.G1MuIjJEBDd88IIu", 
      "email": "dsleecom@futuregen.co.kr" 
    },
```

## Mirror registry

```
OCP_RELEASE=4.6.15
LOCAL_REGISTRY='registry.oss2.fu.igotit.co.kr:5000'
LOCAL_REPOSITORY='ocp4/openshift4'
PRODUCT_REPO='openshift-release-dev'
LOCAL_SECRET_JSON='/root/.docker/config.json'
RELEASE_NAME='ocp-release'
ARCHITECTURE='x86_64'
REMOVABLE_MEDIA_PATH=''

## Review the images and configuration manifests to mirror:
oc adm release mirror -a ${LOCAL_SECRET_JSON}  \ 
     --from=quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-${ARCHITECTURE} \ 
     --to=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} \ 
     --to-release-image=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}-${ARCHITECTURE} --dry-run

## Mirror the images to a directory on the removable media:
oc adm release mirror -a ${LOCAL_SECRET_JSON} --to-dir=${REMOVABLE_MEDIA_PATH}/mirror quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-${ARCHITECTURE}
oc adm release mirror -a /root/.docker/config.json --to-dir=/mirror quay.io/openshift-release-dev/ocp-release:4.6.16-x86_64 

## Take the media to the restricted network environment and upload the images to the local container registry.
oc image mirror -a ${LOCAL_SECRET_JSON} --from-dir=${REMOVABLE_MEDIA_PATH}/mirror "file://openshift/release:${OCP_RELEASE}*" ${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} --registry-config=/opt/registry/pull/pull-secret.json

GODEBUG=x509ignoreCN=0 oc image mirror -a ${LOCAL_SECRET_JSON} --from-dir=${REMOVABLE_MEDIA_PATH}/mirror "file://openshift/release:${OCP_RELEASE}*" ${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}

GODEBUG=x509ignoreCN=0 oc image mirror --insecure=true -a ${LOCAL_SECRET_JSON} --from-dir=${REMOVABLE_MEDIA_PATH}/mirror "file://openshift/release:${OCP_RELEASE}*" ${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} 

## Directly push the release images to the local registry by using following command:
oc adm release mirror -a ${LOCAL_SECRET_JSON}  \
     --from=quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-${ARCHITECTURE} \
     --to=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} \
     --to-release-image=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}-${ARCHITECTURE}

GODEBUG=x509ignoreCN=0 oc adm release mirror -a ${LOCAL_SECRET_JSON}  \ 
     --from=quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-${ARCHITECTURE} \ 
     --to=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} \ 
     --to-release-image=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}-${ARCHITECTURE}

GODEBUG=x509ignoreCN=0 oc adm release extract -a ${LOCAL_SECRET_JSON} --command=openshift-install "${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}-${ARCHITECTURE}"

## Issue logs
1. oc adm release mirror no basic auth credentials

## Solutions
1. oc adm release extract --command=openshift-install "${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}" --registry-config=/root/pull-secret.json - fail
https://access.redhat.com/solutions/4515861

## Test
oc adm -a ${LOCAL_SECRET_JSON} release mirror \ 
--from=quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE} \ 
--to=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} \ 
--to-release-image=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE} \ 
2>&1 | tee ${REGISTRY_BASE}/downloads/secrets/mirror-output.txt

```

## Remark

```
/var/lib/containers/storage
```

## Pull Secret text

```
{"auths":{"cloud.openshift.com":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K29jbV9hY2Nlc3NfNjU1MzYzZmNhYWFkNDkxOWJhNDhmZDBkMzUyYTA3NGE6MUhIV09MTFo1WDE5MVM4SFlOUkQ3Sk9ZVkJBSzI5UFRaMkxWVEc0NDhNRlpKRUVVTEFRMEtBT1E4S1pVV0dVUA==","email":"dsleecom@futuregen.co.kr"},"quay.io":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K29jbV9hY2Nlc3NfNjU1MzYzZmNhYWFkNDkxOWJhNDhmZDBkMzUyYTA3NGE6MUhIV09MTFo1WDE5MVM4SFlOUkQ3Sk9ZVkJBSzI5UFRaMkxWVEc0NDhNRlpKRUVVTEFRMEtBT1E4S1pVV0dVUA==","email":"dsleecom@futuregen.co.kr"},"registry.connect.redhat.com":{"auth":"fHVoYy1wb29sLWI1N2EwOTA4LWViZTktNGVkYi1iMzMyLWI5N2UyNjA2NzlmNzpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSXlZVE5tTWpSa09UTTRNRGswTVdOaVlqZGpOemsxTURabU1qZG1OVGsxTVNKOS5FMnRkcWYyZ2kybllONjM1NXZ4QVhOSjZWckZCS1JDX21fVkd6dDEwNnFrbm54VzBWMVVGYXMzbTNNXzUwM3RtSmVtSWtJd1RZbXQxSUIwaW82R19ETWxPd1VTcnlyUk1DYXNlYzZuMHI5NmlRMzdkRlo0dDVVMFpCTWZYVTBSbVQ2blg0dkI2N3JDd0R1a0xCaFZjZUxWYTlabDFEWG4waVJJU2Rtcm9Iak11OFhzeE1YQXU0d3c1em5zcVd6cXE3NEZjT2JDQW5SeWpKdS1lLVIyc1I5ZWVzNUJhN0oxRkJySzBBWDI1dG04c2lELWVKU2xrSURfMEc1ZnNRazVZdkpRLXo3YW03MnNYakRyWGpiRmc5QjNrb3hGT2t0UTFmNm41X050YVJpR2lZVDdEQTNNQi1oS1VxNkFlam92UkJLNlp6YTREUkR6U3B2MUdvRUM2WUFxTlpnV1U3Wkx6bXZpdzdfV2pTY2tvaFNTOWV1R1k2TGVUQnF0VE1qRjVLWllMSVg3UUxuSkgzTnpzRW5BbFdjbHdQcTNJSE5KUDBwWTdxU1NPcFY2S01iX2s5VnFrcGVfRm9qd0o5RGZTN0dNdEhnOWFQZXNYQXFsUzlNTm0xNUZDYnUzQVl1ejNDMmE5cThqNjkwcUFLSTV1eHhtai1NVUVfMEZaQ010T0xyMnhId3hKc1VsY0ZfQWpmalhzcjRWZWJFcXR2ZFItOUdvV2tsM0xxMnJUbGIxVHR6ZFZYaGg3SF9WQW50UUJ0Y1FxQlJva0RpNl82NFBFR3NmVThlTEx6aktwVkFiTktMY1NXWmZja3NWa09JNy1CTUhKNXJKei0zZmZyMHFvQkZfaUc0OEljY0VpLVVBT00wM0NEZ1o1Y005M3plWkktc1ZJd3EwbFowOA==","email":"dsleecom@futuregen.co.kr"},"registry.redhat.io":{"auth":"fHVoYy1wb29sLWI1N2EwOTA4LWViZTktNGVkYi1iMzMyLWI5N2UyNjA2NzlmNzpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSXlZVE5tTWpSa09UTTRNRGswTVdOaVlqZGpOemsxTURabU1qZG1OVGsxTVNKOS5FMnRkcWYyZ2kybllONjM1NXZ4QVhOSjZWckZCS1JDX21fVkd6dDEwNnFrbm54VzBWMVVGYXMzbTNNXzUwM3RtSmVtSWtJd1RZbXQxSUIwaW82R19ETWxPd1VTcnlyUk1DYXNlYzZuMHI5NmlRMzdkRlo0dDVVMFpCTWZYVTBSbVQ2blg0dkI2N3JDd0R1a0xCaFZjZUxWYTlabDFEWG4waVJJU2Rtcm9Iak11OFhzeE1YQXU0d3c1em5zcVd6cXE3NEZjT2JDQW5SeWpKdS1lLVIyc1I5ZWVzNUJhN0oxRkJySzBBWDI1dG04c2lELWVKU2xrSURfMEc1ZnNRazVZdkpRLXo3YW03MnNYakRyWGpiRmc5QjNrb3hGT2t0UTFmNm41X050YVJpR2lZVDdEQTNNQi1oS1VxNkFlam92UkJLNlp6YTREUkR6U3B2MUdvRUM2WUFxTlpnV1U3Wkx6bXZpdzdfV2pTY2tvaFNTOWV1R1k2TGVUQnF0VE1qRjVLWllMSVg3UUxuSkgzTnpzRW5BbFdjbHdQcTNJSE5KUDBwWTdxU1NPcFY2S01iX2s5VnFrcGVfRm9qd0o5RGZTN0dNdEhnOWFQZXNYQXFsUzlNTm0xNUZDYnUzQVl1ejNDMmE5cThqNjkwcUFLSTV1eHhtai1NVUVfMEZaQ010T0xyMnhId3hKc1VsY0ZfQWpmalhzcjRWZWJFcXR2ZFItOUdvV2tsM0xxMnJUbGIxVHR6ZFZYaGg3SF9WQW50UUJ0Y1FxQlJva0RpNl82NFBFR3NmVThlTEx6aktwVkFiTktMY1NXWmZja3NWa09JNy1CTUhKNXJKei0zZmZyMHFvQkZfaUc0OEljY0VpLVVBT00wM0NEZ1o1Y005M3plWkktc1ZJd3EwbFowOA==","email":"dsleecom@futuregen.co.kr"}}}
```


## refer 
- https://daein.medium.com/restricted-network-bare-metal-upi-installation-on-openshift-v4-6-c796fab3a013

## install-config.yaml

```
apiVersion: v1 
baseDomain: fu.igotit.co.kr
compute: 
- hyperthreading: Enabled    
  name: worker 
  replicas: 0  
controlPlane: 
  hyperthreading: Enabled    
  name: master  
  replicas: 3  
metadata: 
  name: oss2
networking: 
  clusterNetwork: 
  - cidr: 10.128.0.0/14  
    hostPrefix: 23  
  networkType: OpenShiftSDN 
  serviceNetwork:  
  - 172.30.0.0/16 
platform: 
  none: {}  
fips: false  
pullSecret: '{"auths":{"registry.oss2.fu.igotit.co.kr:5000":{"auth":"YWRtaW46YWRtaW4="}}}' 
sshKey: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDRe2jDdYKnA42s4TMgKqLIaM/CpGZuToIeERR0zEuwpWcVb4GDaI+8TocB8FIx+Kkr0KcqC8xSWvqeW1q1+/Zocq6l120VC5DFRXtKe7/1LY70e/qNEIE5z8D4oMlLor4xu61PCUNlrS3KZsjYMEDjQBn0JxRpODevaO4/8JAMbYv6nnSKO1SHqmH23L0pGsxPMy/d/amCcsfKR/QZgb1mJVxFk91QumnFE84dT5tyeXLnyooMKeyDDEifYDLz9P7qqTyJJ6cnR66lFfFK2rvkNvjKLXtF0lCmvVw5a21EBaLsH1pIYwgMRDU1P4yEpiywWmdg8VTj3W1GWV0yHLHCgJ2jREnGG97LjBVak3jsHFHToNsC5vsTFCt6rRGEUMPQPAu3biv3zpgzGVFk/9RgtST9Svgj15vS8KD7rXNxm4x97OE+mI7dmZOfbVGSoqn1n9D+3vgkf9muPLm/ZEEi9Ct1bhq8e7bSuowf7Ikl6F79M0OU4TVr0XvBTjewcEFb/dU+DGTUXZvH2tlzuqFEXN4pnTDUj5EVBNRLRdbUEvwewPWGo5rEVRkNRxPr6PpmswWmE1pyqhsC/3Ka/GUi7zvBM3MeZzsfscuEI5ZvfH/zR9Xrse5EwTIeUGunhFWkz5b/PqBqm9d5/g0ZZE9EdxdDmGo85ev4dAjsmXTZRQ== root@lab1-bastion' 
imageContentSources:
- mirrors:
  - registry.oss2.fu.igotit.co.kr:5000/ocp4/openshift4
  source: quay.io/openshift-release-dev/ocp-release
- mirrors:
  - registry.oss2.fu.igotit.co.kr:5000/ocp4/openshift4
  source: quay.io/openshift-release-dev/ocp-v4.0-art-dev
```

## openshift install

```
manifest dir define : /opt/ocp4/install-20210305
openshift-install create manifests --dir=/opt/ocp4/install-20210305
openshift-install create ignition-configs --dir=/opt/ocp4/install-20210305
```

## Bootstrap

```
sudo nmcli con mod "Wired Connection" ipv4.method manual  ipv4.addresses 192.168.5.100/24 ipv4.gateway 192.168.5.1 ipv4.dns 192.168.5.1 
sudo nmcli con down "Wired Connection" 
sudo nmcli con up   "Wired Connection" 
sudo nmcli general hostname bootstrap.oss2.fu.igotit.co.kr 
sudo coreos-installer install /dev/sda \ 
  --ignition-url=http://192.168.5.10/bootstrap.ign \ 
  --insecure-ignition \ 
  --append-karg "ip=192.168.5.100::192.168.5.1:255.255.255.0:bootstrap.oss2.fu.igotit.co.kr:ens192:none nameserver=192.168.5.1"
```
## init. for Master1

```
sudo nmcli con mod "Wired Connection" ipv4.method manual  ipv4.addresses 192.168.5.101/24 ipv4.gateway 192.168.5.1 ipv4.dns 192.168.5.1 
sudo nmcli con down "Wired Connection" 
sudo nmcli con up   "Wired Connection" 
sudo nmcli general hostname master1.oss2.fu.igotit.co.kr 
sudo coreos-installer install /dev/sda \ 
  --ignition-url=http://192.168.5.10/master.ign \ 
  --insecure-ignition \ 
  --append-karg "ip=192.168.5.101::192.168.5.1:255.255.255.0:master1.oss2.fu.igotit.co.kr:ens192:none nameserver=192.168.5.1"
```

## Verify check

```
openshift-install --dir=/opt/ocp4.6/ocp-install-manifest wait-for bootstrap-complete --log-level=info
export KUBECONFIG=<installation_directory>/auth/kubeconfig

GODEBUG=x509ignoreCN=0 oc image mirror -a ${LOCAL_SECRET_JSON} --keep-manifest-list=true quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:b62cc7b300ec0e9f9c13867437e1880f59e8dab49e5d1f167acce11533bdd294=registry.oss2.fu.igotit.co.kr:5000/ocp4/openshift4
GODEBUG=x509ignoreCN=0 oc image mirror -a ${LOCAL_SECRET_JSON} --keep-manifest-list=true quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:21dec37fcc47ac858cca0a7dacfe360a0e5098e2b3e3e70e2fc777c2455881ff=registry.oss2.fu.igotit.co.kr:5000/ocp4/openshift4

curl -u admin:admin -k https://registry.oss2.fu.igotit.co.kr:5000/v2/_catalog
```

## Operator Mirror 

```
oc patch OperatorHub cluster --type json \ 
    -p '[{"op": "add", "path": "/spec/disableAllDefaultSources", "value": true}]'
```
