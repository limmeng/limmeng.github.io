---
title: Qt5.5+opencv3.0+Mac OSX10.11 Undefined Symbol Error  
date: 2015-10-17 10:15  
layout: post  
category: technology
---  
配置mac上qt+opencv的开发环境，按照[网上的教程](http://blog.sciencenet.cn/blog-702148-657754.html)配置好，在Qt Creator中创建一个简单的读取图片再显示的测试程序一直链接错误，报Undefined symbols for architecture x86_64。  
最后断断续续google了3天终于在stackoverflow的一个角落发现了解决办法：[this](http://stackoverflow.com/questions/14940126/opencv-2-4-3-cant-find-imread-and-surffeaturedetectordetect)  
>On Linux and using OpenCV 3.0.0 I had to link with the opencv_imgcodecs shared lib.   

在*.pro文件中添加了该库后能正常生成了。
