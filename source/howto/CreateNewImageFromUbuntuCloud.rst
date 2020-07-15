=============================
CreateNewImageFromUbuntuCloud
=============================

Documentations coming soon

MT

source/howto/CreateNewImageFromUbuntuCloud.rst

.. code-block:: bash

wget -o /var/lib/libvirt/boot/bionic-mini.iso \
	http://archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/current/images/netboot/mini.iso

chown libvirt-qemu:kvm /var/lib/libvirt/boot/bionic-mini.iso

qemu-img create -f qcow2 /var/lib/libvirt/images/bionic.qcow2 10G

chown libvirt-qemu:kvm /var/lib/libvirt/images/bionic.qcow2

virt-install --virt-type kvm --name bionic --ram 1024 \
	--cdrom=/var/lib/libvirt/boot/bionic-mini.iso \
	--disk /var/lib/libvirt/images/bionic.qcow2,bus=virtio,size=10,format=qcow2 \
	--network network=default \
	--graphics vnc,listen=0.0.0.0 --noautoconsole \
	--os-type=linux --os-variant=ubuntu18.04

######
References
#######
https://docs.openstack.org/heat/queens/template_guide/hot_guide.html

TESDT

