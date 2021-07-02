## mount 실수로 인한 node 재구성

## mirror registry error logs

### podman error

####  mirror.sh
```
[root@bastion ~]# vi /opt/registry/data/mirror.sh

#!/bin/bash

podman run -d --name mirror-registry -p 5000:5000 --restart=always \
   -v /opt/registry/data:/var/lib/registry:z \
   -v /opt/registry/auth:/auth:z \
   -e "REGISTRY_AUTH=htpasswd" \
   -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
   -e REGISTRY_AUTH_HTPASSWD_PATH=/opt/registry/auth/htpasswd \
   -v /opt/registry/certs:/certs:z \
   -e REGISTRY_HTTP_TLS_CERTIFICATE=/opt/registry/certs/registry.yo4.ocp2.fu.igotit.co.kr.crt \
   -e REGISTRY_HTTP_TLS_KEY=/opt/registry/certs/registry.yo4.ocp2.fu.igotit.co.kr.key \
   -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true \
   -d docker.io/library/registry:2
```

### podman error log output
```
### resorce out put ####
To use the new mirrored repository for upgrades, use the following to create an ImageContentSourcePolicy:

apiVersion: operator.openshift.io/v1alpha1
kind: ImageContentSourcePolicy
metadata:
  name: example
spec:
  repositoryDigestMirrors:
  - mirrors:
    - registry.yo4.ocp2.fu.igotit.co.kr:5000/ocp4/openshift4
    source: quay.io/openshift-release-dev/ocp-release
  - mirrors:
    - registry.yo4.ocp2.fu.igotit.co.kr:5000/ocp4/openshift4
mount 실수로 인한 node 재구성mirror registry error logspodman errormirror.sh[root@bastion ~]# vi /opt/registry/data/mirror.sh
```
```
#!/bin/bash

podman run -d --name mirror-registry -p 5000:5000 --restart=always \
   -v /opt/registry/data:/var/lib/registry:z \
   -v /opt/registry/auth:/auth:z \
   -e "REGISTRY_AUTH=htpasswd" \
   -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
   -e REGISTRY_AUTH_HTPASSWD_PATH=/opt/registry/auth/htpasswd \
   -v /opt/registry/certs:/certs:z \
   -e REGISTRY_HTTP_TLS_CERTIFICATE=/opt/registry/certs/registry.yo4.ocp2.fu.igotit.co.kr.crt \
   -e REGISTRY_HTTP_TLS_KEY=/opt/registry/certs/registry.yo4.ocp2.fu.igotit.co.kr.key \
   -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true \
   -d docker.io/library/registry:2
podman error log output### resorce out put ####
To use the new mirrored repository for upgrades, use the following to create an ImageContentSourcePolicy:

apiVersion: operator.openshift.io/v1alpha1
kind: ImageContentSourcePolicy
metadata:
  name: example
spec:
  repositoryDigestMirrors:
  - mirrors:
    - registry.yo4.ocp2.fu.igotit.co.kr:5000/ocp4/openshift4
    source: quay.io/openshift-release-dev/ocp-release
  - mirrors:
    - registry.yo4.ocp2.fu.igotit.co.kr:5000/ocp4/openshift4
    source: quay.io/openshift-release-dev/ocp-v4.0-art-dev
    
[root@bastion data]# podman login --authfile /opt/registry/data/pull-secret.json registry.yo4.ocp2.fu.igotit.co.kr:5000
Authenticating with existing credentials...
Existing credentials are invalid, please enter valid username and password
Username (admin): admin
Password:
Error: error authenticating creds for "registry.yo4.ocp2.fu.igotit.co.kr:5000": error pinging docker registry registry.yo4.ocp2.fu.igotit.co.kr:5000: Get https://registry.yo4.ocp2.fu.igotit.co.kr:5000/v2/: dial tcp 192.168.2.18:5000: connect: connection refused

[root@bastion data]# podman login --authfile /opt/registry/data/pull-secret.json registry.yo4.ocp2.fu.igotit.co.kr:5000
Authenticating with existing credentials...
Existing credentials are invalid, please enter valid username and password
Username (admin):
Password:
Error: error authenticating creds for "registry.yo4.ocp2.fu.igotit.co.kr:5000": error pinging docker registry registry.yo4.ocp2.fu.igotit.co.kr:5000: Get https://registry.yo4.ocp2.fu.igotit.co.kr:5000/v2/: dial tcp 192.168.2.18:5000: connect: connection refused


[root@bastion ~]# podman ps -a
CONTAINER ID  IMAGE                         COMMAND               CREATED            STATUS                     PORTS                   NAMES
b91e00beb503  docker.io/library/registry:2  /etc/docker/regis...  About an hour ago  Up Less than a second ago  0.0.0.0:5000->5000/tcp  mirror-registry

[root@bastion ~]# podman ps -a
CONTAINER ID  IMAGE                         COMMAND               CREATED            STATUS                             PORTS                   NAMES
b91e00beb503  docker.io/library/registry:2  /etc/docker/regis...  About an hour ago  Exited (1) Less than a second ago  0.0.0.0:5000->5000/tcp  mirror-registry
    source: quay.io/openshift-release-dev/ocp-v4.0-art-dev

[root@bastion data]# podman login --authfile /opt/registry/data/pull-secret.json registry.yo4.ocp2.fu.igotit.co.kr:5000
Authenticating with existing credentials...
Existing credentials are invalid, please enter valid username and password
Username (admin): admin
Password:
Error: error authenticating creds for "registry.yo4.ocp2.fu.igotit.co.kr:5000": error pinging docker registry registry.yo4.ocp2.fu.igotit.co.kr:5000: Get https://registry.yo4.ocp2.fu.igotit.co.kr:5000/v2/: dial tcp 192.168.2.18:5000: connect: connection refused

[root@bastion data]# podman login --authfile /opt/registry/data/pull-secret.json registry.yo4.ocp2.fu.igotit.co.kr:5000
Authenticating with existing credentials...
Existing credentials are invalid, please enter valid username and password
Username (admin):
Password:
Error: error authenticating creds for "registry.yo4.ocp2.fu.igotit.co.kr:5000": error pinging docker registry registry.yo4.ocp2.fu.igotit.co.kr:5000: Get https://registry.yo4.ocp2.fu.igotit.co.kr:5000/v2/: dial tcp 192.168.2.18:5000: connect: connection refused


[root@bastion ~]# podman ps -a
CONTAINER ID  IMAGE                         COMMAND               CREATED            STATUS                     PORTS                   NAMES
b91e00beb503  docker.io/library/registry:2  /etc/docker/regis...  About an hour ago  Up Less than a second ago  0.0.0.0:5000->5000/tcp  mirror-registry
[root@bastion ~]#


[root@bastion ~]# podman ps -a
CONTAINER ID  IMAGE                         COMMAND               CREATED            STATUS                             PORTS                   NAMES
b91e00beb503  docker.io/library/registry:2  /etc/docker/regis...  About an hour ago  Exited (1) Less than a second ago  0.0.0.0:5000->5000/tcp  mirror-registry
```


