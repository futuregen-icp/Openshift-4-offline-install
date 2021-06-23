## Issue
```
[root@lab1-bastion ~]# subscription-manager repos \
>     --enable="rhel-7-server-rpms" \
>     --enable="rhel-7-fast-datapath-rpms" \
>     --enable="rhel-7-server-extras-rpms" \
>     --enable="rhel-7-server-optional-rpms" \
>     --enable="rhel-7-server-ose-4.6-rpms"
리포지터리 'rhel-7-server-rpms'이/가 이 시스템에 대해 활성화되어 있습니다.
리포지터리 'rhel-7-server-optional-rpms'이/가 이 시스템에 대해 활성화되어 있습니다.
리포지터리 'rhel-7-server-extras-rpms'이/가 이 시스템에 대해 활성화되어 있습니다.
리포지터리 'rhel-7-fast-datapath-rpms'이/가 이 시스템에 대해 활성화되어 있습니다.
리포지터리 'rhel-7-server-ose-4.6-rpms'이/가 이 시스템에 대해 활성화되어 있습니다.

[root@lab1-bastion ~]# cat /etc/redhat-release 
Red Hat Enterprise Linux Server release 7.9 (Maipo)


[root@lab1-bastion ~]# yum install -y openshift-ansible 
Loaded plugins: product-id, search-disabled-repos, subscription-manager
Resolving Dependencies
--> Running transaction check
---> Package openshift-ansible.noarch 0:4.6.0-202102031649.p0.git.0.bf90f86.el7 will be installed
--> Processing Dependency: ansible >= 2.9.5 for package: openshift-ansible-4.6.0-202102031649.p0.git.0.bf90f86.el7.noarch
--> Processing Dependency: openshift-clients for package: openshift-ansible-4.6.0-202102031649.p0.git.0.bf90f86.el7.noarch
--> Running transaction check
---> Package openshift-ansible.noarch 0:4.6.0-202102031649.p0.git.0.bf90f86.el7 will be installed
--> Processing Dependency: ansible >= 2.9.5 for package: openshift-ansible-4.6.0-202102031649.p0.git.0.bf90f86.el7.noarch
---> Package openshift-clients.x86_64 0:4.6.0-202102120217.p0.git.3834.1ca3c6e.el7 will be installed
--> Processing Dependency: bash-completion for package: openshift-clients-4.6.0-202102120217.p0.git.3834.1ca3c6e.el7.x86_64
--> Running transaction check
---> Package bash-completion.noarch 1:2.1-8.el7 will be installed
---> Package openshift-ansible.noarch 0:4.6.0-202102031649.p0.git.0.bf90f86.el7 will be installed
--> Processing Dependency: ansible >= 2.9.5 for package: openshift-ansible-4.6.0-202102031649.p0.git.0.bf90f86.el7.noarch
--> Finished Dependency Resolution
Error: Package: openshift-ansible-4.6.0-202102031649.p0.git.0.bf90f86.el7.noarch (rhel-7-server-ose-4.6-rpms)
           Requires: ansible >= 2.9.5
           Available: ansible-2.3.1.0-3.el7.noarch (rhel-7-server-extras-rpms)
               ansible = 2.3.1.0-3.el7
           Available: ansible-2.3.2.0-2.el7.noarch (rhel-7-server-extras-rpms)
               ansible = 2.3.2.0-2.el7
           Available: ansible-2.4.0.0-5.el7.noarch (rhel-7-server-extras-rpms)
               ansible = 2.4.0.0-5.el7
           Available: ansible-2.4.1.0-1.el7.noarch (rhel-7-server-extras-rpms)
               ansible = 2.4.1.0-1.el7
           Available: ansible-2.4.2.0-2.el7.noarch (rhel-7-server-extras-rpms)
               ansible = 2.4.2.0-2.el7
**********************************************************************
yum can be configured to try to resolve such errors by temporarily enabling
disabled repos and searching for missing dependencies.
To enable this functionality please set 'notify_only=0' in /etc/yum/pluginconf.d/search-disabled-repos.conf
**********************************************************************

Error: Package: openshift-ansible-4.6.0-202102031649.p0.git.0.bf90f86.el7.noarch (rhel-7-server-ose-4.6-rpms)
           Requires: ansible >= 2.9.5
           Available: ansible-2.3.1.0-3.el7.noarch (rhel-7-server-extras-rpms)
               ansible = 2.3.1.0-3.el7
           Available: ansible-2.3.2.0-2.el7.noarch (rhel-7-server-extras-rpms)
               ansible = 2.3.2.0-2.el7
           Available: ansible-2.4.0.0-5.el7.noarch (rhel-7-server-extras-rpms)
               ansible = 2.4.0.0-5.el7
           Available: ansible-2.4.1.0-1.el7.noarch (rhel-7-server-extras-rpms)
               ansible = 2.4.1.0-1.el7
           Available: ansible-2.4.2.0-2.el7.noarch (rhel-7-server-extras-rpms)
               ansible = 2.4.2.0-2.el7
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
 ```

## Solution
```
+ Configure for the bastion subscription
subscription-manager repos\
    --enable="rhel-7-server-rpms"\
    --enable="rhel-7-server-extras-rpms"\
    --enable="rhel-7-server-ansible-2.9-rpms"\
    --enable="rhel-7-server-ose-4.6-rpms"
```