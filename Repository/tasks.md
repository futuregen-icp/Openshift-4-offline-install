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

- Definitions
```
download_path=/var/ftp/pub/repos
server: 172.10.20.11 (l2-11-nfs1.lab2.dslee.lab)
```
