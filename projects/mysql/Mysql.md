
## 一、不符合密码策略

```
SHOW VARIABLES LIKE 'validate_password%';
```

2、首先需要设置密码的验证强度等级，设置 validate_password_policy 的全局参数为 LOW 即可

```
set global validate_password_policy=LOW;
set global validate_password_length=6;
```

3、

```
grant all privileges on *.* to root@"%" identified by "123456";
```

4、

```
FLUSH PRIVILEGES;
```



## 二、函数

### 1.创建函数

1.列名

2.赋予用户的可使用权限



## 三、创建用户

### 1、创建用户

create user 用户名@'IP地址' IDENTIFIED BY '密码';注意：'IP地址'可以设置为localhost(代表本机) 或者 '%'(代表允许所有IP地址登录)，示例：

```
create user xxljob@'%' identified by 'xxljob';
```

### 2、删除用户

drop user 用户名 @'IP地址';

```
drop user lotus @'localhost' 注意：'IP地址'可以设置为localhost(代表本机)或者'%'(代表允许所有IP地址登录)
```

### 3、grant用户授权

```
权限列表：
all或者all privileges --------所有权限
usage  				 ------无权限
select,update,insert,create,drop --------个别权限
select,update(字段1,....，字段N)----指定字段

库名
*.*    -----所有库所有表
库名.* -------一个库
库名.表名 ------一张表

客户端地址
%                 ----所有主机
192.168.2.%       ----网段内的所有主机
192.168.2.1       ----一台主机
localhost         ----数据库服务器本机
```

1.  添加用户并设置权限 命令格式：

```
grant  权限列表 on  库名.表名 to  用户名@"客户端地址"  identified  by  "密码"   with grant option;
with grant  option 有授权权限，可选项。 
```

应用示例1：创建用户mydb1,对所有库所有表有完全权限。允许从任何客户端连接，密码：123456，且有授权权限。

```
grant all on *.* to mydb1@"%" identified by "1234" with grant option;
```

GRANT ALL ON `ry_product`.* TO bug@"%" ;

应用示例2：创建admin用户，允许客户端192.168.2.0/24 网段连接，对db3库的b3表有查询权限，密码是：123456

```
grant select on db3.b3 to admin@"192.168.2.%" identified by "123456";
```

应用示例3：创建用户bug，允许本机连接 ，允许bugstack库的所有表有查询、更新、插入、删除记录权限，密码 bug

```
grant select,update,insert,delete on bugstack.* to bug@"localhost" identified by "bug";
```

1.4、刷新授权
flush privileges;

1.5、查看授权
show GRANTS for xxljob @'%'

1.6、撤销授权
revoke 权限1,权限2,...... on 数据库名.* to 用户名@'ip地址';
示例：
revoke all on bank.* from lotus @'localhost';

撤销 admin用户在db3.b3表中的select 权限。
revoke select on db3.b3 from admin@"192.168.2.%";

1.7、修改密码

修改密码 set password = password ( 'wy123' );
登录授权 grant all privileges on *.* to 'lotus' @'%' identified by 'wy123';
刷新授权 flush privileges;

1.8、忘记密码
1、可以在配置文件(mysql.ini)里加上 skip-grant-tables ,注意写到[mysqld]参数组下，表示跳过授权
2、重启MySQL(net stop mysql 先闭关  net start mysql )再登录就不需要密码，进去改密码，改完后，直接 FLUSH PRIVILEGES; 就可以使用新密码来登录了
示例：(update mysql.user set password=password("root") where user="root" and host="localhost";)
3、改完后记得去掉配置文件例的 skip-grant-tables,重新启动MySQL服务
4、再使用新的密码登录就可以了







CREATE DATABASE lottery_01;
GRANT ALL PRIVILEGES ON lottery_01.* TO 'bug'@'%';
FLUSH PRIVILEGES;