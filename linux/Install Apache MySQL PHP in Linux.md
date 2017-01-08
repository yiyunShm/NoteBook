### 关于LAMP
LAMP 是一组用于构建web服务器的开源软件集合代表，它们分别是Linux, Apache, MySQL和PHP的首字母缩写。

-----------------------------------------------------------------
### LAMP安装(eg: Ubuntu)
#### Apache
Apache是一款开源产品，目前全球已在全球50%以上的服务器运行着
打开终端，并输入以下命令:
``` 
sudo apt-get update
sudo apt-get install apache2
```
这就完成了。接下来是检测Apache是否安装成功：打开浏览器并输入你的服务器IP地址(例：http://127.0.0.1). 网页就会显示"It works"，表示安装成功。

#### MySQL
MySQL是一个强大的数据库管理系统，能有效的管理和检索数据
打开终端，并输入以下命令:
```
sudo apt-get install mysql-server libapache2-mod-auth-mysql php5-mysql
```
在安装期间，MySQL将会要求你设置root的访问密码 (如果你暂时不想设置，也可以往后手动设置)
在安装完成后，我们需要启动MySQL:
```
sudo mysql_install_db
```
或者通过运行MySQL脚本来完成
```
sudo /usr/bin/mysql_secure_installation
```
将会弹出上一步设置的密码验证
```
Enter current password for root(enter for none):
OK,successfully used password, moving on...
```
继续弹出询问是否要重置root的访问密码，如果不想重置请输入N，并进行下一步
接下来还有一些基本设置，都可以输入Y对MySQL进行初始化
```
By default, a MySQL installation has an anonymous user, allowing anyone to log into MySQL without having to have a user account created for them.  This is intended only for testing, and to make the installation go a bit smoother.  You should remove them before moving into a production environment.

Remove anonymous users? [Y/n] y
... Success!

Normally, root should only be allowed to connect from 'localhost'. This ensures that someone cannot guess at the root password from the network.
Disallow root login remotely? [Y/n] y
... Success!

By default, MySQL comes with a database named 'test' that anyone can access.  This is also intended only for testing, and should be removed before moving into a production environment.
Remove test database and access to it? [Y/n] y
Dropping test database...
... Success!
Removing privileges on test database...
... Success!

Reloading the privilege tables will ensure that all changes made so far will take effect immediately.
Reload privilege tables now? [Y/n] y
... Success!
Cleaning up...
```
现在，你完成MySQL的安装了.

#### PHP
PHP是开源的web脚本语言，被广泛的应用于动态网站的搭建
打开终端，并输入以下命令:
```
sudo apt-get install php5 libapache2-mod-php5 php5-mcrypt
```
在完成两次提示回应后，PHP将会自动安装完成
在安装完成后，可以设置Apache的启动文件
```
sudo vim /etc/apache2/mods-enabled/dir.conf
```
查看index.php是否是默认的启动文件，像下面这样
```
<IfModule mod_dir.c>
	DirectoryIndex **index.php** index.html index.cgi index.pl index.php index.xhtml index.htm
</IfModule>
```
PHP含有丰富的库和模块资源，你可以将他们选择性的添加到你的服务器上使用
```
apt-cache search php5-
```
终端会根据上面的命令罗列出查找到的库，就像下面这样：
```
php5-cgi - server-side, HTML-embedded scripting language (CGI binary)
php5-cli - command-line interpreter for the php5 scripting language
php5-common - Common files for packages built from the php5 source
php5-curl - CURL module for php5
php5-dbg - Debug symbols for PHP5
php5-dev - Files for PHP5 module development
php5-gd - GD module for php5
php5-gmp - GMP module for php5
php5-ldap - LDAP module for php5
php5-mysql - MySQL module for php5
php5-odbc - ODBC module for php5
php5-pgsql - PostgreSQL module for php5
php5-pspell - pspell module for php5
php5-recode - recode module for php5
php5-snmp - SNMP module for php5
php5-sqlite - SQLite module for php5
php5-tidy - tidy module for php5
php5-xmlrpc - XML-RPC module for php5
php5-xsl - XSL module for php5
php5-adodb - Extension optimising the ADOdb database abstraction library
php5-auth-pam - A PHP5 extension for PAM authentication
[...]
```
找到你想要的模块并安装
```
sudo apt-get install [name of the module]
```

------------------------------------------------------------------
### LAMP基本操作
#### Apache
1.假设Apache安装目录为/usr/local/apache2：
```
/usr/local/apache2/bin/apachectl start    //启动
/usr/local/apache2/bin/apachectl stop     //停止
/usr/local/apache2/bin/apachectl restart  //重启 
```
2.service命令
```
sudo service apache2 start
sudo service apache2 stop
sudo service apache2 restart
```
3.脚本执行
```
sudo /etc/init.d/apache2 start    //启动
sudo /etc/init.d/apache2 stop     //停止
sudo /etc/init.d/apache2 restart  //重启
```
#### MySQL
1.service命令
```
sudo service mysql start
sudo service mysql stop
sudo service mysql restart
```
2.脚本执行
```
sudo /etc/init.d/mysql start
sudo /etc/init.d/mysql stop
sudo /etc/init.d/mysql restart
```
3.登录SQL
```
mysql -u root -p [password]
```

--------------------------------------------------------------
### 开启web服务之旅
在Apache服务根目录下创建一个php文件
```
sudo vim /var/www/index.php
```
在文件内容加上下面的语句
```
<?php
	phpinfo();
?>
```
保存后退出vim，启动Apache
```
sudo service appache2 restart
```
打开你的浏览器输入你的[IP地址](http://127.0.0.1)，享受成功的喜悦吧～