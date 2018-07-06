# 关于Linux服务器部署laravel项目

这篇说下在Linux Ubuntu服务器中部署laravel项目吧

```
1.下载laravel5.5最新版(推荐使用composer)
composer create-project --prefer-dist laravel/laravel laravel-wechat
我这里的laravel-wechat是我的项目名，你可以随便定义
```
```
2.接下来配置一个站点
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/laravel-wechat

打开文件
sudo vim /etc/nginx/sites-available/laravel-wechat

根据情况，修改自己的内容，（root目录指向项目目录下public）
server {
        listen 80 ;
        listen [::]:80 ;
        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
       
        # include snippets/snakeoil.conf;

        root /var/www/html/laravel-wechat/public;

        # Add index.php to the list if you are using PHP
        index index.php index.html index.htm index.nginx-debian.html;

        server_name ;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php$ {
                try_files $uri /index.php =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
}
保存  :wq
```
```
3.激活站点，添加软连接
sudo ln -s /etc/nginx/sites-available/laravel-wechat    /etc/nginx/sites-enabled/
```

```
4.检查下nginx配置文件是否正确:
sudo nginx -t
5.重启nginx使修改生效:
sudo systemctl restart nginx
```

```
6.设置目录访问权限
sudo chown -R :www-data /var/www/html/laravel-wechat
sudo chmod -R 775 /var/www/html/laravel-wechat/storage/

```
OK，接下来，输入你的域名进行访问吧，不足之处，还有指正
