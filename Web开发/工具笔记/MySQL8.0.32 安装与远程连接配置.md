[TOC]

## 安装环境

在 *Ubuntu 22.04* 操作系统下使用 `apt` 安装：

```shell
sudo apt install -y mysql-server mysql-client mysql-common
```

## 一、进入数据库

目前主要有三个方式能首次进入数据库，进入数据库之后修改密码即可。具体的方式与 `MySQL` 版本有关，没有测试：

* 无密码
* 日志密码
* 跳过密码

主要针对之后两种方法说一下操作方式。

### 1.1 日志密码

首次登录时，默认情况下在安装过程中会提醒在错误日志的文件中可以看到：

```shell
sudo grep 'temporary password' /var/log/mysql/error.log
```

可以看到：

```
[Note] [MY-010454] [Server] A temporary password is generated for root@localhost: xxxxxxxx
```

然后使用命令登录即可：

```shell
mysql -u root -p
```

> 该方式在已经安装过 `MySQL` 的情况下是无效的，如果之前已经修改过密码，并且忘记了，或者由于某些原因无法进入数据库，则需要采用另一个方式。

### 1.2 跳过密码

这种方式在任何情况下都可以生效（最起码 `8.0.32` 版本下是可以的）。

首先找到 *MySQL 8.0.32* 服务的配置文件：

```shell
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

然后在 `[mysqld]` 之后的一行添加：

````
[mysqld]
skip-grant-tables

# 其他配置项
````

然后重新使用命令登录即可：

```shell
mysql -u root -p
```

无需密码直接进入数据库，然后进行修改密码等操作即可。

> 记得将配置文件还原，为了安全考虑。

## 二、远程连接

### 2.1 用户权限配置

远程连接需要的步骤比较多，基本上按照 [`MySQL 8.0` 远程连接相关博客](https://zhuanlan.zhihu.com/p/587097435)就可以。

### 2.2 地址绑定配置

但是**在 `MySQL 8.0.32` 版本还需要多配置一下绑定的 `IP` 地址**。具体的配置项在配置文件中：

```shell
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

找到配置项 `bind-address` ，如果值为 `127.0.0.1` ，则**只能主机访问**（即使已经修改过 `mysql/user` 库里的数据并设置了远程连接权限），**需要修改为想要绑定的网卡地址**（`ifconfig` 或其他工具查看就行）。

***

至此，已经完成初始的配置，可验证远程连接了。