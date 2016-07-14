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
  `email` varchar(100) NOT NULL DEFAULT '' COMMENT '邮箱',
  `password` varchar(64) NOT NULL DEFAULT '' COMMENT '密码',
  `nickname` varchar(20) NOT NULL DEFAULT '' COMMENT '昵称',
  `type` tinyint(2) NOT NULL DEFAULT '2' COMMENT '管理员类型 1-超级管理员，2-普通管理员',
  `remember_token` varchar(62) NOT NULL DEFAULT '' COMMENT 'token',
  `created_at` int(11) NOT NULL DEFAULT '0' COMMENT '创建时间',
  `updated_at` int(11) NOT NULL DEFAULT '0' COMMENT '最后更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_email` (`email`),
  UNIQUE KEY `idx_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='账号表';
```

Model 直接借用框架中的 `User` 类，注意 move 到 Models 目录下之后改一下 `namespace` 。

在开始写登录逻辑之前先来聊另一件事：密码。

一般来说，简单点我们会把用户密码进行加密，比如Md5或者哈希加密。而 Laravel 本身已经提供了较为可靠的哈希加密和校验方法：`Hash` 类。

```
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

