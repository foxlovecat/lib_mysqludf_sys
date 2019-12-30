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
