title: apt-deb 相关问题记录
date: 2017-12-14 10:42:00
toc: true
tags: [ubuntu, deb, apt]
categories: linux
keywords: [ubuntu, linux, deb, apt, package, control, dependency, dependencies, build dependency, source.list, build-dep, backports, 编译依赖, 查看, /boot, full, 满]
description: 记录 apt / deb 使用过程中的各种问题及解决方法。
---

##  修改源

## 查看依赖关系

* 编译依赖，`apt-rdepends --build-depends package_name`

## 问题记录
### apt-get -y build-dep linux 失败

log：

```
root@c60c282d2d76:/# apt-get -y build-dep linux
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Build-Depends dependency for linux cannot be satisfied because candidate version of package debhelper can't satisfy version requirements
```

分析：debhelper 的版本满足不了编译 linux 的依赖。

* 通过 `apt-rdepends --build-depends linux > linux.build-depends.txt` 查看 linux 的编译依赖。linux 的编译依赖好多，要等很长时间。
* 通过 `cat linux.build-depends.txt | grep debhelper` 确认对 debhelper 的版本依赖发现有软件对其的版本依赖 >= 10，而 debian jessie 的默认 debhelper 版本为 9.20xxx，在 repo 中可以看到 debhelper 当前最新版本为 10.10.9。

```
root@c60c282d2d76:/# cat linux.build-depends.txt | grep ">= 10"  
  Build-Depends: debhelper (>= 10~)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10~)
  Build-Depends: debhelper (>= 10~)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
```

* 查看 debhelper 所有版本

通过 `apt-cache madison debhelper` 查看 debhelper 的所有版本

```
root@c60c282d2d76:/# apt-cache madison debhelper
 debhelper | 10.2.5~bpo8+1 | http://ftp.cn.debian.org/debian/ jessie-backports/main amd64 Packages
 debhelper | 9.20150101+deb8u2 | http://ftp.cn.debian.org/debian/ jessie/main amd64 Packages
 debhelper | 9.20150101+deb8u2 | http://security.debian.org/ jessie/updates/main amd64 Packages
 debhelper | 9.20150101+deb8u2 | http://ftp.cn.debian.org/debian/ jessie/main Sources
 debhelper | 10.2.5~bpo8+1 | http://ftp.cn.debian.org/debian/ jessie-backports/main Sources
 debhelper | 9.20150101+deb8u2 | http://security.debian.org/ jessie/updates/main Sources
```

尝试安装 debhelper 版本 10.2.5~bp08+1，发现有新的依赖关系问题：

```
root@c60c282d2d76:/# apt-get install debhelper=10.2.5~bpo8+1
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 debhelper : Depends: dh-autoreconf (>= 12~) but 10 is to be installed
             Depends: dh-strip-nondeterminism (>= 0.028~) but 0.003-1 is to be installed
E: Unable to correct problems, you have held broken packages.
root@c60c282d2d76:/# 
```

发现比较高版本的包都是来自 jessie-backports:

