

+ Add dns record
```
infra1		IN	A	192.168.5.121
infra2		IN	A	192.168.5.122
infra3		IN	A	192.168.5.123
```

+ Configure for the bastion subscription
```
subscription-manager repos\
    --enable="rhel-7-server-rpms"\
    --enable="rhel-7-server-extras-rpms"\
    --enable="rhel-7-server-ansible-2.9-rpms"\
    --enable="rhel-7-server-ose-4.6-rpms"
```

+ Configure for the node subscription
```
subscription-manager repos
subscription-manager register
subscription-manager refresh

subscription-manager list --available --matches="*Openshift*"
subscription-manager attach --pool=8a85f9997385090b0173b342862a50b8

subscription-manager repos --disable="*"
subscription-manager repos \
    --enable="rhel-7-server-rpms" \
    --enable="rhel-7-fast-datapath-rpms" \
    --enable="rhel-7-server-extras-rpms" \
    --enable="rhel-7-server-optional-rpms" \
    --enable="rhel-7-server-ose-4.6-rpms"
```

+ install yum package on bastion for the run ansible 
```
yum install -y openshift-ansible openshift-clients jq
```
+ setting hostname for the all node as infra
+ copy ssh key
+ edit /etc/ansible/hosts
```
[all:vars] 
ansible_user=root 
#ansible_become=True 
openshift_kubeconfig_path="/root/.kube/kubeconfig"

[new_workers] 
infra1.oss2.fu.igotit.co.kr
infra2.oss2.fu.igotit.co.kr
infra3.oss2.fu.igotit.co.kr
```

+ infra firewall for the all infra node as infra
```
firewall-cmd --permanent --add-port=9000-9999/tcp --add-port=10250-10259/tcp --add-port=10256/tcp --add-port=4789/udp --add-port=6081/udp --add-port=9000-9999/udp --add-port=30000-32767/tcp --add-port=30000-32767/udp  
firewall-cmd --reload
```

+ run ansible playbook
```
ansible-playbook -i /etc/ansible/hosts /usr/share/ansible/openshift-ansible/playbooks/scaleup.yml -k -K

```

+ verify check
+ remove node
  - https://access.redhat.com/solutions/5152031

+ configure machine config pool as infra node

+ vi infra-mcp.yaml 
```
apiVersion: machineconfiguration.openshift.io/v1 
kind: MachineConfigPool 
metadata: 
  name: infra 
spec: 
  machineConfigSelector: 
    matchExpressions: 
    - {key: machineconfiguration.openshift.io/role, operator: In, values: [worker,infra]} 
  nodeSelector: 
    matchLabels: 
      node-role.kubernetes.io/infra: ""
```


+ Configure taint for the infra nodes.
```
oc adm taint nodes -l node-role.kubernetes.io/infra infra=reserved:NoSchedule infra=reserved:NoExecute
```
+ File ad cm-monitor.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |+
    alertmanagerMain:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: infra
        value: reserved
        effect: NoSchedule
      - key: infra
        value: reserved
        effect: NoExecute
    prometheusK8s:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: infra
        value: reserved
        effect: NoSchedule
      - key: infra
        value: reserved
        effect: NoExecute
    prometheusOperator:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: infra
        value: reserved
        effect: NoSchedule
      - key: infra
        value: reserved
        effect: NoExecute
    grafana:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: infra
        value: reserved
        effect: NoSchedule
      - key: infra
        value: reserved
        effect: NoExecute
    k8sPrometheusAdapter:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: infra
        value: reserved
        effect: NoSchedule
      - key: infra
        value: reserved
        effect: NoExecute
    kubeStateMetrics:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: infra
        value: reserved
        effect: NoSchedule
      - key: infra
        value: reserved
        effect: NoExecute
    telemeterClient:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: infra
        value: reserved
        effect: NoSchedule
      - key: infra
        value: reserved
        effect: NoExecute
    openshiftStateMetrics:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: infra
        value: reserved
        effect: NoSchedule
      - key: infra
        value: reserved
        effect: NoExecute
  ```
