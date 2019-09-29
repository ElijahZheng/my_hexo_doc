title: macOS Mojave 安装双系统 bug 总结
author: Elijah Zheng
tags:
  - macOS
categories:
  - 系统
date: 2019-03-23 09:24:00
---
  在 mac 上装 windows10 双系统一般是由于	

1. 开发的需要，有些系统级的架构只支持windows，不适合在 mac 上开发。
2. 运行的游戏只支持 windows，无法在 mac 上运行。	
 
macOS Mojave 只支持安装 windows10，往下的版本不再支持。``启动转移助理（BootCamp）``的功能目前也限制于制作双系统和将双系统恢复为单一分区，取消了之前支持windows多版本和制作u盘启动盘的选项。不过目前仍可用代码来制作u盘启动盘。

在 mac 上用 windows10 u盘启动盘来安装双系统是行不通的，因为会导致缺少驱动而不能往下安装，所以就只能走``启动转移助理``这条路。	
windows10 经常更新版本，导致 mac 使用``启动转移助理``安装双系统时会出现各种各样的问题。列举如下：	
1. 版本问题。用 macOS Mojave 10.14版本只能安装 windows10 4月版的，安装10月版的会出现错误。将macOS Mojave升级到 10.14.3版本解决版本问题。
2. 外接设备导致无法安装 windows 的问题。当进入到 windows 安装界面时，必须把所有外接设备（鼠标、u盘等）全部拔出，不然会出现分区找不到的错误导致无法安装 windows。
3. 一旦将 windows 安装完毕之后，分配的硬盘空间将无法调整，所以分磁盘空间的时候必须考虑清楚。
4. 用 mac 上的``磁盘工具``直接将 windows 分区抹除再合并分区，可能会导致下次安装 windows 失败，所以要删掉之前安装好的 windows 的办法是用``启动转移助理``将 windows 合并成单一分区。