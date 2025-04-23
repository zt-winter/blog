title: ssh配置
tags:
	- ssh
	- linux
date: 2023/02/05
categories: ssh
---

# ssh配置与相关错误解决

1. 错误信息:`git@github.com: Permission denied (publickey,password,keyboard-interactive)`
环境:linux,在一个星期前,原本github配置还能够使用,突然需要输入密码,且在输入密码后依然无法连接
原因:连接github无法找到对应的密钥
排查过程:使用`ssh -Tv git@github.com`报错依然是该信息,重新生成密钥后再次添加到github.com的ssh公钥管理中,依然报错.怀疑是ssh配置出现问题,查看~/.ssh/config文件,相关信息如下所示
```
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github
```
解决方法:在stackoverflow中找到解决方法,将github.com(HostName)改为ssh.github.com. 尝试后解决.怀疑github关于https与ssh两种连接方式更新后,ssh需要单独指向ssh.github.com

2. 错误信息:`git@github.com: Permission denied (publickey,password,keyboard-interactive)`
环境:windows,第一次配置github,无法使用.
原因:连接github无法找到对应的密钥
```
Host github
HostName ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
```
解决方法:将github(Host)改为github.com.问题在于对于一般的ssh远程连接,Host就是一个昵称,在命令行`ssh xxx`使用,但在github的连接中,应该使用github.com,否则ssh无法找到对应的配置.
