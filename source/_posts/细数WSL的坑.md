---
title: 细数linux子系统WSL的坑
date: 2019-12-11 10:17:17
tags: WSL windows linux
---
# 背景

习惯用mac在类unix系统下开发，目前没配mac被迫用window下开发。用Windows开发效率太低了，无法用需要的命令和写shell脚本，还有一些与linux系统差异导致的坑。为了用上类unix之前尝试过：

* 公司申请mac，好像没有回应了，不过配macmini也不够完美，习惯了mac触控板了。
* 装黑苹果，没装成功，而且运维说有N卡问题会黑屏，放弃。
* 装unbuntu虚拟机，太卡了。还有虚拟机无法启动的情况。
* wsl目前最好的方案，估计是比较新的技术，有一些坑，而且坑的解决方案不好找。

# 具体的坑

* ssh和rsync的远程连接异常慢。
* 由于是远程连接开发，这种连接有时会出现无法连上的情况，需要重启vscode。
* linux子系统好像与window共享系统时间，linux下 date命令，ntp命令都无法修改系统时间，window修改系统时间linux系统也会变为对应时间。
* 毕竟是linux子系统的文件路径，在svn和日志中有一些文件路径会出现无法找到的情况。
* linux子系统中windows硬盘会被挂载在mnt目录下，反过来linux的文件会放在windows的 " C:\Users\<用户名>\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState\rootfs " 路径中，但直接对路径中的文件修改linux不会立即同步修改,最后还是重启电脑。还有windows下创建的文件会有权限问题无法直接读写，需要改权限。

# 总结

折腾得好累，人生苦短，给我配台 15寸的macbook pro 吧。