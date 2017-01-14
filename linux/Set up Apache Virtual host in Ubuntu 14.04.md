[TOC]

### 简介
记录安装配置Apache遇到的一些坑

---------------------------------------------------
### 前提
在修改默认配置前请确认你已安装Apache
如果还没有安装的打开终端，输入以下命令:
```
sudo apt-get update
sudo apt-get apache2
```
在终端执行成功以后，Apache将会正确的安装在我们的系统里

---------------------------------------------------
#### 说明
这篇文章主要目的是引导建立虚拟站点(example.com)用于本地访问，你也可以用你喜欢的域名来替换，当然，也要遵循相同的配置规则

#### 创建目录
第一步是要创建虚拟域名的根目录，用来存放web站点的所有资源，开放给用户访问
Apache的默认根目录是`/var/www`，我们可以在主目录下创建避免Linux的读写权限问题而造成无法访问
```
sudo mkdir -p /home/self/www/example
```
其中文件夹命名为example是为了方便多域名时的管理

#### 授予权限
假设你在其他地方设置根目录，那我们就需要为其他的用户赋予读写这个目录的权限
```
sudo chown -R $USER:$USER /var/www/example
```
这样，我们的常用用户也能自由访问这个目录，接下来我们还要普通用户赋予读和执行的权限
```
sudo chmod -R 755 /var/www
```
这样，你的服务器就有足够的权限给用户访问和浏览了

#### 创建Virtual host
Virtual host是指定虚拟主机实际配置的文件，并指定Apache Web服务器如何响应各种域请求。
Apache有一个默认的虚拟主机文件`/etc/apache2/sites-available/000-default.conf`
我们将已它为模板，为我们的每一个域名创建虚拟主机文件
创建第一个Virtual host:
```
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/example.com.conf
```
打开conf文件，并对里面的内容按照我们想要的去修改:
```
sudo nano /etc/apache2/sites-available/example.com.conf
```
像这样:
```
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	ServerName example.com
	DocumentRoot /home/self/www/example
	ErrorLog /home/self/www/example/Log/error.log
	CustomLog /home/self/www/example/Log/access.log combined
</VirtualHost>
```
*ServerAdmin 是站点管理员能有效接受的邮件地址*
*ServerName 是我们需要创建的域名*
*DocumentRoot 是站点访问的根目录*
*ErrorLog 和 CustomLog 分别是错误日志和站点访问日志*

保存文件并退出
这样修改完以后并不会直接起效，我们需要去激活这个Virtual host
```
sudo a2ensite example.com.conf
```

#### 目录访问权限
在上面两步提到的，我们可以在不同的地方设置站点根目录，而Apache的默认根目录是`/var/www`
当站点目录在默认根目录下时，我们在**授予权限**后就可以跳过这一步了
当站点目录是在主目录下时，我们就需要修改Apache的访问权限，避免服务器绕过站点目录而返回403(没有权限访问)
```
sudo vim /etc/apache2/apache2.conf
```
在访问目录下添加一个新的目录节点，并开放访问权限
```
<Directory /home/self/www/>
	Options Indexes FollowSymLinks
	AllowOverride None
	Require all granted
</Directory>
```
保存退出后重启Apache
```
sudo service apache2 restart
```

#### 修改本地hosts
我们的虚拟域名并不是实际存在的域名，所以在浏览器访问的时候DNS无法有效解析
可以通过修改hosts来改变浏览器的地址指向，使得成功访问Apache服务
```
sudo vim /etc/hosts

// hosts文件内容
127.0.0.1   localhost
127.0.0.1   example.com
```
*127.0.0.1为Apache2的IP地址*

#### 访问测试
打开浏览器，并输入域名:
```
http://example.com
```
访问成功～