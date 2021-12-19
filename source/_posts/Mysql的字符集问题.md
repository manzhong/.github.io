---
title: Mysql的字符集问题
tags: Mysql
categories: Mysql
abbrlink: 53399
date: 2021-11-19 18:04:46
summary_img:
encrypt:
enc_pwd:
---

## 一 问题背景

 报错:

```
'\\xF0\\x9F\\x91\\xAC' for column 'counselor_user_name' at row 1","record":[{"byteSize":2,"index":0,"rawData":55,"type":"LONG"},{"byteSize":3,"index":1,"rawData":565,"type":"LONG"},{"byteSize":4,"index":2,"rawData":"美伊👬","type":"STRING"}
```

​	表情符号等,向MySQL导数据报错,数据库编码格式，表编码格式和字段编码格式的时候，一般设置为“utf-8”，这对于汉字来说足够了，在mysql中utf8占3个字节，但是对于移动端的特殊表情符号来说，三个字节是不够的，他需要四个字节。这个时候我们使用utf8就会出现‘\xF0\x9F\x8F\x80’的问题。

## 二 解决

1. 可以对4字节的字符进行编码存储，然后取出来的时候，再进行解码。但是这样做会使得任何使用该字符的地方都要进行编码与解码。

2. **更改数据库的编码为utf8mb4:**utf8mb4编码是utf8编码的超集，兼容utf8，并且能存储4字节的表情字符。 
   采用utf8mb4编码的好处是：存储与获取数据的时候，不用再考虑表情字符的编码与解码问题。

   注意: 需要数据库和相关表都修改为utf8mb4

   ```mysql
   #查看数据库的字符集:
   	show variables like '%character%';
   #查看每个库的字符集;
   	select schema_name,default_character_set_name from information_schema.schemata;
   ```

## 三 修改

```
修改mysql的配置文件,my.cnf (windows为my.ini)
my.cnf 一般在etc/mysql/my.cnf
修改:
	[client]  
default-character-set=utf8mb4  
[mysqld]  
character-set-server = utf8mb4  
collation-server = utf8mb4_unicode_ci  
init_connect='SET NAMES utf8mb4'  
skip-character-set-client-handshake = true  
[mysql]  
default-character-set = utf8mb4
```

8个配置文件

```mysql
character_set_client
	主要用来设置客户端使用的字符集。
character_set_connection
	主要用来设置连接数据库时的字符集，如果程序中没有指明连接数据库使用的字符集类型则按照这个字符集设置。
character_set_database
	主要用来设置默认创建数据库的编码格式，如果在创建数据库时没有设置编码格式，就按照这个格式设置。
character_set_filesystem
	文件系统的编码格式，把操作系统上的文件名转化成此字符集，即把 character_set_client转换character_set_filesystem， 默认binary是不做任何转换的。
character_set_results
	数据库给客户端返回时使用的编码格式，如果没有指明，使用服务器默认的编码格式。
character_set_server
	服务器安装时指定的默认编码格式，这个变量建议由系统自己管理，不要人为定义。
character_set_system
	数据库系统使用的编码格式，这个值一直是utf8，不需要设置，它是为存储系统元数据的编码格式。
character_sets_dir
	这个变量是字符集安装的目录。
```

修改后重启数据库:

```mysql
#重启
service mysql restart
#停止
service mysql stop
#启动
service mysql start
```

```mysql
一般只修改:
	character_set_client
	character_set_connection
	character_set_database
	character_set_results
	character_set_server  ## 必改
这三个不用动:
	character_set_filesystem
	character_set_system
	character_sets_dir
```

**借助网上的一个完整的用户请求的字符集转换流程来更好的理解上述几个变量：**
mysql Server收到请求时将请求数据从 character_set_client 转换为 character_set_connection
进行内部操作前将请求数据从 character_set_connection 转换为内部操作字符集,步骤如下
　　A. 使用每个数据字段的 CHARACTER SET 设定值；
　　B. 若上述值不存在，则使用对应数据表的字符集设定值
　　C. 若上述值不存在，则使用对应数据库的字符集设定值；
　　D. 若上述值不存在，则使用 character_set_server 设定值。
最后将操作结果从内部操作字符集转换为 character_set_results;

```
1 在命令提示符窗口中输入MySQL命令或sql语句，回车后，这些MySQL命令或sql语句由“命令提示符窗口字符集”转换为“character_set_client”定义的字符集

2、  使用命令提示符窗口成功连接MySQL服务器后，就建立了一条“数据通信链路”，MySQL命令或sql语句沿着“数据链路”传向MySQL服务器，由“character_set_client”定义的字符集转换为character_set_connection定义字符集
3、  MySQL服务实例收到数据通信链路中的MySQL命令或sql语句，将MySQL语句或sql语句从character_set_connection定义的字符集转换为character_set_server定义的字符集

4、  若MySQL命令或sql语句针对于某个数据库进行操作，此时将MySQL命令或sql命令从character_set_server定义的字符集转换为character_set_database定义的字符集

5、  MySQL命令或sql语句执行结束后，将执行结果设置为character_set_results定义字符集

6、  执行结果沿着打开的数据通信链路原路返回，将执行结果又character_set_results定义的字符集转character_set_client定义的字符集，最终转换为命令提示符窗口字符集显示到命令提示符窗口中。
```



