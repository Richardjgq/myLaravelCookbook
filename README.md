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

#### 二、关于参考文章

>   * [Laravel中文官网](http://www.golaravel.com/)

>   * [JohnLui的系列博客 ： Laravel 5 系列入门教程【最适合中国人的 Laravel 教程】](https://lvwenhan.com/laravel/432.html)

>   * [JohnLui的系列博客 ： 深入理解 Laravel Eloquent](https://lvwenhan.com/laravel/421.html)

>   * 安居客 一些前辈整理在内部GitLab上的资料

#### 三、关于环境

>   Mac OS EI Capitan 10.11.5

>   Nginx   v1.8.1

>   PHP     v5.6.17

>   MySQL   v5.7.12

>   Laravel v5.1

#### 四、关于内容

    不能保证纯原创。毕竟文档来源是本人的学习过程中的心得，如果有些解决问题的方案有现成的、前辈整理好的，我可能会直接copy过来做笔记用。

    有参考会备注，所以如需转载，请备注来源。

------------------------------

# Part II ： 文档目录

一、框架搭建

* [看到第一个页面](./route.md#route-firstview)

* [简单的路由配置](./route.md)

* [路由到Controller](./route.md)

* [Post & \_csrfVerify](./route.md)

二、数据库配置 & EloquentORM

* [.env文件 & databse.php](./db.md)

* [简单增删查改](./db.md)

* [部分高级应用](./db.md)

三、中间件/拦截器

* [AuthMiddleware](./Middleware.md)

四、所谓艺术家：Artisan脚手架
