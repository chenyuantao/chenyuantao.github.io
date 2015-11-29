---
layout: post
author: chenyuantao
title: The connection to adb is down, and a severe error has occured.
category: android
tag: [markdown]
---

有时候我们用边用酷狗听歌边用eclipse连接android模拟器的时候，会报The connection to adb is down, and a severe error has occured的错误，网友说的通过任务管理器关闭adb.exe再重启eclipse的方法治标不治本，我们把kadb.exe关闭之后，可以到酷狗的安装文件夹里找到kadb.exe然后删除，以后就没事了。
除了酷狗以外，其它会占用adb的程序如豌豆荚，金山手机助手，各种手机助手等软件，我们在任务管理器里发现了相应的adb进程，都可以关闭之然后删除相应的adb.exe文件。
