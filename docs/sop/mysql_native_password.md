---
tag: mysql
---

# 填坑 Plugin 'mysql_native_password' is not loaded

数据库从 mysql5.7 升级到 mysql8.4，部分场景出现以下错误提示：

```shell
Plugin 'mysql_native_password' is not loaded
```

## 原因

原因是：mysql_native_password 插件(模式)在新版本中被弃用了，新模式为 `caching_sha2_password`，需要启用一下旧模式。

## 解决

1.连接到数据库后，用命令 `SHOW PLUGINS` 查看插件列表

可以看到插件【mysql_native_password】的状态为【DISABLED】
我们的需要将其状态改为【ACTIVE】，修改步骤在后面。

2.找到配置文件 my.ini 进行修改

配置文件位置举例：C:\ProgramData\MySQL\MySQL Server 8.4\my.ini
在my.ini最后一行添加：
mysql_native_password=ON

3.重启mysql服务，命令示例：

net start mysql84
net stop mysql84

4.再次连接到数据库，用命令 SHOW PLUGINS 查看插件列表

可以看到插件【mysql_native_password】的状态为【ACTIVE】

5.为用户启用旧的身份验证插件【mysql_native_password】

```shell
ALTER USER '账号'@'主机' IDENTIFIED WITH 'mysql_native_password' BY '密码';
FLUSH PRIVILEGES;
```

举例：

```shell
ALTER USER 'root'@'localhost' IDENTIFIED WITH 'mysql_native_password' BY 'root';
FLUSH PRIVILEGES;
```

此时，问题就解决了。
