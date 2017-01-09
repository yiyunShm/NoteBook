从Windows换成Linux大概有一个多月，因为不熟悉，所以一直在折腾。之前虽然有安装过双系统，但是因为惰性，也或者是心里不太情愿跳出舒适区，双系统中的Linux一直就被搁置，也没有好好学习。

经过了多次的挣扎，终于狠下心来。把电脑重装了，换的系统是Elementary OS，基于Ubuntu14.04，界面是仿Mac的，看着倒也不错。

接下来安装软件的过程是各种懵逼...

不知道有哪些软件好用，不知道那些软件的开发配置怎么修改。google了很多Linux的教程，但是不同版本也会有不一样的做法，就在那里倒腾了几天。安装了，不满意，卸载，再找一个重新安装、配置。刚开始的时候很苦闷，后来看着系统工具慢慢完善，也多了点欣慰和喜悦，对命令也慢慢熟悉起来。

------------------------------------------------------------
以下是我在倒腾过程中发现的比较好的软件列表
#### 目录
* [开发工具](#开发工具)
* [编辑器](#编辑器)
* [图像处理](#图像处理)
* [文档处理](#文档处理)
* [音乐视频播放器](#音乐视频播放器)
* [文档阅读](#文档阅读)
* [网盘工具](#网盘工具)
* [其他](#其他)

#### 开发工具
__Java: __
* [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 需配置环境变量
* [Android Developers](https://developer.android.com/index.html) android开发官网
* [Eclipse](http://developer.android.com/sdk/index.html) 直接下载ADT Bundle，省时省事，需先有java环境。SDK Manager更新缓慢的解决办法：设置代理

__PHP: __
* [PHP](http://php.net/manual/zh/install.unix.debian.php)
* [Netbean](https://netbeans.org/downloads/) 支持PHP/Java等环境，不够要先安装了开发环境才能调试


__Node: __
* [NodeJs](https://nodejs.org/en/download/package-manager/) 包安装的话版本可能会有点久，不过也可以先安装好后再升级

__MySQL: __
* [MySQL](http://dev.mysql.com/downloads/os-linux.html) 经典的关系数据库管理软件

__MongoDB: __
* [mongodb](https://www.mongodb.com/download-center?jmp=docs&_ga=1.69178008.827425485.1483933422#community) NoSQL

#### 编辑器
* [Sublime Text 3](http://www.sublimetext.com/3) 很强大的编辑器，有丰富的插件，收费但是可以无限期试用，需要注意的是Ubuntu下的Sublime无法输入中文，需要额外配置
* [Vim](http://www.vim.org/) 非常强大...
* MarkDown编辑器[Typora](https://typora.io/#linux) 

#### 图像处理
* [Krita](https://krita.org/zh/)  概念艺术图  数字绘景  插画和漫画
* [Gimp](https://www.gimp.org/)

#### 文档处理
* [WPS](http://community.wps.cn/download/) 64位用户安装32位依赖包`sudo apt-get install ia32-libs`。需要注意的是跟Win平台的MS-Office兼容不太好，建议编辑完后转为其他格式(pdf)。

#### 音乐视频播放器
* [VLC media player](http://www.videolan.org/vlc/#download)
* [SMPlayer](http://smplayer.org/) 视频播放器

#### 文档阅读
* [Adobe Reader](http://www.adobe.com/support/downloads/product.jsp?product=10&platform=unix)
* [ChmSee](https://code.google.com/p/chmsee/) 阅读帮助文档必备
* [Calibre](http://www.calibre-ebook.com/) 不仅仅是阅读器，还可以做电子文档管理

#### 网盘工具
* [Dropbox](https://www.dropbox.com/install-linux)
* [百度网盘](https://github.com/LiuLang/bcloud) 开源爱好者[LiuLang](https://github.com/LiuLang)所写

#### 其他
* [搜狗输入法](http://pinyin.sogou.com/linux/?r=pinyin) 官方出品
* [Virtual](https://www.virtualbox.org/) 有时候还是要用一下windows的
* [Shadowsocks](https://shadowsocks.com/client.html) 佛跳墙～
* [XLCloudClient](https://github.com/CaledoniaProject/XLCloudClient) 迅雷离线客户端
* [xunlei-lixian](https://github.com/iambus/xunlei-lixian)