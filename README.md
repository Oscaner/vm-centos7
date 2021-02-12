# vm-centos7

```shell
$ vagrant up
```

if show error - `[centos] No Virtualbox Guest Additions installation found.`

```shell
$ vagrant upload /Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso /tmp/VBoxGuestAdditions.iso
$ vagrant ssh
$ sudo su
$ yum update kernel -y
$ yum install kernel-headers kernel-devel gcc make -y
$ mkdir /mnt/vbguest
$ mount -t iso9660 -o loop /tmp/VBoxGuestAdditions.iso /mnt/vbguest/
$ sh /mnt/vbguest/VBoxLinuxAdditions.run
```
