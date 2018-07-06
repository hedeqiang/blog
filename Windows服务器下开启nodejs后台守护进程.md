# Windows服务器下开启nodejs后台守护进程

windows服务器部署node项目 启动node服务以后，CMD窗口不能关闭 关闭以后服务就挂掉了
因此需要开启守护进程，使其后台执行

``` 
安装forever
npm install forever -g
```
```
运行
forever start bin/www
```
```
注意上述运行命令是基于nodejs express框架运行的 