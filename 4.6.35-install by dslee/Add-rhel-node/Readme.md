## http repogitory 구성

```
[root@l2-11-nfs1 ~]# cat repos.sh 
#!/bin/sh
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

for repo in \
   rhel-7-server-rpms \
   rhel-7-server-extras-rpms \
   rhel-7-server-ansible-2.9-rpms \
   rhel-7-server-ose-4.6-rpms \
   rhel-7-fast-datapath-rpms" \
   ehrl-7-server-optional-rmps"
do
   reposync --gpgcheck -lm --repoid=${repo} --download_path=/var/www/html/repos
   createrepo -v /var/www/html/repos/${repo} -o /var/www/html/repos/${repo}
done

-----

reposync --gpgcheck -lmn --repoid=rhel-7-server-rpms --download_path=var/www/html/repos
createrepo -v /var/www/html/repos/rhel-7-server-rpms -o /var/www/html/repos/rhel-7-server-rpms
reposync --gpgcheck -lmn --repoid=rhel-7-fast-datapath-rpms --download_path=var/www/html/repos
createrepo -v /var/www/html/repos/rhel-7-fast-datapath-rpms -o /var/www/html/repos/rhel-7-fast-datapath-rpms
reposync --gpgcheck -lmn --repoid=rhel-7-server-extras-rpms --download_path=var/www/html/repos
createrepo -v /var/www/html/repos/rhel-7-server-extras-rpms -o /var/www/html/repos/rhel-7-server-extras-rpm
reposync --gpgcheck -lmn --repoid=rhel-7-server-optional-rpms --download_path=var/www/html/repos
createrepo -v /var/www/html/repos/rhel-7-server-optional-rpms -o /var/www/html/repos/rhel-7-server-optional-rpms
```

## Add node
```
- Disk configured
Source: D:\Virtual Machines\t-redhat7\Virtual Hard Disks\t-redhat7-1.vhdx

Destination: D:\Virtual Machines\l2-51-router1\Virtual Hard Disks￦l2-51-router1-0.vhdx
Destination: E:\Virtual Machines\l2-52-router2\Virtual Hard Disks￦l2-52-router2-0.vhdx

- Network configered
/etc/sysconfig/network-script/ifcfg-eth0
hostnamectl set-hostname l2-51-router1.lab2.dslee.lab
hostnamectl set-hostname l2-52-router2.lab2.dslee.lab

- Congigure repos
subscription-manager repos
subscription-manager repos --disable="*"

cat /etc/yum.repos.d/ocp4-6.repo
[rhel-7-server-rpms]
name=rhel-7-server-rpms
baseurl=http://172.10.20.11/repos/rhel-7-server-rpms
enabled=1
gpgcheck=0
[rhel-7-server-extras-rpms]
name=rhel-7-server-extras-rpms
baseurl=http://172.10.20.11/repos/rhel-7-server-extras-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.9-rpms]
name=rhel-7-server-ansible-2.9-rpms
baseurl=http://172.10.20.11/repos/rhel-7-server-ansible-2.9-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ose-4.6-rpms]
name=rhel-7-server-ose-4.6-rpms
baseurl=http://172.10.20.11/repos/rhel-7-server-ose-4.6-rpms
enabled=1
gpgcheck=0
[rhel-7-fast-datapath-rpms]
name=rhel-7-fast-datapath-rpms
baseurl=http://172.10.20.11/repos/rhel-7-fast-datapath-rpms
enabled=1
gpgcheck=0
[rhel-7-server-optional-rpms]
name=rhel-7-server-optional-rpms
baseurl=http://172.10.20.11/repos/rhel-7-server-optional-rpms
ebabled=1
gpgcheck=0

## Check selinux
sestatus

## Disable selinux
setenforce 0
vi /etc/selinux/config # change enforce to permissive

## Disalbe firewalld
[root@l2-51-router1 yum.repos.d]# systemctl stop firewalld
[root@l2-51-router1 yum.repos.d]# systemctl disable firewalld

```

## Edit inventory file
```
[all:vars]
ansible_user=root
#ansible_become=Tru
openshift_kubeconfig_path="~/.kube/config"

[new_workers]
l2-51-router1.lab2.dslee.lab
l2-52-router2.lab2.dslee.lab
l2-61-log1.lab2.dslee.lab
```

## Run ansible playbook
ansible-playbook -i /root/inventory.yaml /usr/share/ansible/openshift-ansible/playbooks/scaleup.yml

- Refer link
https://github.com/futuregen-icp/Openshift-4-online-install/blob/master/openshift%204.4%20-%20adding%20RHEL%20node%20to%20openshift%20cluster.md
