---
layout: post
author: chenyuantao
title: 使用okhttp时url特殊字符自动转义的解决办法
category: android
tag: [okhttp]
---
##使用okhttp时url特殊字符被自动转义的解决办法##
###起因###
在调用网易云音乐的接口时，发现接口传递的参数含有“%”字符，直接使用okhttp发送请求的时候，会被自动转义成“%25”，导致请求返回400。
接口：String url = "http://music.163.com/api/song/detail/?id="+music_id+"&ids=%5B"+$music_id+"%5D";    
![返回结果](http://ww2.sinaimg.cn/large/72f96cbagw1f7qm8af45ij21160bo0ww.jpg)
###解决###
Google和StackOverFlow一番之后找到问题所在，就是特殊字符会被转义掉
![原因](http://ww4.sinaimg.cn/large/72f96cbagw1f7qmc27fpoj214m08ggpk.jpg)
但是这个解决办法是解决编码不匹配问题的，我的需求是让特殊字符不被转义。     
自己捣鼓一番之后，发现URL解码可以解决问题

    URLDecoder.decode("%5B"+musicId+"%5D","UTF-8");

结果如图，%没有被转义成%25了     
![解决](http://ww1.sinaimg.cn/large/72f96cbagw1f7qmfifsvkj20we0dojv1.jpg)
