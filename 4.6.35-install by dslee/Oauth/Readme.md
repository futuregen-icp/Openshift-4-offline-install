## Definitions
- User management example
  + Admins : ocpadmin, dslee -> /opt/ocp4/install-20210623/oauth/htpasswd/ocpadmin.htpasswd
  + Users : ocpuser1, kdhong -> /opt/ocp4/install-20210623/oauth/htpasswd/ocpuser1.htpasswd

## Add  Admins

- Add user as ocpadmin, dslee for ocpadmin 
```
htpasswd -cBb /opt/ocp4/install-20210623/oauth/htpasswd/ocpadmin.htpasswd ocpadmin ocpadmin.!
htpasswd -Bb /opt/ocp4/install-20210623/oauth/htpasswd/ocpadmin.htpasswd dslee dslee.!
cat /opt/ocp4/install-20210623/oauth/htpasswd/ocpadmin.htpasswd
```

- Create secret
```
oc create secret generic htpass-secret-ocpadmin --from-file=htpasswd=/opt/ocp4/install-20210623/oauth/htpasswd/ocpadmin.htpasswd -n openshift-config
```

- Create Oauth
```
vi /opt/ocp4/install-20210623/oauth/htpasswd/oauth.cr.yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: Ocpadmin
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret-ocpadmin
```

- Login to openshift console or command using ocpadmin, user . because created user accounts as ocpadmin, dslee.

- Assign role for rbac
```
 oc adm policy add-cluster-role-to-user cluster-admin ocpadmin
 oc adm policy add-role-to-user admin ocpadmin
 oc adm policy add-cluster-role-to-user cluster-admin dslee
 oc adm policy add-role-to-user admin dslee
```

## Add users

- Add user as ocpuser1, kdhong for ocpuser
```
htpasswd -cBb /opt/ocp4/install-20210623/oauth/htpasswd/ocpuser1.htpasswd ocpuser1 ocpuser1.!
htpasswd -Bb /opt/ocp4/install-20210623/oauth/htpasswd/ocpuser1.htpasswd kdhong kdhong.!
cat /opt/ocp4/install-20210623/oauth/htpasswd/ocpuser1.htpasswd
```

- Create secret
```
oc create secret generic htpass-secret-ocpuser1 --from-file=htpasswd=/opt/ocp4/install-20210623/oauth/htpasswd/ocpuser1.htpasswd -n openshift-config
```

- Create Oauth as file update
```
vi /opt/ocp4/install-20210623/oauth/htpasswd/oauth.cr.yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: Ocpadmin
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret-ocpadmin
  - name: Ocpuser1
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret-ocpuser1
```

- Update Oauth
```
oc apply -f /opt/ocp4/install-20210623/oauth/htpasswd/oauth.cr.yaml
```

- Login to openshift console or command using ocpadmin, user . because created user accounts as ocpadmin, dslee.

- Assign role for rbac