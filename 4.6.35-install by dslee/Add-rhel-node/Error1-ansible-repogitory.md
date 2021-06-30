## Issue 1 - repo sync from http protocol
```
[root@l2-51-router1 yum.repos.d]# yum update
Loaded plugins: product-id, search-disabled-repos, subscription-manager
Repository rhel-7-server-extras-rpms is listed more than once in the configuration
Repository rhel-7-server-ansible-2.9-rpms is listed more than once in the configuration
Repository rhel-7-server-ose-4.6-rpms is listed more than once in the configuration
Repository rhel-7-server-rpms is listed more than once in the configuration
ftp://172.10.20.11/pub/repos/rhel-7-server-ansible-2.9-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory
Trying other mirror.
ftp://172.10.20.11/pub/repos/rhel-7-server-extras-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory
Trying other mirror.
ftp://172.10.20.11/pub/repos/rhel-7-server-ose-4.6-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory
Trying other mirror.
ftp://172.10.20.11/pub/repos/rhel-7-server-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory
Trying other mirror.
No packages marked for update
```

## Validation check
```
- http repogitory 구성

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


- repo

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
```

## Issue 2 - enable repos as 'rhel-7-fast-datapath-rpms' and 'rhel-7-server-optional-rpms'
```
TASK [openshift_node : Install openshift support packages] *******************************************************************************************
FAILED - RETRYING: Install openshift support packages (3 retries left).
FAILED - RETRYING: Install openshift support packages (3 retries left).
FAILED - RETRYING: Install openshift support packages (2 retries left).
FAILED - RETRYING: Install openshift support packages (2 retries left).
FAILED - RETRYING: Install openshift support packages (1 retries left).
FAILED - RETRYING: Install openshift support packages (1 retries left).
fatal: [l2-52-router2.lab2.dslee.lab]: FAILED! => {"ansible_job_id": "158393074102.9833", "attempts": 3, "changed": false, "finished": 1, "msg": "No package matching 'NetworkManager-ovs' found available, installed or updated", "rc": 126, "results": ["All packages providing kernel are up to date", "All packages providing systemd are up to date", "All packages providing selinux-policy-targeted are up to date", "All packages providing dracut-network are up to date", "All packages providing passwd are up to date", "All packages providing openssh-server are up to date", "All packages providing openssh-clients are up to date", "All packages providing NetworkManager are up to date", "No package matching 'NetworkManager-ovs' found available, installed or updated"]}
fatal: [l2-51-router1.lab2.dslee.lab]: FAILED! => {"ansible_job_id": "327657261158.9737", "attempts": 3, "changed": false, "finished": 1, "msg": "No package matching 'NetworkManager-ovs' found available, installed or updated", "rc": 126, "results": ["All packages providing kernel are up to date", "All packages providing systemd are up to date", "All packages providing selinux-policy-targeted are up to date", "All packages providing dracut-network are up to date", "All packages providing passwd are up to date", "All packages providing openssh-server are up to date", "All packages providing openssh-clients are up to date", "All packages providing NetworkManager are up to date", "No package matching 'NetworkManager-ovs' found available, installed or updated"]}

PLAY RECAP *******************************************************************************************************************************************
l2-51-router1.lab2.dslee.lab : ok=2    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
l2-52-router2.lab2.dslee.lab : ok=2    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
localhost                  : ok=1    changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
```

## Resolution
```
 subscription-manager repos \
    --enable="rhel-7-server-rpms" \
    --enable="rhel-7-fast-datapath-rpms" \
    --enable="rhel-7-server-extras-rpms" \
    --enable="rhel-7-server-optional-rpms" \
    --enable="rhel-7-server-ose-4.6-rpms" \
    --enable="rhel-7-server-ansible-2.9-rpms"

reposync --gpgcheck -lmn --repoid=rhel-7-server-rpms --download_path=var/www/html/repos
createrepo -v /var/www/html/repos/rhel-7-server-rpms -o /var/www/html/repos/rhel-7-server-rpms
reposync --gpgcheck -lmn --repoid=rhel-7-fast-datapath-rpms --download_path=var/www/html/repos
createrepo -v /var/www/html/repos/rhel-7-fast-datapath-rpms -o /var/www/html/repos/rhel-7-fast-datapath-rpms
reposync --gpgcheck -lmn --repoid=rhel-7-server-extras-rpms --download_path=var/www/html/repos
createrepo -v /var/www/html/repos/rhel-7-server-extras-rpms -o /var/www/html/repos/rhel-7-server-extras-rpm
reposync --gpgcheck -lmn --repoid=rhel-7-server-optional-rpms --download_path=var/www/html/repos
createrepo -v /var/www/html/repos/rhel-7-server-optional-rpms -o /var/www/html/repos/rhel-7-server-optional-rpms
```