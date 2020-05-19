# Prepare Bare Metal Host

All the instructions (except where indicated) are assumed to be running as `root` on the bare metal host machine.

1. From a terminal on your local workstation, shell into the baremetal machine.  If your public keys were used during provisioning, you shouldn't need a password.  If you are prompted for a password, you either have the wrong machine name (typo), the DNS records have not propogated yet (so just wait anhour or so), or you didn't add your keys while provisioning (you'll have to recreate the bare metal machine).
```shell
ssh root@mycluster.example.com

```

2. Update and install some utils.
```shell
apt update
apt install -y net-tools curl nano tree wget jq nfs-common

```

3. Install the checker so that we can make sure virtualization is turned on.  Then run the command `kvm-ok`.
```shell
apt install -y cpu-checker
kvm-ok 

``` 

4. If the result of this command is NOT the following, then virtualization can not be used on this machine.  A ticket will have to be created to update the BIOS.
```shell
INFO: /dev/kvm exists
KVM acceleration can be used
```

5. Set the hostname for the machine.
```shell
hostnamectl set-hostname mycluster.example.com

```

6. Logout and then back in, verifying the new hostname in the command prompt.

## Install KVM

1. Install the KVM service, enable and start it.
```shell
apt install -y libosinfo-bin qemu qemu-kvm libvirt-bin bridge-utils virt-manager 

```

2. Enable and start the KVM service.
```shell
systemctl enable libvirtd; systemctl start libvirtd

```

3. Install the Ubuntu `uvtool` which will make some of the virtual machine operations easier to run.
```shell
apt install -y uvtool

```

4. Install the basic ubuntu image (Bionic Beaver) for `uvtool` to create the load balancer and nfs server from.
```shell
uvt-simplestreams-libvirt --verbose sync release=bionic arch=amd64

```

5. Create the key for the metal root user to use to access the VMs on the host machine command line.
```shell
ssh-keygen -b 4096 -t rsa -f ~/.ssh/id_rsa -N ""

```

6. Get the generated public key and update the [`install-config.yaml`](./files/install-config.yaml) file in this repository.
```shell
cat ~/.ssh/id_rsa.pub

```

## Set up VM network

This defines a separate network that the VMs will use. It explicitly disables the
DNS and DHCP, so that our separate instance of dnsasq can take over.

> Useful links:\
>   <a href="https://wiki.libvirt.org/page/Libvirtd_and_dnsmasq" target="_blank">Libvirtd and dnsmasq</a></br>
>   <a href="https://unix.stackexchange.com/questions/425028/libvirt-kvm-how-to-get-the-ip-address-of-virtual-machines-on-a-bridged-netwo" target="_blank">libvirt / kvm - how to get the ip address of virtual machines on a bridged network?</a>


1. Copy the network definition file to the bare metal host.
```shell
*** LOCAL MACHINE ***

scp ./files/net_ocp.xml root@mycluster.example.com:~

```

2. Create this network, start it and have it autostart.
```shell
virsh net-define net_ocp.xml
virsh net-autostart ocp
virsh net-start ocp

```

3. Restart KVM.
```shell
systemctl restart libvirt-bin

```

4. Verify existence of network with.
```shell
ifconfig  br-ocp

```

## Install DNSMasq/ipxe/tftp

On the KVM host (as root in home), install the Dnsmasq, the required tftp, and ipxe.

> Useful links:\
>   <a href="https://linuxhint.com/dnsmasq_ubuntu_server/" target="_blank">How to Configure dnsmasq on Ubuntu Server 18.04 LTS</a>

1. Install DNSMasq, enable and start it.
```shell
apt install -y dnsmasq
systemctl enable dnsmasq; systemctl start dnsmasq 

```

2. Install the iPXE server and TFTP.
```shell
apt -y install ipxe
mkdir -p /var/lib/tftp
cp /usr/lib/ipxe/{undionly.kpxe,ipxe.efi} /var/lib/tftp
chown dnsmasq:nogroup /var/lib/tftp/\*

```

3. Backup the original DNSmasq configuration file.
```shell
mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig

```

4. Edit the file [`dnsmasq`](./files/dnsmasq/dnsmasq.conf), replacing the URL at the bottom of the file with the URL of your bare metal machine.  Then copy the file to the bare metal host.
```shell
*** LOCAL MACHINE ***

scp ./files/dnsmasq/dnsmasq.conf root@mycluster.example.com:/etc/dnsmasq.conf

```

5. If this is not the first time you have edited this file, you will want to clean up any existing DHCP lease information, and reset that file with an empty one.
```shell
rm -rf /var/lib/misc/dnsmasq.leases 
touch /var/lib/misc/dnsmasq.leases 

```

7. Restart the DNSmasq service.
```shell
systemctl restart dnsmasq

```

## Configure nameservers

> Useful links: \
>     <a href="https://www.techrepublic.com/article/how-to-set-dns-nameservers-in-ubuntu-server-18-04/" target="_blank">How to set DNS nameservers in Ubuntu Server 18.04</a>


1. Edit the file `/etc/netplan/01-netcfg.yaml`.  Under the sections for `bond0` and `bond1` replace the two nameserver addresses with just one address; `127.0.0.1`.  This will make all DNS traffic going though that network interface to use the local DNSmasq server insteacd of the IBM Public Cloud assigned one. 
```shell 
nano /etc/netplan/01-netcfg.yaml

```
for example;
```
...
    bond1:
      interfaces:
        - eth1
        - eth3
      addresses: [169.47.234.183/28]
      gateway4: 169.47.234.177
      nameservers:
        addresses:
          - 127.0.0.1
...
```

2. Apply the changes made to `netplan`.
```shell
netplan apply

```

3. Edit the file `resolved.conf` so that the machine uses the local DNS server by uncommenting the property for `DNS` and setting its value to `127.0.0.1`.
```shell
nano /etc/systemd/resolved.conf

```
The section will look something like this:
```ini
...
[Resolve]
DNS=127.0.0.1
...
```

6. Restart `systemd-resolved`.
```shell
systemctl restart systemd-resolved 

```

7. Verify that only the 127.0.0.1 nameserver value is under the interface definition for the public network interface.
```shell
systemd-resolve --status

``` 
Make sure Global DNS Servers are set to teh lopback addres (127.0.0.1), at the top, and in the bond0 and bond1 sections.

## Matchbox setup

Matchbox is used to determine which images to respond with when getting a PXE request we download it, then install and configure.

1. As root in home folder (i.e. `cd ~`), download the matchbox binaries.
```shell
cd ~
wget https://github.com/poseidon/matchbox/releases/download/v0.8.3/matchbox-v0.8.3-linux-amd64.tar.gz

```

2. Unpackage it.  Copy the binary to the local bin folder.
```shell
tar zxvf matchbox-v0.8.3-linux-amd64.tar.gz
cd matchbox-v0.8.3-linux-amd64
cp matchbox /usr/local/bin

```

3. Create a special user account for matchbox, and make it the owner of the `/var/lib/matchbox` folder.  Then copy service definition to system services.
```shell
useradd -U matchbox
mkdir -p /var/lib/matchbox/{assets,groups,ignition,profiles}
cp contrib/systemd/matchbox-local.service /etc/systemd/system/matchbox.service

```

4. Enable and start matchbox servie.
```shell
systemctl enable matchbox; systemctl start matchbox

```

5. The folder `/var/lib/matchbox/assets` is where we put the files that the PXE server will dish up.  Get the RHCOS files and place them in there.
```shell
cd /var/lib/matchbox/assets
wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.3/latest/rhcos-4.3.8-x86_64-installer-initramfs.x86_64.img
wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.3/latest/rhcos-4.3.8-x86_64-installer-kernel-x86_64
wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.3/latest/rhcos-4.3.8-x86_64-metal.x86_64.raw.gz


```

6. Edit the three matchbox group files; [`bootstrap.json`](./files/matchbox/profiles/bootstrap.json), [`master.json`](./files/matchbox/profiles/master.json), [`worker.json`](./files/matchbox/profiles/worker.json) and update the hostname at the bottom of the files (two instances each).

7. Copy the matchbox group and profile files to the bare metal host.
```shell
*** LOCAL MACHINE ***

scp ./files/matchbox/groups/*.json root@mycluster.example.com:/var/lib/matchbox/groups/

scp ./files/matchbox/profiles/*.json root@mycluster.example.com:/var/lib/matchbox/profiles/

```

8. Change ownership of these files with the following command.
```shell
chown -R matchbox:matchbox /var/lib/matchbox

```

9. Clean up the matchbox files and folders.
```shell
cd ~
rm -rf matchbox-v0.8.3-linux-amd64

```

## Openshift Installer install

This is where we get the OCP installer and client programs and add to our host.  We already grabbed the RHCOS images in the previous section. 

1. Obtain the installer, and most importantly get the pull secret for your entitled instance of OCP4 follow these [instructions to get it](https://docs.openshift.com/container-platform/4.2/installing/installing_bare_metal/#installation-obtaining-installer_installing-bare-metal).  Which has you navigate to [https://cloud.redhat.com/openshift/install](https://cloud.redhat.com/openshift/install) (select BareMetal).  Copy (or download the pull secret and save it somewhere, we'll use it soon).

2. Download the client and installer into root's home on the bare metal host.
```shell
cd ~
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.3/openshift-client-linux.tar.gz

```

3. Unpackage the file, and then copy the `oc` and `kubectl` binaries to `\usr\local\bin`, and finally clean up the remaining files.
```shell
tar xvf openshift-client-linux.tar.gz
cp oc /usr/local/bin/
cp kubectl /usr/local/bin
rm openshift-client-linux.tar.gz README.md

```

4. Now download the installer and put in `/usr/local/bin`.
```shell
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.3/openshift-install-linux.tar.gz

```

5. Unpack the file, and copy the binary to the local bin folder and then clean up the remaining files.
```shell
tar xvf openshift-install-linux.tar.gz
cp openshift-install /usr/local/bin/
rm openshift-install-linux.tar.gz README.md

```


## Openshift Installer setup

We need to use the installer at this point to create the ignition files which are required to create VMs with.

1. Create a new directory to work in (delete it if it already there - to ensure a fresh set of ignition files).
```shell
cd ~
rm -rf ocp-install
mkdir -p ocp-install
cd ocp-install
```

2. Edit the [`install-config.yaml`](./files/install-config.yaml) in the files section of this Git repository.  Update the `baseDomain` (i.e. coc-ibm.com) and the cluster `metadata.name` (i.e. mycluster), the pull secret (the one you got from the RedHat site earlier), and finally the host's root public key that was generated generated on the host machine (i.e. `/root/.ssh/id_rsa.pub`).

3. Copy the edited `install-config.yaml` file to the home directory on the host.
```shell
*** LOCAL MACHINE ***

scp ./files/install-config.yaml root@mycluster.example.com:~

```

4. Back on the host machine, copy the file to the working directory.  We do this, because the Red Hat OCP installer will consume (remove) the install config file when it builds the ignition files.  By copying it to the home first, then copying it to the ocp-install directory we ensure that a copy of it exists on the host, incase we need to repeat later when troubleshooting.
```shell
cd ~/ocp-install
cp ../install-config.yaml .

```

4. Run the installer to create the ignition files.
```shell
openshift-install create ignition-configs

```

5. Remove any exisiting ignition files (from a previous install attempt), copy the files to the matchbox location, and set the permissions to read.
```shell
rm -f /var/lib/matchbox/ignition/*.ign
cp *.ign /var/lib/matchbox/ignition/
chmod +r /var/lib/matchbox/ignition/*

```


<table align="center">
<tr>
  <td align="left" width="9999"><a href="domain.md">Previous - Register and Configure Domain</a> </td>
  <td align="right" width="9999"><a href="create_vms.md">Next - Create VMs</a> </td>
</tr>
</table>
