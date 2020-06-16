# Openshift4.4-offline install


## 설치 환경

### 1. 서버 구성 사항
|role|수량|OS|IP|core / memory / disk|필수패키지|비고|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
WEB|&nbsp;&nbsp;&nbsp;1&nbsp;&nbsp;&nbsp;|RHEL 7.6|10.0.10.220|2/8/100G|Haproxy
GW|1|RHEL 7.6|10.0.10.221|2/8/100G|NDS<br>LDAP
repository |1|RHEL 7.6|10.0.10.222|2/8/300G|podman<br>openshift-client<br>httpd-tools<br>mirror-registry
storage|1|RHEL 7.6|10.0.10.223|2/8/300G|NFS
bastion|1|RHEL 7.6|10.0.10.229|2/8/100G|FTP<br>openshift-client<br>openshift-install
bootstrap|1|coreOS 4.4.3|192.168.10.11|4/16/120G||마스터 노드 설치후 삭제
master|3|coreOS 4.4.3|192.168.10.21~|4/16/120G||
worker|n|coreOS 4.4.3|192.168.10.31~|2/8/120G||
worker|n|RHEL 7.6|192.168.10.31~|2/8/120G|ansible<br>openshift-clients<br>jq|ansible 설치

### 2. 서버 구성도
[클릭하여 이미지 확인 하세요](https://www.processon.com/view/link/5ee84782f346fb1ae56574a6)
<iframe src="https://www.processon.com/embed/5ee847811e085326372b8055" id="the_iframe" onload="calcHeight();" name="WrittenPublic" title="게시판뷰" frameborder="0" scrolling="no" style="overflow-x:hidden; overflow:auto; width:100%; min-height:500px;"></iframe>