> [Backports](https://wiki.debian.org/Backports): Backports are recompiled packages from testing (mostly) and unstable (in a few cases only, e.g. security updates), so they will run without new libraries (wherever it is possible) on a stable Debian distribution. It is recommended to pick out single backports which fit your needs, and not to use all backports available.

即，Backports 存放的软件为不稳定或仅用于测试目的的编译结果。

```
root@c60c282d2d76:/# apt-cache madison dh-autoreconf        
dh-autoreconf |  12~bpo8+1 | http://ftp.cn.debian.org/debian/ jessie-backports/main amd64 Packages
dh-autoreconf |         10 | http://ftp.cn.debian.org/debian/ jessie/main amd64 Packages
dh-autoreconf |         10 | http://ftp.cn.debian.org/debian/ jessie/main Sources
dh-autoreconf |  12~bpo8+1 | http://ftp.cn.debian.org/debian/ jessie-backports/main Sources
root@c60c282d2d76:/# apt-cache madison dh-strip-nondeterminism 
dh-strip-nondeterminism | 0.034-1~bpo8+1 | http://ftp.cn.debian.org/debian/ jessie-backports/main amd64 Packages
dh-strip-nondeterminism |    0.003-1 | http://ftp.cn.debian.org/debian/ jessie/main amd64 Packages
strip-nondeterminism |    0.003-1 | http://ftp.cn.debian.org/debian/ jessie/main Sources
strip-nondeterminism | 0.034-1~bpo8+1 | http://ftp.cn.debian.org/debian/ jessie-backports/main Sources
root@c60c282d2d76:/# 
```

OK，是时候把 Backports 的源踢掉了。

故障时的源，被 [163 的 Debian镜像使用帮助](http://mirrors.163.com/.help/debian.html)误导了。

```
deb http://mirrors.163.com/debian/ jessie main non-free contrib
deb http://mirrors.163.com/debian/ jessie-updates main non-free contrib
deb http://mirrors.163.com/debian/ jessie-backports main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie-updates main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie-backports main non-free contrib
deb http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
deb-src http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
```

在调试的时候，一开始已注意到可能是源的问题，但是还是一遍一遍傻逼式地试不同的源，比如上面的 debian 官方中国源 ftp.cn.debian.org。

**烂习惯。。。。。调东西还是乱试一通，没去找根本原因。。。**
教训是一个下午没了，正常已经半个小时就可以发现根本原因了。

### /boot 目录满了，无法增删内核 image

如果 /boot 目录满了，无法正常地 apt-get intall / remove / purge 等操作。解决方法如下，来自：

* https://gist.github.com/ipbastola/2760cfc28be62a5ee10036851c654600
* https://askubuntu.com/questions/585736/cant-clean-a-full-boot-because-of-unmet-dependencies, msa 的回答

解决步骤:

* sudo apt autoremove --purge
* sudo apt autoremove
* sudo apt-get -f install

异常 log：

```
jeromesun@km:~$ df -h
Filesystem                   Size  Used Avail Use% Mounted on
...
/dev/sda1                    472M  468M     0 100% /boot
...
jeromesun@km:~$ 
jeromesun@kmc-b0232:~$ sudo apt-get autoremove 
[sudo] password for jeromesun: 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
You might want to run 'apt-get -f install' to correct these.
The following packages have unmet dependencies:
 linux-image-extra-4.13.0-37-generic : Depends: linux-image-4.13.0-37-generic but it is not installed
 linux-image-generic-hwe-16.04 : Depends: linux-image-4.13.0-37-generic but it is not installed
E: Unmet dependencies. Try using -f.
jeromesun@km:~$ sudo apt-get -f
E: Command line option 'f' [from -f] is not understood in combination with the other options.
jeromesun@km:~$ 
jeromesun@km:~$ sudo apt-get -f install
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Correcting dependencies... Done
The following packages were automatically installed and are no longer required:
...
  linux-headers-4.10.0-37 linux-headers-4.10.0-37-generic linux-headers-4.10.0-40 linux-headers-4.10.0-40-generic linux-headers-4.10.0-42 linux-headers-4.10.0-42-generic linux-headers-4.13.0-26
  linux-headers-4.13.0-26-generic linux-headers-4.13.0-31 linux-headers-4.13.0-31-generic linux-image-4.10.0-37-generic linux-image-4.10.0-40-generic linux-image-4.10.0-42-generic
  linux-image-4.13.0-26-generic linux-image-4.13.0-31-generic linux-image-extra-4.10.0-37-generic linux-image-extra-4.10.0-40-generic linux-image-extra-4.10.0-42-generic linux-image-extra-4.13.0-26-generic linux-image-extra-4.13.0-31-generic 
...  
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  linux-image-4.13.0-37-generic
Suggested packages:
  fdutils linux-tools
The following NEW packages will be installed:
  linux-image-4.13.0-37-generic
0 upgraded, 1 newly installed, 0 to remove and 205 not upgraded.
9 not fully installed or removed.
Need to get 0 B/20.9 MB of archives.
After this operation, 72.6 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
(Reading database ... 574107 files and directories currently installed.)
Preparing to unpack .../linux-image-4.13.0-37-generic_4.13.0-37.42~16.04.1_amd64.deb ...
Done.
Unpacking linux-image-4.13.0-37-generic (4.13.0-37.42~16.04.1) ...
dpkg: error processing archive /var/cache/apt/archives/linux-image-4.13.0-37-generic_4.13.0-37.42~16.04.1_amd64.deb (--unpack):
 cannot copy extracted data for './boot/vmlinuz-4.13.0-37-generic' to '/boot/vmlinuz-4.13.0-37-generic.dpkg-new': failed to write (No space left on device)
No apport report written because the error message indicates a disk full error
                                                                              dpkg-deb: error: subprocess paste was killed by signal (Broken pipe)
Examining /etc/kernel/postrm.d .
run-parts: executing /etc/kernel/postrm.d/initramfs-tools 4.13.0-37-generic /boot/vmlinuz-4.13.0-37-generic
run-parts: executing /etc/kernel/postrm.d/zz-update-grub 4.13.0-37-generic /boot/vmlinuz-4.13.0-37-generic
Errors were encountered while processing:
 /var/cache/apt/archives/linux-image-4.13.0-37-generic_4.13.0-37.42~16.04.1_amd64.deb
E: Sub-process /usr/bin/dpkg returned an error code (1)
jeromesun@km:~$ 
jeromesun@km:~$ uname -a
Linux km 4.10.0-38-generic #42~16.04.1-Ubuntu SMP Tue Oct 10 16:32:20 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
jeromesun@km:~$ sudo dpkg --list 'linux-image*'|awk '{ if ($1=="ii") print $2}'|grep -v `uname -r`
linux-image-4.10.0-37-generic
linux-image-4.10.0-40-generic
linux-image-4.10.0-42-generic
linux-image-4.13.0-26-generic
linux-image-4.13.0-31-generic
linux-image-4.13.0-32-generic
linux-image-4.13.0-36-generic
linux-image-extra-4.10.0-37-generic
linux-image-extra-4.10.0-40-generic
linux-image-extra-4.10.0-42-generic
linux-image-extra-4.13.0-26-generic
linux-image-extra-4.13.0-31-generic
linux-image-extra-4.13.0-32-generic
jeromesun@km:~$ 
jeromesun@km:~$ sudo apt-get purge linux-image-4.10.0-37-generic
Reading package lists... Done
Building dependency tree       
Reading state information... Done
You might want to run 'apt-get -f install' to correct these:
The following packages have unmet dependencies:
 linux-image-extra-4.10.0-37-generic : Depends: linux-image-4.10.0-37-generic but it is not going to be installed
 linux-image-extra-4.13.0-37-generic : Depends: linux-image-4.13.0-37-generic but it is not going to be installed
 linux-image-generic-hwe-16.04 : Depends: linux-image-4.13.0-37-generic but it is not going to be installed
E: Unmet dependencies. Try 'apt-get -f install' with no packages (or specify a solution).
jeromesun@km:~$ sudo dpkg --force-all -P linux-image-4.10.0-37-generic
dpkg: linux-image-4.10.0-37-generic: dependency problems, but removing anyway as you requested:
 linux-image-extra-4.10.0-37-generic depends on linux-image-4.10.0-37-generic.

(Reading database ... 574105 files and directories currently installed.)
Removing linux-image-4.10.0-37-generic (4.10.0-37.41~16.04.1) ...
Examining /etc/kernel/postrm.d .
run-parts: executing /etc/kernel/postrm.d/initramfs-tools 4.10.0-37-generic /boot/vmlinuz-4.10.0-37-generic
update-initramfs: Deleting /boot/initrd.img-4.10.0-37-generic
run-parts: executing /etc/kernel/postrm.d/zz-update-grub 4.10.0-37-generic /boot/vmlinuz-4.10.0-37-generic
Generating grub configuration file ...
Warning: Setting GRUB_TIMEOUT to a non-zero value when GRUB_HIDDEN_TIMEOUT is set is no longer supported.
Found linux image: /boot/vmlinuz-4.13.0-36-generic
Found initrd image: /boot/initrd.img-4.13.0-36-generic
Found linux image: /boot/vmlinuz-4.13.0-32-generic
Found initrd image: /boot/initrd.img-4.13.0-32-generic
Found linux image: /boot/vmlinuz-4.13.0-31-generic
Found initrd image: /boot/initrd.img-4.13.0-31-generic
Found linux image: /boot/vmlinuz-4.13.0-26-generic
Found initrd image: /boot/initrd.img-4.13.0-26-generic
Found linux image: /boot/vmlinuz-4.10.0-42-generic
Found initrd image: /boot/initrd.img-4.10.0-42-generic
Found linux image: /boot/vmlinuz-4.10.0-40-generic
Found initrd image: /boot/initrd.img-4.10.0-40-generic
Found linux image: /boot/vmlinuz-4.10.0-38-generic
Found initrd image: /boot/initrd.img-4.10.0-38-generic
Found memtest86+ image: /memtest86+.elf
Found memtest86+ image: /memtest86+.bin
done
Purging configuration files for linux-image-4.10.0-37-generic (4.10.0-37.41~16.04.1) ...
Examining /etc/kernel/postrm.d .
run-parts: executing /etc/kernel/postrm.d/initramfs-tools 4.10.0-37-generic /boot/vmlinuz-4.10.0-37-generic
run-parts: executing /etc/kernel/postrm.d/zz-update-grub 4.10.0-37-generic /boot/vmlinuz-4.10.0-37-generic
jeromesun@km:~$ 
```
