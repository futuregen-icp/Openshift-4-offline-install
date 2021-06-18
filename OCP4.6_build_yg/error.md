
### yum install error
```
# yum install gcc

failure: repodata/repomd.xml from rhel-7-server-ansible-2.6-rpms: [Errno 256] No more mirrors to try.
ftp://192.168.3.20/repos/rhel-7-server-ansible-2.6-rpms/repodata/repomd.xml: [Errno 14] curl#7 - "Failed connect to 192.168.3.20:21; No route to host"
ftp://192.168.3.20/repos/rhel-7-server-ansible-2.6-rpms/repodata/repomd.xml: [Errno 14] curl#7 - "Failed connect to 192.168.3.20:21; No route to host"
ftp://192.168.3.20/repos/rhel-7-server-ansible-2.6-rpms/repodata/repomd.xml: [Errno 14] curl#7 - "Failed connect to 192.168.3.20:21; No route to host"
ftp://192.168.3.20/repos/rhel-7-server-ansible-2.6-rpms/repodata/repomd.xml: [Errno 14] curl#7 - "Failed connect to 192.168.3.20:21; No route to host"
ftp://192.168.3.20/repos/rhel-7-server-ansible-2.6-rpms/repodata/repomd.xml: [Errno 14] curl#7 - "Failed connect to 192.168.3.20:21; No route to host"
ftp://192.168.3.20/repos/rhel-7-server-ansible-2.6-rpms/repodata/repomd.xml: [Errno 14] curl#7 - "Failed connect to 192.168.3.20:21; No route to host"
ftp://192.168.3.20/repos/rhel-7-server-ansible-2.6-rpms/repodata/repomd.xml: [Errno 14] curl#7 - "Failed connect to 192.168.3.20:21; No route to host"                                              ftp://192.168.3.20/repos/rhel-7-server-ansible-2.6-rpms/repodata/repomd.xml: [Errno 14] curl#7 - "Failed connect to 192.168.3.20:21; No route to host"
ftp://192.168.3.20/repos/rhel-7-server-ansible-2.6-rpms/repodata/repomd.xml: [Errno 14] curl#7 - "Failed connect to 192.168.3.20:21; No route to host"
ftp://192.168.3.20/repos/rhel-7-server-ansible-2.6-rpms/repodata/repomd.xml: [Errno 14] curl#7 - "Failed connect to 192.168.3.20:21; No route to host"


```
### vsftpd status error
```
● vsftpd.service - Vsftpd ftp daemon
   Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Thu 2021-06-17 17:13:29 KST; 5s ago
  Process: 8463 ExecStart=/usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf (code=exited, status=2)

Jun 17 17:13:29 repository.yo4.ocp3.fu.igotit.co.kr systemd[1]: Starting Vsftpd ftp daemon...
Jun 17 17:13:29 repository.yo4.ocp3.fu.igotit.co.kr vsftpd[8463]: 500 OOPS: unrecognised variable in config file: connect_from_port_21
Jun 17 17:13:29 repository.yo4.ocp3.fu.igotit.co.kr systemd[1]: vsftpd.service: control process exited, code=exited status=2
Jun 17 17:13:29 repository.yo4.ocp3.fu.igotit.co.kr systemd[1]: Failed to start Vsftpd ftp daemon.
Jun 17 17:13:29 repository.yo4.ocp3.fu.igotit.co.kr systemd[1]: Unit vsftpd.service entered failed state.
Jun 17 17:13:29 repository.yo4.ocp3.fu.igotit.co.kr systemd[1]: vsftpd.service failed.
```

### ssh error (w.bastion)
```
#ssh root@192.168.3.19

Permissions 0644 for '/home/yghong/.ssh/id_rsa' are too open.
It is required that your private key fxxiles are NOT accessible by others.
This private key will be ignored.
Load key "/home/yghong/.ssh/id_rsa": bad permissions
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).

soution
check IP number
bastion IP : 192.168.3.18

```


