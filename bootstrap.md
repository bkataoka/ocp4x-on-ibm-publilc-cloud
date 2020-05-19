# Bootstrap

1. start up the VMs. 
```shell
virsh start bootstrap
virsh start master1
virsh start master2
virsh start master3
virsh start worker1
virsh start worker2
virsh start worker3
virsh start worker4
virsh start worker5
virsh start worker6

```



> At this poing it might be usful to open up another terminal window, log into the bare metal hub, and watch the system logs.  You can do this with the command `watch "tail /var/log/syslog"`. \
> Look for errors.  A few are expected initially, but recurring errors should not appear.  If they do, investigate.

2. Before we can install OCP we need to bootstrap the system.  Starting from the ocp-install working directory.
```shell
cd ~/ocp-install

```

3. Kick off the bootstrap (the step prior to install).
```shell 
openshift-install --dir=. wait-for bootstrap-complete --log-level=debug

```

5. You should see some activity like the following.  It will take about 15 min for api server to respond, then another 20 min for the bootstrap to finish.

```shell
DEBUG OpenShift Installer 4.3.18
DEBUG Built from commit db4411451af55e0bab7258d25bdabd91ea48382f
INFO Waiting up to 30m0s for the Kubernetes API at https://api.mycluster.example.com:6443...
DEBUG Still waiting for the Kubernetes API: Get https://api.blue.coc-ibm.com:6443/version?timeout=32s: EOF
DEBUG Still waiting for the Kubernetes API: an error on the server ("") has prevented the request from succeeding
DEBUG Still waiting for the Kubernetes API: the server could not find the requested resource
DEBUG Still waiting for the Kubernetes API: the server could not find the requested resource
DEBUG Still waiting for the Kubernetes API: the server could not find the requested resource
DEBUG Still waiting for the Kubernetes API: the server could not find the requested resource
DEBUG Still waiting for the Kubernetes API: the server could not find the requested resource
DEBUG Still waiting for the Kubernetes API: the server could not find the requested resource
DEBUG Still waiting for the Kubernetes API: the server could not find the requested resource
DEBUG Still waiting for the Kubernetes API: the server could not find the requested resource
DEBUG Still waiting for the Kubernetes API: Get https://api.mycluster.example.com:6443/version?timeout=32s: EOF
INFO API v1.16.2 up
INFO Waiting up to 30m0s for bootstrapping to complete...
DEBUG Bootstrap status: complete
INFO It is now safe to remove the bootstrap resources
```

6. When the bootstrap is completed, we don't need the bootstrap VM anymore so we can remove it from the load balancer (gobetween) configuration, and DNSmasq configuration files.  First remove the bootstrap entries from the DNSMasq configuration file, and then restart the service
```shell
nano /etc/dnsmasq.d/cluster.conf 
systemctl restart dnsmasq

```

7. Edit the GoBetween configuration file (remove bootstrap entries), then restart the service.
```shell
nano /etc/gobetween/gobetween.toml
systemctl restart gobetween

```


<table align="center">
<tr>
  <td align="left" width="9999"><a href="load_balancer.md">Previous - Load Balancers</a> </td>
  <td align="right" width="9999"><a href="install.md">Next - Install</a>  </td>
</tr>
</table>
