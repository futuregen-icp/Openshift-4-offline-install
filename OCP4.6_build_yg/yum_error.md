### yum install pip (_TEST_)
```
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager

This system is not registered with an entitlement server. You can use subscription-manager to register.

ftp://192.168.3.20/pub/repos/rhel-7-server-ansible-2.9-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory                                                                                                        Trying other mirror.
ftp://192.168.3.20/pub/repos/rhel-7-server-extras-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory
Trying other mirror.
ftp://192.168.3.20/pub/repos/rhel-7-server-ose-4.6-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory
Trying other mirror.
ftp://192.168.3.20/pub/repos/rhel-7-server-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory                                                                                                                    Trying other mirror.
No package pip available.
Error: Nothing to do
[root@bastion ~]# yum install ansible
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager

This system is not registered with an entitlement server. You can use subscription-manager to register.

ftp://192.168.3.20/pub/repos/rhel-7-server-ansible-2.9-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory
Trying other mirror.
ftp://192.168.3.20/pub/repos/rhel-7-server-extras-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory
Trying other mirror.
ftp://192.168.3.20/pub/repos/rhel-7-server-ose-4.6-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory
Trying other mirror.
ftp://192.168.3.20/pub/repos/rhel-7-server-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory
Trying other mirror.
Resolving Dependencies
--> Running transaction check
---> Package ansible.noarch 0:2.9.22-1.el7ae will be installed
--> Processing Dependency: python-jinja2 for package: ansible-2.9.22-1.el7ae.noarch
--> Processing Dependency: python-paramiko for package: ansible-2.9.22-1.el7ae.noarch
--> Processing Dependency: python2-cryptography for package: ansible-2.9.22-1.el7ae.noarch
--> Processing Dependency: sshpass for package: ansible-2.9.22-1.el7ae.noarch
--> Running transaction check
---> Package python-jinja2.noarch 0:2.7.2-4.el7 will be installed
--> Processing Dependency: python-babel >= 0.8 for package: python-jinja2-2.7.2-4.el7.noarch                                      
--> Processing Dependency: python-markupsafe for package: python-jinja2-2.7.2-4.el7.noarch
---> Package python-paramiko.noarch 0:2.1.1-9.el7 will be installed
--> Processing Dependency: python2-pyasn1 for package: python-paramiko-2.1.1-9.el7.noarch
---> Package python2-cryptography.x86_64 0:1.7.2-2.el7 will be installed
--> Processing Dependency: python-cffi >= 1.4.1 for package: python2-cryptography-1.7.2-2.el7.x86_64
--> Processing Dependency: python-idna >= 2.0 for package: python2-cryptography-1.7.2-2.el7.x86_64
--> Processing Dependency: python-enum34 for package: python2-cryptography-1.7.2-2.el7.x86_64
---> Package sshpass.x86_64 0:1.06-2.el7 will be installed
--> Running transaction check
---> Package python-babel.noarch 0:0.9.6-8.el7 will be installed
---> Package python-cffi.x86_64 0:1.6.0-5.el7 will be installed
--> Processing Dependency: python-pycparser for package: python-cffi-1.6.0-5.el7.x86_64
---> Package python-enum34.noarch 0:1.0.4-1.el7 will be installed
---> Package python-idna.noarch 0:2.4-1.el7 will be installed
---> Package python-markupsafe.x86_64 0:0.11-10.el7 will be installed
---> Package python2-pyasn1.noarch 0:0.1.9-7.el7 will be installed
--> Running transaction check
---> Package python-pycparser.noarch 0:2.14-1.el7 will be installed                                                               
--> Processing Dependency: python-ply for package: python-pycparser-2.14-1.el7.noarch
--> Running transaction check
---> Package python-ply.noarch 0:3.4-11.el7 will be installed                                                                      
--> Finished Dependency Resolution

Dependencies Resolved                                                                                                              
===================================================================================================================================
 Package                          Arch               Version                      Repository                                  Size 
 ===================================================================================================================================
Installing:
 ansible                          noarch             2.9.22-1.el7ae               rhel-7-server-ansible-2.9-rpms              17 M
Installing for dependencies:
 python-babel                     noarch             0.9.6-8.el7                  rhel-7-server-rpms                         1.4 M
 python-cffi                      x86_64             1.6.0-5.el7                  rhel-7-server-rpms                         218 k
 python-enum34                    noarch             1.0.4-1.el7                  rhel-7-server-rpms                          52 k
 python-idna                      noarch             2.4-1.el7                    rhel-7-server-rpms                          94 k
 python-jinja2                    noarch             2.7.2-4.el7                  rhel-7-server-rpms                         519 k
 python-markupsafe                x86_64             0.11-10.el7                  rhel-7-server-rpms                          25 k
 python-paramiko                  noarch             2.1.1-9.el7                  rhel-7-server-rpms                         269 k
 python-ply                       noarch             3.4-11.el7                   rhel-7-server-rpms                         123 k
 python-pycparser                 noarch             2.14-1.el7                   rhel-7-server-rpms                         105 k
 python2-cryptography             x86_64             1.7.2-2.el7                  rhel-7-server-rpms                         503 k  
 python2-pyasn1                   noarch             0.1.9-7.el7                  rhel-7-server-rpms                         100 k
 sshpass                          x86_64             1.06-2.el7                   rhel-7-server-ansible-2.9-rpms              21 k
 Transaction Summary
===================================================================================================================================
Install  1 Package (+12 Dependent packages)                                                                                        
Total download size: 20 M
Installed size: 119 M                                                                                                              
Is this ok [y/d/N]: ^[^[^Cn
Exiting on user command
Your transaction was saved, rerun it with:
 yum load-transaction /tmp/yum_save_tx.2021-06-18.16-23.YFNhfN.yumtx
```

### error code
```
This system is not registered with an entitlement server. You can use subscription-manager to register.

ftp://192.168.3.20/pub/repos/rhel-7-server-ansible-2.9-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory                                                                                                        Trying other mirror.
ftp://192.168.3.20/pub/repos/rhel-7-server-extras-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory
Trying other mirror.
ftp://192.168.3.20/pub/repos/rhel-7-server-ose-4.6-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory
Trying other mirror.
ftp://192.168.3.20/pub/repos/rhel-7-server-rpms/repodata/repomd.xml: [Errno 14] FTP Error 550 - Server denied you to change to the given directory 
 ```
 
 ### check 
 ```
 $ ssh root@192.0.0.20
 
 [root@repository rhel-7-server-extras-rpms]# pwd
/var/ftp/pub/repos/rhel-7-server-extras-rpms
[root@repository rhel-7-server-extras-rpms]# ls -al
total 16
drwxr-xr-x.  3 root root 4096 Jun 16 18:19 .
drwxr-xr-x.  6 root root 4096 Jun 16 18:31 ..
-rw-r--r--.  1 root root  124 Jun 16 18:19 comps.xml
drwxr-xr-x. 17 root root 4096 Jun 16 18:19 Packages
[root@repository rhel-7-server-extras-rpms]#

-> it not created, so check up the created repo


```

