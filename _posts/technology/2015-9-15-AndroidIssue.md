---
title: Android Release 崩溃而 Debug 正常
date: 2015-9-15 21:44
layout: post
category: technology
---   
正准备发布首个个人独立开发的App，但是导出的Release版本老是因为null pointer而crush，但在Debug模式下一切正常，从下午调到晚上，最后发现是Android Studio中设置`minifyEnabled true`会使引用的java库出问题，我的项目里用到了joda time，需要在 proguard-rules.pro 文件中加上  
`-keep class org.joda.time.** { *; } `
`-dontwarn org.joda.time.**`  
两句。之后就正常了。