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
  `sex` tinyint(3) NOT NULL DEFAULT 1 COMMENT '性别：1-男，2-女',
  `birthday` date NOT NULL DEFAULT '0000-00-00' COMMENT '生日',
  `grade` int(5) NOT NULL DEFAULT 0 COMMENT '入学年份',
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
$userId = User::create($data);

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

# 查询类型四：与或条件复杂
User::where('key','value')->Where(function($query){
	$query->where('k1','v1')->orWhere('k2','v2');
})->get();
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

###### 3.关联模型

`EloquentORM` 还依赖其强大的 [查询语句构造器](http://laravel-china.org/docs/5.1/queries) 提供了同样强大的外键查询功能。传送门：[关联查询](http://laravel-china.org/docs/5.1/eloquent-relationships)

设计如下四张表

>|rooms|||students|||subjects|||chooses||
|:--|:--|:--|:--|:--|:--|:--|:--|:--|:--|:--|
|字段|意义| - |字段|意义| - | 字段|意义 | - |字段|意义|
|room_id|宿舍号||student_id|学号||subject_id|科目ID||id|关联ID|
||||room_id|所属宿舍|||||student_id|选课学生|
||||||||||subject_id|所选课程|

`rooms` 表，记录宿舍信息, 以 `room_id` 宿舍号为索引            
`students` 表， 记录学生信息， 以 `student_id` 学号为索引， `room_id` 记录所属宿舍                
`subjects` 表， 记录课程信息， 以 `subject_id` 课程ID为索引       
`choosees` 表， 主键ID无意义， 以 `student_id` 和 `subject_id` 关联选课关系

故而：

	Rooms 		(1) ---------------- (n) Students 
	Students 	(n) ---------------- (n) Subjects
	
先做一批测试数据，初始化数据的代码在 `routes.php` 中，`/try/init/`

*	一对一关系：`HasOne` & `BelongsTo` 
*  一对多关系：`HasMany` & `BelongsTo`

一对一和一对多比较类似，以一对多为例介绍。

Rooms 与 Students 是一对多关系，面上看来，一个 `room` 会 `HasMany` `Students`,反之，每个 `student` 都 `belongsTo` 一个 `Room`

```
# Model Room
public function students()
{
    return $this->hasMany('App\Models\Student');
}

# Model Student 
public function room(){
    return $this->belongsTo('App\Models\Room');
}
```
	
`hasMany` 和 `belongsTo` 的参数是相似的，第一个是对应的 `Model`,第二个是外键名称，默认为表名的单数形式(不带前缀)加`_id`。比如此处省略了第二个参数`room_id`。

调用如下：

```
# routes.php
Route::get('/try/hasmany', function() {
    $room = \App\Models\Room::find(3);
    var_dump($room->students->toArray());
});

Route::get('/try/belongsto', function() {
    $data = \App\Models\Student::find(1);
    var_dump($data->room->toArray());
});
```

`belongsTo` 方法，还可以配合 `with` 进行查询，叫做“预加载”：

```
Route::get('/try/with', function() {
    $data = \App\Models\Student::with('room')->whereIn('id',[1,2,3])->get();
    foreach ($data as $v) {
        var_dump($v->toArray());
    }
});
```

这样打印出来，结果如下：

```

array(11) {
  ["id"]=>
  int(1)
  ["name"]=>
  string(5) "Silov"
  ["sex"]=>
  int(1)
  ["birthday"]=>
  string(10) "1999-10-10"
  ["grade"]=>
  int(2016)
  ["class"]=>
  int(2)
  ["created_at"]=>
  string(10) "1465986707"
  ["updated_at"]=>
  string(10) "1467603414"
  ["deleted_at"]=>
  NULL
  ["room_id"]=>
  int(1)
  ["room"]=>
  array(3) {
    ["room_id"]=>
    int(1)
    ["created_at"]=>
    string(10) "1466496573"
    ["updated_at"]=>
    string(10) "1466496573"
  }
}
```


* 多对多关系，相对复杂，因为还有一个中间关联表

```
# Model Subject
public function students()
{
    return $this->belongsToMany('App\Models\Student','subject_choose', 'subject_id', 'student_id');
}
```
参数：Model、中间表名（不带前缀）、本表关联字段、对面表关联字段

调用方法：

```
# routes.php
Route::get('try/belongstomany', function() {
    $data = App\Models\Subject::find(1);
    foreach($data->students as $student){
        var_dump($student->toArray());
    }
});
```

单次循环打印结果输出如下：

```
array(11) {
  ["id"]=>
  int(1)
  ["name"]=>
  string(5) "Silov"
  ["sex"]=>
  int(1)
  ["birthday"]=>
  string(10) "1999-10-10"
  ["grade"]=>
  int(2016)
  ["class"]=>
  int(2)
  ["created_at"]=>
  string(10) "1465986707"
  ["updated_at"]=>
  string(10) "1467603414"
  ["deleted_at"]=>
  NULL
  ["room_id"]=>
  int(1)
  ["pivot"]=>
  array(2) {
    ["subject_id"]=>
    int(1)
    ["student_id"]=>
    int(1)
  }
}
```

其中最后一个Key：`pivot`, 是关联表 `subject_choose` 的内容。

所以循环中如果想获取中间表的某个数据，比如关联关系建立时间，可以这样：

```
echo $student->pivot->created_at;
```

除此之外还有一种[远层一对多](http://laravel-china.org/docs/5.1/eloquent-relationships#%E8%BF%9C%E5%B1%82%E4%B8%80%E5%AF%B9%E5%A4%9A)关系，即比如：

一个 room ---- 多个students --- 跟 student一对一的床位

如果想直接根据宿舍查床位信息，则可以在 `Room` 使用 `hasManyThrough` 方法，通过中间一层表来实现的一对多关系。
这里不给出具体的方法和代码，详情自行参考官方文档~

###### 4.多态关联 

所谓多态关联，简单点理解就是某个外键所对应的表可能不止一个，由另外一个type一类的字段来判断这个外键对应哪张表。

比如：

如果给学生和课程，都加上图片，所有的图片都存在一张表里。

则图片表如下：

>|字段|类型|备注|
|:--|:--|:--|
|id|int(11)|图片ID|
|path|varchar(100)| 图片地址|
|bind_type|varchar(30)| ModelName,比如：App\Models\Student|
|bind_id|int(11)|外键：student\_id/subject\_id|

```
CREATE TABLE `fl_images` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '图片ID',
  `path` varchar(100) NOT NULL DEFAULT '' COMMENT '图片链接',
  `bind_type` varchar(30) NOT NULL DEFAULT '' COMMENT 'model name',
  `bind_id` int(11) NOT NULL DEFAULT '0' COMMENT 'student_id or subject_id',
  `created_at` int(11) NOT NULL DEFAULT '0' COMMENT '创建时间',
  `updated_at` int(11) NOT NULL DEFAULT '0' COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='图片';
```

Model中绑定模型的方法：

```php
# Model Image
public function bind()
{
    return $this->morphTo();
}
# Model Student & Model Subject 
public function images()
{
    return $this->morphMany('App\Models\Image', 'bind');
}
```

调用数据的方法类似上文的 `hasMany` 和 `belongsTo`

```php
# routes.php 
# 查看某个学生的照片
Route::get('try/morphto', function() {
    $student = \App\Models\Student::find(13);
    foreach($student->images as $image) {
        var_dump($image->toArray());
    }
});

# 查看某张照片的宿主信息，学生/科目
Route::get('try/bind', function() {
    $image = \App\Models\Image::find(1);
    var_dump($image->bind->toArray());
});
```

多态多对多关联，比多态关联和多对多关系更复杂一层。具体的例子参考[中文官网文档：多态多对多关系](http://laravel-china.org/docs/5.1/eloquent-relationships#%E5%A4%9A%E6%80%81%E5%A4%9A%E5%AF%B9%E5%A4%9A%E5%85%B3%E8%81%94)

###### 5.关联查询

关联关系可以作为查询条件来查找符合条件的数据。

比如，上文提到宿舍和学生的一对多关系，可以查找哪些宿舍有学生、哪些宿舍学生有几个等等。

```php
# 获取学生数量不为0的宿舍信息
$data = \App\Models\Room::has('students')->get();
# 获取学生数量超过4个的宿舍
$data = \App\Models\Room::has('students', '>', 4)->get();
```

或者，可以查询某个学生所在宿舍的信息

```php
# 获取学生名字叫做 野原新之助 所在的宿舍信息
$data = \App\Models\Room::whereHas('students', function($query){
    $query->where('name','野原新之助');
})->first();
```

###### 6.预加载

效率&查询次数！非常有用！

举个例子，前面提到过一个 `with` 方法的预加载

```
# 预加载模式
Route::get('/try/preload', function() {
    $data = \App\Models\Student::with('room')->whereIn('id', [1,5,10,16])->get();
    var_dump($data->toArray());
    foreach ($data as $v) {
        var_dump($v->room->toArray());
    }
});

# 非预加载模式
Route::get('/try/unpreload', function() {
    $data = \App\Models\Student::whereIn('id', [1,5,10,16])->get();
    var_dump($data->toArray());
    foreach ($data as $v) {
        var_dump($v->room->toArray());
    }
});
```

两种方式结果对比可以发现，`foreach` 循环部分输出的内容是一样的，但是循环前面的 `var_dump` 输出的就不一样：预加载方法中，每条数据都多了个 `room` 字段。这就是差别了,两种方法的查询本质是这样的：

```sql
# 预加载
select * from fl_students;
select * from fl_rooms where id in(1,2,3,4);

# 非预加载模式
select * from fl_students;
//foreach:
	select * from fl_room where id=1/2/3/4;
```

以上可以看出，在这个例子中仅依靠 `belongsTo` 和 `hasOne`/`hasMany` 这类方法实现的关联数据查询，时间复杂度O(n)，而加上预加载之后就变成了 O(1)。而众所周知PHP最大的瓶颈就在Mysql查询，所以预加载有效地减少了查询次数，提高了查询的效率。

预加载还有多种模式，比如：`App\Models\Student` 中同时定义了`room`、`images` 两种关联模型，那么就可以同时预加载两组数据：

```php
Route::get('/try/preload/students', function() {
    $data = \App\Models\Student::with('room','images')->get();
    var_dump($data->toArray());
});
```

反过来，查询宿舍信息时，预加载一个宿舍所有人的头像信息，这叫做嵌套预加载：

```
Route::get('/try/preload/roomimages', function() {
    $data = \App\Models\Room::with('students.images')->find(1)->toArray();
    var_dump($data);
});
```

输出格式是这样的

```
[
	'room_id' 	=> 1,
	....
	'students'	=> [//该宿舍学生信息
		....
		1	=> [
			...
			'images'	=> [
				//图片信息
			]
		]
		....
	]
]
```


预加载也是可以有条件的加载，比如，查询宿舍的时候，预加载某个人的个人信息：

```
Route::get('/try/preload/search', function() {
    $data = \App\Models\Room::with(['students' => function($query){
        $query->where('name', '野原新之助');
    }])->get()->toArray();
    var_dump($data);
});
```

###### 7.延迟预加载

实际使用的情况可能比较多，比如：预加载的数据不一定有用，不使用预加载的话，循环一次次查询又太耗时。有没有折中的办法？

都说到这里了，肯定是有的，那就是延迟预加载：

比如原来预加载：

```
$data = \App\Models\Student::with('room','images')->get();
```

延迟写法：

```
$data = \App\Models\Student::all();
if (#判断是否需要预加载数据) {
    $data->load('room', 'images');
}
```

`load` 方法也可以设置预加载条件，如官网文档例子：

```
$books->load(['author' => function ($query) {
    $query->orderBy('published_date', 'asc');
}]);
```

###### 8.Scopes查找范围

假设我们有一个文章列表，其中有个 `is_published` 字段，标明文章是否发布。

那么取出已发布的文章是这样的：

```
Post::where('is_published', true)->get();
```

其中 `where('is_published',true)` 这个条件，可能在很多查询的时候重复出现。

根据 **Don't Repeat Yourself** 的原则， `Laravel` 提供了一个预设查询范围的方式。

```
class Post extentds Model
{
	public function scopePublished($query)
	{
		return $this->where('is_published',true);
	}
}
```
然后所有需要带着个条件的查询都可以直接使用 `published` 方法：

```
Post::published()->get();
```

###### 9.预加载指定字段


