TinyCrossLinux
==============

> Be careful!
>
> I am currently updating lots of stuff, this currently means bootloader generation is broken, which results in unbootable ISO images on UEFI. The BIOS ISO works. The resulting rootfs directory can still be used to build an initramfs or chroot into.

This repository contains scripts, build definitions and patches to build a minimal linux distribution running from initramfs. It's primary purpose is to teach the minimal requirements of a working linux system. If you just run our scripts you will end up with a small linux (between 5MB and 50MB) that allows you to login via SSH. Your phantasy and some scripts will expand this linux to a great deployment system or a platform for forensic analysis.

Our TinyCrossLinux is initially based on "Cross Linux from Scratch" (embedded). It uses an initramfs instead of a classic root file system. The boot scripts use a layout that is inspired by Berkeley Unix (BSD). Boot the system and take a look at `/etc/rc.d`. The scripts here will be executed in the order they appear (they will be called with the argument "start"). At shutdown the scripts will be called in reverse order (with argument stop).

Scripts in this repository might be eventually moved to LessLinux' build process (I feel tempted not do to so to keep it simple). Currently only x86_64 is supported. ia32 and armhf (Raspberry Pi) will follow, but are of no major priority.

What is in stage01?
--------

Scripts to build the toolchain itself. You probably will not need to add/modify scripts here.

What is in stage02?
--------

Scripts to build binaries for the target filesystem. Add as you want, but do not forget that you are cross compiling. Define `PKGNAME=...` and `PKGVERSION=...` since those are read by the script `build_stage0n.sh`. 

What is in stage02.extras?
--------

These are scripts for tools and libraries that are not part of the base system. Softlink scripts of software you want to include to stage02. But be warned, you have to manually track dependencies, for example ntfs-3 needs fuse.

Building everything!
-------------

Script to assemble a bootable ISO image (isohybrid, might be dd'ed to an USB thumb drive). Currently BIOS only, UEFI will follow. Feel free to ad an overlay or similar sick things to this script - it is less than 40 lines anyway.

So to build TinyCrossLinux run

	bash build_stage0n.sh # compiles everything
	bash build_iso.sh     # builds two ISO images
	                      # one for BIOS - one for UEFI
	
## FAQ

### Build packages with dependencies to linux-headers

I move the Linux headers from "include/linux" to "include/kernel/linux" to prevent buildscripts finding them. The simple reason for this is that some programs are not aware which structs are provided by musl and which by the kernel. In case you need the kernel headers, first try with gcc -I path/to/kernel. If this fails, take a look at the busybox build script: there I copy a subset of the kernel headers and simple touch a few others. 

### Start services

Currently the following services provided by BusyBox or dropbearmulti are available. They are inactive by default, so use the bootparameters specified to enable them:

 * SSHD `sshd=1` - you might want to add a permanent key to `/etc/dropbear`
 * HTTPD `httpd=1` - searches for files in `/srv/www`
 * TFTPD `tftpd=1` - serves files from `/srv/tftp` 
 * UDHCPD `udhcpd=1` - starts only in `/etc/udhcpd.conf` is present

### Login with password?

Run the command
	
	openssl passwd -1
	
on an arbitary machine to create a password hash and replace the '!' in roots line in `patches/etc-shadow` with the output.

### Remove login without password?

Take a look at `patches/etc-inittab`. Replace lines like

	ttyN::respawn:/bin/ash
	
by

	ttyN::respawn:/sbin/getty -l /bin/login 38400 ttyN

### I want to add more services!

The build script `stage02/0001_basefiles.sh` installs some of the startup scripts. Many of them are copied from the `patches directory`. You might want to create `stage02/0002_mybasefiles.sh`. Or you just expand the basefiles script.

### Do you plan to add features?

This linux distribution will stay small and compact, thus I do not plan to add many features. I will add some build scripts to further expand the system for some rescue and forensic purposes:

 * ddrescue
 * ntfs-3g (ntfsclone etc.)
 
Changes to the boot scripts that will be eventually made:

 * more flexible network configuration (allow specification of static IP via boot command line)
 * more flexible module loading (blacklisting and allow to specify additional modules via boot command line)
 
I will also write a script to select certain modules and subdirectories from the kernels module tree to be able to reduce the size of the resulting ISO image without having to reconfigure the kernel.

### Are there bugs?

Probably there are some. I already ran into those:

 * You have to boot with a connected ethernet cable, otherwise udhcpc will try nearly forever to get a lease.
 * Some firmware is missing - I will provide you with a script to automatically install firmware for the linux-firmware collection for kernel modules installed, in the meantime add those blobs by hand.
