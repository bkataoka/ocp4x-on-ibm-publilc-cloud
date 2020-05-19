# Install 

In this section the actual OCP install happens.

1. Change to the `ocp-install` folder you created when the ignition files were first created.<pre>
cd ~/ocp-install
</pre>

2. Run the installer.<pre>
openshift-install --dir=. wait-for install-complete --log-level=debug
</pre>

You should see the debug logs, and eventually they will finish with a URL, user and pwd that you can use:

```
DEBUG OpenShift Installer 4.3.12
DEBUG Built from commit db4411451af55e0bab7258d25bdabd91ea48382f
DEBUG Fetching Install Config...
DEBUG Loading Install Config...
DEBUG   Loading SSH Key...
DEBUG   Loading Base Domain...
DEBUG     Loading Platform...
DEBUG   Loading Cluster Name...
DEBUG     Loading Base Domain...
DEBUG     Loading Platform...
DEBUG   Loading Pull Secret...
DEBUG   Loading Platform...
DEBUG Using Install Config loaded from state file
DEBUG Reusing previously-fetched Install Config
INFO Waiting up to 30m0s for the cluster at https://api.mycluster.example.com:6443 to initialize...
DEBUG Still waiting for the cluster to initialize: Working towards 4.3.12: 97% complete
DEBUG Still waiting for the cluster to initialize: Working towards 4.3.12: 97% complete, waiting on authentication, console, monitoring, openshift-samples
DEBUG Still waiting for the cluster to initialize: Working towards 4.3.12: 97% complete, waiting on authentication, console, monitoring, openshift-samples
DEBUG Still waiting for the cluster to initialize: Working towards 4.3.12: 99% complete
DEBUG Still waiting for the cluster to initialize: Working towards 4.3.12: 100% complete, waiting on authentication, monitoring
DEBUG Still waiting for the cluster to initialize: Working towards 4.3.12: 100% complete, waiting on authentication, monitoring
DEBUG Still waiting for the cluster to initialize: Working towards 4.3.12: 100% complete, waiting on authentication, monitoring
DEBUG Cluster is initialized
INFO Waiting up to 10m0s for the openshift-console route to be created...
DEBUG Route found in openshift-console namespace: console
DEBUG Route found in openshift-console namespace: downloads
DEBUG OpenShift console route is created
INFO Install complete!
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/root/ocp-install/auth/kubeconfig'
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.mycluster.example.com
INFO Login to the console with user: kubeadmin, password: 2ZDmp-PiaUG-YRMjS-PXk8Q
```

If the installation is complete, you can verify by logging into the console with the URL and credentials given.

3. Make sure that the kubeconfig is set and optionally set the default editor to nano.
```shell
export KUBECONFIG=/root/ocp-install/auth/kubeconfig
export EDITOR=nano

```

<table align="center">
<tr>
  <td align="left" width="9999"><a href="bootstrap.md">Previous - Bootstrap</a> </td>
  <td align="right" width="9999"><a href="nfs.md">Next - NFS</a> </td>
</tr>
</table>