```
#### mirror registry curl ### 


[root@bastion anchors]# curl -u admin:admin -k https://registry.yo4.ocp2.fu.igotit.co.kr:5000/v2/_catalog -vv
* About to connect() to registry.yo4.ocp2.fu.igotit.co.kr port 5000 (#0)
*   Trying 192.168.2.18...
* 연결이 거부됨
* Failed connect to registry.yo4.ocp2.fu.igotit.co.kr:5000; 연결이 거부됨
* Closing connection 0
curl: (7) Failed connect to registry.yo4.ocp2.fu.igotit.co.kr:5000; 연결이 거부됨

[root@bastion ~]# podman login --authfile /opt/registry/data/pull-secret.json registry.yo4.ocp2.fu.igotit.co.kr:5000
Authenticating with existing credentials...
Existing credentials are invalid, please enter valid username and password
Username (admin): admin
Password:
Error: error authenticating creds for "registry.yo4.ocp2.fu.igotit.co.kr:5000": error pinging docker registry regist  ry.yo4.ocp2.fu.igotit.co.kr:5000: Get https://registry.yo4.ocp2.fu.igotit.co.kr:5000/v2/: dial tcp 192.168.2.18:5000  : connect: connection refused

[root@bastion ~]# podman login --tls-verify registry.yo4.ocp2.fu.igotit.co.kr:5000                                    Username: admin
Password:
Error: error authenticating creds for "registry.yo4.ocp2.fu.igotit.co.kr:5000": error pinging docker registry regist  ry.yo4.ocp2.fu.igotit.co.kr:5000: Get https://registry.yo4.ocp2.fu.igotit.co.kr:5000/v2/: dial tcp 192.168.2.18:5000  : connect: connection refused

[root@bastion ~]# podman login --tls-verify=false registry.yo4.ocp2.fu.igotit.co.kr:5000
Username: admin
Password:
Error: error authenticating creds for "registry.yo4.ocp2.fu.igotit.co.kr:5000": error pinging docker registry regist  ry.yo4.ocp2.fu.igotit.co.kr:5000: Get http://registry.yo4.ocp2.fu.igotit.co.kr:5000/v2/: dial tcp 192.168.2.18:5000:   connect: connection refused

[root@bastion ~]# podman login --authfile /opt/registry/data/pull-secret.json registry.yo4.ocp2.fu.igotit.co.kr:5000
Authenticating with existing credentials...
Existing credentials are invalid, please enter valid username and password
Username (admin): admin
Password:
Error: error authenticating creds for "registry.yo4.ocp2.fu.igotit.co.kr:5000": error pinging docker registry regist  ry.yo4.ocp2.fu.igotit.co.kr:5000: Get https://registry.yo4.ocp2.fu.igotit.co.kr:5000/v2/: dial tcp 192.168.2.18:5000  : connect: connection refused
```
