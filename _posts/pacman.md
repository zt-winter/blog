title: pacman常用命令
tags:
    - pacman
date: 2025/04/17
categories: pacman
---

# pacman常用命令

pacman -Rc packagename, -c = --cascade; 删除该软件以及所有依赖该软件的软件

pacman -Rs packagename, -s = --recursive; 删除该软件，并递归删除该软件的不需要的依赖。在安装该软件时，可能安装一些依赖软件，如果这些依赖软件没有被其他软件依赖，则递归删除。

pacman -Rsc packagename; 可以删除该软件的上下游依赖关系

pacman -S --overwrite \\* packagename; 覆盖性强制安装软件，当错误删除/var/lib/pacman后，导致pacman无法识别已安装的软件，可以用--overwrite \\*进行强制覆盖，之后pacman会识别到重新安装的软件