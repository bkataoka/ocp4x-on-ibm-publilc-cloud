# Users

For now we will just use Apache htaccess file for users.


1. Starting from the root home on the bare metal host, install the Apache utils.
```shell
cd ~
apt install -y apache2-utils

```

2. Create a password file with the user `admin` and an appropriate password.
```shell
htpasswd -cBb htpasswd.txt admin PassW0rd

```

3. You can add additional users with the commands:
```shell
htpasswd -b htpasswd.txt user1 passW1erd
htpasswd -b htpasswd.txt user2 PassWor1d

```

4. Create a secret called `htpass-secret` from the password file.
```shell
oc create secret generic htpass-secret  --from-file=htpasswd=htpasswd.txt -n openshift-config
```

5. Edit the OAuth configuration resource with the command. 
```shell
oc edit OAuth cluster

```

6. Scroll down in the file and paste the following contents after the `spec.identityProviders` property, then save the it and exit.
```
spec:
  identityProviders:
  - name: local 
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret

```

7. Make the user (and/or all the users) an admin user with the following command:
```shell
oc adm policy add-cluster-role-to-user cluster-admin admin

```


<table align="center">
<tr>
  <td align="left" width="9999"><a href="nfs.md">Previous - NFS</a> </td>
  <td align="right" width="9999"><a href="oc_cluster_storage.md">Next - OCP Cluster Storage</a></td>
</tr>
</table>
