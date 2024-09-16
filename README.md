# Palo Alto Networks VM-Series on Proxmox

## Getting Started

### Download the PAN-OS image file

Download the qcow2 image from **Palo Alto Networks Customer Support Portal** and upload the image to your Proxmox server.

*Updates -> Software Updates -> Content type: PAN-OS for VM-Series KVM Base Images*

```shell
# Transfer the image to your Proxmox server with SCP
scp PA-VM-KVM-11.2.0.qcow2 <username>@pve01.lab.internal:~

# If you have a download URL you can download the image directly to your Proxmox node
wget https://download.somewhere.com/PA-VM-KVM-11.2.0.qcow2
```

### Create the VM-Series firewall

```shell
# Access your Proxmox node
ssh <username>@pve01.lab.internal

# Change to root account
su -
```

> [!INFO]
> Update all variables to suit your needs and adjust the network interfaces to match your environment.
> 
> The Proxmox `net0` interface is paired with the Palo Alto VM's management interface. The remaining interfaces are paired one-to-one as follows:
> `net1` -> `ethernet1/1`
> 
> If you do not need this many interfaces you can remove some of them.

```bash
VM_ID=5000
VM_NAME=panw-fw01
VM_MEMORY=6144
qm create $VM_ID \
--name $VM_NAME \
--agent enabled=1 \
--machine q35 \
--bios seabios \
--numa 0 \
--cpu x86-64-v2-AES \
--ostype l26 \
--cores 2 \
--sockets 1 \
--memory $VM_MEMORY \
--serial0 socket \
--scsihw virtio-scsi-pci \
--boot order='virtio0' \
--hotplug disk,network,usb,cpu \
--net0 virtio,bridge=vmbr0,tag=10 \
--net1 virtio,bridge=vmbr0,tag=20 \
--net2 virtio,bridge=vmbr0,tag=30 \
--net3 virtio,bridge=vmbr0,tag=40
```

### Import the VM-Series disk

Update your storage path accordingly. In my home lab I am using `local-zfs` for VM storage.

```bash
# First specify where the VM should be stored - default Proxmox location is local-lvm
VM_STORAGE=local-zfs
qm importdisk $VM_ID PA-VM-KVM-11.2.0.qcow2 $VM_STORAGE --format qcow2
```

> [!NOTE]
> You may get the following message, depending on how your storage is configured:
> `format 'qcow2' is not supported by the target storage - using 'raw' instead`

### Attach the disk to your VM

```bash
qm set $VM_ID --virtio0 $VM_STORAGE:vm-$VM_ID-disk-0,discard=on,cache=writeback
```

### Start your VM

```bash
qm start $VM_ID
```

### Attach to the console to show or configure the management IP (optional)

```shell
# Default login is admin / admin
# The password must be changed on the first login

# The management interface is configured with DHCP by default
show interface management
```
