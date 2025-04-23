title: modsecurity环境配置
tags:
    - modsecurity 
    - WAF
date: 2025/04/22
categories: security
---

# modsecurity+nginx+dvwa安装配置
下载nginx与modsecurity
```bash
# 下载nginx和modsecurity的nginx插件
wget http://nginx.org/download/nginx-1.26.1.tar.gz
tar -zxvf nginx-1.26.1.tar.gz
git clone https://github.com/SpiderLabs/ModSecurity-nginx
cd nginx-1.26.1
# 编译安装
./configure --add-module=../Modsecurity-nginx
make -j 8
sudo make install 
```
安装dvwa及相关依赖
```bash
cd ~/git
wget https://github.com/digininja/DVWA/archive/master.zip
unzip master.zip -d dvwa
mv dvwa /usr/local/nginx/html
sudo pacman -S apache php php-apache mariadb php-gd php-sqlite
#启动apache服务
systemctl enable httpd
systemctl start httpd
sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
#安装mariadb数据库
systemctl enable mariadb
systemctl start mariadb
sudo mysql_secure_installation
sudo mysql -u root -p
# 去除extension中mysqli\gd\sqlite3\pdo_mysql注释
sudo vim /etc/php/php.ini
sudo chown -R http:http /usr/local/nginx/html/dvwa
sudo chown -R 755 /usr/local/nginx/html/dvwa
# 修改php的用户和分组，保持和nginx一致
sudo vim /etc/php/php-fpm.d/www.conf
# 加上LoadModule php_module modules/libphp.so
sudo vim /etc/httpd/conf/httpd.conf
# dvwa的配置文件，修改用户、密码、IP端口、防护级别等
sudo vim /usr/local/nginx/html/dvwa/config/config.inc.php
```

