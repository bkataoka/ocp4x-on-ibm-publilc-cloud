# Create Virtual Machines

In this section we manually create each VM in the cluster, including the bootstrap machine.  We need to do this now, in order to get the IP addresses assigned by the DHCP server, which we will use in the load balancer configuration and elsewhere.

We are going to use the volume pool `uvtool`, since it already exists (by installing `uvtool`).

1. make sure this pool exists, at this point there should only be one entry.<pre>
virsh vol-list uvtool
</pre>

2. For each VM we will create a disk volume with the `virsh vol-create-as` command.  Then create the VM with the `virt-install` command, passing in as one of the arguments the path to that volume.  But what is most important here, is that we pass in a manually created MAC address.  By pre-defining the MAC address we make it easier to modify other configuration files later.
```shell

virsh vol-create-as uvtool bootstrap.qcow2 100G
virt-install --name=bootstrap --ram=16000 --vcpus=8 --mac=52:54:00:02:85:01 \
    --disk path=/var/lib/uvtool/libvirt/images/bootstrap.qcow2,bus=virtio \
    --pxe --noautoconsole --graphics=vnc --hvm \
    --network network=ocp,model=virtio --boot hd,network

virsh vol-create-as uvtool master1.qcow2 200G
virt-install --name=master1 --ram=65536 --vcpus=16 --mac=52:54:00:02:86:01 \
    --disk path=/var/lib/uvtool/libvirt/images/master1.qcow2,bus=virtio \
    --pxe --noautoconsole --graphics=vnc --hvm \
    --network network=ocp,model=virtio --boot hd,network

virsh vol-create-as uvtool master2.qcow2 200G
virt-install --name=master2 --ram=65536 --vcpus=16 --mac=52:54:00:02:86:02 \
    --disk path=/var/lib/uvtool/libvirt/images/master2.qcow2,bus=virtio \
    --pxe --noautoconsole --graphics=vnc --hvm \
    --network network=ocp,model=virtio --boot hd,network

virsh vol-create-as uvtool master3.qcow2 200G
virt-install --name=master3 --ram=65536 --vcpus=16 --mac=52:54:00:02:86:03 \
    --disk path=/var/lib/uvtool/libvirt/images/master3.qcow2,bus=virtio \
    --pxe --noautoconsole --graphics=vnc --hvm \
    --network network=ocp,model=virtio --boot hd,network

virsh vol-create-as uvtool worker1.qcow2 200G
virt-install --name=worker1 --ram=65536 --vcpus=16 --mac=52:54:00:02:87:01 \
    --disk path=/var/lib/uvtool/libvirt/images/worker1.qcow2,bus=virtio \
    --pxe --noautoconsole --graphics=vnc --hvm \
    --network network=ocp,model=virtio --boot hd,network

virsh vol-create-as uvtool worker2.qcow2 200G
virt-install --name=worker2 --ram=65536 --vcpus=16 --mac=52:54:00:02:87:02 \
    --disk path=/var/lib/uvtool/libvirt/images/worker2.qcow2,bus=virtio \
    --pxe --noautoconsole --graphics=vnc --hvm \
    --network network=ocp,model=virtio --boot hd,network

virsh vol-create-as uvtool worker3.qcow2 200G
virt-install --name=worker3 --ram=65536 --vcpus=16 --mac=52:54:00:02:87:03 \
    --disk path=/var/lib/uvtool/libvirt/images/worker3.qcow2,bus=virtio \
    --pxe --noautoconsole --graphics=vnc --hvm \
    --network network=ocp,model=virtio --boot hd,network

virsh vol-create-as uvtool worker4.qcow2 200G
virsh vol-create-as uvtool worker4b.qcow2 10G
virsh vol-create-as uvtool worker4c.qcow2 100G
virt-install --name=worker4 --ram=65536 --vcpus=16 --mac=52:54:00:02:87:04 \
    --pxe --noautoconsole --graphics=vnc --hvm \
    --network network=ocp,model=virtio --boot hd,network \
    --disk path=/var/lib/uvtool/libvirt/images/worker4.qcow2,bus=virtio \
    --disk path=/var/lib/uvtool/libvirt/images/worker4b.qcow2,bus=virtio \
    --disk path=/var/lib/uvtool/libvirt/images/worker4c.qcow2,bus=virtio

virsh vol-create-as uvtool worker5.qcow2 200G
virsh vol-create-as uvtool worker5b.qcow2 10G
virsh vol-create-as uvtool worker5c.qcow2 100G
virt-install --name=worker5 --ram=65536 --vcpus=16 --mac=52:54:00:02:87:05 \
    --pxe --noautoconsole --graphics=vnc --hvm \
    --network network=ocp,model=virtio --boot hd,network \
    --disk path=/var/lib/uvtool/libvirt/images/worker5.qcow2,bus=virtio \
    --disk path=/var/lib/uvtool/libvirt/images/worker5b.qcow2,bus=virtio \
    --disk path=/var/lib/uvtool/libvirt/images/worker5c.qcow2,bus=virtio

virsh vol-create-as uvtool worker6.qcow2 200G
virsh vol-create-as uvtool worker6b.qcow2 10G
virsh vol-create-as uvtool worker6c.qcow2 100G
virt-install --name=worker6 --ram=65536 --vcpus=16 --mac=52:54:00:02:87:06 \
    --pxe --noautoconsole --graphics=vnc --hvm \
    --network network=ocp,model=virtio --boot hd,network \
    --disk path=/var/lib/uvtool/libvirt/images/worker6.qcow2,bus=virtio \
    --disk path=/var/lib/uvtool/libvirt/images/worker6b.qcow2,bus=virtio \
    --disk path=/var/lib/uvtool/libvirt/images/worker6c.qcow2,bus=virtio

```

3. Monitor VMs.  Eventually all these VMs will shutdown themselves, so that the next time they have started the CoreOS will be loaded.  During this intial run, a simple in memory OS is running, formatting the disks and such so that the next time the VM starts it will be a RHCOS machine.
```shell
watch "virsh list --all"

````

4. When all the VMs have shut down, you can stop the watch (Ctrl-C).  If the machines don't shut down within 2 minutes somethin is probably wrong.  If it takes 5 minutes or longer to shutdown something is definitely wrong.  You should probably review your previous steps, before trying again.


<table align="center">
<tr>
  <td align="left" width="9999"><a href="prepare_host.md">Previous - Prepare Host</a> </td>
  <td align="right" width="9999"><a href="load_balancer.md">Next - Load Balancers</a> </td>
</tr>
</table>
