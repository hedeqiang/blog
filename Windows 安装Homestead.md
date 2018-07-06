# Windows 如何安装Homestead

## 简要安装步骤
1. 安装 VirtualBox
2. 安装 Vagrant
3. 安装 Git
4. 安装 Homestead Box 虚拟机盒子
5. 安装 Homestead 管理脚本
6. 配置 Homestead.yaml 文件
7. 启动 Homestead 虚拟机

大体就是以上7步，Git 其实有时候可以略过，接下来围绕这几步 进行安装

## 安装 VirtualBox
[VirtualBox 官网](https://www.virtualbox.org/wiki/Downloads)

下载完成之后，双击安装包进行安装，默认下一步就可以，当然你也可以更换系统盘符

## 安装 Vagrant
[Vagrant 官网](https://www.vagrantup.com/downloads.html)

同样傻瓜式直接下一步即可

==以上两个软件安装 Windows可能弹出需要管理员运行等操作，请直接运行（最好将各种杀毒软件关掉）==

## 安装Git
`Windows` 上有一个图形化界面可以下载安装  [Git客户端](https://git-scm.com/)
安装好他你只需要使用他的命令行操作即可，不要使用他的图形化界面，难用的要死

另外 `Windows` 上推荐一款软件 很好用 `git` `composer` `yarn` 等等 他都已经集成了，`nginx` `Apache`可以任意切换 最主要的是 `Linux` 中大部分命令他都可以使用 ,同样你也不需要安装连接 `Linux`的客户端了  。直接命令 `ssh root@47.256.111.111` 即可，是不是非常方便？省去了你大部分软件需要安装

## 安装 Homestead Vagrant Box
命令行下输入以下命令，注意，国内使用 以下命令 80%会出现问题，你也可以使用第三方进行下载，但是我觉得最后还是会遇到问题
所以我的建议是 如果出现错误继续运行以下命令。
```
vagrant box add laravel/homestead
```

## 下载 Homestead 管理脚本
```
cd C:\Users\你的用户名     //注意最好不要使用中文
git clone https://github.com/laravel/homestead.git Homestead
```
接着

```
cd Homestead
git checkout v6.1.0

init.bat
```

基于以上 Hmoestead 就安装成功了，接下来进行配置


## 配置 Homestead.yaml 文件
在配置之前，我们先在任意磁盘 新建一个文件夹 `Code`，用来存放我们的 `PHP`代码，比如 `laravel`等
```
cd D:\php
mkdir Code
```
接着
```
cd C:\users\你的用户名\Homestead
```
打开 `Homestead.yaml` 文件  修改 `folders`  *map 为刚才新建Code文件夹的路径*
```
folders:
    - map: D:\php\Code
      to: /home/vagrant/Code
```

比如我们现在要创建一个新的 `laravel` 项目 项目名为laravel-blog,接下来配置 Nginx 站点

对 `Nginx` 不熟悉吗？没关系。`sites` 属性可以帮助你可以轻松地将 域名 映射到 `homestead` 环境中的文件夹。`Homestead.yaml` 文件中已包含示例站点配置。同样的，你也可以增加多个站点到你的 `Homestead` 环境中。 `Homestead` 可以同时为多个 `Laravel` 应用提供虚拟化环境：

```
sites:
    - map: laravel-blog.test
      to: /home/vagrant/Code/laravel-blog/public
```



注意使用 `.test` 作为域名后缀 ，当然也可以使用别的 比如.work什么的  ，随你爱好,但是，不要使用 `.dev` `.app` 这两个了 ，因为被收买了，而且 谷歌浏览器 会自动跳转 `HTTPS` 的

## 启动 Vagrant Box

```
vagrant up
```
OK,进入到 `Code` 目录生成一个全新的 `laravel` 项目

```
cd D:php\Code
composer create-project --prefer-dist laravel/laravel laravel-blog
```

## 修改 hosts 文件
```
192.168.10.10  laravel-blog.test
```

ok  打开浏览器访问 `http://laravel-blog.test` 



