# lib_mysqludf_sys
mysql调用系统命令
### window
里面有x86和x64位两个解压后放到对应的 mysql 目录中的 lib/plubin 中  一定要放这个目录中  mysql5.1 以上版本不放这个目录中无法使用

打开mysql可以执行命令行的地方创建自定义函数,下面创建两个函数一个是返回执行结果,另一个是返回命令行的字符串
``` sql
DROP FUNCTION IF EXISTS sys_exec;
DROP FUNCTION IF EXISTS sys_eval;
CREATE FUNCTION sys_exec RETURNS integer SONAME 'lib_mysqludf_sys_x64.dll';
CREATE FUNCTION sys_eval RETURNS string SONAME 'lib_mysqludf_sys_x64.dll';
SELECT sys_eval("ipconfig/all");
``` 

### linux

lib_mysqludf_sys.so复制到mysql/lib/plugin目录下。

b) 在mysql中创建函数(根据需要选取)：
``` sql
Drop FUNCTION IF EXISTS lib_mysqludf_sys_info;
Drop FUNCTION IF EXISTS sys_get;
Drop FUNCTION IF EXISTS sys_set;
Drop FUNCTION IF EXISTS sys_exec;
Drop FUNCTION IF EXISTS sys_eval;
Create FUNCTION lib_mysqludf_sys_info RETURNS string SONAME 'lib_mysqludf_sys.so';
Create FUNCTION sys_get RETURNS string SONAME 'lib_mysqludf_sys.so';
Create FUNCTION sys_set RETURNS int SONAME 'lib_mysqludf_sys.so';
Create FUNCTION sys_exec RETURNS int SONAME 'lib_mysqludf_sys.so';
Create FUNCTION sys_eval RETURNS string SONAME 'lib_mysqludf_sys.so';
``` 
使用此函数
例：在select语句调用mkdir命令
``` sql
Select sys_exec('mkdir -p /home/user1/aaa')
```

例：在触发器中调用外部的脚本(脚本需要可执行权限)
``` sql
Create TRIGGER trig_test AFTER Insert ON <table1>
FOR EACH ROW 
BEGIN
    DECLARE redata INT;
    Select sys_exec('/home/user1/test.sh') INTO redata;
END
``` 
判断更新后指定的字段有变化时才进行一些操作,如下判断价格有没有变化
``` sql
CREATE TRIGGER `updateorder` AFTER UPDATE ON `order`
FOR EACH ROW 
begin
   DECLARE redata INT;
   if old.price<>new.price  then
     select sys_exec('E:/*****/socket.bat') into redata;
    end if;
end;
``` 



mysql-udf-http 是一款简单的MySQL用户自定义函数，具有http_get()、http_post()、http_put()、http_delete()四个函数，可以在MySQL数据库中利用HTTP协议进行REST相关操作，它的安装方式如下：

下载并安转curl：

    wget http://curl.haxx.se/download/curl-7.21.1.tar.gz
	tar zxvf curl-7.21.1.tar.gz
	cd curl-7.21.1/
	./configure --prefix=/usr
	make && make install
	cd ../
1
2
3
4
5
6
下载mysql-udf-http-1.0.tar.gz：（百度网盘：http://pan.baidu.com/s/1nuYZqR3）

	tar zxvf mysql-udf-http-1.0.tar.gz
	cd mysql-udf-http-1.0/
	./configure --prefix=/usr/local/mysql --with-mysql=/usr/bin/mysql_config
	make && make install
1
2
3
4
注意：prefix记得改成你自己的mysql安装目录

正常的情况mysql-udf-http.so等文件将安装至/usr/local/mysql/lib/plugin下，
如果不是，那么先找到这个文件的位置：

find / -name mysql-udf-http.so
1
加一个软连接：

ln -s /usr/lib64/mysql/lib/mysql/plugin/mysql-udf-http.so /usr/lib64/mysql/plugin/mysql-udf-http.so
1
安装成功后，进到mysql控制台，注册相关函数：

create function http_get returns string soname 'mysql-udf-http.so';
create function http_post returns string soname 'mysql-udf-http.so';
create function http_put returns string soname 'mysql-udf-http.so';
create function http_delete returns string soname 'mysql-udf-http.so'; 
1
2
3
4
在业务表中加入更新操作的触发器：

DELIMITER $$ 
DROP TRIGGER IF EXISTS test_update$$
CREATE TRIGGER test_update  
AFTER UPDATE ON sp_bal  
FOR EACH ROW BEGIN  
    SET @tt_re = (SELECT http_get(CONCAT('http://192.168.99.200:8005/test?id=', OLD.id)));
END; 
$$
1
2
3
4
5
6
7
8
常见错误解决：

1、如果mysql_config文件不存在(执行 find / -name mysql_config，提示没有找到)：yum install mysql-devel；
2、如果执行./configure --prefix=/usr/local/mysql --with-mysql=/usr/local/mysql/bin/mysql_config 时候报错：checking for DEPS… configure: error: Package requirements (libcurl >= 7.15) were not met:No package ‘libcurl’ found

如果是第一步安装cur成功，那么执行下面的命令：

[root@localhost mysql-udf-http-1.0]# which curl
/usr/bin/curl
[root@localhost mysql-udf-http-1.0]# find / -name libcurl.pc
/home/mysql/curl-7.21.1/libcurl.pc
/usr/lib/pkgconfig/libcurl.pc
[root@localhost mysql-udf-http-1.0]# cp /usr/lib/pkgconfig/libcurl.pc /usr/lib64/pkgconfig/
1
2
3
4
5
6
如果提示缺少libcurl，就安装curl：yum install curl*
————————————————
版权声明：本文为CSDN博主「原影远行」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/sheyongjun1990/article/details/89887175
