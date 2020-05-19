# NFS

In this section we create an NFS server VM and use it for persistent storage in the cluster.

> Useful Links:\
> <a href="https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-18-04">How To Set Up an NFS Mount on Ubuntu 18.04</a>

1. First create a small VM running Ubuntu 18.04, name it `nfs-server`, and assign it at least 400GB of disk space.
```shell
cd ~
uvt-kvm create nfs-server release=bionic --memory 4096 --cpu 4 --disk 400 --bridge br-ocp 
 
```

2. Wait a minute or so for the virtual machine to create, then check the leases to see what IP address it has been assigned.
```shell
cat /var/lib/misc/dnsmasq.leases 

```

3. There will be a new line in the leases (with a long client-id value).  Note the assigned IP address.
```shell
1580550335 52:54:00:d1:23:c2 192.168.10.128 ubuntu ff:b5:5e:67:ff:00:02:00:00:ab:11:a8:3a:e0:01:6e:9c:5f:ff

```

4. Log into the VM, and switch to the super user account (root).
```shell
ssh ubuntu@192.168.10.128
ubuntu@nfs-server:~$ sudo su

```

5. While in the VM as root, install the NFS server components.
```shell
apt install -y nfs-kernel-server

```

6. Make a directory to use as the file share (`/var/nfs/general`). Change the owner to be `nobody`.
```shell
mkdir /var/nfs/general -p
chown nobody:nogroup /var/nfs/general

```

7. Edit the `exports` file.
```shell
nano /etc/exports

```

8. Add the following line to `exports` file.  Then save the file.
```shell
/var/nfs/general    *(rw,sync,no_subtree_check,no_root_squash)

```

9. Restart the NFS server.
```shell
systemctl restart nfs-kernel-server

```

10. Exit super user, and logout of VM.
```shell
exit
logout

```

## Configuring NFS as a Managed Storage Class

In this section we configure the NFS to be dynamically provisionable in the cluser.

> Useful links:\
>  <a href="https://medium.com/faun/openshift-dynamic-nfs-persistent-volume-using-nfs-client-provisioner-fcbb8c9344e
" target="_blank">openshift dynamic NFS persistent volume using NFS-client-provisioner</a>

1. As root on the bare metal host machine, create a test mount point for the NFS share.
```shell
mkdir /var/nfsshare

```

2. Mount the volume using the IP address of the nfs-server VM created earlier.
```shell
mount -t nfs 192.168.10.128:/var/nfs/general /var/nfsshare

```

3. In the home directory of the root user (`~`), create a working directory called `nfs`.
```shell
cd ~
mkdir nfs 
cd nfs
```

4. On your local machine edit the [`deployment.yaml`](./files/nfs/deployment.yaml) file. Replace `<< IP ADDRESS OF NFS SERVER VM>>` with the actual IP address of the NFS server VM.

5. From your **local machine** copy all the `.yaml` files in the `files/nfs/` folder to the `~/nfs` folder on the bare metal host.
```shell
*** LOCAL MACHINE ***

scp ./files/nfs/*.yaml root@mycluster.example.com:~/nfs

```

6. Back on the bare metal host, as root, create a new OCP project called `nfs-fs`.
```shell
oc new-project nfs-fs

```

7. Create the service account, roles and role bindings. 
```shell
oc create -f rbac.yaml

```

8. Update the security policies.
```shell
oc adm policy add-scc-to-user hostmount-anyuid system:serviceaccount:nfs-fs:nfs-client-provisioner

```

9. Create the storage class and deployment.
```shell
oc create -f class.yaml
oc create -f deployment.yaml

```

10. Verify the creation of the storage class.
```shell
oc get sc

```

11. Patch the storage class to make it the default storage class.
```shell
oc patch storageclass managed-nfs-storage -p '{"metadata": {"annotations":  {"storageclass.kubernetes.io/is-default-class": "true"}}}'

```

12. Verify the storage class is default (will have a `(default)` after its name).
```shell
oc get sc

```

13.  Now lets test the storage class with a simple claim.
```shell
oc create -f test-claim.yaml

```

14. List the PVC, and verify it has been created. This may take a minute or two the first time.
```shell
oc get pvc

```

15. Once the PVC is created, we can test it with a simple pod that will create an empty file called `SUCCESS` in it.
```shell
oc create -f test-pod.yaml

```

16. Verify the pod has completed.
```shell
oc get pod

```

17. Verify that a directory got created in the NFS share.  Look in that directory for a file called `SUCCESS`.
```shell
ls /var/nfsshare/*

```

18. If the file is there, the all is well and we can clean up the test objects.
```shell
oc delete -f test-pod.yaml
oc delete -f test-claim.yaml

```

## Update OpenShift Image Registry

*Special thanks to Philippe Thomas for figuring out this part.*

1. Create a PVC for the image registry.
```shell
oc create -f  nfspvc.yaml 

```

2. Edit the image registry configuration.
```shell
oc edit configs.imageregistry.operator.openshift.io/cluster

```
Change the value of  `managementState` from `Removed` to `Managed`.  Next change the value of `storage` from `{}` to
```yaml
  storage:
    pvc:
      claim: nfspvc

```
Save the file.

3. Monitor the change by watching the cluster operators.  The image registry operator will change from False to True.
```shell
watch "oc get clusteroperators"

```

<table align="center">
<tr>
  <td align="left" width="9999"><a href="install.md">Previous - Install</a> </td>
  <td align="right" width="9999"><a href="users.md">Next - Users</a> </td>
</tr>
</table>
