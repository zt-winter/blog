title: linux-wifi
tags:
	- linux
categories: linux
---

主机上装了双系统，昨天晚上使用windows10打完游戏后，今天进入Linux发现没有无线网卡设备，报错信息为`kernel iwlwifi: probe of failed with error -110`。

搜索相关报错信息后发现原因为windows10为了快速启动，即使关闭windows也不会释放网卡设备，以便下次启动时不用再次加载网卡驱动，但这导致linux启动时发现网卡设备被占用，无法使用网卡设备。

解决方法：windows10进入控制面板，打开电源设置，关闭快速启动。
