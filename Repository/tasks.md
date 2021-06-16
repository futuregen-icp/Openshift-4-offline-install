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
