
### VM server Config
  
```
domain: repository.yo4.fufufuf.co.kr
IP: 192.0.0.20
partion[ / :400GB /swap: 20GB]
spac[ 2cpu/8GB/500GB]
```

### DNS recode internal.zone
```
$TTL 60
@       IN SOA  dns.ocp3.fu.igotit.co.kr. root.ocp3.fu.igotit.co.kr. (
                                        1       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
                IN      NS      ns.ocp3.fu.igotit.co.kr.;
                IN      A       192.0.0.1
ns              IN      A       192.0.0.1
api             IN      A       192.0.0.1
api-int         IN      A       192.0.0.1
*.apps          IN      A       192.0.0.1
;

api.yo4         IN      A       192.0.0.21
api-int.yo4     IN      A       192.0.0.21
apps.yo4        IN      A       192.0.0.21
bastion.yo4     IN      A       192.0.0.18
bootstrap.yo4   IN      A       192.0.0.19
repository.yo4  IN      A       192.0.0.20
master01.yo4    IN      A       192.0.0.21
master02.yo4    IN      A       192.0.0.22
master03.yo4    IN      A       192.0.0.23
worker01.yo4    IN      A       192.0.0.121
worker02.yo4    IN      A       192.0.0.122
etcd-0          IN      A       192.0.0.21
etcd-1          IN      A       192.0.0.22
etcd-2          IN      A       192.0.0.23
_etcd-server-ssl._tcp IN SRV 0 10 2380 etcd-0
_etcd-server-ssl._tcp IN SRV 0 10 2380 etcd-1
_etcd-server-ssl._tcp IN SRV 0 10 2380 etcd-2
```

###  Reverse domain

```
$TTL 20
@       IN      SOA     ns.igotit.co.kr.   root (
                        2019070700      ; serial
                        3H              ; refresh (3 hours)
                        30M             ; retry (30 minutes)
                        2W              ; expiry (2 weeks)
                        1W )            ; minimum (1 week)
        IN      NS      ns.igotit.co.kr.
;
; syntax is "last octet" and the host must have fqdn with trailing dot

19      IN      PTR     bootstrap.yo4.ocp3.fu.igotit.co.kr.
20      IN      PTR     repository.yo4.ocp3.fu.igotit.co.kr
21      IN      PTR     master01.yo4.ocp3.fu.igotit.co.kr.
22      IN      PTR     master02.yo4.ocp3.fu.igotit.co.kr.
23      IN      PTR     master03.yo4.ocp3.fu.igotit.co.kr.
121     IN      PTR     worker01.yo4.ocp3.fu.igotit.co.kr.
122     IN      PTR     worker02.yo4.ocp3.fu.igotit.co.kr.
;
```

###  Node spac

|node|IP|cpu|ram|hdd|
|:--:|:--:|:--:|:--:|:--:|
|bastion|192.0.0.18|2|8|300GB|
|bootstrap|192.0.0.19|4|16|200GB|
|repository|192.0.0.20|2|8|500GB|
|master01|192.0.0.21|4|16|200GB|
|master02|192.0.0.22|4|16|200GB|
|master03|192.0.0.23|4|16|200GB|
|worker01|192.0.0.121|2|8|160GB|
|worker02|192.0.0.122|2|8|160GB|


###  Subscription registered
```
subscription-manager register --username=**** --password=****
subscription-manager refresh
subscription-manager list --available --matches '*OpenShift*'
subscription-manager attach --pool=8a85f999738............... 
--enable="rhel-7-server-rpms" 
--enable="rhel-7-server-extras-rpms" 
--enable="rhel-7-server-ose-4.6-rpms" 
--enable="rhel-7-server-ansible-2.9-rpms"
```

###  Tools install
```
yum -y install yum-utils createrepo podman git vsftpd
```

###  Repo synchronization 
```
for repo in rhel-7-server-rpms rhel-7-server-extras-rpms rhel-7-server-ansible-2.9-rpms rhel-7-server-ose-4.6-rpms
do
	reposync --gpgcheck -lm --repoid=${repo} --download_path=/var/ftp/pub/repos
	createrepo -v /var/ftp/pub/repos/${repo} -o /var/ftp/pub/repos${repo}
done
```

### remake /etc/yum.repos.d/redhat.repo
```
cat /etc/yum.repos.d/ocp3-11.repo
[rhel-7-server-rpms]
name=rhel-7-server-rpms
baseurl=ftp://192.0.0.20/pub/repos/rhel-7-server-rpms
enabled=1
gpgcheck=0
[rhel-7-server-extras-rpms]
name=rhel-7-server-extras-rpms
baseurl=ftp://192.0.0.20/pub/repos/rhel-7-server-extras-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.9-rpms]
name=rhel-7-server-ansible-2.9-rpms
baseurl=ftp://192.0.0.20/pub/repos/rhel-7-server-ansible-2.9-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ose-3.11-rpms]
name=rhel-7-server-ose-3.11-rpms
baseurl=ftp://192.0.0.20/pub/repos/rhel-7-server-ose-3.11-rpms
enabled=1
gpgcheck=0
```

###  haproxy config
```
 6443, 22623, 443, 80

backend yo4_ocp3_api 
    balance source 
    mode tcp 
    server bootstrap 192.0.0.19:6443 check 
    server master01 192.0.0.21:6443 check 
    server master02 192.0.0.22:6443 check 
    server master03 192.0.0.23:6443 check 
    
backend yo4_ocp3_mc
    #balance leastconn
    balance source
    mode tcp
    server bootstrap 192.0.0.19:22623 check 
    server master01 192.0.0.21:22623 check 
    server master02 192.0.0.22:22623 check 
    server master03 192.0.0.23:22623 check 

acl yo4_ocp3_https req_ssl_sni -m end -i yo4.ocp3.fu.igotit.co.kr
use_backend yo4_ocp3_https if yo4_ocp3_https

backend yo4_ocp3_https
    balance leastconn
    # balance source
    mode tcp
    #option tcplog
    server worker01 192.0.0.121:443 check
    server worker02 192.0.0.122:443 check

acl yo4_ocp3_http hdr_end(host) -i yo4.ocp3.fu.igotit.co.kr
    use_backend yo4_ocp3_http if yo4_ocp3_http

backend yo4_ocp3_http
    balance leastconn
    # balance source
    # mode tcp
    # option tcplog
    server worker01 192.0.0.121:80 check
    server worker02 192.0.0.122:80 check

```

### registry mirrorring
```
podman run -d --name mirror-registry -p 5000:5000 --restart=always \
   -v /opt/registry/data:/var/lib/registry \
   -v /opt/registry/auth:/auth \
   -e "REGISTRY_AUTH=htpasswd" \
   -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
   -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
   -v /opt/registry/certs:/certs \
   -e REGISTRY_HTTP_TLS_CERTIFICATE=/opt/registry/certs/registry.yo4.ocp3.fu.igotit.co.kr.crt \
   -e REGISTRY_HTTP_TLS_KEY=/opt/registry/certs/registry.yo4.ocp3.fu.igotit.co.kr.key \
   docker.io/library/registry:2
   
```   
### registry mirrorring 참고 내용
```
https://chhanz.github.io/container/2020/03/02/podman
위 링크의 내용을 참고하여 podman의 이해와 작업을 진행 하여 습니다.
```
