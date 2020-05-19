# Load Balancer

In this section we create the load balancer, after we update the DNSMAsq configuration.

## Update DNSMasq configuration file

1. Edit the file [`cluster.conf`](./files/dnsmasq/cluster.conf), updating the URLs for all the entries.

2. Copy the updated file to the bare metal host.
```shell
*** LOCAL MACHINE ***

scp ./files/dnsmasq/cluster.conf root@mycluster.example.com:/etc/dnsmasq.d/cluster.conf

```

3. Restart the dnsmasq service.
```shell
systemctl restart dnsmasq

```


## LoadBalancer on Host

In this section we will install and configure a load balancer (GoBetween) that will accept all traffic from outside and send it to either the master or worker nodes, based on port.  Any request to port 6443 (and during bootstrap process port 22623) will get distributed to the master nodes.  Traffic to ports 80 and 443 will get sent to the worker nodes.

1. Change to the root home directory on the bare metal hub, and download the load balancer component.
```
cd ~
wget https://github.com/yyyar/gobetween/releases/download/0.7.0/gobetween_0.7.0_linux_amd64.tar.gz

```

2. Create a temporary directory to work in.  Then change to it, and unpack the files into it.  Finally, copy the binary to the VM's `/usr/local/bin/` folder.
```
mkdir ~/gobetween
cd ~/gobetween
tar xvf ../gobetween_0.7.0_linux_amd64.tar.gz
cp gobetween /usr/local/bin/

```

7. Clean up the temporary folder and remove the doanloaded file.
```
cd ~
rm -rf gobetween gobetween_0.7.0_linux_amd64.tar.gz

```

8. Copy the service definition file for the load balancer from the local workstation to the bare metal host.
```shell
*** LOCAL MACHINE ***

scp ./files/gobetween/gobetween.service root@mycluster.example.com:/etc/systemd/system/gobetween.service

```

9. Make a directory for the gobetween configuration file.
```
mkdir -p /etc/gobetween

```

10. Copy the GoBetween configuration file [`gobetween.toml`](./files/gobetween/gobetween.toml) from the local machine to the bare metal host.
```shell
*** LOCAL MACHINE ***

scp ./files/gobetween/gobetween.toml root@mycluster.example.com:/etc/gobetween/gobetween.toml

```

12. Enable and start the service
``` 
systemctl enable gobetween; systemctl start gobetween

```



<table align="center">
<tr>
  <td align="left" width="9999"><a href="create_vms.md">Previous - Create VMs</a> </td>
  <td align="right" width="9999"><a href="bootstrap.md">Next - Bootstrap</a> </td>
</tr>
</table>
