---
layout: post
title: "Mac上搭建Phonegap环境"
date: 2014-10-02 11:36:33 +0800
comments: true
categories: [mobile, Mac, phonegap]
---
话说好久没有更新Blog了。八九两个月各种事情，工作也是一换再换。现在终于能够以远程的形式重新加入一家创业公司，为心理咨询的事业奋斗。公司给配了台Macbook Pro，近期要开始着力移动端的开发，简单粗暴的phonegap第一版已经发布了，我这边也需要尽快对安卓、iOS开发熟悉下。先从phonegap入手吧。

### Node.js
phonegap是一个node.s的程序，所以需要先安装node.js。
安法也很简单，到其官网首页，点'INSTALL'下载其预编译好的安装包，直接装就行。
装完会说明安装位置并提示需要`/usr/local/bin`在path里。可以用`echo $PATH`查看
```
Node was installed at

   /usr/local/bin/node

npm was installed at

   /usr/local/bin/npm

Make sure that /usr/local/bin is in your $PATH.
```
安装成的的话，在控制台键入`node -v`应该能看到版本信息

### Cordova CLI
`sudo npm install -g cordova`

### Build Android Platform
[下载安装jdk 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，执行`java -version`会有相应版本信息
```
java version "1.8.0_20"
Java(TM) SE Runtime Environment (build 1.8.0_20-b26)
Java HotSpot(TM) 64-Bit Server VM (build 25.20-b23, mixed mode)
```

下载安装 Android SDK, `brew install android-sdk`

[下载 Eclipse](http://www.eclipse.org/downloads/), 解压即可用。 启动eclipse，选择`Help > Install New Software`，输入下面的URL: `https://dl-ssl.google.com/android/eclipse/`, 安装插件。之后会要求重启Eclipse，设定android-sdk位置,`/usr/local/opt/android-sdk`。

为了使用 Android 模拟器，还需要安装 Ant。`brew install ant`

### Build iOS Platform
安装iOS模拟器，`sudo npm install -g ios-sim`




