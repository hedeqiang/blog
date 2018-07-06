# Ubuntu下LNMP安装

今天说一下 关于Ubuntu16下LNMP安装方式吧 PHP7.2、MySQL5.7、Nginx1.13，貌似这三个是目前最新的了吧
哈哈，废话不说，开始正题
原文链接：
[CODECASTS](https://www.codecasts.com/discuss/laravel/laravel-project-from-scratch-deployment-752)

```
首先先更新下Ubuntu
sudo apt update
sudo apt upgrade
```

```#FFFACD
1.安装Nginx
sudo apt-get install nginx
```
```
2.安装MySQL5.7
sudo apt-get install mysql-server mysql-client
过程当中会弹出输入，密码 确认以后应该就可以了
```

```
安装PHP7.2
sudo apt-get update 
sudo apt-get install -y language-pack-en-base
locale-gen en_US.UTF-8


sudo apt-get install software-properties-common 
sudo LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php
sudo apt-get update 


sudo apt-get -y install php7.2
sudo apt-get -y install php7.2-mysql
sudo apt-get install php7.2-fpm

apt-get install php7.1-curl php7.2-xml  php7.2-json php7.2-gd php7.2-mbstring

```
```
配置PHP设置
sudo vim /etc/php/7.2/fpm/php.ini
找到cgi.fix_pathinfo,修改为：
 cgi.fix_pathinfo=0  去掉注释
```

```
重启PHP
sudo service php7.2-fpm restart
```


接下来配置Nginx 使其支持PHP
```
打开nginx配置文件
sudo vim /etc/nginx/sites-available/default
```
```
修改文件
sudo vim /etc/nginx/sites-available/default

location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ /index.php?$query_string;
}

location ~ \.php$ {
                try_files $uri /index.php =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php/7.2-fpm.sock;  ~~#注意这里是个坑  加上php~~
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
}
主要是上述两个模块，至于root servername 根据情况自己配置
```

```
保存 可以使用nginx -t 检查有没有错误，如果看到ok  、success就说明准确无误

之后重启nginx  两种方式
1.sudo service nginx restart
2.sudo systemctl reload nginx
```

至此LNMP就安装成功了，但是Nginx版本是1.10. 这就不爽了 ，既然PHP是7.2，MySQL5.7都是最新版，Nginx怎么能out呢。既然如此那就更新nginx吧

有两种方式更新nginx
```
1.源码安装，但是好费劲啊  麻烦，所以这里就不用了了
2.升级
方法:
在 /etc/apt/sources.list.d/ 下添加一个 nginx.list 文件，内容如下：
deb http://nginx.org/packages/mainline/ubuntu/ xenial nginx  
deb-src http://nginx.org/packages/mainline/ubuntu/ xenial nginx  

添加 nginx 的 key，并更新 apt
curl http://nginx.org/keys/nginx_signing.key | sudo apt-key add  
sudo apt update  
需要注意的是，Ubuntu 自带的 nginx 系列模组会干扰nginx本体安装，所以先备份配置文件，删除ubuntu的默认模组，再重装nginx
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak  
sudo apt remove nginx nginx-common nginx-full nginx-core  
sudo apt install nginx  
sudo rm /etc/nginx/nginx.conf  
sudo cp /etc/nginx/nginx.conf.bak /etc/nginx/nginx.conf  

另外一点是此时 nginx 被 mask 了……解除并重启它：
sudo systemctl unmask nginx  
sudo systemctl start nginx  
测试无误后，加上重启自启动

sudo systemctl enable nginx  
```

更新nginx的方法当然是我百度到的 ，下方给出地址
[Kouga's blog.](https://kouga.us/update-nginx-under-ubuntu-16-04/)

```
ok,至此安装完毕，不足之处，欢迎指正。
```

