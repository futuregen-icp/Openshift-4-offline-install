﻿### haproxy error

```
[root@fu yghong]#
Message from syslogd@localhost at Jun 21 13:57:38 ...
 haproxy[6290]: backend yo4_ocp3_api has no server available!

Message from syslogd@localhost at Jun 21 13:57:39 ...
 haproxy[6293]: backend yo4_ocp3_mc has no server available!

Message from syslogd@localhost at Jun 21 13:57:39 ...
 haproxy[6293]: backend yo4_ocp3_http has no server available!

Message from syslogd@localhost at Jun 21 13:57:40 ...
 haproxy[6293]: backend yo4_ocp3_https has no server available!

Message from syslogd@localhost at Jun 21 13:57:41 ...
 haproxy[6293]: backend nas-http has no server available!

Message from syslogd@localhost at Jun 21 13:57:41 ...
 haproxy[6293]: backend nas-https has no server available!

[root@fu yghong]#

```


### check 

```

- 3자와 동시간때 작업이 진행되면서 충돌 의심 
-haporxy에 acl과 backend 값을 넣으면서 재원이 틀려서일수도 있음

```

### soluthion

```
- 재원 확인(acl, backend IP) 후 재설정 

```
