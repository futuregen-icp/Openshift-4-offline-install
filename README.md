[tco]
# Openshift4.4-offline install


## 1설치 환경

### 1.1 서버 구성도
![OCP 환경 구성도.png](https://i.loli.net/2020/06/19/EJr2AbLuBy4oUlF.png)
### 1.2 서버 구성 사항
|role|수량|OS|IP|core / memory / disk|필수패키지|비고|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
Gateway|&nbsp;&nbsp;&nbsp;1&nbsp;&nbsp;&nbsp;|RHEL 7.6|10.0.10.220<br>192.168.10.220|2/8/100G|Haproxy<br>DNS<br>LDAP
registry |1|RHEL 7.6|10.0.10.222<br>192.168.10.222|2/8/300G|podman<br>openshift-client<br>httpd-tools<br>mirror-registry<br>FTP
storage|1|RHEL 7.6|10.0.10.223<br>192.168.10.223|2/8/300G|NFS
bastion|1|RHEL 7.6|10.0.10.229<br>192.168.10.229|2/8/100G|FTP<br>openshift-client<br>openshift-install
bootstrap|1|coreOS 4.4.3|192.168.10.11|4/16/120G||마스터 노드 설치후 삭제
master|3|coreOS 4.4.3|192.168.10.21~|4/16/120G||
worker|n|coreOS 4.4.3|192.168.10.31~|2/8/120G||
worker|n|RHEL 7.6|192.168.10.31~|2/8/120G|ansible<br>openshift-clients<br>jq|ansible 설치

### 1.3 DNS 요구 사항
Table 3. Required DNS records
Component|Record|Description
|:--:|:--:|:--:|
Kubernetes API|api.<cluster_name>.<base_domain>.|This DNS A/AAAA or CNAME record must point to the load balancer for the control plane machines. This record must be resolvable by both clients external to the cluster and from all the nodes within the cluster.
||api-int.<cluster_name>.<base_domain>.|This DNS A/AAAA or CNAME record must point to the load balancer for the control plane machines. This record must be resolvable from all the nodes within the cluster.
Routes|*.apps.<cluster_name>.<base_domain>.|A wildcard DNS A/AAAA or CNAME record that points to the load balancer that targets the machines that run the Ingress router pods, which are the worker nodes by default. This record must be resolvable by both clients external to the cluster and from all the nodes within the cluster.
etcd|etcd-<index>.<cluster_name>.<base_domain>.|OpenShift Container Platform requires DNS A/AAAA records for each etcd instance to point to the control plane machines that host the instances. The etcd instances are differentiated by  `<index>`  values, which start with  `0`  and end with  `n-1`, where  `n`  is the number of control plane machines in the cluster. The DNS record must resolve to an unicast IPv4 address for the control plane machine, and the records must be resolvable from all the nodes in the cluster.
||_etcd-server-ssl._tcp.<cluster_name>.<base_domain>.|For each control plane machine, OpenShift Container Platform also requires a SRV DNS record for etcd server on that machine with priority `0`, weight `10` and port `2380`. A cluster that uses three control plane machines requires the following records

### 1.4 방화벽 요구 사항
Table 1. **All machines** to **all machines**
Protocol|Port|Description
|:--:|:--:|:--:|
ICMP|N/A|Network reachability tests
TCP|`9000`-`9999`|  Host level services, including the node exporter on ports  `9100`-`9101`  and the Cluster Version Operator on port  `9099`
||`10250`-`10259`|The default ports that Kubernetes reserves
||10256|openshift-sdn
UDP|4789|  VXLAN and GENEVE
||6081|VXLAN and GENEVE
||`9000`-`9999`|Host level services, including the node exporter on ports `9100`-`9101`
TCP/UDP|`30000`-`32767`|Kubernetes NodePort

Table 2. **All machines** to **control plane**
Protocol|Port|Description
|:--:|:--:|:--:|
TCP|`2379`-`2380`|etcd server, peer, and metrics ports
||6443|Kubernetes API

### 1.5 Proxy 정책
Port|Machines|Internal|External|Description
|:--:|:--:|:--:|:--:|:--:|
6443|Bootstrap and control plane. You remove the bootstrap machine from the load balancer after the bootstrap machine initializes the cluster control plane.|x|x|Kubernetes API server
22623|Bootstrap and control plane. You remove the bootstrap machine from the load balancer after the bootstrap machine initializes the cluster control plane.|x||Machine Config server
443|The machines that run the Ingress router pods, compute, or worker, by default.|x|x|HTTPS traffic
80|The machines that run the Ingress router pods, compute, or worker by default.|x|x|HTTPS traffic


## 2 사전 준비
### 2.1 Getaway 서버 설치
#### 2.1.1  OS 설치
 - IP 정보 
 
|망| IP address |
|--|:--:|
| 외부IP | 10.0.10.220 |
| 내부IP | 192.168.10.220 |

 - VM 설정

![2.1.1-vm.jpg](https://i.loli.net/2020/06/17/iWJGmVnUYEkt4L3.jpg)

- repo 설정 (192.168.10.222동신 되야 함)
```
cat /etc/yum.repos.d/ocp4-4.repo
[rhel-7-server-rpms]
name=rhel-7-server-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-rpms
enabled=1
gpgcheck=0
[rhel-7-server-extras-rpms]
name=rhel-7-server-extras-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-extras-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.8-rpms]
name=rhel-7-server-ansible-2.8-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-ansible-2.8-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.9-rpms]
name=rhel-7-server-ansible-2.9-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-ansible-2.9-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ose-4.4-rpms]
name=rhel-7-server-ose-4.4-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-ose-4.4-rpms
enabled=1
gpgcheck=0
```

#### 2.1.2  DNS 서버 설치

 - Name 규젹
 
|목적|예  |
|--|--|
| base domain | `ocp44.fu.com` |
|api server|`api.ocp44.fu.com`|
|api server 연결|`api-int.ocp44.fu.com`|
|applications 주소|`*.apps.ocp44.fu.com`|
|설치용 임시서버|`bootstrap.ocp44.fu.com`|
|master nodes|`master*.ocp44.fu.com`|
|worker nodes|`worker*.ocp44.fu.com`|
|etcd servers|`etcd-*.ocp44.fu.com`|
|etcd간 통신 주소|`_etcd-server-ssl._tcp.ocp44.fu.com`|

 - DNS 설치
```
#yum install bind  -y
```
 - DNS configration
```
# vi /etc/named.conf <= replace
options {
        listen-on port 53 { ANY; };
        listen-on-v6 port 53 { ANY; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { ANY; };

#  vi /etc/named.rfc1912.zones  <= append
zone "ocp44.fu.com" IN {
        type master;
        file "ocp44.fu.com.zone";
        allow-update { none; };
};

zone "10.168.192.in-addr.arpa" IN {
        type master;
        file "ocp44.fu.com.rr";
        allow-update { none; };
};

# vi /var/named/ocp44.fu.com.zone
$TTL 60
@       IN SOA  dns.ocp44.fu.com. root.ocp44.fu.com. (
                                        1       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
                IN      NS      ns.ocp44.fu.com.;
IN      A       192.168.10.220;
ns              IN      A       192.168.10.220;
registry        IN      A       192.168.10.222;
nfs             IN      A       192.168.10.223;
bastion         IN      A       192.168.10.229;
;
bootstrap       IN      A       192.168.10.11;
;
master01        IN      A       192.168.10.21;
master02        IN      A       192.168.10.22;
master03        IN      A       192.168.10.23;
;
worker01        IN      A       192.168.10.31;
worker02        IN      A       192.168.10.32;
;
api             IN      A       192.168.10.220;
;
api-int         IN      A       192.168.10.220;
;
etcd-0          IN      A       192.168.10.21;
etcd-1          IN      A       192.168.10.22;
etcd-2          IN      A       192.168.10.23;
;
*.apps          IN      A       192.168.10.220;
;
_etcd-server-ssl._tcp  86400  IN    SRV 0        10     2380 etcd-0
_etcd-server-ssl._tcp  86400  IN    SRV 0        10     2380 etcd-1
_etcd-server-ssl._tcp  86400  IN    SRV 0        10     2380 etcd-2
;

vi /var/named/ocp44.fu.com.rr
$TTL 20
@       IN      SOA     ns.ocp44.fu.com.  root (
                        2019070700      ; serial
                        3H              ; refresh (3 hours)
                        30M             ; retry (30 minutes)
                        2W              ; expiry (2 weeks)
                        1W )            ; minimum (1 week)
        IN      NS      ns.ocp44.fu.com.
;
; syntax is "last octet" and the host must have fqdn with trailing dot
21      IN      PTR     master01.ocp44.fu.com.
22      IN      PTR     master02.ocp44.fu.com.
23      IN      PTR     master03.ocp44.fu.com.
;
11      IN      PTR     bootstrap.ocp44.fu.com.
;
220      IN      PTR     api.ocp44.fu.com.
220      IN      PTR     api-int.ocp44.fu.com.
;
31      IN      PTR     worker01.ocp44.fu.com.
32      IN      PTR     worker02.ocp44.fu.com.

```
 - DNS 서버 시작
```
DNS서버를 부팅시 자동 시작하도록 등록
$ systemctl enable named

DNS서버 시작
$ systemctl start named

상태확인
$ systemctl status named
```


 - DNS 설정 검증
```
[root@gateway ~]# nslookup master01.ocp44.fu.com. 192.168.10.220
Server:         192.168.10.220
Address:        192.168.10.220#53

Name:   master01.ocp44.fu.com
Address: 192.168.10.21

[root@gateway ~]# nslookup api.ocp44.fu.com. 192.168.10.220
Server:         192.168.10.220
Address:        192.168.10.220#53

Name:   api.ocp44.fu.com
Address: 192.168.10.220
```
#### 2.1.3  HAProxy 서버 설치
 - L/B 설치
```
#yum install haproxy.x86_64 -y
```

 - L/B 설정
```
vi /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2        info

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

##
#  balancing for OCP Kubernetes API Server
##
frontend openshift-api-server
    bind *:6443
    mode http
    option tcplog
    #acl
    acl ocp44 hdr_dom(host) -i ocp44.fu.com
    #call
    use_backend ocp44-openshift-api-server if ocp44
    

backend ocp44-openshift-api-server
    balance source
    server bootstrap 192.168.10.11:6443 check
    server master01 192.168.10.21:6443 check
    server master02 192.168.10.22:6443 check
    server master03 192.168.10.23:6443 check
##
# balancing for OCP Machine Config Server
##
frontend machine-config-server
    bind *:22623
    mode http
    option tcplog
    #acl
    acl ocp44 hdr_dom(host) -i ocp44.fu.com
    #call
    use_backend ocp44-machine-config-server if ocp44

backend ocp44-machine-config-server
    balance source
    server bootstrap 192.168.10.11:22623 check
    server master01 192.168.10.21:22623 check
    server master02 192.168.10.22:22623 check
    server master03 192.168.10.23:22623 check

##
# balancing for OCP Ingress Insecure Port & Admin Page
##
frontend ingress-http
    bind *:80
    mode http
    option tcplog
    #acl
    acl ocp44 hdr_dom(host) -i ocp44.fu.com
    #call
    use_backend ocp44-ingress-http if ocp44

backend ocp44-ingress-http
    balance source
    server worker01 192.168.10.31:80 check
    server worker02 192.168.10.32:80 check

##
# balancing for OCP Ingress Secure Port
##
frontend ingress-https
    bind *:443
    mode http
    option tcplog
    #acl
    acl ocp44 hdr_dom(host) -i ocp44.fu.com
    #call
    use_backend ocp44-ingress-https if ocp44

backend ocp44-ingress-https
    balance leastconn
#    balance source
    server worker01 192.168.10.31:443 check
    server worker02 192.168.10.32:443 check
```

 - selinux config
```
setsebool -P haproxy_connect_any 1
```


- Log 설정
```
vi /etc/rsyslog.conf
# Provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514
local2.*              /var/log/haproxy.log
---

service rsyslog restart
service haproxy restart
systemctl enable haproxy
```
- HAProxy  정상 작동 첵크
```
netstat -na |egrep ":80 |6443|443|22623"
tcp        0      0 0.0.0.0:6443            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22623           0.0.0.0:*               LISTEN

curl http://api.ocp44.fu.com:80
로그 확인
ingress-http ocp44-ingress-http/<NOSRV>

curl http://api.ocp44.fu.com:6443
로그 확인
openshift-api-server ocp44-openshift-api-server/<NOSRV>
```

#### 2.1.4  LDAP 서버 설치
- 나중에 추가

#### 2.1.5   IP Masquerade 설정
- ip forword를 위한 kernel 설정
```
sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.d/ip_forward.conf
```
- 인터페이스 설정
```
#외부망 ip를 external setting
firewall-cmd --set-default-zone=external
#내부망 ip를 internal setting
firewall-cmd --permanent --zone=internal --add-interface=ens224
```
- ip Masquerade 설정
```
#192.168.10.0 나갈떄 ens192(외부망 인터페이스 통하게 한다)
firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o ens192 -j MASQUERADE -s 192.168.10.0/24
firewall-cmd --reload
```
- interface matric change
```
#외부망 interface metric값을 내부망 interface metric값보다 작게 한다
#Metric Ref    Use Iface
#100    0        0 ens192
#101    0        0 ens224

nmcli connection modify ens192 ipv4.route-metric 100
```
#### 2.1.6   방화벽 설정
- external zone firewall
```
firewall-cmd --permanent --zone=external --add-port=80/tcp
firewall-cmd --permanent --zone=external --add-port=443/tcp
```
- internal zone firewall
```
firewall-cmd --permanent --zone=internal --add-service=dns
firewall-cmd --permanent --zone=internal --add-service=ssh
firewall-cmd --permanent --zone=internal --add-port=80/tcp
firewall-cmd --permanent --zone=internal --add-port=443/tcp
firewall-cmd --permanent --zone=internal --add-port=6443/tcp
firewall-cmd --permanent --zone=internal --add-port=8443/tcp
firewall-cmd --permanent --zone=internal --add-port=10256/tcp
firewall-cmd --permanent --zone=internal --add-port=2379-2380/tcp
firewall-cmd --permanent --zone=internal --add-port=9000-9999/tcp
firewall-cmd --permanent --zone=internal --add-port=10249-10259/tcp
firewall-cmd --permanent --zone=internal --add-port=22623/tcp
firewall-cmd --permanent --zone=internal --add-port=4789/udp
firewall-cmd --permanent --zone=internal --add-port=6081/udp
firewall-cmd --permanent --zone=internal --add-port=9000-9999/udp
firewall-cmd --permanent --zone=internal --add-port=30000-32767/udp

firewall-cmd --reload
```

### 2.2 Registry서버 설치
#### 2.2.1  OS 설치
 - IP 정보 
 
|망| IP address |
|--|:--:|
| 외부IP | 10.0.10.222 |
| 내부IP | 192.168.10.222 |

 - VM 설정
![regsssss-vm.jpg](https://i.loli.net/2020/06/17/AzBZjHLu2msxcif.jpg)

- 서브 스크립션 등록
```
subscription-manager register --username=***** --password=*******
subscription-manager refresh
subscription-manager list --available --matches '*OpenShift*'
subscription-manager attach --pool=8a85f99a71a877730171ecdc0ba5093d
subscription-manager repos\
    --enable="rhel-7-server-rpms"\
    --enable="rhel-7-server-extras-rpms"\
    --enable="rhel-7-server-ansible-2.8-rpms"\
    --enable="rhel-7-server-ose-4.4-rpms"
```
- 서브 스크립션 검증
```
[root@registry ~]# yum repolist
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
repo id                                            repo name                                                                           status
rhel-7-server-ansible-2.8-rpms/x86_64              Red Hat Ansible Engine 2.8 RPMs for Red Hat Enterprise Linux 7 Server                   16
rhel-7-server-extras-rpms/x86_64                   Red Hat Enterprise Linux 7 Server - Extras (RPMs)                                    1,285
rhel-7-server-ose-4.4-rpms/x86_64                  Red Hat OpenShift Container Platform 4.4 (RPMs)                                        226
rhel-7-server-rpms/7Server/x86_64                  Red Hat Enterprise Linux 7 Server (RPMs)                                            29,127
repolist: 30,654
```
#### 2.2.2 Docker Registry 구성
- install package
```
yum -y install podman httpd-tools
```

- Create folders for the registry
```
mkdir -p /opt/registry/{auth,certs,data}
```
- Create certificate for the registry
```
cd /opt/registry/certs
openssl req -newkey rsa:4096 -nodes -sha256 -keyout registry.ocp44.fu.com.key -x509 -days 365 -out registry.ocp44.fu.com.crt
...
Country Name (2 letter code) [XX]:KR
State or Province Name (full name) []:Seoul
Locality Name (eg, city) [Default City]:gurogu
Organization Name (eg, company) [Default Company Ltd]:futuregen
Organizational Unit Name (eg, section) []:ICS
Common Name (eg, your name or your server's hostname) []:registry.ocp44.fu.com
Email Address []:luoqs@futuregen.co.kr
```

- registry username and password 생성
```
htpasswd -bBc /opt/registry/auth/htpasswd admin admin
cat /opt/registry/auth/htpasswd
admin:$2y$05$StVswQVO.hVYzkuFuyrrbuyNIY8COy0vaWfq/7MdLQvxLtuaVO/IG
```
- mirror-registry container 생성
```
podman run --name mirror-registry -p 5000:5000 \
     -v /opt/registry/data:/var/lib/registry:z \
     -v /opt/registry/auth:/auth:z \
     -e "REGISTRY_AUTH=htpasswd" \
     -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
     -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
     -v /opt/registry/certs:/certs:z \
     -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.ocp44.fu.com.crt \
     -e REGISTRY_HTTP_TLS_KEY=/certs/registry.ocp44.fu.com.key \
     -d docker.io/library/registry:2
```
- docker login용으로 인증서 디폴트 위치로 변경
```
cp /opt/registry/certs/registry.ocp44.fu.com.crt /etc/pki/ca-trust/source/anchors/
#bastion 노드에도 복사 안하면 # x509 에러 나타남
update-ca-trust
```
- 확인
```
[root@gateway var]# curl -u admin:admin -k https://registry.ocp44.fu.com:5000/v2/_catalog
{"repositories":[]}
```

#### 2.2.3 yum repository 구성

- ftp 설치
```
yum install vsftpd -y
```
-ftp 기동
```
systemctl enable vsftpd
systemctl start vsftpd

curl ftp://<ip or hostname>/pub/
```

- repo 동기화 및 구성
``` 
yum -y install createrepo

for repo in rhel-7-server-rpms rhel-7-server-extras-rpms rhel-7-server-ansible-2.8-rpms rhel-7-server-ose-4.4-rpms
do
  reposync --gpgcheck -lm --repoid=${repo} --download_path=/var/ftp/pub/repos
  createrepo -v /var/ftp/pub/repos/${repo} -o /var/ftp/pub/repos${repo}
done

chmod -R +r ./repos/
```
- repo 설정(yum repo 필요한 서버 설정이 필요함)
```
cat /etc/yum.repos.d/ocp4-4.repo
[rhel-7-server-rpms]
name=rhel-7-server-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-rpms
enabled=1
gpgcheck=0
[rhel-7-server-extras-rpms]
name=rhel-7-server-extras-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-extras-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.8-rpms]
name=rhel-7-server-ansible-2.8-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-ansible-2.8-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ose-4.4-rpms]
name=rhel-7-server-ose-4.4-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-ose-4.4-rpms
enabled=1
gpgcheck=0
```
#### 2.2.4   방화벽 설정
- 인터페이스 설정
```
#외부망 ip를 external setting
firewall-cmd --set-default-zone=external
#내부망 ip를 internal setting
firewall-cmd --permanent --zone=internal --add-interface=ens224
```
- external zone firewall
- internal zone firewall
```
firewall-cmd --permanent --zone=internal --add-service=ftp
firewall-cmd --permanent --zone=internal --add-port=5000/tcp

firewall-cmd --reload
```
### 2.3 Storage 서버 설치
#### 2.3.1 NFS 서버 구성
-나중에 추가

### 2.4 Bastion 서버 설치
#### 2.4.1 NFS 서버 구성
 - IP 정보 
 
|망| IP address |
|--|:--:|
| 외부IP | 10.0.10.229 |
| 내부IP | 192.168.10.229 |

 - VM 설정
![bastion.jpg](https://i.loli.net/2020/06/17/mZxvdiV3BrUaOL4.jpg)

- repo 설정
```
vi /etc/yum.repos.d/ocp4-4.repo
[rhel-7-server-rpms]
name=rhel-7-server-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-rpms
enabled=1
gpgcheck=0
[rhel-7-server-extras-rpms]
name=rhel-7-server-extras-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-extras-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.8-rpms]
name=rhel-7-server-ansible-2.8-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-ansible-2.8-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.9-rpms]
name=rhel-7-server-ansible-2.9-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-ansible-2.9-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ose-4.4-rpms]
name=rhel-7-server-ose-4.4-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-ose-4.4-rpms
enabled=1
gpgcheck=0
```
#### 2.4.2   FTP 서버 설치
- ftp 설치
```
yum install vsftpd -y

systemctl enable vsftpd
systemctl start vsftpd
```
#### 2.4.3   설치 준비
- Compressed metal RAW 파일 다운로드
```
cd /var/ftp/pub/
wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.4/4.4.3/rhcos-4.4.3-x86_64-metal.x86_64.raw.gz
mv rhcos-4.4.3-x86_64-metal.x86_64.raw.gz rhcos-443.raw.gz
chmod +r rhcos-443.raw.gz
```
- ssh 인증키 생성
```
## bastion 일반 계정에서 생성
adduser core
su - core
ssh-keygen -t rsa -b 4096 -N ''

## 인증키 복사 방법
ssh-copy-id core@[serverip or hostname]
```
- openshift 관련 패키지 설치
```
yum install openshift-clients jq podman httpd httpd-tools buildah skopeo  -y

##wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux.tar.gz
#버전 이슈로 openshift-install는 다운받지 말고 ocp image repository에서 닫아야 한다 아래 문서 참고
```
- pull-secret down
```
https://cloud.redhat.com/openshift/install/metal/user-provisioned pull-secret download
cd /opt/ocp44/pull
cat ./pull-secret.text | jq . > pull-secret.json
```
- mirror registry 구성시 추가 부분
```
mirror-registry 등록 
echo -n 'admin:admin' | base64 -w0  # admin:admin ( mirror registry 접속 계정 )
**YWRtaW46YWRtaW4=**
vi pull-secret.json  (아래 부부분 추가)
    }**,
    "registry.ocp44.fu.com:5000": {
      "auth": "YWRtaW46YWRtaW4=",
      "email": "luoqs@futuregen.co.kr"
    }**
  }
}
```
- authenticate with the target mirror registry
```
podman login -u admin -p admin registry.ocp44.fu.com:5000
podman login -u qingsong1989 -p xxxx registry.redhat.io
```
- Mirroring OCP Image repository
```
#[https://quay.io/repository/openshift-release-dev/ocp-release?tab=tags](https://quay.io/repository/openshift-release-dev/ocp-release?tab=tags)
export OCP_RELEASE="4.4.5-x86_64"
export LOCAL_REGISTRY='registry.ocp44.fu.com:5000' 
export LOCAL_REPOSITORY='ocp44/openshift4' 
export PRODUCT_REPO='openshift-release-dev' 
export LOCAL_SECRET_JSON='/opt/ocp44/pull/pull-secret.json' 
export RELEASE_NAME="ocp-release"
```
- Mirror the repository
```
oc adm -a ${LOCAL_SECRET_JSON} release mirror \
  --from=quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE} \
  --to=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} \
  --to-release-image=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}

output 꼭 저장
---
  - mirrors:
    - registry.ocp44.fu.com:5000/ocp4/openshift445
    source: quay.io/openshift-release-dev/ocp-release
  - mirrors:
    - registry.ocp44.fu.com:5000/ocp4/openshift445
    source: quay.io/openshift-release-dev/ocp-v4.0-art-dev
---
```

- openshift-install 가져옴
```
oc adm -a ${LOCAL_SECRET_JSON} release extract --command=openshift-install "${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}" && chmod +x openshift-install
mv openshift-install /usr/local/bin/

#버전 확인
[root@bastion pull]# openshift-install version
openshift-install 4.4.5
built from commit 15eac3785998a5bc250c9f72101a4a9cb767e494
release image registry.ocp44.fu.com:5000/ocp4/openshift445@sha256:4a461dc23a9d323c8bd7a8631bed078a9e5eec690ce073f78b645c83fb4cdf74
```

- Building an Operator catalog image (offline 에서 사용한 Operator hub 자원 )
```
cp /opt/ocp44/pull/pull-secret.json ${XDG_RUNTIME_DIR}/containers/auth.json
export REG_CREDS=/opt/ocp44/pull/pull-secret.json

oc adm catalog build \
  --appregistry-org redhat-operators \
  --from=registry.redhat.io/openshift4/ose-operator-registry:v4.4 \
  --filter-by-os="linux/amd64" \
  --to=registry.ocp44.fu.com:5000/olm/redhat-operators:v1 \
  -a ${REG_CREDS}
```
- oc adm catalog mirror
```
oc adm catalog mirror \
  -a ${REG_CREDS} \
  registry.ocp44.fu.com:5000/olm/redhat-operators:v1 \
  registry.ocp44.fu.com:5000

여기서 error가 납니다.
I0409 08:04:48.342110 11331 mirror.go:231] wrote database to /tmp/db-225652515/bundles.db 
W0409 08:04:48.347417 11331 mirror.go:258] errors during mirroring. the full contents of the catalog may not have been mirrored: couldn't parse image for mirroring (), skipping mirror: invalid reference format 
I0409 08:04:48.385816 11331 mirror.go:329] wrote mirroring manifests to redhat-operators-manifests

해결 방법1 권장 하지 않음
#최신 manifest 찾아서 만약게 /tmp/cache-320634009 최신이라면 거기에 모든 파일의 value을 image 로 바꿘다.
ls -l /tmp/cache-*/
sed -i "s/value: registry/image: registry/g" $(egrep -rl "value: registry" /tmp/cache-320634009/)
권장 하지 않음 기제 하는이유는 에라 원인을 이해

해결 방법2 권장 
cat redhat-operators-manifests/mapping.txt | while  read line; do 
  origin=$(echo  $line | cut -d= -f1) 
  target=$(echo  $line | cut -d= -f2) 
  if [[ "$origin" =~ "sha256" ]]; then 
    tag=$(echo  $origin | cut -d: -f2 | cut -c -8) 
    skopeo copy --all docker://$origin docker://$target:$tag  
  else 
    skopeo copy --all docker://$origin docker://$target  
  fi  
done
```
- openshift 설치 directory 준비
```
cd /opt/ocp44
chown core.core /opt/ocp44 -R
su - core
cd /opt/ocp44
```
- install-config.yaml 생성 - offline
```
vi install-config.yaml
apiVersion: v1
baseDomain: fu.com
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: ocp44
networking:
  clusterNetwork:
  - cidr: 10.136.0.0/14
    hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.36.0.0/16
platform:
  none: {}
fips: false
pullSecret: '{"auths":{"registry.ocp44.fu.com:5000": {"auth": "YWRtaW46YWRtaW4=","email": "luoqs@futuregen.co.kr"}}}'
sshKey: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDVKNP2qSJboP6B1U+ickuqYJbqXVLoBQ+PtbjAhnPWkRz8fpmZUIz1EoTjfU3zd14AVZ3uz6iJuiuC7200P1cOleG04xPCybpLCQlrzERx9/Q+gEmq0XRPh3NhmgEZHGxXi03bur3xoqpe+e4SBI0GaZNc+qiqN5kKRSrsXDBXNUbAeNnfrRtvfW45rRj3HoXs3/68TGh/TrxOpiMGpRR8YOq5R6kCOKGB4ujH4D9JpFUkuymuRzLqAqzPo3hW8sB8yXhBnd7HXnJwvtvsinMdUmaFQA1WW7EXCqEPepcFhiKPPUqjImN3fcc3B11nIiBXcixA3zEK2ekPf4HYau4guv3Tod31D04Jvy+ObU08x/mzk/e51OO4jXkLWFrXE1ftySVc9TZiPqVNAB0PQiAfkWGLc5N5fw1Ef0ix76oBgDzvK83/cJA4F4gUx5/pP1ACYkYIhfm8mtuhU58RxBQ/j5BYAnIZgAEZRsP/IIxFbtWbRUB9DJePhK0ju6zZ5EvPzsSa7NxnhXFslPZT7+YB9FXKc9Con03bSxOb8Pi9HW/vQa62nLb2z4r6N2Fc4AOi+Y44uRnAdejw/f/xeKGNjoOdNn3Pfxn7RSiGZcCT3IY5R/orHj6n2Vj5Erv5A1DLJTBWIlG+0OHma0/D/IegOEro7mwRp0FktDu5aELcXw== core@bastion.ocp44.fu.com'
additionalTrustBundle: |
  -----BEGIN CERTIFICATE-----
  MIIGATCCA+mgAwIBAgIJAJkw5HvRMJdLMA0GCSqGSIb3DQEBCwUAMIGWMQswCQYDVQQGEwJLUjEOMAwGA1UECAwFU2VvdWwxDzANBgNVBAcMBmd1cm9ndTESMBAGA1UECgwJZnV0dXJlZ2VuMQwwCgYDVQQLDANJQ1MxHjAcBgNVBAMMFXJlZ2lzdHJ5Lm9jcDQ0LmZ1LmNvbTEkMCIGCSqGSIb3DQEJARYVbHVvcXNAZnV0dXJlZ2VuLmNvLmtyMB4XDTIwMDYxNzEyMDc1NVoXDTIxMDYxNzEyMDc1NVowgZYxCzAJBgNVBAYTAktSMQ4wDAYDVQQIDAVTZW91bDEPMA0GA1UEBwwGZ3Vyb2d1MRIwEAYDVQQKDAlmdXR1cmVnZW4xDDAKBgNVBAsMA0lDUzEeMBwGA1UEAwwVcmVnaXN0cnkub2NwNDQuZnUuY29tMSQwIgYJKoZIhvcNAQkBFhVsdW9xc0BmdXR1cmVnZW4uY28ua3IwggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIKAoICAQCyN1lQ/8ZclSzfXnpX3kj8/ejQgGyf8Vl2obtugPUl5fyzfKGh3ejmC8OHr0QN5Qvmjk7j0zNO5l20Tps3wWigZu4gLV6sLDCSfGRriwB7TtbfpmxPu8UoBh6pAhRAPEYNdNyveI4woJ6MY0xsOv2Br95e6Du8Sel6h6WUNTcH9XcXGHqnnpxDDR6RZ2F9b/us9wtzJj00NJS1Mg/rcIMKPiIjyTYfy3T4m7Rng07VhdVa5Uu2z1J+zpkpq+g1pQ/mc8na6amVlLZZ6FJHan4FxrtOHGTy7+dYioh/b0cZti+gPXuevU3E2yohzHDLc+MEA91roem5F+7LmPg2hxwLRCaJIxICfLdUFY0UNQ+K3yG3X6er9hl3Wdyl6YWb+8iL1Dl0Vd5KIWm+KqEibuUgbNbQLfcarrfrrjxiYOP1CRQOtvAa4h3F9Vqew2JTXuP2I3KwQDpgk9DQL9V00GgP+MeJPR7OsBUS2FiOjCEIh2OZgIWcneUr1hYEUr7iNB0CazEQFDA/RWMtsNIC9RnpyscDQBvEMYdGhYDy/oT/sdYYpDeKb9DfXan+khifHnidTl2vS+Zoa5eCvxR0JOBkEK4DDBvAwZ9YMrBzznikg0IQUO8HP4N4lcnpf6yy00V9/h8NO5fldYGx4XYuCRnjf+zJGJDn1Gnavs3wXUb+OwIDAQABo1AwTjAdBgNVHQ4EFgQUao294BXgvRxvAE5vO2ymVYbKoGkwHwYDVR0jBBgwFoAUao294BXgvRxvAE5vO2ymVYbKoGkwDAYDVR0TBAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAgEAiGRV3VXH8b+D2JDWnIFxGlHsGQLC3NY4MPIYI+YwA0jGzCuobUeUuiHYms+3BJEf6vfvUQ3I/ru+z0FIAgzk9o8QeCiNpqvDLTVMAAPa1QOPc4oFZIKjzAVTIXvItRUAXzBWKOfkXhXwsIEUoNlooxeXgDOrhx9L+WjIi58PkU5KyDyzwJmOYfYpRAx7WbB906OX45982eTSHdRIaPqxLPBXHE/MrqcOtKp/b6qpr4hinsUGLMTTQfGbNXAD3Fwtz13BaFKDizaCCjflLOTF6LaG/AgJx1kPe03i6PqdL/JOenwSO6jllmwJr5pE5qZ76iGyh0wS8ozX++66KRqXQgLMzhw5uWn7aFaDyuZ78QMGEEesVvV8DaNGI0wTscy/R4Xdeh5KmdYVniZ8/4dup5W2luZd0ysThD8FlSflSqf3ZaJXRPLn1Jhunys+SwutCRn/pDsuZQIcz+QnjjySOn5KyvD13I2HeD9TB1IZ2tpeZ60McNx63m8PlIFXLJWKHOZwXhZMJ57lvxDjDPfU01gbT90yGR/fao7LJCetKMYg/4ifK3fCy95avyCynxCXSvXpP1WBZ4BxC6sZzdbedAWP5vP2oY3vk9y5KIGrUraCefnRjTyEcarc6EQr7cLhoAgTed6NE546mlBpz+UBZvQyjHvxwG7HuK/ZThU/uxY=
  -----END CERTIFICATE-----
imageContentSources:
- mirrors:
  - registry.ocp44.fu.com:5000/ocp4/openshift445
  source: quay.io/openshift-release-dev/ocp-release
- mirrors:
  - registry.ocp44.fu.com:5000/ocp4/openshift445
  source: quay.io/openshift-release-dev/ocp-v4.0-art-dev


**## pullSecret는 mirror registry의 pullsecret 입력
## sshKey  /home/core/.ssh/ras.pub
##** additionalTrustBundle: **mirror registry생성시 만든 인증서 
##** imageContentSources **mirror registry 생성후 oc adm으로 설치 이미지 미러후 나오는 메세지 내용 추가** 
```

#### 2.4.4   방화벽 설정
- 인터페이스 설정
```
#외부망 ip를 external setting
firewall-cmd --set-default-zone=external
#내부망 ip를 internal setting
firewall-cmd --permanent --zone=internal --add-interface=ens224
```
- external zone firewall
- internal zone firewall
```
firewall-cmd --permanent --zone=internal --add-service=ftp

firewall-cmd --reload
```

## 3 OCP 설치
### 3.1 Kubernetes manifest 및 ignition 파일 생성
#### 3.1.1 Kubernetes manifests 생성
- bastion node login 
`ssh core@bastion.ocp44.fu.com`

- Generate the Kubernetes manifests for the cluster:
```
mkdir /opt/ocp44/install-`date +%Y%m%d$h`
[core@bastion ocp44]$ openshift-install create manifests --dir=/opt/ocp44/install-`date +%Y%m%d$h`
INFO Consuming Install Config from target directory
WARNING Making control-plane schedulable by setting MastersSchedulable to true for Scheduler cluster settings

# cluster-scheduler-02-config 수정  control-plane에 사용자 파드 배포되지않게
vi /opt/ocp44/install-20200618/manifests/cluster-scheduler-02-config.yml
---
spec:
  mastersSchedulable: false
  policy:
---
```
#### 3.1.2 ignition 파일 생성
- Obtain the Ignition config files
```
openshift-install create ignition-configs --dir=/opt/ocp44/install-`date +%Y%m%d$h`

.
├── auth
│   ├── kubeadmin-password (okd login password)
│   └── kubeconfig
├── bootstrap.ign
├── master.ign
├── metadata.json
└── worker.ign
```
- ignition 파일을 ftp에 노출
```
chmod +r /opt/ocp44/install-`date +%Y%m%d$h`/*ign
sudo cp /opt/ocp44/install-`date +%Y%m%d$h`/*ign /var/ftp/pub
```
### 3.2 Kubernetes manifest 및 ignition 파일 생성
#### 3.2.1 사전 필요 사항
 - [ ] ISO: `rhcos-<version>-installer.<architecture>.iso`
 - [ ] Compressed metal RAW: `rhcos-<version>-metal.<architecture>.raw.gz`
 - [ ] ftp 통신

#### 3.2.2 bootstrap VM 준비
 - IP 정보 
 
|망| IP address |
|--|:--:|
| 외부IP | X |
| 내부IP | 192.168.10.11 |

 - VM 설정
![bootstrap.jpg](https://i.loli.net/2020/06/18/9vb3j2PFRM7W8eh.jpg)

- VM 기동 하여 부팅 화면에서 TAB을 눌러서 kernel command 편집
```
coreos.inst=yes
coreos.inst.install_dev=sda 
coreos.inst.image_url=ftp://192.168.10.229/pub/rhcos-443.raw.gz
coreos.inst.ignition_url=ftp://192.168.10.229/pub/bootstrap.ign
ip=192.168.10.11::192.168.10.1:255.255.255.0:bootstrap.ocp44.fu.com:ens192:none nameserver=192.168.10.220
```
![bootstrap01.jpg](https://i.loli.net/2020/06/18/wO9oYgZeqnvprBL.jpg)

![bootstrap02.jpg](https://i.loli.net/2020/06/18/QMgjDSRqWful2i1.jpg)

- 서저 자동 재부팅후 ssh 접속 가능하면 성곡 적으로 설치
```
su - core
ssh bootstrap.ocp44.fu.com
```
#### 3.2.3 Creating the cluster
- Monitor the bootstrap process
```
openshift-install --dir=/opt/ocp44/install-`date +%Y%m%d$h` wait-for bootstrap-complete -log-level=debug
```

#### 3.2.4 master VM 준비
- master01 서버 설치
 - IP 정보 
 
|망| IP address |
|--|:--:|
| 외부IP | X |
| 내부IP | 192.168.10.21 |

 - VM 설정
![bootstrap.jpg](https://i.loli.net/2020/06/18/9vb3j2PFRM7W8eh.jpg)

- VM 기동 하여 부팅 화면에서 TAB을 눌러서 kernel command 편집
```
coreos.inst=yes
coreos.inst.install_dev=sda 
coreos.inst.image_url=ftp://192.168.10.229/pub/rhcos-443.raw.gz
coreos.inst.ignition_url=ftp://192.168.10.229/pub/master.ign
ip=192.168.10.21::192.168.10.1:255.255.255.0:master01.ocp44.fu.com:ens192:none nameserver=192.168.10.220
```
![bootstrap01.jpg](https://i.loli.net/2020/06/18/wO9oYgZeqnvprBL.jpg)

![master01.jpg](https://i.loli.net/2020/06/18/k4FnA5orGEq1cVP.jpg)

- master02 서버 설치
 - IP 정보 
 
|망| IP address |
|--|:--:|
| 외부IP | X |
| 내부IP | 192.168.10.22 |
-절차는 master01과 동일함 

- master03 서버 설치
 - IP 정보 
 
|망| IP address |
|--|:--:|
| 외부IP | X |
| 내부IP | 192.168.10.23 |
-절차는 master01과 동일함 

#### 3.2.5 클러스터 상태 확인
- 마스터노드 배포 완료
```
[core@bastion ~]$ openshift-install --dir=/opt/ocp44/install-`date +%Y%m%d$h` wait-for bootstrap-complete --log-level=info
INFO Waiting up to 20m0s for the Kubernetes API at https://api.ocp44.fu.com:6443...
INFO API v1.17.1 up
INFO Waiting up to 40m0s for bootstrapping to complete...
INFO It is now safe to remove the bootstrap resources

#bootstrap 삭제 해도 됨
#삭제시 haproxy 도 같이 삭제
```
- kubeconfig 환경변수에 적용
```
[core@bastion ~]$ id
uid=1000(core) gid=1000(core) groups=1000(core)
vi ~/.bashrc
[core@bastion ~]$ oc get nodes
NAME                    STATUS   ROLES    AGE     VERSION
master01.ocp44.fu.com   Ready    master   9m36s   v1.17.1
master02.ocp44.fu.com   Ready    master   9m30s   v1.17.1
master03.ocp44.fu.com   Ready    master   9m31s   v1.17.1

#마스트 노드 배포 끝
```

#### 3.2.6 노드 추가 - CoreOS
- worker01 서버 설치
 - IP 정보 
 
|망| IP address |
|--|:--:|
| 외부IP | X |
| 내부IP | 192.168.10.31 |

 - VM 설정
![bootstrap.jpg](https://i.loli.net/2020/06/18/9vb3j2PFRM7W8eh.jpg)

- VM 기동 하여 부팅 화면에서 TAB을 눌러서 kernel command 편집
```
coreos.inst=yes
coreos.inst.install_dev=sda 
coreos.inst.image_url=ftp://192.168.10.229/pub/rhcos-443.raw.gz
coreos.inst.ignition_url=ftp://192.168.10.229/pub/master.ign
ip=192.168.10.31::192.168.10.1:255.255.255.0:worker01.ocp44.fu.com:ens192:none nameserver=192.168.10.220
```

```
[core@bastion ~]$ oc get csr
NAME        AGE    REQUESTOR                                                                   CONDITION
csr-682gd   47m    system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Approved,Issued
csr-8txnw   6m1s   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
csr-c6nbn   47m    system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Approved,Issued
csr-nn9wb   47m    system:node:master03.ocp44.fu.com                                           Approved,Issued
csr-pqssb   47m    system:node:master02.ocp44.fu.com                                           Approved,Issued
csr-rnnnm   47m    system:node:master01.ocp44.fu.com                                           Approved,Issued
csr-whvvp   47m    system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Approved,Issued
csr-xtmfr   21m    system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending


oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve

[core@bastion ~]$ oc get nodes
NAME                    STATUS   ROLES    AGE   VERSION
master01.ocp44.fu.com   Ready    master   49m   v1.17.1
master02.ocp44.fu.com   Ready    master   49m   v1.17.1
master03.ocp44.fu.com   Ready    master   49m   v1.17.1
worker01.ocp44.fu.com   Ready    worker   43s   v1.17.1

[core@bastion ~]$ oc get co
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
authentication                             4.4.5     True        False         False      3m28s
cloud-credential                           4.4.5     True        False         False      97m
cluster-autoscaler                         4.4.5     True        False         False      46m
console                                    4.4.5     True        False         False      5m11s
csi-snapshot-controller                    4.4.5     True        False         False      9m4s
dns                                        4.4.5     True        False         False      52m
etcd                                       4.4.5     True        False         False      51m
image-registry                             4.4.5     True        False         False      47m
ingress                                    4.4.5     True        False         False      9m6s
insights                                   4.4.5     True        False         False      47m
kube-apiserver                             4.4.5     True        False         False      50m
kube-controller-manager                    4.4.5     True        False         False      50m
kube-scheduler                             4.4.5     True        False         False      50m
kube-storage-version-migrator              4.4.5     True        False         False      9m23s
machine-api                                4.4.5     True        False         False      52m
machine-config                             4.4.5     True        False         False      51m
marketplace                                4.4.5     True        False         False      47m
monitoring                                 4.4.5     True        False         False      8m5s
network                                    4.4.5     True        False         False      53m
node-tuning                                4.4.5     True        False         False      52m
openshift-apiserver                        4.4.5     True        False         False      48m
openshift-controller-manager               4.4.5     True        False         False      47m
openshift-samples                          4.4.5     True        False         False      43m
operator-lifecycle-manager                 4.4.5     True        False         False      52m
operator-lifecycle-manager-catalog         4.4.5     True        False         False      51m
operator-lifecycle-manager-packageserver   4.4.5     True        False         False      47m
service-ca                                 4.4.5     True        False         False      52m
service-catalog-apiserver                  4.4.5     True        False         False      52m
service-catalog-controller-manager         4.4.5     True        False         False      52m
storage                                    4.4.5     True        False         False      47m

```
#### 3.2.7 노드 추가 - RHEL
- worker02 서버 설치
 - IP 정보 
 
|망| IP address |
|--|:--:|
| 외부IP | X |
| 내부IP | 192.168.10.32 |

- **시스템 requirements**

**provisioning server (여기서는 bastion 사용)**

     - ansible 실행 가능한 서버
     - kubeconfig 파일
     - worker node에 대한 ssh 접근 권한

**worker node**

     - Base OS: [RHEL 7.6]   
       Only RHEL 7.6 is supported in OpenShift
     - NetworkManager 1.0 or later
     - 1 vCPU.
     - Minimum 8 GB RAM.
     - 20GB hard disk
 2. 사전 준비

**worker node**

- repo 설정

```
cat /etc/yum.repos.d/ocp4-4.repo
[rhel-7-server-rpms]
name=rhel-7-server-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-rpms
enabled=1
gpgcheck=0
[rhel-7-server-extras-rpms]
name=rhel-7-server-extras-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-extras-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.8-rpms]
name=rhel-7-server-ansible-2.8-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-ansible-2.8-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.9-rpms]
name=rhel-7-server-ansible-2.9-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-ansible-2.9-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ose-4.4-rpms]
name=rhel-7-server-ose-4.4-rpms
baseurl=ftp://192.168.10.222/pub/repos/rhel-7-server-ose-4.4-rpms
enabled=1
gpgcheck=0
```
-  방화벽 오픈 
```
systemctl disable --now firewalld.service
```

 **bastion 에서 실행**

- inventory 파일 생성
```
vi /home/core/inventory/hosts/inventory.yaml 
-------------------------- 
[all:vars] 
ansible_user=root 
#ansible_become=True 
openshift_kubeconfig_path="/opt/ocp44/install-20200618/auth/kubeconfig" 

[workers] 
worker01.ocp44.fu.com 

[new_workers] 
worker02.ocp44.fu.com
#mycluster-rhel7-3.example.com (다중 노드 추가 시)
```
- ssh key 복사
```
ssh-copy-id root@worker02.ocp44.fu.com
```
- ansible playbook 실행
```
cd /usr/share/ansible/openshift-ansible
ansible-playbook -K -i /home/core/inventory/hosts/inventory.yaml playbooks/scaleup.yml
```    
- 노드 확인 
```
[core@bastion openshift-ansible]$ oc get nodes
NAME                    STATUS   ROLES    AGE   VERSION
master01.ocp44.fu.com   Ready    master   17h   v1.17.1
master02.ocp44.fu.com   Ready    master   17h   v1.17.1
master03.ocp44.fu.com   Ready    master   17h   v1.17.1
worker01.ocp44.fu.com   Ready    worker   16h   v1.17.1
worker02.ocp44.fu.com   Ready    worker   53m   v1.17.1
[core@bastion openshift-ansible]$ oc get co
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
authentication                             4.4.5     True        False         False      16h
cloud-credential                           4.4.5     True        False         False      18h
cluster-autoscaler                         4.4.5     True        False         False      17h
console                                    4.4.5     True        False         False      16h
csi-snapshot-controller                    4.4.5     True        False         False      16h
dns                                        4.4.5     True        False         False      17h
etcd                                       4.4.5     True        False         False      17h
image-registry                             4.4.5     True        False         False      17h
ingress                                    4.4.5     True        False         False      16h
insights                                   4.4.5     True        False         False      17h
kube-apiserver                             4.4.5     True        False         False      17h
kube-controller-manager                    4.4.5     True        False         False      17h
kube-scheduler                             4.4.5     True        False         False      17h
kube-storage-version-migrator              4.4.5     True        False         False      16h
machine-api                                4.4.5     True        False         False      17h
machine-config                             4.4.5     True        False         False      16h
marketplace                                4.4.5     True        False         False      16h
monitoring                                 4.4.5     True        False         False      16h
network                                    4.4.5     True        False         False      17h
node-tuning                                4.4.5     True        False         False      17h
openshift-apiserver                        4.4.5     True        False         False      17h
openshift-controller-manager               4.4.5     True        False         False      17h
openshift-samples                          4.4.5     True        False         False      17h
operator-lifecycle-manager                 4.4.5     True        False         False      17h
operator-lifecycle-manager-catalog         4.4.5     True        False         False      17h
operator-lifecycle-manager-packageserver   4.4.5     True        False         False      152m
service-ca                                 4.4.5     True        False         False      17h
service-catalog-apiserver                  4.4.5     True        False         False      17h
service-catalog-controller-manager         4.4.5     True        False         False      17h
storage                                    4.4.5     True        False         False      17h

```

- 이슈 정리 
```
TASK [openshift_node : Get cluster nodes]
fatal: [localhost]: FAILED! => {"msg": "The conditional check 'oc_get.stdout != ''' failed. The error was: error while evaluating conditional (oc_get.stdout != ''): 'dict object' has no attribute 'stdout'"}

체크 포인트: 작업노드에서 계정 core가 sudo 권한 있어야 한다.
```

#### 3.2.8 update operator hub catalog 
Apply the manifests

```
oc apply -f ./redhat-operators-manifests
```

CatalogSource 정의

```
vi  catalogsource.yaml
apiVersion:  operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: my-operator-catalog
  namespace: openshift-marketplace
spec:
  sourceType: grpc
  image: registry.ocp44.fu.com:5000/olm/redhat-operators:v1
  #image: <registry_host_name>:<port>/olm/redhat-operators:v1
  displayName: My Operator Catalog
  publisher: grpc
```
CatalogSource apply

```
oc create -f catalogsource.yaml
```

Verify the CatalogSource

```
# oc get pods -n openshift-marketplace
NAME READY STATUS RESTARTS AGE
my-operator-catalog-6njx6 1/1 Running 0 28s
marketplace-operator-d9f549946-96sgr 1/1 Running 0 26h

# oc get catalogsource -n openshift-marketplace
NAME DISPLAY TYPE PUBLISHER AGE
my-operator-catalog My Operator Catalog grpc 5s
```


error 정리
```
my-operator-catalog pod 에서 인증서로 인해 image pull 실패시  사용자 저의 registry의 인증서 복사 필요 함.
실패한 노에만 해당 됨
scp -r registry.ocp44.fu.com:/xxxxx/registry.ocp44.fu.com\:5000/{REGISTRY_HTTP_TLS_CERTIFICAT} worker01.ocp44.fu.com:/etc/containers/certs.d

podman login registry.ocp44.fu.com:5000 -u admin
```

## 4 OCP 설치끝 
![login access.jpg](https://i.loli.net/2020/06/18/Pz51yxgbD3LUmWn.jpg)

