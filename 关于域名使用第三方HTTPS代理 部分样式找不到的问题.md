# 关于域名使用第三方HTTPS代理 部分样式找不到的问题

今天把网站上传到服务器中，忽然发现样式找不到，F12一看，静态资源默认链接都是http 因此找不到

这种情况在laravel中就很好解决了
```
在app\Providers\下修改AppServiceProvider.php
/**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
 
        $this->app['request']->server->set('HTTPS',true);
    }

```
OK，这样就没问题了。