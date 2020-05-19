# OpenShift Cluster Storage

In this section we will configure OpenShift Cluster Storage which will give us Ceph file and block storage (a requirement for some cloud paks).

> Useful links:\
>   <a href="https://access.redhat.com/documentation/en-us/red_hat_openshift_container_storage/4.2/html/deploying_openshift_container_storage/deploying-openshift-container-storage#installing-openshift-container-storage-operator-using-the-operator-hub_rhocs" target="_blank">Deploying OpenShift Container Storage</a></br>
>   <a href="https://blog.openshift.com/ocs-4-2-in-ocp-4-2-14-upi-installation-in-rhv/" target="_blank">OCS 4.2 in OCP 4.2.14 - UPI installation in RHV</a>

## Local Storage

1. On you local machine edit the files `local-storage-block` and `local-storage-filesystem` located in the `files/storage/` folder of the project.  Update all instances of the cluster domain (i.e. replace `mycluster.example.com` with your actual domain name).

2. On the bare metal host create a working directory called `storage` and change to it.
```shell
mkdir ~/storage
cd ~/storage

```

3. Copy all the yaml files from your local machine to the bare metal host.
```shell
*** LOCAL MACHINE ***

scp ./files/storage/*.yaml root@mycluster.example.com:~/storage

```

4. On the bare metal host create a new project called `local-storage`.
```shell
oc new-project local-storage

```

5. Login into the OpenShift user interface.  The URL was provided to you at the completion of the installation step (i.e. `https://console-openshift-console.apps.mycluster.example.com`).  Select the `local` identity provider at the initial login, and use the password that you created when creating the users.

6. Go to the Operator Hub (**Operators > Operator Hub**).  Filter the operators with the keyword `local`.  Select the **Local Storage** operator.  Press the Install button.

7. Select the option to use a specific namespace, and select the `local-storage` namespace that you just created.  Select the latest stable version (i.e. v4.4).  Press the Subscribe button.

8. In the bare metal host terminal (and in the working storage directory), create an instance of local storage for the filesystem type.
```shell
oc create -f local-storage-filesystem.yaml

```

9. Wait for all the pods to finish creating (will take a few minutes).  You can monitor them with the commadnd:
```shell
watch "oc get po"

```

10. Verify the storage class was created.
```shell
oc get sc

```

11. Verify the creation of the persistent volume.
```shell
oc get pv

```

12. Create the class and volume to be used for block storage.
```shell
oc create -f local-storage-block.yaml

```

13. Verify the storage class and persistent volume was created.
```shell
oc get sc
oc get pv

```

## Openshift Cluster Storage

1. Label the nodes so that the storage components can be deployed to them.
```shell
oc label nodes worker4.mycluster.coc-ibm.com cluster.ocs.openshift.io/openshift-storage=""  --overwrite=true
oc label nodes worker4.mycluster.coc-ibm.com topology.rook.io/rack=rack1 --overwrite=true

oc label nodes worker5.mycluster.coc-ibm.com cluster.ocs.openshift.io/openshift-storage=""  --overwrite=true
oc label nodes worker5.mycluster.coc-ibm.com topology.rook.io/rack=rack2 --overwrite=true

oc label nodes worker6.mycluster.coc-ibm.com cluster.ocs.openshift.io/openshift-storage=""  --overwrite=true
oc label nodes worker6.mycluster.coc-ibm.com topology.rook.io/rack=rack3 --overwrite=true


```

2. Create a namespace/project for cluster storage.
```shell
oc create -f rhocs-namespace.yaml

```
3. Use the user interface to install the Red Hat Storage operator.  Go to the operator hub and filter by 'storage`.  Select the operator and press Install.  Select the option to install in a specific namespace, and select `openshift-storage`.  Select version 4.3 of the operator.  Press Subscribe.


4. Create the operator group.
```shell
oc create -f rhocs-operatorgroup.yaml

```

5. In the bare metal host terminal, change to the `openshift-storage` project.
```shell
oc project openshift-storage

```

6. Check to make sure the pod is running.
```shell
watch "oc get po"

```

7. Wait for all the pods to get into the Running state.
```shell
watch "oc get po"

```

9. Create the cluster storage service. 
```shell
oc create -f ocs-cluster-service.yaml

```

10. It will take some time for all the pods to come up running (about 15 minutes). You can monitor thier progress with:
```shell
watch "oc get po"

```



<table align="center">
<tr>
  <td align="left" width="9999"><a href="users.md">Previous - Users</a> </td>
  <td align="right" width="9999"><a href="overview.md">Back to Overview</a></td>
</tr>
</table>
