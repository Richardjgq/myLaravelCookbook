# README

----------------

## Part I ： 我的LaravelCookBook

#### 一、关于读者群体

Laravel 官方号称本框架为：The PHP Framework For Web Artisans -- 为 WEB 艺术家创造的 PHP 框架。

然而这个框架学起来并不是特别容易上手，尤其是对一个像我这样PHP技能点并不算高的人而言 -- 如果你直接看官方文档的话，可能会绕些路子。所以这里先提一句话，谁适合看接下来的东西：

>   有一定PHP基础、对PHP框架有所了解但并不高深的PHP工程师。

谁不适合看：

>   (1) PHP初学者，尚且不知道PHP框架是什么。这类人看接下来的文档可能会比较吃力，不过配合搜索引擎应该还是可以理解的。

>   (2) PHP大牛。请直接移步官方文档。这里的文档只是一个普通PHP开发者的自我学习过程，对你可能不会太有帮助。

OK，差不多就是一个半吊子的人写的东西，仅供半吊子的人读。

PS：半吊子的东西自然也很容易有出错的地方，欢迎一起讨论交流~     
个人QQ：1136652998

***补充于2016、7、14：***    
***再次强烈建议，学PHP框架之前，请先搞懂什么叫面向对象编程，语言基础都不够的话学框架会心累***

#### 二、关于参考文章

>   * [Laravel中文官网](http://www.golaravel.com/)

>   * [JohnLui的系列博客 ： Laravel 5 系列入门教程【最适合中国人的 Laravel 教程】](https://lvwenhan.com/laravel/432.html)

>   * [JohnLui的系列博客 ： 深入理解 Laravel Eloquent](https://lvwenhan.com/laravel/421.html)

>   * 安居客 一些前辈整理在内部GitLab上的资料

#### 三、关于环境

>   Mac OS EI Capitan 10.11.5

>   Nginx   v1.8.1

>   PHP     v7.0.6

>   MySQL   v5.7.12

>   Laravel v5.1

#### 四、关于内容

    不能保证纯原创。毕竟文档来源是本人的学习过程中的心得，如果有些解决问题的方案有现成的、前辈整理好的，我可能会直接copy过来做笔记用。

    有参考会备注，所以如需转载，请备注来源。

------------------------------

# Part II ： 文档目录

一、框架搭建

* [看到第一个页面](./route.md#一第一个laravel页面)

* [简单的路由配置](./route.md#二laravel路由的简单应用)

* [路由到Controller](./route.md#三路由到Controller)

* [Controller到view](./route.md#四Controller到view)

* [Post & \_csrfVerify](./route.md#2表单--_csrf验证)

* [表单验证](./route.md#3表单校验)

二、数据库配置 & EloquentORM

* [.env文件 & databse.php](./db.md##1-env--databasephp)

* [创建Model类](./db.md#2创建model)

* [简单增删查改](./db.md#二eloquent-orm的基本语法)

* [部分高级应用](./db.md#三orm-提升)
	
	* [关联模型](./db.md#3关联模型)             
	
	* [多态关联](./db.md#4多态关联)                 
	
	* [关联查询](./db.md#5关联查询)
	
	* [预加载](./db.md#6预加载) 
	
	* [延迟预加载](./db.md#7延迟预加载)	         

三、中间件/拦截器

* AuthMiddleware : 直接参考[实战篇-账户系统部分](./student.md)

四、所谓艺术家：Artisan脚手架

* Artisan基本功能介绍

五、脚本 

* 脚本部署 : 直接参考[实战篇-脚本程序部分](./student.md)

六、实战篇：一个简易学生信息管理系统

* [新增学生](./student.md#4新增学生页面)

* [数据校验](./student.md#5表单数据校验)

* [Blade模板简介](./student.md#6Blade模板简介)

* [使用Bootstrap美化页面](./student.md#7使用Bootstrap美化页面)

* [学生列表查询](./student.md#8学生列表查询)

* [两种删除：休学与退学](./student.md#9删除学生休学与退学)

* [修改学生信息](./student.md#10编辑学生信息)

七、实战篇二：账户系统

* [登录](./account.md)

* 中间件校验用户权限

* 忘记密码邮件 

* 权限系统

* 脚本程序