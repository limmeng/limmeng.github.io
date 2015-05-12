---
title: 通过源代码学习——Android反编译
date: 2015-5-12 12:45
layout: post
category: technology
---
阅读源代码是很好的学习方式，过程虽有些痛苦与枯燥，但认真看下来往往能够经验大增。
最近在做一个Android效率工具的小项目，在调研的过程中下载试用了很多同类型的应用。在有的应用里看到很好的效果，想自己也实现这样的效果，但是思路不是很明确，于是就去学习了一下Android反编译的东西，好看看别人到底是怎样实现的。

##方法一：APK Studio   
工具下载：[Apk Studio](https://apkstudio.codeplex.com/)  
使用方法：十分简单，直接打开.apk文件就可了，java源文件以及资源文件都能看到，但是在实际的使用过程中只成功破解过一个app，可能是稍作处理后的app就不能用此工具直接反编译了。

##方法二：dex2jar+jd-gui+Apktool
工具下载：[dex2jar](http://sourceforge.net/projects/dex2jar/files/)，[jd-gui](http://jd.benow.ca/)，[Apktool](https://ibotpeaches.github.io/Apktool/)  
使用方法：  
1. 将.apk文件后缀改为.zip，然后解压可以得到.dex文件；  
2. 将得到的.dex文件（一般是classes.dex），放到dex2jar-x.x文件夹中(x.x代表dex2-jar的版本号，在dex2jar-x.x文件中应该有一个名为d2j-dex2jar.bat的文件），在该文件夹中打开命令行（空白处shift+鼠标右键 选择在此处打开命令窗口），运行<code>d2j-dex2jar.bat classes.dex</code>之后将在同一文件夹下得到名为classes-dex2jar.jar的文件；  
3. 将得到的classes-dex2jar.jar文件用jd-gui直接打开即可看到原app的java源码了；  
4. 接下来使用Apktool获取原app中的资源文件（布局文件等），按照Apktool官网上的说明下载好apktool.bat及apktool.jar文件后，将这两个文件放入同一文件夹下，再将.apk文件也放在该文件夹下，在该文件夹下打开命令行，运行<code>apktool.bat d xxx</code>（xxx代表.apk文件全名，d代表decode），即可获得原app中资源文件。  

好好学习，天天向上:)



