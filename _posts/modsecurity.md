title: modsecurity环境配置
tags:
    - modsecurity 
    - WAF
date: 2025/05/20
categories: security
---

# modsecurity+nginx+dvwa安装配置
下载nginx与modsecurity
```bash
# 下载modsecurity并安装
git clone git@github.com:owasp-modsecurity/ModSecurity.git

# 下载nginx和modsecurity的nginx插件
wget http://nginx.org/download/nginx-1.26.1.tar.gz
tar -zxvf nginx-1.26.1.tar.gz
git clone https://github.com/SpiderLabs/ModSecurity-nginx
cd nginx-1.26.1
# 编译安装
./configure --add-module=../Modsecurity-nginx --with-http_ssl_module
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
# 启动apache服务
systemctl enable httpd
systemctl start httpd
sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
# 安装mariadb数据库
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

#apache服务在启动时会在/var/run/中创建文件夹，可以在启动服务中添加创建目录命令，避免找不到目录和文件
vim /usr/lib/systemd/system/php8.4-fpm.service
[Service]
ExecStartPre=/bin/install -d /var/run/php-fpm -o root -g root -m 751 
```
modsecurity规则文件整合及配置
```bash
mkdir /usr/local/nginx/conf/modsec
git clone git@github.com:coreruleset/coreruleset.git
cd coreruleset/rules
cat *.conf >> modsecurity-crs.conf
cp * /usr/local/nginx/conf/modsec/

cd ModSecurity
cp modsecurity.conf-recommanded /usr/local/nginx/conf/modsec/
cd coreruleset
cp crs-setup.conf.example /usr/local/nginx/conf/modsec/
cd /usr/local/nginx/conf/modsec
mv modsecurity.conf-recommanded modsecurity.conf
mv crs-setup.conf.example crs-setup.conf
```



nginx配置文件
```conf
server {
    #监控本地80端口
	listen 80;
	server_name localhost;
	modsecurity on;
    #加载核心规则集启动配置
	modsecurity_rules_file /usr/local/nginx/conf/modsec/crs-setup.conf;
    #加载配置文件
	modsecurity_rules_file /usr/local/nginx/conf/modsec/modsecurity.conf;
	modsecurity_rules_file /usr/local/nginx/conf/modsec/modsecurity-crs.conf;
	#nginx加载modsecurity不支持使用通配符匹配多个文件
	#modsecurity_rules_file /usr/local/nginx/conf/modsec/crs/rules/*.conf;
    #设置代理
	location / {
		proxy_pass http://localhost:8080;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_set_header ModSecurity-enabled "1";
	}
}
```
