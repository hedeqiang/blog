# 自定义laravel 辅助函数
> Laravel 包含各种各样的全局「辅助」PHP 函数，你可以找到并使用它们，但是 ，可能并不是所有的内置方法都能满足你，因此我们需要自定义一个辅助方法。

#### 方法如下：

我们将自定义的方法存放在 `bootstrap/helpers.php` 文件中。

在 `bootstrap/` 文件下创立 `helpers.php` 。

    ```
    touch bootstrap/helpers.php
    ```
测试方法，写入内容
 
    ```
    function hello() {
        return 'hello word';
    }
    ```
 接下来我们使用 `tinker` 命令来验证线下我们的方法
    ```
    php artisan tinker
    ```
    
    
 然后在 `tinker` 交互中输入我们的测试方法 `hello()`
 
 ```
 PHP Fatal error:  Call to undefined function hello() in eval()'d code on line 1
 ```
 发现报错，提示找不到这个函数，这是因为我们还没有引入这个 helpers.php 文件，我们可以使用 composer 的 autoload 功能来自动引入：、
 
 打开 `composer.json` 文件，并找到 `autoload` 段，将其修改为：
 ```
 "autoload": {
        "classmap": [
            "database/seeds",
            "database/factories"
        ],
        "psr-4": {
            "App\\": "app/"
        },
        "files": [
            "bootstrap/helpers.php"
        ]
    },
 ```
 
 最后在项目根目录中执行 `composer dumpautoload` 命令。做了这些工作，我们的辅助方法，就可以正常运行了,继续使用 `tinker` 输入 `hello()` ,应该会输出 `hello word` 字样

以上就是 `laravel` 如何自定义辅助函数的内容。 更多方法请参考 [辅助函数](https://laravel-china.org/docs/laravel/5.6/helpers/1391)
 

