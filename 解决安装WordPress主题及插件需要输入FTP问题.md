# 解决安装WordPress主题及插件需要输入FTP问题


安装一个WordPress好像挺简单，但是默认主题不喜欢，想更换一个，无奈本地可以更换，但是服务器更换的时候需要设置FTP 。OK，设置呗，好像我的用户名密码之类的都是正确的，就是不让我通过，因此，找了一下解决方案
```
1. 进入WordPress根目录
vim wp-config.php
添加以下三句代码
define("FS_METHOD", "direct");
define("FS_CHMOD_DIR", 0777);
define("FS_CHMOD_FILE", 0777);
```
接着重新访问你的网站，重新安装主题，可能就不需要输入密码了。哈哈
OK，不足之处，欢迎指正