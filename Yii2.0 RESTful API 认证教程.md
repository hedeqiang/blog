# Yii2.0 RESTful API 认证教程

隔了怎么长时间，终于到了 `Yii2.0`  `RESTful`  `API`  认证介绍了.

废话不多说，直接正文开始

# 认证介绍
和Web应用不同，`RESTful` `APIs` 通常是无状态的， 也就意味着不应使用 `sessions` 或 `cookies`， 因此每个请求应附带某种授权凭证，因为用户授权状态可能没通过 `sessions` 或 `cookies` 维护， 常用的做法是每个请求都发送一个秘密的 `access token` 来认证用户， 由于 `access token` 可以唯一识别和认证用户，**`API` 请求应通过 `HTTPS` 来防止man-in-the-middle (MitM) 中间人攻击**.

# 认证方式
* [HTTP 基本认证](https://en.wikipedia.org/wiki/Basic_access_authentication) ：access token 当作用户名发送，应用在access token可安全存在API使用端的场景， 例如，API使用端是运行在一台服务器上的程序。
* 请求参数： `access token` 当作API URL请求参数发送，例如 `https://example.com/users?access-token=xxxxxxxx`， 由于大多数服务器都会保存请求参数到日志， 这种方式应主要用于`JSONP` 请求，因为它不能使用HTTP头来发送 `access token`
* [OAuth 2](https://oauth.net/2/) : 使用者从认证服务器上获取基于 `OAuth2` 协议的 `access token`， 然后通过 `HTTP Bearer Tokens` 发送到 `API` 服务器。


上方进行简单介绍，内容来自 [Yii Framework 2.0 权威指南](http://www.yiichina.com/doc/guide/2.0/rest-authentication)


# 实现步骤


我们都知道 `Yii2.0` 默认的认证类都是 `User`,前后台都是共用一个认证类，因此我们要把`API` 认证类 单独分离出来，达到前、后、API都分离，
继上一章：(这里暂时使用默认User数据表，正式环境请分离不同的数据表来进行认证)

> 准备条件

继上篇的 `User` 数据表，我们还需要增加一 个`access_token` 的字段，
1. 直接在你的数据库中新增 `access_token` 字段。
2. 使用数据迁移的方式
    * 进入项目根目录打开控制台输入以下命令:
    ```
    php yii migrate/create add_access_token_to_user
    ```
    * 打开 `你的项目目录/console/migrations/m180704_054630_add_access_token_to_user.php` 修改如下内容:
    
    ```
    public function safeUp()
    {
        $this->addColumn('user', 'access_token', $this->string());
    }

    public function safeDown()
    {
        $this->dropColumn('user', 'access_token');
    }
    ```
    * 执行迁移命令
    ```
    php yii migrate
    ```
    

## 配置
打开 `api\config\main.php` 
#### 配置 `user` 应用组件:
    * 设置 `identityClass` 属性为哪个认证类
    * 设置 `enableSession` 属性为 `false`
    * 设置 `enableAutoLogin` 属性为 `true`
#### 将 `session` 组件注释掉，或删掉
```
'user' => [
            'identityClass' => 'api\models\User',
            'enableAutoLogin' => true,
            'enableSession'=>false,
            //'identityCookie' => ['name' => '_identity-backend', 'httpOnly' => true],
        ],
//'session' => [ // this is the name of the session cookie used for login on the backend
//            'name' => 'advanced-backend',
//        ],
```

#### 编写 `api\models\User.php` 实现认证类，继承 `IdentityInterface`

将 `common\models\User` 类拷贝到 `api\models\`目录下，修改命名空间为`api\models`
```
<?php
namespace api\models;

use Yii;
use yii\base\NotSupportedException;
use yii\behaviors\TimestampBehavior;
use yii\db\ActiveRecord;
use yii\web\IdentityInterface;
...
class User extends ActiveRecord implements IdentityInterface
{
    ...
    ...
}
```

#### 将 `common\models\LoginForm.php` 类拷贝到`api\models\`目录下，修改命名空间,并重写login方法：
```
<?php
namespace api\models;

use Yii;
use yii\base\Model;
...
...

public function login()
{
    if ($this->validate()) {
        $access_token=$this->_user->generateAccessToken();
        $this->_user->save();
        return $access_token;
    } else {
        return false;
    }
}

```
#### 上方代码给User模型添加了一个`generateAccessToken()`方法，因此我们到`api\models\User.php`中添加此方法
```
namespace api\models;

use Yii;
use yii\base\NotSupportedException;
use yii\behaviors\TimestampBehavior;
use yii\db\ActiveRecord;
use yii\web\IdentityInterface;
...
...
class User extends ActiveRecord implements IdentityInterface
{
    ...
    ...
    
    /**
     * 生成accessToken字符串
     * @return string
     * @throws \yii\base\Exception
     */
    public function generateAccessToken()
    {
        $this->access_token=Yii::$app->security->generateRandomString();
        return $this->access_token;
    }
}
```

#### 接下来打开 之前的`User` 控制器编写登录方法
```
use api\models\LoginForm;
...
... //省略一些代码


/**
 * 登陆
 * @return array
 * @throws \yii\base\Exception
 * @throws \yii\base\InvalidConfigException
 */
public function actionLogin()
{
    $model = new LoginForm();
    if ($model->load(Yii::$app->getRequest()->getBodyParams(), '') && $model->login()) {
        return [
            'access_token' => $model->login(),
        ];
    } else {
        return $model->getFirstErrors();
    }
}
...
```




#### 最后新增一条URL规则

打开 `api\config\main.php` 修改 `components`属性，添加下列代码:
```
'urlManager' => [
    'enablePrettyUrl' => true,
    'enableStrictParsing' => true,
    'showScriptName' => false,
    'rules' => [
        ['class' => 'yii\rest\UrlRule', 
            'controller' => 'user',
            'extraPatterns'=>[
                'POST login'=>'login',
            ],
        ],
    ],
]
```
#### 使用一个调试工具来进行测试 `http://youdomain/users/login` 记住是`POST` 请求发送，假如用`POSTMAN`有问题的话指定一下 `Content-Type:application/x-www-form-urlencoded`。

ok,不出意外的话，相信你已经可以收到一个access_token了，接下来就是如何使用这个token，如何维持认证状态，达到不携带这个token将无法访问，返回401


# 维持认证状态
实现认证只需两步：

1. 在你的 `REST` 控制器类中配置 `authenticator` 行为来指定使用哪种认证方式
2. 在你的 [user identity class](https://www.yiichina.com/doc/api/2.0/yii-web-user#$identityClass-detail) 类中实现 [yii\web\IdentityInterface::findIdentityByAccessToken()](https://www.yiichina.com/doc/api/2.0/yii-web-identityinterface#findIdentityByAccessToken()-detail) 方法.


接下来我们围绕这两步来实现：

#### 添加一个REST控制器

* 因我这里暂未设计其他数据表 所以我们暂且还使用`User` 数据表吧

1. 在`api\controllers\`新加一个控制器 命名为 `ArticleController` 并继承 `yii\rest\ActiveController`，配置认证方式代码:代码如下：

```
<?php
namespace api\controllers;

use yii\rest\ActiveController;
use Yii;
use yii\filters\auth\CompositeAuth;
use yii\filters\auth\HttpBasicAuth;
use yii\filters\auth\HttpBearerAuth;
use yii\filters\auth\QueryParamAuth;

class ArticleController extends ActiveController
{
    public $modelClass = 'api\models\User';
    public function behaviors()
    {
        $behaviors = parent::behaviors();
        $behaviors['authenticator'] = [
            'class' => CompositeAuth::className(),
            'authMethods' => [
                HttpBasicAuth::className(),
                HttpBearerAuth::className(),
                QueryParamAuth::className(),
            ],
        ];
        return $behaviors;
    }
}
```
> #### 注意：这个控制器并非真正的Article，实则还是User

2. 实现 `findIdentityByAccessToken()` 方法：

打开 `api\models\User.php` 重写 `findIdentityByAccessToken()` 方法

```
...
...
class User extends ActiveRecord implements IdentityInterface
{
    ...
    ...
    
    public static function findIdentityByAccessToken($token, $type = null)
    {
        return static::findOne(['access_token' => $token]);
    }
    ...
}
```
3. 为刚才新加的控制器添加路由规则(ps:好像多了一步......)


修改 `api\config\main.php`
```
'urlManager' => [
    'enablePrettyUrl' => true,
    'enableStrictParsing' => true,
    'showScriptName' => false,
    'rules' => [
        ['class' => 'yii\rest\UrlRule',
            'controller' => 'user',
            'extraPatterns'=>[
                'GET send-email'=>'send-email'
                'POST login'=>'login',
            ],
        ],
        ['class' => 'yii\rest\UrlRule',
            'controller' => 'article',
            'extraPatterns'=>[

            ],
        ],
    ],
]
```

接下来访问一下你的域名 `http://youdomain/articles`,不携带任何参数是不是返回 401了？

ok，这里介绍两种访问方式，一种是URL访问，另一种是通过`header` 来进行携带

1. http://youdomain/articles?access-token=y3XWtwWaxqCEBDoE-qzZk0bCp3UKO920
2. 传递 `header`头信息
```
Authorization:Bearer y3XWtwWaxqCEBDoE-qzZk0bCp3UKO920
```
> 注意 Bearer 和你的token中间是有 一个空格的，很多同学在这个上面碰了很多次

好啦，基于YII2.0 RESTful 认证就此结束了，

更过完整的功能 请移步官方文档
[授权验证](https://www.yiichina.com/doc/guide/2.0/rest-authentication)
另外还有[速率验证](https://www.yiichina.com/doc/guide/2.0/rest-rate-limiting)，就自行发觉吧
另外，如果看不懂，或者写的不好，请移步 **魏曦** 老师的视频教程，本人所有内容都是跟随 **魏曦老师** 学的
[魏曦教你学](http://www.weixistyle.com/api.php)



写完认证发现我们的接口返回的数据不是很直观，现实生活中通常也不是这样子的，我们可能会返回一些特定的格式

## 自定义响应内容

打开 `api\config\main.php` 在 `components`数组里面添加如下内容分
```

'response' => [
    'class' => 'yii\web\Response',
    'on beforeSend' => function ($event) {
        $response = $event->sender;
        $response->data = [
            'success' => $response->isSuccessful,
            'code' => $response->getStatusCode(),
            'message' => $response->statusText,
            'data' => $response->data,
        ];
        $response->statusCode = 200;
    },
],
```
这里的状态码统一设为 200 ，具体的可另行配置，假如登陆操作 密码错误或者其他，我们可以在控制器中这样使用：

```
    $response = Yii::$app->response;
    $response->setStatusCode(422);
    return [
        'errmsg' => '用户名或密码错误!'
    ];
```

水平有限，难免有纰漏，请不吝赐教，在下会感激不尽








