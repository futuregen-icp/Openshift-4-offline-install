## Issue
```
TASK [openshift_node : Pull release image] **********************************************************************************************************
FAILED - RETRYING: Pull release image (12 retries left).
FAILED - RETRYING: Pull release image (12 retries left).
FAILED - RETRYING: Pull release image (11 retries left).
FAILED - RETRYING: Pull release image (11 retries left).
FAILED - RETRYING: Pull release image (10 retries left).
FAILED - RETRYING: Pull release image (10 retries left).
FAILED - RETRYING: Pull release image (9 retries left).
FAILED - RETRYING: Pull release image (9 retries left).
FAILED - RETRYING: Pull release image (8 retries left).
FAILED - RETRYING: Pull release image (8 retries left).
FAILED - RETRYING: Pull release image (7 retries left).
FAILED - RETRYING: Pull release image (7 retries left).
FAILED - RETRYING: Pull release image (6 retries left).
FAILED - RETRYING: Pull release image (6 retries left).
FAILED - RETRYING: Pull release image (5 retries left).
FAILED - RETRYING: Pull release image (5 retries left).
FAILED - RETRYING: Pull release image (4 retries left).
FAILED - RETRYING: Pull release image (4 retries left).
FAILED - RETRYING: Pull release image (3 retries left).
FAILED - RETRYING: Pull release image (3 retries left).
FAILED - RETRYING: Pull release image (2 retries left).
FAILED - RETRYING: Pull release image (2 retries left).
FAILED - RETRYING: Pull release image (1 retries left).
FAILED - RETRYING: Pull release image (1 retries left).
fatal: [l2-51-router1.lab2.dslee.lab]: FAILED! => {"attempts": 12, "changed": true, "cmd": ["podman", "pull", "--tls-verify=False", "--authfile", "/tmp/ansible.ApzQ9v/pull-secret.json", "mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d"], "delta": "0:00:00.144348", "end": "2021-06-29 00:38:41.667461", "msg": "non-zero return code", "rc": 125, "start": "2021-06-29 00:38:41.523113", "stderr": "Trying to pull mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d...\n  Get http://mirrorregistry.lab2.dslee.lab:5000/v2/: dial tcp 172.10.20.10:5000: connect: no route to host\nError: error pulling image \"mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d\": unable to pull mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d: unable to pull image: Error initializing source docker://mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d: error pinging docker registry mirrorregistry.lab2.dslee.lab:5000: Get http://mirrorregistry.lab2.dslee.lab:5000/v2/: dial tcp 172.10.20.10:5000: connect: no route to host", "stderr_lines": ["Trying to pull mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d...", "  Get http://mirrorregistry.lab2.dslee.lab:5000/v2/: dial tcp 172.10.20.10:5000: connect: no route to host", "Error: error pulling image \"mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d\": unable to pull mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d: unable to pull image: Error initializing source docker://mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d: error pinging docker registry mirrorregistry.lab2.dslee.lab:5000: Get http://mirrorregistry.lab2.dslee.lab:5000/v2/: dial tcp 172.10.20.10:5000: connect: no route to host"], "stdout": "", "stdout_lines": []}
fatal: [l2-52-router2.lab2.dslee.lab]: FAILED! => {"attempts": 12, "changed": true, "cmd": ["podman", "pull", "--tls-verify=False", "--authfile", "/tmp/ansible.TkhVmV/pull-secret.json", "mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d"], "delta": "0:00:00.152929", "end": "2021-06-29 00:38:41.770858", "msg": "non-zero return code", "rc": 125, "start": "2021-06-29 00:38:41.617929", "stderr": "Trying to pull mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d...\n  Get http://mirrorregistry.lab2.dslee.lab:5000/v2/: dial tcp 172.10.20.10:5000: connect: no route to host\nError: error pulling image \"mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d\": unable to pull mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d: unable to pull image: Error initializing source docker://mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d: error pinging docker registry mirrorregistry.lab2.dslee.lab:5000: Get http://mirrorregistry.lab2.dslee.lab:5000/v2/: dial tcp 172.10.20.10:5000: connect: no route to host", "stderr_lines": ["Trying to pull mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d...", "  Get http://mirrorregistry.lab2.dslee.lab:5000/v2/: dial tcp 172.10.20.10:5000: connect: no route to host", "Error: error pulling image \"mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d\": unable to pull mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d: unable to pull image: Error initializing source docker://mirrorregistry.lab2.dslee.lab:5000/ocp4/openshift4@sha256:8a57df33243359a71e6a4aa835b0423ff280d59ecc332816ba2143842adcb28d: error pinging docker registry mirrorregistry.lab2.dslee.lab:5000: Get http://mirrorregistry.lab2.dslee.lab:5000/v2/: dial tcp 172.10.20.10:5000: connect: no route to host"], "stdout": "", "stdout_lines": []}

PLAY RECAP ******************************************************************************************************************************************
l2-51-router1.lab2.dslee.lab : ok=34   changed=24   unreachable=0    failed=1    skipped=5    rescued=0    ignored=0   
l2-52-router2.lab2.dslee.lab : ok=34   changed=25   unreachable=0    failed=1    skipped=5    rescued=0    ignored=0   
localhost                  : ok=1    changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0  
```

## Resolutions
[root@l2-10-base1 works]# podman ps
CONTAINER ID  IMAGE   COMMAND  CREATED  STATUS  PORTS   NAMES
[root@l2-10-base1 works]# podman ps -a
CONTAINER ID  IMAGE                         COMMAND               CREATED     STATUS                 PORTS                   NAMES
ceb17c7ccb28  docker.io/library/registry:2  /etc/docker/regis...  7 days ago  Exited (2) 4 days ago  0.0.0.0:5000->5000/tcp  mirror-registry
[root@l2-10-base1 works]# podman start ceb17c7ccb28
