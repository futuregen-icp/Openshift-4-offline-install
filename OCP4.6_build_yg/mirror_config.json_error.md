###  Mirror registry
```
OCP_RELEASE=4.6.15
LOCAL_REGISTRY='registry.yo4.ocp3.fu.igotit.co.kr:5000'
LOCAL_REPOSITORY='ocp4/openshift4'
PRODUCT_REPO='openshift-release-dev'
LOCAL_SECRET_JSON='/home/core/.docker/config.json'
RELEASE_NAME='ocp-release'
ARCHITECTURE='x86_64'
REMOVABLE_MEDIA_PATH=''
``` 
### check commend
```
oc adm release mirror -a ${LOCAL_SECRET_JSON} --from=quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-${ARCHITECTURE} --to=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} --to-release-image=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}-${ARCHITECTURE} --dry-run
error: unable to retrieve release image info: unable to load --registry-config: error occurred while trying to unmarshal json
```

### error list
```
-rw-r--r--. 1 root root 3014 Jun 23 10:10 config.json
-rw-r--r--. 1 root root 3014 Jun 23 13:55 config.json.old
[root@bastion .docker]# vi config.json
[root@bastion .docker]# oc adm release mirror -a ${LOCAL_SECRET_JSON} --from=quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-${ARCHITECTURE} --to=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} --to-release-image=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}-${ARCHITECTURE} --dry-run
error: unable to retrieve release image info: unable to read image quay.io/openshift-release-dev/ocp-release:4.6.15-x86_64: Head "https://quay.io/v2/openshift-release-dev/ocp-release/manifests/4.6.15-x86_64": unauthorized: Invalid Username or Password

 solution
 미러레지스트리 부분 인증 부분에 'admin'추가
```

```
[root@bastion .docker]# vi config.json  
[root@bastion .docker]# vi config.json                                                                   
[root@bastion .docker]# echo -n 'admin:admin' | base64 -w0                                               
                        YWRtaW46YWRtaW4=[root@bavi config.json 
[root@bastion .docker]# oc adm release mirror -a ${LOCAL_SECRET_JSON} --from=quay.io/
                        ${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-${ARCHITECTURE} --to=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} --to-release-image=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}-${ARCHITECTURE} --dry-run
error: unable to retrieve release image info: unable to load --registry-config: error occurred while trying to unmarshal json
```
```
 solution
 방화벽 추가 및  pullsecret 재구성
firewall-cmd --add-port = 5000 / tcp --zone = internal --permanent 
firewall-cmd --add-port = 5000 / tcp --zone = public --permanent
```
###  pull secret
```
"auths": {                                                                                                                                      
   "cloud.openshift.com": {                                                                                                                    
"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K2h5ZzY1NjUxY3FyZXo4ZnZ...................................................................................                                                                                                   "email": "hyg6565@futuregen.co.kr"                                                                                                          },                                                                                                                                              

"quay.io": {                                                                                                                                    "auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K2h5ZzY1NjUxY3FyZXo4Zn................................................................................... "email": "hyg6565@futuregen.co.kr"


[root@bastion .docker]# oc adm release mirror -a ${LOCAL_SECRET_JSON} --from=quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-${ARCHITECTURE} --to=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} --to-release-image=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}-${ARCHITECTURE} --dry-run
error: must specify a release image with --from
```

```
solution
 pull secret 다운 (기존 paste 하여 작업)
 ```
 
