
```
-------------------------------------------------------------------------------
● Building a Repository Server (dns)
-------------------------------------------------------------------------------
tar zcvf - /var/www/html/repos |split -b 4200M - /volumes/repo_data/repos.tar.gz
cat repos.tar.gz* | tar zxvf -

# repodata 생성
cd /var/www/html/repos/rhel-7-server-rpms
createrepo -v .
나머지 package도 같이 진행

cp local.repo /etc/yum.repos.d/
yum -y install yum-utils createrepo

# apache install
yum -y install httpd
cp -a /root/.example/repos /var/www/html/
chmod -R +r /var/www/html/repos
restorecon -vR /var/www/html

iptables -I INPUT -m state --state NEW -p tcp -m tcp --dport 80 -j ACCEPT;
iptables-save > /etc/sysconfig/iptables;
firewall-cmd --permanent --add-service=http;
firewall-cmd --reload;
```
