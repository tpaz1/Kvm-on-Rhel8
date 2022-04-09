# Perparing Red Hat Enterprise Linux 8.4 Server For KVM Installation

This procedure described how to prepare the installation for QEMU KVM on a Red hat Enterprise Linux 8.4 Server. 

## Preparing Storage For Virtual Machines 

Check how many disks you have in your server using the `lsblk -l` command: 

```bash
$ lsblk -l

NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda         8:0    0   20G  0 disk 
sda1        8:1    0    1G  0 part /boot
sda2        8:2    0   19G  0 part 
sdb         8:16   0    5G  0 disk 
sdc         8:32   0    5G  0 disk 
sr0        11:0    1  9.4G  0 rom  /run/media/test/RHEL-8-4-0-BaseOS-x86_64
rhel-root 253:0    0   17G  0 lvm  /
rhel-swap 253:1    0    2G  0 lvm  [SWAP]
```

We'll create `lvm` using the available devices, which are `sdb` and `sdc` using Cockpit. 

Go to Cockpit and use the `Storage` tab on the left. Under `Devices` choose `Volume Group`: 

![MarineGEO circle logo](/images/cockpit-devices.png "MarineGEO logo")

Now give your Volume Group a name and choose the disks that will be managed under that Volume Group, then hit the `Create` button: 

![MarineGEO circle logo](/images/choose-disks.png "MarineGEO logo")

Click that volume Group under the `Devices` tab. and then `Create New Logical Drive`: 

![MarineGEO circle logo](/images/create-logical-volume.png "MarineGEO logo")

Under the Logical Volume, click `Format` to create a filesystem on that device, choose the filesystem type for that device and give it a mountpoint: 

![MarineGEO circle logo](/images/format-lv.png "MarineGEO logo")

Make sure that this mountpoint was created on your server: 

```bash
$ df -h | grep qemu

/dev/mapper/qemu--volume--group-qemu--logical--volume   10G  104M  9.9G   2% /var/lib/libvirt/images
```

## Installing KVM Locally Using Local DVD Repository 

Mount the DVD ISO file to your machine (can be CDROM/USB): 

```bash 
$ ll /dev/sr0

brw-rw----+ 1 root cdrom 11, 0 Apr  9 11:48 /dev/sr0
```

Create a directory for mounting the ISO file, then mount the ISO to the directory you have created: 

```bash 
$ mkdir /mnt/disc

$ mount /dev/sr0 /mnt/disc/
mount: /mnt/disc: WARNING: device write-protected, mounted read-only.
```

Make sure the ISO file is actually mounted: 

```bash
$ mount /dev/sr0 /mnt/disc/

mount: /mnt/disc: WARNING: device write-protected, mounted read-only.
```

Create a local repository from the filders mounted under the ISO used before: 

```bash
$ cat <<EOF > /etc/yum.repos.d/rhel8-local.repo  

[AppStream]
name=Red Hat Enterprise Linux 8.4.0 AppStream
mediaid=None
metadata_expire=-1
enabled=1
baseurl=file:///mnt/disc/AppStream
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
gpgcheck=0
cost=500

[BaseOS]
name=Red Hat Enterprise Linux 8.4.0 BaseOS
mediaid=None
metadata_expire=-1
enabled=1
baseurl=file:///mnt/disc/BaseOS
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
gpgcheck=0
cost=500
EOF
```

MAke sure those repositories were created using `yum repolist` command: 

```bash
$ yum repolist

Updating Subscription Management repositories.
Unable to read consumer identity

This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.

repo id                                                   repo name
AppStream                                                 Red Hat Enterprise Linux 8.4.0 AppStream
BaseOS                                                    Red Hat Enterprise Linux 8.4.0 BaseOS
```

Install KVM locally on your machine using the local repos created from the ISO: 

```bash
$ yum install qemu-kvm qemu-img libvirt virt-install libvirt-client virt-manager -y
```

Start and enable the libvirtd process to start interacting with KVM: 

```bash
$ systemctl start libvirtd && systemctl enable libvirtd
```

Install the virtual machines plugin to Cockpit using the `Yum` command: 

```bash 
$ yum install -y cockpit-machines
```

## Create Storage Pool for KVM 

Head off to the `Virtual Machines` section based on the plugin you've installed in the previous stage, and create a storage pool using /var/lib/libvirt/images` logical volume: 

![MarineGEO circle logo](/images/create-storage-pool.png "MarineGEO logo")
