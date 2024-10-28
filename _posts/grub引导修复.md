---
title: Linux 和 Windows 双系统下的 grub 引导修复
date: 2024-10-28 21:53:23
tags:
    - "linux"
excerpt: "在安装了 Linux 和 Windows 双系统的情况下，有时候重装了 Windows 系统，就会导致 Linux 的 grub 引导找不到 Windows 的 efi 引导文件，本篇博客讲述解决方案。（EndeavourOS + Windows10）"
---

# 1.出现的问题

如下图所示：
[![引导出错.jpg](https://s21.ax1x.com/2024/10/28/pA05Jit.jpg)](https://imgse.com/i/pA05Jit)
提示找不到`/efi/Microsoft/Boot/bootmgfw.efi`，`no such device: 4458-2764`。后面这一串数据其实是`Windows`引导分区的`UUID`。一般情况下是因为重装系统，导致`UUID`改变了，但是 Linux 下的 Grub 引导并没有进行修改，这时候是需要手动修改的。

# 2.解决方案

首先需要知道 Windows 的引导分区是否存在：
```bash
lsblk
```
或者使用：
```bash
sudo fdisk -l
```
显示的信息更详细。

输出：
```txt
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme1n1     259:0    0 476.9G  0 disk 
├─nvme1n1p1 259:1    0   300M  0 part 
├─nvme1n1p2 259:2    0    16M  0 part 
├─nvme1n1p3 259:3    0   100G  0 part 
└─nvme1n1p4 259:4    0 376.6G  0 part 
nvme0n1     259:5    0 238.5G  0 disk 
├─nvme0n1p1 259:6    0   301M  0 part /boot/efi
└─nvme0n1p2 259:7    0 238.2G  0 part /
```
根据自己电脑磁盘情况查找，我的是`nvme1n1p1`（一般都是`300M`的那个分区）。然后记住这个分区名。

查看分区`UUID`：
```bash
sudo blkid /dev/nvme1n1p1
```

输出：
```txt
/dev/nvme1n1p1: UUID="9252-7D2A" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI system partition" PARTUUID="c9ce23fc-7bde-40a9-8cce-8da0df9079cb"
```

把这个`UUID`记下，我的这里是`9252-7D2A`。

打开并编辑`/boot/grub/grub.cfg`文件：
```bash
sudo nano /boot/grub/grub.cfg
```

找到这样的一个内容：
```txt
menuentry 'Windows Boot Manager (on /dev/nvme0n1p1)' --class windows --class os $menuentry_id_option 'osprober-efi-4458-2764>
        insmod part_gpt
        insmod fat
        search --no-floppy --fs-uuid --set=root 9252-7D2A
        chainloader /efi/Microsoft/Boot/bootmgfw.efi
```
把`--set=root`后面的`UUID`改为你自己电脑的，然后重启电脑就行了。