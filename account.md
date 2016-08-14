# Laravel 实战练习：账号系统

账号系统，简单点说要做三件事：

*	第一件事：登录逻辑；        
*	第二件事：在部分页面判断登录才能访问；              
*	第三件事：退出的时候注销登录状态。           

其中，第二部分就是这里我们要用中间件来实现的事情。【PS：如果不用中间件的话，给所有的Controller一个基类，在基类的构造方法里面实现也是可以的。】

#### 一、登录模块

表结构按照框架提供的思路设计，多加两个字段

```sql
CREATE TABLE `fl_users` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT 'user_id',
  `username` varchar(20) NOT NULL DEFAULT '' COMMENT '登录账号',
  `password` varchar(64) NOT NULL DEFAULT '' COMMENT '密码',
  `email` varchar(100) NOT NULL DEFAULT '' COMMENT '邮箱',
  `remember_token` varchar(100) DEFAULT NULL COMMENT 'token',
  `created_at` int(11) NOT NULL DEFAULT '0' COMMENT '创建时间',
  `updated_at` int(11) NOT NULL DEFAULT '0' COMMENT '最后更新时间',
  `nickname` varchar(20) NOT NULL DEFAULT '' COMMENT '昵称',
  `type` tinyint(2) NOT NULL DEFAULT '2' COMMENT '管理员类型 1-超级管理员，2-普通管理员',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_email` (`email`),
  UNIQUE KEY `idx_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='账号表';
```

要注意这里，`remember_token` 字段 必须设置为默认null、长度100+。这个字段将会被用来保存「记住我」 session 的令牌。

Model 直接借用框架中的 `User` 类，注意 move 到 Models 目录下之后改一下 `namespace` 。

在开始写登录逻辑之前先来聊另一件事：密码。

一般来说，简单点我们会把用户密码进行加密，比如Md5或者哈希加密。而 Laravel 本身已经提供了较为可靠的哈希加密和校验方法：`Hash` 类。

```php
# Hash Bcrypt 加密：
$password = Hash::make('secret');
//上句等同于： $password = bcrypt('secret');

# 校验加密
$bool = Hash::check('secret', $hashedPassword);

# 检查密码是否需要重新散列
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('secret');
}
```



密码说完了，回到正题。

Laravel 自带 的Auth认证，有一个配置文件，位置在 `app/config/auth.php` ，在使用 `auth` 之前，需要先在这个文件 中进行配置：

```
'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\Models\User::class, //尤其是laravel框架刚下载的时候，自带User.php是放在Models目录外的，这里需要注意→_→
        ],

        // 如果你使用的是DB而非ORM，那就配置如下：
        // 'users' => [
        //     'driver' => 'database',
        //     'table' => 'users',
        // ],
    ],
```



下一步：创建Controller

> 据说可以不创建Controller，直接在路由中添加如下并创建对应视图文件就好：

>```php
>Route::get('auth/login', 'Auth\AuthController@getLogin');
>Route::post('auth/login', 'Auth\AuthController@postLogin');
>Route::get('auth/logout', 'Auth\AuthController@getLogout');
>```

详情不介绍，这种方法直接参考 [Laravel-china/Auth](http://laravel-china.org/docs/5.1/authentication) 提供的文档。



```php
# LoginController
namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class LoginController extends Controller
{
    /**
     * Login Page
     */
    public function getLogin()
    {
        return view('login.login');
    }

    /**
     * Login Logic
     * @param Request $request
     * @return \Illuminate\Http\RedirectResponse|\Illuminate\Routing\Redirector
     */
    public function postLogin(Request $request)
    {
        $params = $request->all();
        if (Auth::attempt(['username'   => $params['username'],'password' => $params['password']])) {
            $user = Auth::user();
            var_dump($user);
        } else {
            return redirect('/login/login');
        }
    }
}
```



并创建对应的 login.blade.php 视图文件。

注意验证方法： 

```php
Auth::attempt(['username'   => $params['username'],'password' => $params['password']]
```

这个Auth是facade实现的。

具体方法定义位置：	

```php
# ~/vendor/laravel/framework/src/Illuminate/Auth/SessionGuard.php
/**
 * Attempt to authenticate a user using the given credentials.
 *
 * @param  array  $credentials
 * @param  bool   $remember
 * @param  bool   $login
 * @return bool
 */
public function attempt(array $credentials = [], $remember = false, $login = true){
  //略
}
```

这个方法首先用你提供的username/email 、password等字段判断用户登录信息是否正确，如果正确的话会继续将登录信息存在session中。

当第二个参数传 `$remember = true` 时，它还负责进行“记住账号”,这里会用到 `users` 表中的 `remember_token` 字段。

第三个参数 `$login` ，看了下实现方法，`$login = true` (默认条件)时，程序会更新登录信息 `session` 和根据 `$remember`  判断是否记住账号。`false` 时不进行这些操作，只返回登录信息是否正确。所以这个 `$login` 应该是在改密码判断旧密码时，才会用到 `false` 的情况~具体是否如此，留待做密码修改操作时再来看。



然后，看下Controller中判断登录之后的处理：

```php
$user = Auth::user();
```

这个，Auth中提供的获取当前登录用户基本信息的方法。返回值是个User对象。



#### 二、部分页面判断是否登录

好了到这里登录操作完成，我们来给原来的学生信息账号加上登录判断：如果没有登录，请跳转到登录页~



```
Waiting......
```



