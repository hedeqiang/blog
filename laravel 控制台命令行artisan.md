# laravel 控制台命令行artisan

`Artisan` 是 `Laravel`自带的命令行接口，它提供了许多实用的命令来帮助你构建` Laravel` 应用。要查看所有可用的 `Artisan` 命令的列表，可以使用 `list` 命令：
```php
php artisan list
```
每个命令包含了「帮助」界面，它会显示并概述命令的可用参数及选项。只需要在命令前面加上 help 即可查看命令帮助界面：
```
php artisan help migrate
```
编写命令#
除 `Artisan` 提供的命令之外，还可以构建自己的自定义命令。命令默认存储在` app/Console/Commands` 目录，你也可以修改 `composer.json` 文件来指定你想要存放的目录。
生成命令#
要创建一个新的命令，可以使用 Artisan 命令` make:command`。这个命令会在 `app/Console/Commands` 目录中创建一个新的命令类。 不必担心应用中不存在这个目录，因为它会在你第一次运行 Artisan 命令 make:command 时创建。生成的命令会包括所有命令中默认存在的属性和方法：
```
php artisan make:command HelloWord
```
命令生成后，应先填写类的 `signature` 和 `description` 属性，这会在使用` list` 命令的时候显示出来。执行命令时会调用 `handle` 方法，你可以在这个方法中放置命令逻辑。
修改生成的文件 如下:
```
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class HelloWorld extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'HelloWorld:say {name ?}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Command description';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
        $password = $this->secret('What is the password?');
        if ($this->confirm('Do you wish to continue?')) {
            Log::info( $name  .$password);
            $this->info('hello '. $name .$this->argument('name'));
        }
    }
}
```
修改完代码 执行`php artisan list`查看当前命令是否生成
接着运行`php artisan HelloWorld:say hedeqiang`,应该会依次让你输入name 以及password 接着输入yes，
上面的代码其实并没有什么实际作用，这里只不过是演示一下命令如何生成，应用场景可能有好多，比如发送邮件，生成特定的文件等等......
ok,这就是如何使用`laravel` 生成`artisan` 命令
更多完整功能，请查阅官方文档[laravel5.5中文文档](https://d.laravel-china.org/docs/5.5/artisan)