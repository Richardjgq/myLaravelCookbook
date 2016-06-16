# Laravel 实战练习：学生信息管理系统

环境搭建、数据库配置依旧使用该学习文档中的一套，so，我的本地开发环境域名还是：http://flaravel.me

#### 一、基础功能

###### 1. 设计学生信息表

```sql
CREATE TABLE `fl_students` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT 'ID as 学号',
  `name` varchar(30) NOT NULL DEFAULT '' COMMENT '姓名',
  `sex` tinyint(3) NOT NULL DEFAULT 1 COMMENT '性别：1-男，2-女',
  `birthday` date NOT NULL DEFAULT '0000-00-00' COMMENT '生日',
  `grade` tinyint(3) NOT NULL DEFAULT 1 COMMENT '年纪',
  `class` tinyint(3) NOT NULL DEFAULT 1 COMMENT '班级',
  `created_at` int(11) NOT NULL DEFAULT 0 COMMENT '创建时间',
  `updated_at` int(11) NOT NULL DEFAULT 0 COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='学生信息表';
```

（1）依旧 前缀 `fl_`, 数据库继续使用 `flaravel_db`        
（2）`created_at` . `updated_at` 字段，记录数据变化的时间，必备    

###### 2.创建Model

不确定我们的系统慢慢会不会需要两个数据库的程度，但是为了良好的可拓展性和代码的高效重用，还是先设计一层Model基类，定义一些常规属性 `Models/BaseModel.php`：

```php
<?php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;
class BaseModel extends Model
{
    protected $connection = 'mysql';
    protected $dateFormat = 'U';
    public $timestamps = true;
}
```

继续，创建 `Models/Student.php` :

```php
<?php
namespace App\Models;
class Student extends BaseModel
{
    protected $primaryKey = 'id';
    protected $table = 'students';
}
```

可以看到这里我们继承的是BaseModels，而BaseModels中定义的属性除非特殊也不再重新定义。

###### 3.创建Controller，并添加统一route 

创建Controller文件 `app/Http/Controllers/StudentController.php` ：

```php
<?php
namespace App\Http\Controllers;
use Illuminate\Http\Request;
use App\Http\Requests;
class StudentController extends Controller
{
    //
}
```

或者使用脚手架，在根目录下执行命令行：

```shell
php artisan make:controller StudentController
```

然后，在 `routes.php` 中添加对该Controller的监听：

```php
Route::controller('student', 'StudentController');
```

OK,添加完成，可在Controller中添加如下方法看是否可以访问：

```php
public function getIndex(){echo "Student.Index";}
```
然后访问 http://flaravel.me/student/ 看能否打印出内容就好了~

###### 4.新增学生页面

首页需要一个数据入口页面：

```php
# StudentController
public function getAddv()
{
    return view("student.addv");
}
```

`resources/views` 目录下，创建 `student` 子目录，并创建 `student/addv.blade.php` 文件

内容如下：

```html
<!DOCTYPE html>
<html>
<head>
    <title>Add Student - Students's Info Manage System</title>
</head>
<body>
<div class="container">
    <div class="content">
        <h3>新增学生</h3>
        <hr>
        <form method="post" action="/student/add">
            <label>姓名:</label>
            <input type="text" name="name" /><br>
            <label>性别:</label>
            <select name="sex">
                <option value="1">男</option>
                <option value="2">女</option>
            </select><br>
            <label>生日:</label>
            <input type="text" name="birthday"/><br>
            <label>年级:</label>
            <input type="text" name="grade"/><br>
            <label>班级:</label>
            <input type="text" name="class"/><br>
            <input type="submit" value="提交">
            {{csrf_field()}}
        </form>
    </div>
</div>
</body>
</html>
```

如此，访问 `http://flaravel/student/addv` ，页面如下：

![addv_origin](./images/shoot7.jpg)

然后，再在Controller中添加一个方法 `postAdd(Request $request)` ：

```php
public function postAdd(Request $request)
{
    $params = $request->all();

    $student = new Student;
    $student->name  = $params['name'];
    $student->sex   = $params['sex'];
    $student->birthday  = $params['birthday'];
    $student->grade     = $params['grade'];
    $student->class     = $params['class'];
    $student->save();
    var_dump($student->toArray());
}
```
使用Model `Student` ,记得添加`use` : 

```
use App\Models\Student;
```

然后，你可以在页面上填写数据（尚未做数据校验，请填写合法数据）并提交，页面显示是这样的：

![submit_add](./images/shoot8.jpg)

忽略第一行的 `var_dump` 提示,会发现返回的内容是你提交之后数据库里整个保存的样子：`id/updated_at/created_at` 等字段均已自动补齐。这表明，数据已经成功存储到数据库中了~

But。。。。

个人提议，最好将创建数据的过程，在Model中重新封装一下，我是这么做的：

```php 
# app/Models/Student.php
public function addStudent($name, $sex = 1, $birthday = '0000-00-00', $grade = 1, $class = 1)
{
    $student = new static;
    $student->name      = $name;
    $student->sex       = $sex;
    $student->birthday  = $birthday;
    $student->grade     = $grade;
    $student->class     = $class;
    $student->save();
    return empty($student->id) ? false : $student->toArray();
}
```

so，`StudentController` 中的 `postAdd` 方法变成了这样：

```php
public function postAdd(Request $request)
{
    $params = $request->all();
    $studentModel = new Student();
    $student = $studentModel->addStudent($params['name'], $params['sex'], $params['birthday'], $params['grade'], $params['class']);
    var_dump($student);
}
```

看起来简单了不少。。。

But，上面说了，没做数据校验，那，校验一下怎么做呢？

###### 5.表单数据校验

文档传送门：[Laravel 表单验证](http://laravel-china.org/docs/5.1/validation)

这里，非AJAX请求的话，推荐一种方式：

```php
public function postAdd(Request $request)
{
	//定义数据校验规则
	$rules = [
		'name'  => 'required|max:30',
		'sex'   => 'required|integer|between:1,2',
		'birthday'  => 'required|date',
		'grade' => 'required|int',
		'class' => 'required|int'
	];
	//校验数据
	$this->validate($request, $rules);
	
	//校验通过后的数据处理
	...
}
```

其中 `rules` 中的数据校验规则，参考上面Laravel表单验证传送门的列表。

这里可以查看 `ValidatesReuqest` 中 `validate` 方法的定义，再处理失败时，会直接抛出异常，并返回数据来源页面。

所以，在 `student.addv` 这个页面上，还要做两件事：

* 显示数据校验错误信息 : validator 已经定义的 $error 对象
	
	```
	@if (count($errors) > 0)
        <div class="alert alert-danger">
            <ul>
                @foreach ($errors->all() as $k=>$error)
                    <li>{{$k}}:{{ $error }}</li>
                @endforeach
            </ul>
        </div>
        <hr>
    @endif
	```
* 回填数据 : `old` 方法
	
	```
    <input type="text" name="name" value="{{ old('name') }}"/>
	```

校验失败后页面如下：

![validate_err](./images/shoot9.jpg)

OK，下一步，调整顺序：将不合法数据信息，显示在对应的输入位置:

```
<input type="text" name="name" value="{{ old('name') }}"/>
{{$errors->first('name')}}
```

如此，新增数据包括数据校验的过程就完成了。

###### 6.Layout & 使用 Bootstrap 美化页面

我们的页面中，像 `<head>` 这一类的标签，以及可能会有的头部导航栏等等，几乎所有页面都会出现的东西，没必要每个页面写一次。而且，目前这样的页面，也。。。着实有点丑。。。

