# aliyun服务器MySQL开启远程连接

说下阿里云服务器开启MySQL远程连接吧
每次执行SQL命令都得去服务器上操作  很是不爽。所以。。。。。。
根据度娘的搜索 总结如下:
```
1.登陆MySQL
mysql -u root -p
2.设置MySQL远程访问
grant all on *.* to ‘root’@'%' identified by 'root' with grant option;
解释下：第一个root表示用户名 ；第二个root表示“远程连接”的密码 ；% 表示所有的IP都可以访问登录；如果只希望特定的IP可以在这里将特定IP替换掉%；
```
```
3.刷新权限信息
flush privileges;
```

```
4.编辑MySQL配置文件：
vim /etc/mysql/mysql.conf.d/mysqld.cnf
注释掉bind-address = 127.0.0.1
```

```
5.重启MySQL服务
sudo /etc/init.d/mysql restart
```
注意：注意：注意：一定要使用flush privileges; 刷新权限否则不生效

你以为这样就完了吗？哈哈，错啃爹的阿里MySQL 3306没有设置访问权限

```
6.登陆阿里云服务器  进入控制台-》云服务器ECS-》网络和安全-》安全组
选择你服务器所在大区（这里不得不吐槽一下，我只有一个大区，你还让我选啊。。。。。不人性化。。。。）
点击“配置规则”
接着点击快速创建规则    
规则方向：入方向；
授权策略：  允许；
常用端口：MySQL（3306);
自定义端口：TCP;
授权类型：地址段访问;
授权对象：0.0.0.0/0；
优先级：1（我这里设置的1）
```

再次提醒 如果上述操作完毕还是不能连接，那么再次执行
```
 flush privileges; 
```
OK，至此MySQL就开启远程访问了，不足之处，欢迎指正