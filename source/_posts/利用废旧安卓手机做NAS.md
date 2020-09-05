---
title: 利用废旧安卓手机做NAS的尝试
date: 2020-07-31 09:40:22
tags: 
- NAS 
- 安卓 
- samba 
- Linux
---

## 背景

之前买了希捷的移动机械硬盘，但是接口接触不良，导致在高速读写时，硬盘损坏，而且正好有一台只是碎屏的小米 6x，于是就想将这个手机通过 otg 功能做为 NAS 用。这样好处就是：

1、手机占用空间小，耗电量低。
2、降低机械硬盘移动导致的意外损坏。
3、废物利用不用去购买一些额外的外设。

## 方案

之前用过 ftp 传输，但是 ftp 的缺点是无法在线浏览文件，使用起来不方便，最后选择了使用 samba 服务来实现，samba 服务支持 mac，windows，安卓，ios 使用，需要注意的是 windows 无法使用默认端口以外的端口，如果安卓需要用这个默认端口，就需要对安卓手机 root。

samba 服务有两种实现

一、个比较简单的方案是安装实现 samba 服务的 app，我找到的一个是叫 LAN drive 的 app，这个 app 比自己搭建 samba 服务简单多了，但是缺点是免费版速度被限制为 0.5MB/S，还有就是我未 root 时 mac 也出现连接失败的情况，root 后就可以了。

二、就是比较折腾的方案，在安卓上装上 linux 环境，然后在 linux 中运行 samba 服务实现，好处就是速度不受软件层面限制，而且自由度高也可以安装一些实现 NAS 功能的开源项目。缺点就是需要对 linux 命令等有一定了解，比较折腾。

综上，我选择了二方案，同时记录下一些遇到的问题。

## 实现

1、安装 linux 环境
先安装 Linux deploy 这个 app，然后先做如下配置，
然后点右上角菜单的安装，安装完成后，点配置，配置好就可以启动了，需要看下启动的日志中 ssh 服务是否启动成功，不成功的话一般是 linux 系统没装好，我试了几次翻墙重复安装才成功的。成功后可以 ssh 连接到手机 linux 安装 samba 服务。
需要注意下
1、镜像大小设为 5000MB。
2、启用 ssh。
3、将 otg 存储与手机内部存储挂载到 Linux。
4、最好在翻墙环境下载 linux 镜像。我出现了几次下载不成功的情况。
2、安装 samba 服务
安装不同 linux 发行版会有些区别，我用默认的 Debian，出现了 vim 方向键变字母的问题需要先升级下 vim。

然后通过ssh连接上后进行下面的操作：

```bash
sudo apt -y update
```

安装后出现方向键变字母的问题，需要升级下vim

```bash
sudo apt-get remove vim-common
```

```bash
 sudo apt-get install vim
```

然后安装 samba

```bash
 sudo apt -y install samba
```

添加 samba 用户,需要注意的是这里的用户名需要是 linux 系统的用户名，否则会失败

```bash
sudo smbpasswd -a root
```

配置/etc/samba/smb.conf，我的配置是在配置文件最后加上以下内容

```bash
[nas]
    comment = share all
    path = /mnt
    browseable = yes
    public = yes
    writable = yes
```

最后启动 samba 服务生效

```bash
/etc/init.d/smbd restart
```

然后 mac 的话在 finder 用快捷键 comand+k 输入地址就可以就可以访问手机的存储了。

## 使用体验

由于我的路由器垃圾，只有 2-3MB/s的读写速度，换为手机热点就可以达到 16-18MB/s 的读写速度，都不是很快，但也够。最大的问题是手机存储不足，用otg挂机械硬盘好像供电不足，我买的带充电otg的线也只能挂U盘，很难成功带起机械硬盘，所以总体还是比较鸡肋的。感觉还是用开发板来做会更好。不过都装了Linux了,就可以把手机做服务器或者内网穿透工具了。
