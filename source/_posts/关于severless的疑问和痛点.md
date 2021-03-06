---
title: 关于severless的疑问和痛点
date: 2020-09-05 14:25:08
tags: severless aws
---

1 、一个接口调用 severless 运行需要将所有函数接口代码加载到内存，运行玩之后就释放，还有对于数据库之类的连接就不是用连接池的方式要不断创建断开，这样运行效率不会比较低吗？

2 、使用了 severless 无法用之前开发的断点调试方式，也许应该是有办法的断点的，但是之前的开发者没有用断点，都是打日志的形式，这样调试起来效率就比较低了。

3 、接手的项目日志只有本地调试有打，部署后由于是 severless 无法存日志于本机，项目并没有做日志功能，如果是用户端反馈 bug，则排查起来比较困难。

4 、接手的项目并没有用主流的 web 框架来开发，而是自己对接口调用进来的像 url，get/post 进行 swich case 或 if 来处理的，这样无法与主流技术保持一致，我想对 koa 之类的框架应该有一些中间件实现对 severless 的支持吧，然后如果后面不想用来，可以比较方便地切换 severless 服务商或者运行在自己的服务器。

5 、部署不方便，要在 aws 的后台传 apigetway 文件，然后再传函数实现部分的代码，无法沿用我以前的 shell 脚本上传方式，加上 aws 网站太慢了，效率有点低，需要额外学习 aws 的开发文档然后写个 shell 脚本。

6 、项目中的 apigetway 文件是 swagger 的 yaml 文件，和接口实现部分是分开的，这样开发需要两个文件间跳转查看修改并保持一致，效率有点低，容易出错，我希望吧 swagger 的接口文档写在对应接口实现的注释中，然后转换成 aws 需要的 yaml 文件。

最后 severless 开发起来不够自由，无法内存长驻，无法使用自建服务器的技术，虽然降低了服务器开发维护的门槛，但是开发与云服务商强相关了，各家serveless的技术实现不统一，资料目前比较少。
