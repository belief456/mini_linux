build a minimal bootable linux

requirements:
	vmware
	running linux (centos 7 in my case)

refer: https://revcode.wordpress.com/2012/02/25/booting-a-minimal-busybox-based-linux-distro/


1:  add a new hard disk partition

	in vmware, add a hard disk to linux, assume it is /dev/sdb
	
	using fdisk or similar tools to create a primary partition /dev/sdb1, 

	mkfs.ext4 /dev/sdb1
	
	mount /dev/sdb1 /mnt

2:  build the kernel
	
	note:   
		if when booting, kernel panics with messages like "unable to mount root fs on unknown-block(0,0)",
		it seems that the kernel lacks modules to the hdd controller. Use lspci -k on the host linux to see 
		the modules in use, and reconfig/recompile the kernel. See:
		https://www.linuxquestions.org/questions/linux-from-scratch-13/unable-to-mount-root-fs-on-unknown-block-0-0-a-4175589372/

3:  busybox
	get busybox from https://busybox.net/downloads/binaries/

4:  root fs

	mkdir rootfs 
	cd rootfs
	mkdir dev proc sys tmp bin
	mknod dev/console c 5 1

	cat > init << "EOF" 
	#!/bin/ash 

	mount -t proc none /proc 
	mount -t sysfs none /sys

	/bin/ash
	EOF 

	(note: remember to trim the leading spaces when copying...)
	chmod +x init

	
	cd bin

	cp ../../busybox .
	ln -s busybox ash
	ln -s busybox ls
	...

	cd ..
	
	find . | cpio -H newc -o | gzip > ../rootfs.cpio.gz
	cd ..

	cp vmlinuz /mnt
	cp rootfs.cpio.gz /mnt

5:  grub
	
	grub-install --boot-directory=/mnt/boot /dev/sdb	

	cat > /mnt/boot/grub/grub.cfg << "EOF" 
	set timeout=5
	menuentry 'Linux LL' {
		set root=(hd0,msdos1)
		linux /bzImage
		initrd /rootfs.cpio.gz
	}
	EOF 

	(note: remember to trim the leading spaces when copying...)

	(note: if grub(2) does not boot automatically, i.e. goes to the grub cmd line,
	you can cp host linux's grub.cfg, modify it, and boot. TODO...).

6 : boot
	
	create a new virtual machine, using the previous hard disk, and enjoy!
	


