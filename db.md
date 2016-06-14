# Laravel学习笔记（二） Eloquent ORM 

Eloquent ORM 是 Laravel 自带的 ORM , 提供了一个优雅的、简单的 数据库 ActiveRecord 实现。每一个数据库的表有一个对应的 "Model" 用来与这张表交互。葡萄的Kerisy框架中的ORM直接采用了Eloquent. 

当然，Laravel除了ORM之外还提供了一套可以直接使用原生SQL来处理数据库操作的接口，需要学习的同学直接点击 [Laravel DB 传送门](http://laravel-china.org/docs/5.1/database), 这里只介绍关于Eloquent ORM的一些常见用法。

#### 一、数据库配置

###### 1. .env & database.php

laravel项目根目录下，有一 `.env` 文件，其格式为 一行一配置， `KEY=VALUE`

其中 ，以 `DB_` 开头的KEY，为数据库相关配置

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=flaravel_db
DB_USERNAME=root
DB_PASSWORD=root
```

然后，找到文件 `app/config/database.php` 这里才是数据库配置文件所在地。

```php
'default' => env('DB_CONNECTION', 'mysql'),
'connections' => [
    'mysql' => [
        'driver' => 'mysql',
        'host' => env('DB_HOST', 'localhost'),
        'port' => env('DB_PORT', '3306'),
        'database' => env('DB_DATABASE', 'flaravel_db'),
        'username' => env('DB_USERNAME', 'root'),
        'password' => env('DB_PASSWORD', 'root'),
        'charset' => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix' => 'fl_', //统一数据表前缀
        'strict' => false,
        'engine' => null,
    ]
]
```

如上，这里 `env` 方法，就是在取 `.env` 文件中的配置信息。当然你也可以直接在 这个文件中修改配置项，不去读取 `.env` 文件的内容。

另外，在实际使用中，一个项目可能会需要连接多个数据库，或者主从库等等，这时候，只需在 `connections` 数组中，继续往后加就好了， 格式 同 `mysql`, 譬如：

```
'default' => 'mysql',
'connections' => [
    'mysql' => [
        'driver' => 'mysql',
        'host' => env('DB_HOST', 'localhost'),
        'port' => env('DB_PORT', '3306'),
        'database' => env('DB_DATABASE', 'flaravel_db'),
        'username' => env('DB_USERNAME', 'root'),
        'password' => env('DB_PASSWORD', 'root'),
        'charset' => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix' => 'fl_',
        'strict' => false,
        'engine' => null,
    ],
    'slave'   => [
        'driver' => 'mysql',
        'host' => env('DB_HOST', 'localhost'),
        'port' => env('DB_PORT', '3306'),
        'database' => env('DB_DATABASE', 'flaravel_db'),
        'username' => env('DB_USERNAME', 'root'),
        'password' => env('DB_PASSWORD', 'root'),
        'charset' => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix' => 'fl_',
        'strict' => false,
        'engine' => null,
    ]
]
```

这里，`mysql` 和 `slave` 就是两个 `connection` 的 `connection_name`。这里留坑，下面 介绍 Model 的时候在讲如何用。

###### 2.创建Model

为了便于维护，现在 `app` 目录下，创建 `Models` 目录，用于 Model 文件的管理

然后，在数据库 `flaravel_db` 中，创建表 `fl_students`

```sql
CREATE TABLE `fl_students` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT 'ID as 学号',
  `name` varchar(30) NOT NULL DEFAULT '' COMMENT '姓名',
  `sex` tinyint(3) NOT NULL DEFAULT 1 COMMENT '性别：1-男，0-女',
  `age` tinyint(3) NOT NULL DEFAULT 0 COMMENT '年龄',
  `grade` tinyint(3) NOT NULL DEFAULT 1 COMMENT '年纪',
  `class` tinyint(3) NOT NULL DEFAULT 1 COMMENT '班级',
  `created_at` int(11) NOT NULL DEFAULT 0 COMMENT '创建时间',
  `updated_at` int(11) NOT NULL DEFAULT 0 COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='学生信息表';
```

再在Models中，创建文件 `Student.php`

```php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;
class Student extends Model
{
    //
}
```

默认情况下：

（1）这个 model 对应的表明是 `students` ，即 className 的复数。或者使用 `table` 属性自定义。      
（2）这个表的主键是 `id` , 或者使用 `primaryKey` 属性自定义。      
（3）这个表会在增、改操作时，自动更新 `created_at` & `updated_at` 两个字段，或者使用 `timestamps` （bool） 字段来控制开关。          
（4）上一条中自动更新的时间字段为 `datetime` 格式，可以设置 `dateFormat` 属性值为 `'U'` ，将其格式改为时间戳。              
（5）若 `database.php` 中配置了多个 `connection`, 这里默认的取的 `mysql`， 可设置 `connection` 属性，自定义对应的数据库链接。

>	第（5）条中提到，默认取 `mysql` 对应的链接，原因可查看 `vendor/framework/src/Illuminate/Database/DatabaseManager.php` 中，`getDefaultConnection` 方法的定义。也可自行修改。

So，这个 Model 的相对常用的完整版是可以改为如下：

```
namespace App\Models;
use Illuminate\Database\Eloquent\Model;
class Student extends Model
{
    protected $connection = 'mysql';
    protected $primaryKey = 'id';
    protected $table = 'students';
    protected $dateFormat = 'U';
    public $timestamps = true;
}
```

对于相对成熟的多数据库业务，可以针对不同的数据库，各自封装一层基类，将connection/dataFormat等通用的属性定义在基类中，避免重复劳动，提高代码有效复用。

#### 二、Eloquent ORM的基本语法

注：以下调用方法，可以用静态调用，也可以将 `User` 类实例化之后用 `$model` 对象调用。

###### 1.增（add）
```
# 方法一
$data = [
	'username'	=> 'name',
	'sex'		=> 'boy'
];
$userId = User::add($data);

# 方法二
$obj = new User;
$obj->username = 'name';
$obj->sex = 'boy';
$obj->save();
$userId = $obj->id
```
###### 2.删（delete）

```php
# 删除方法一：find/where & delete
User::find(1)->delete();
User::where('id', 1)->delete();

# 删除方法二：destroy【主键】
User::destroy(1);
User::destroy(1,2,3);
User::destroy([1,2,3]);
```

关于删除，上面两种方法都是直接从数据库中删除掉该记录，但是一般生产环境，我们大多数删除操作都是以某个字段标记数据状态为已删除、未删除等。EloquentOrm提供了一系列处理这种删除的方法，我们称之为“软删除”。它所使用的，标记“软删除”状态的字段，是 `deleted_at` ，即删除时间。此字段默认为空(NULL)，若 ` deleted_at not null ` 则证明该记录已被软删除。

Model声明：
```php
class User extends Model 
{
	use SoftDeletes;
    /**
     * 需要被转换成日期的属性。
     *
     * @var array
     */
    protected $dates = ['deleted_at'];
}
```

此处关于 `deleted_at` 字段：laravel默认将之处理为 datetime 类型， 若在model中设置属性：`protected $dateFormat = 'U';` , 则会将至处理为时间戳。

关于软删除的一系列操作：

```php
$user = find(1);
# 删除
$user->softDeletes();

# 判断是否软删除
if ( $user->trashed() ) {
	//
}

# 查询
## 默认where查询，会过滤掉已软删除的数据，若想查出，则需如下
User::withTrashed()// 强制查找已被软删除的模型，包括软删除和未软删除的所有数据
	->where('id', '>', 0)
	->get();
## 仅查询被软删除的
User::onlyTrashed()// 强制查找范围为已软删除的数据	->where('id', '>', 0)
	->get();

# 回复软删除的数据
$user->restore();

# 彻底删除已被软删除的数据
$user->forceDelete();
```

###### 3.查（select）

Model定义时可以设置主键 `$primaryKey` ，默认为 `id`

```php
# 查询方法一：根据主键查询一条
$userInfo = User::find($id);

# 查询类型二：条件查询 - 列表
$list = User::where('username', 'name')
	->where('created_at','<',time() - 3600) //不等条件
	->orderBy('id', 'desc')	//排序
	->take(3)	//limit
	->skip(2)	//offset
	->get();  //取出结果并返回

# 查询类型三：条件查询 - 一条结果
$user = User::where('username', 'name')
	->where('created_at','<',time() - 3600) //不等条件
	->orderBy('id', 'desc')	//排序
	->first();
	
# 查询类型三：返回全部结果
$list = User::all();

# 查询类型四：一些数字的查询
$count = User::where('key','value')->count();
$total = User::count();
$sumAge = User::where('key','value')->sum('age');
$maxAge = User::max('age');
$minAge = User::min('age');
```

这里注意，查出来的结果除了第四项都是int型之外，其他都是 Object 类型的，如需使用数组，则结果继续调用 `->toArray()` 方法即可。但是调用之前必须确保结果非空，否则会报错。

###### 4.改（update）

```php
# 修改方式一： Object
$user = User::find(1);
$user->username = 'newname';
$user->increment('age', 5);
$user->save();

# 修改方式二：Object
User::where('id', 1)->update(['nickname' => 'newname']);
```

#### 三、ORM 提升

###### 1.查询自动过滤字段

```php 
class User extends Eloquent {
	...
	...
}
```

###### 2.查询结果404

```php
$model = App\Flight::findOrFail(1);

$model = App\Flight::where('legs', '>', 100)->firstOrFail();
```

如此，在找不到符合条件的数据时，会抛出 `HTTP 404` 异常给用户