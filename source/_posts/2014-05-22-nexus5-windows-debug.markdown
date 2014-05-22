---
layout: post
title: "解决Windows下Android Device Manager无法找到Nexus5"
date: 2014-05-22 21:50:49 +0800
comments: true
categories: Android
---
一直用公司的MAC开发Android，在将Nexus5连接到自己的Windows上时，发现ADT中无法找到Nexus5，adb devices命令也无法检测到设备，虽然设备能够以磁盘的方式显示在Windows Explorer中，但是似乎设备的驱动仍然没有正确的安装：

*	打开windows 设备管理器可以发现，Nexus5设备标识上有一黄色叹号，表示驱动没有正确安装
*	下载Nexus5 [设备驱动](http://developer.android.com/sdk/win-usb.html#top)，并解压至某一目录
*	在设备管理器中，右键点击设备，并更新驱动
*	选中刚才解压的目录，即可正确的安装驱动

Issue Fixed ：）