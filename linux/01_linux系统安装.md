# linux系统安装


## 概述

单机大概步骤：

* 下载镜像
* 制作启动盘
* u盘启动
* 下一步下一步

批量安装解决方案：

* PXE
* 部分发行版 cobbler

## 单机系统安装

### 镜像下载

中科大镜像站地址 [点我访问-->](https://mirrors.ustc.edu.cn/)

### 制作启动盘

略

### U盘启动

略

### 开始安装

...



## 批量安装

### kickstart

#### kickstart简介

`kickstart ` 他是在安装过程中记录人工干预填写的各种参数，并生成一个名为ks.cfg的文件。他的作用就是根据文件里面的参数进行批量化系统安装，且保证每个安装出来的是一模一样的。 简单来说就是一个安装模板，根据这个模板安装出来的系统都是一样的。同时有遇预设相关参数，安装过程中不需要手动干预，可以实现批量化系统安装。

`kickstart` 文件可以手动创建、也可以自动创建(系统安装完成后会生成一个)， 也可以通过工具生成。

`kickstart` 优势： 

* 批量自动化
* 一致性
* 减少人为安装的部署失误

使用`kickstart` 的安装步骤

* 创建一个kickstart文件
* 创建有kickstart文件的引导介质或者使这个文件在网络上可用
* 开始ks安装：anconda自身启动 -->选取ks安装模式--> 从ks文件读取配置 --> 最后安装

#### 不同介质使用kickstart

- 本地CDROM 

  ```shell
  ks=cdrom:/path/to/kickstart_file
  ```

- 磁盘驱动器

  ```shell
  ks=hd:/device/path/to/kickstart_file
  ```

- NFS

- HTTP

- FTP

  ```shell
   ks=[http://]/path/kickstart_file
  ```

#### 制作kickstart的方式

制作方式：

* 使用安装后自动生成文件
* 修改安装后自动生成文件
* 图形化界面生成`system-config-kickstart`



#### kickstart文件组成部分

* 命令必填项
  - 配置系统的属性及安装中的各种必要设置信息，如语言、键盘类型、时区等信息
* 设置软件包
  * 使用`%packages`指明开始，`%end`指明结束。包组以`@`开始标明，软件名可以直接写，但每个之间需要换行
* 安装前脚本
  * 使用`%pre`指明开始，`%end`指明结束，将每个命令写一行
* 安装后脚本
  * 使用`%post`指明开始，`%end`指明结束，将每个命令写一行

#### 部分参数说明

| 参数       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| Install    | 可选，指明此次是全新安装                                     |
| cdrom      | 可选，指明安装源为本地的光驱                                 |
| url        | 可选，指明安装源为远程的ftp、http方式安装 url  --url http:// |
| harddrive  | 可选，指明安装源为本地的硬盘                                 |
| nfs        | 可选，指明安装源为NFS服务                                    |
| bootloader | 设定bootloader安装选项                                                        `bootloader  --location=mbr  --append=“rhgb quiet” --driveorder=sda,sdb` |
| clearpart  | 在建立新分区前清空系统上原有的分区表，默认不删除分区               clearpart --all 擦除系统上原有所有分区； |
| firewall   | 可选，配置防火墙选项                                         |
| selinux    | 可选，配置selinux选项，默认选项为enf                         |
| reboot     | 可选，在系统成功安装完成后默认自动重启系统（kickstart方法时）；在装系统完成后，会提示按任意键进行重启；在配置中没有指明其他方法时，默认完成就会reboot<br /> |
| keyboard   | 设置键盘类型，一般为us   keyboard us                         |
| lang       | 设置系统语言，默认为en_US  lang en_US                        |
| timezone   | 设置系统时区 timezone Asia/Shanghai                          |
| auth       | 设置系统的认证方式  auth --useshadow --passalgo=sha512       |
| rootpw     | 设置root的密码  rootpw abcde123 <br/> rootpw rootpw --iscrypted $1$iRHppr42$VMesh73wBqhUTjKp6OYOD. <br/>可以使用 `openssl passwd -6 -salt 盐值 密码` 生成密文密码 |
| service    | 可选，设置禁用或允许列出的服务                               |

最小化的文件示例：

```shell
[root@centos7 ~]# cat anaconda-ks.cfg 
#version=DEVEL
# System authorization information 系统认证方式 512加密
auth --enableshadow --passalgo=sha512
# Use CDROM installation media
cdrom
# Use graphical install 安装方式， 字符界面改成text
graphical
# Run the Setup Agent on first boot  
firstboot --enable

# Keyboard layouts 键盘布局
keyboard --vckeymap=us --xlayouts='us'
# System language 系统语言
lang en_US.UTF-8

# Network information 网络信息
network  --bootproto=dhcp --device=enp0s3 --onboot=off --ipv6=auto --no-activate
network  --hostname=localhost.localdomain

# Root password root密码
rootpw --iscrypted $6$0z465Zw04B6eijX9$gYVQRsEaKbg.mVxpYyFGhxdr1AH.drQCM48FI.0rbU60CmA0yus27O8lgO9J1Vsna.0vttJLcR/GMvGnK7Gl1.
# System services 服务管理
services --enabled="chronyd"
# System timezone 时区设置
timezone America/New_York --isUtc
# System bootloader configuration 
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
autopart --type=lvm
# Partition clearing information
clearpart --none --initlabel
# 最小化安装
%packages 
@^minimal
@core
chrony
kexec-tools

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end
# 密码复杂度
%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end

```





