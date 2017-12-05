# 《PostgreSQL修炼之道 --从小工到专家》

## 第二章 PostgreSQL安装与配置
### Ubuntu 下安装
1. apt
```bash
sudo apt-get install postgresql
su - postgres
psql
\l # 列出所有数据库
\q # 退出psql
sudo service postgresql status
sudo service postgresql stop
sudo service postgresql start
```
2. source
一般选择`.bz2`的压缩包，这种格式体积小

1> 依赖`zlib1g-dev`, `libreadline6-dev`, `libperl-dev`, `python-dev`.
```bash
aptitude search zlib | grep dev
aptitude search readline | grep dev
...

sudo apt-get install zlib1g-dev
sudo apt-get install libreadline6-dev
sudo apt-get install libperl-dev
sudo apt-get install python-dev
```
2> 安装
```bash
./configure --prefix=/usr/local/pgsql9.5.5 --with-perl --with-python
make
sudo make install
cd /usr/local/psql9.5.5
sudo ln -sf /usr/local/pgsql9.5.5 /usr/local/pgsql

# /etc/profile
export PATH=/usr/local/pgsql/bin:$PATH
export LD_LIBRARY=/usr/local/gpsql/lib:$LD_LIBRARY
```

### 配置
1. 命令配置
```bash
# 创建数据库蔟
mkdir /home/wuxiaolong/pgdata
export PGDATA=/home/wuxiaolong/pgdata
initdb

# 安装contrib目录下工具
cd /usr/local/postgresql9.5.5
make
sudo make isntall

pg_ctl start -D $PGDATA
pg_ctl stop -D $PGDATA [-m <smart|fast|immediate>]
```
2. `postgresql.conf` 在`$PGDATA`下
监听IP和端口：（修改后要重启数据库）
```
# liste_addresses='localhost'
# port = 5432
```
log相关的参数：
```
logging_collector = on
log_directory = 'pg_log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_truncate_on_rotation = off # 是否覆盖
log_rotation_age = 0 # 覆盖周期
log_rotation_size = 0 # 每当日志写满一定大小，则切换下一个日志
```
内存参数
```
shared_buffers: 共享存储大小，主要用于共享数据块。
work_mem：单个SQL执行时，排序、hash join所使用的内存，SQL运行完后，内存就释放了。
```
## 第三章 SQL语言入门
### 简介
1. 分类
SQL命令一般分为DQL(数据查询语言; select)，DML(数据操纵语言; insert、update、delete)，DDL(数据定义语言; create等)
2. 语法结构
命令由`;`结束。可以有注释，注释相当于空白。

### DDL
1. 建表
语法：
```SQL
create table table_name (
  col01_name data_type,
  col02_name data_type,
  ...
);
```
例子：
```SQL
create table score (
  student_name varchar(40),
  chineses_score int,
  math_score int,
  test_date date,
);
\d # 显示所有表格
\d score # 显示score表格的定义情况
```
建表的时候，可以指定主键，主键是表中行的唯一标识，不可重复：
```SQL
create table student (no int primary key, student_name varchar(40), age int);
```
2. 删除表格：`drop table table_name;`

### DML
1. 插入语句
```SQL
insert into student values(1, '张三', 14);
insert into student (no, age, student_name) values (2, 13, '李四');
insert into student (no, student_name) values (3, '王二'); # 缺失值会被置空
```
2. 更新语句
```SQL
update student set age = 15;
update student set age = 14 where no = 3;
update student set age=13, student_name='王明充' where no=3; # 同时更新多个值
```
3. 删除语句
```SQL
delete from student where no = 3;
delete from student; # 删除表中所有数据
```

### DQL
1. 单表查询语句
```SQL
select no, student_name, age from student;
select age+5 from student;
select no, 3+5 from student;
select 10*2, 3*5+2;
select * from student;
```
2. 过滤条件查询(where)
```SQL
select * from student where age >= 15;
```
3. 排序(order by)
```SQL
select * from student order by age;

# 排序子句"order by"应该在"where"子句之后：
select * from student where age >= 15 order by age;

select * from student order by age, student_name;
select * from student order by age desc;
select * from student order by age desc, student_name;
```
4. 分组查询(group by)
```SQL
select age, count(*) from student group by age;
```
使用"group by"语句时，需要使用聚合函数，常用的聚合函数为"count"、"sum"等。
5. 关联表查询(join)
```SQL
create table class (no int primary key, class_name varchar(40));
insert into class (no, class_name) values
  (1, '初二（1）班'),
  (2, '初二（2）班'),
  (3, '初二（3）班')，
  (4, '初二（4）班');

create table student (no int primary key, student_name, varchar(40), age int, class_no int);
insert into student (no, student_name, age, class_no) values
  (1, '张三', 14, 1),
  (2, '吴二', 15, 1),
  (3, '李四', 13, 2),
  (4, '吴三', 15, 2),
  (5, '王二', 15, 3),
  (6, '李三', 14, 3),
  (7, '吴二', 15, 4),
  (8, '张四', 14, 4);

select student_name, class_name from student, class where student.class_no = class.no;
select student_name, class_name from student a, class b where a.class_no = b.no; # 别名
select student_name, class_name from student a, class b where a.class_no = b.no and a.age > 14;
```
### 其他SQL语句
1. insert into ... select
该语句可以把数据从一张表插入到另一张表中。
```SQL
create table student_bak (no int primary key, student_name varchar(40), age int, class_no int);
insert into student_bak select * from student;
```
2. union
```SQL
select * from student where no = 1 union select * from student_bak where no =2;
# union会把相同的记录合并成一条，若不想合并，使用union all：
select * from student where no = 1 union all select * from student_bak where no =1;
```
3. truncate table
truncate table的用途是清空表内容。执行起来比delete快， delete是把数据一条条的删除，而truncate是直接把原先表的内容丢弃了。
`truncate table student_bak`

## psql工具的使用介绍
### psql的简单使用
直接输入`psql`，安装PostgreSQL的数据库时，会建立一个与初始化数据库时的操作系统用户同名的数据库用户，该用户是数据库的超级用户，免密。可通过修改`pg_hba.conf`文件来要求输入密码。
`psql -l`查看有哪些数据库。
```SQL
\l # 查看数据库
\d # 列出当前数据库中的所有表
create database testdb;
\c testdb; # 连接到testdb
```
```bash
psql -h <hostname|ip> -p <port> [database_name] [user_name]

# 也可以直接用环境变量指定
export PGDATABASE=database_name
export PGHOST=ip
export PGPORT=port
export PGUSER=user_name
```
### psql的常用命令
- \d
\d [ pattern ]
\d [ pattern ] +

1. 什么都不带，将列出当前数据库中的所有表
2. 跟一表名，显示该表的结构定义
3. "表名_pkey"，显示该表的索引信息
4. 跟通配符`*`或`?`，多表查询
5. \d+ 显示比\d更详细的信息，包括与列表关联的注释，以及表中出现的OID。
6. 匹配不同对象类型的\d命令
只显示匹配的表： \dt
只显示索引： \di
只显示序列： \ds
只显示视图： \dv
只显示函数： \df
7. 显示SQL已执行的时间，\timing on
8. 列出所有的schema： \dn
9. 显示所有的表空间： \db
> 实际上PostgreSQL中的表空间就是对应一个目录，放在这个表空建的表，就是把表的数据文件放到这个表空间下。
10. 列出数据库中所有角色或用户：\du或\dg
> PostgreSQL数据库中用户和角色不分的。
11. \dp或\z命令用于显示表的权限分配情况

- 指定字符集编译的命令 `\enconding gbk;`, `\enconding utf8;`
- \pset 设置输出的格式
\pset border 0: 无边框
\pset border 1: 边框在内部
\pset border 2: 内外都有边框
- \x 把表中每一行的每列数据都拆分为单行展示
- \i <filename> 执行存储在外部文件中的sql语句或命令
也可以使用`psql -f <filename>`来执行SQL脚本文件中的命令。
- \echo 输出一行信息
- \? 显示更多命令用法

### psql使用技巧和注意事项
1. 补全：<tab><tab>
2. 自动提交方面的技巧
在psql中事务是自动提交的。如果不想自动提交：
1> 运行`begin;`命令，然后执行DML语句，最后在执行`commit;`或`rollback;`命令
2> 关闭自动提交功能 `\set AUTOCOMMIT off` (9.5.5不起作用)
3. 获得psql中命令实际执行的SQL
1> 启动psql的命令行中加"-E"参数，就可以把psql中各种以"\\"开头的命令执行的实际SQL打印出来。(`psql -E postgres`)
2> 在已运行的psql中，可以使用`\set ECHO HIDDEN on | off`命令

## 第五章 数据类型
### 类型介绍
#### 类型的分类：
1. 布尔类型(boolean)
2. 数值类型(整数类型有2字节smallint、4字节int、8字节bigint; 十进制精确类型有numeric; 浮点类型有real和double precision; 8字节的货币类型money)
3. 字符类型(varchar(n), char(n), text)
4. 二进制数据类型(bytea)
5. 位串类型(位串就是一串1和0的字符串，有bit(n)、bit varying(n))
6. 日期和时间类型(date、time、timestamp，而time和timestamp又分是否包括时区的两种类型)
7. 枚举类型(枚举类型是一种包含一系列有序静态值集合的数据类型，等于某些编程语言中的enum类型。使用时要先用`create type`创建这个类型)
8. 几何类型(点point、直线line、线段lseg、路径path、多边形polygon、圆cycle等类型)
9. 网络地址类型(cidr、inet、macaddr)
10. 数组类型
11. 复合类型
12. xml类型
13. json类型
14. range类型
15. 对象标识符类型(oid类型、regproc类型、regclass类型等)
16. 伪类型(不可作为字段的数据类型，可用于声明一个函数的参数或者结果类型。有any、anyarray、anyelement、cstring、internal、language_handler、record、trigger、void、opaque)
17. 其他类型(UUID类型、pg_lsn类型)
#### 类型输入与转换
简单的类型：
```SQL
select 1, 1.1421, 'hello world';
```
复杂类型：使用“类别名”加上单引号括起来
```SQL
select bit '11110011';
select int '1' + int '2';
```
使用类型转换函数`CAST`和双冒号`::`进行类型转换：
```SQL
select cast('5' as int), cast('2014-07-17' as date);
select '5'::int, '2014-07-17'::date;
```
### 布尔类型
#### 布尔类型解释
boolean在SQL中可以用不带引号的TRUE、FALS表示，也可以用更多的表示真和假的带引号的字符表示，如'true'、'false'、'yes'、'no'等等。'unknown'(未知)状态用NULL表示
```SQL
create table t (id int, col1 boolean, col2 text);
insert into t values (1, true, 'true');
insert into t values (2, false, 'falser');
insert into t values (3, 'true', '''true''');
insert into t values (4, 'false', '''false''');
insert into t values (5, 't', '''t''');
insert into t values (6, 'f', '''f''');
insert into t values (7, 'y', '''y''');
insert into t values (8, 'n', '''n''');
insert into t values (9, 'yes', '''yes''');
insert into t values (10, 'no', '''no''');
insert into t values (11, '1', '''1''');
insert into t values (12, '0', '''0''');
select * from t;
select * from t where col1;
select * from t where not col2;
```
#### 布尔类型的操作符 and，or， not， is
布尔类型可以使用"IS"比较运算符：
```
expression is true
expression is not true
expression is false
expression is not false
expression is unknown
expression is not unknown
```
### 数值类型
#### 数值类型解释
| 类型名称         | 存储空间         |    范围           |
| :-------------: | :-------------: | :-------------: |
| smallint        | 2字节           | $-2^{15} \sim 2^{15}-1$ |
| int 或 integer  | 4字节           | $-2^{31} \sim 2^{31}-1$ |
| bigint          | 8字节           | $-2^{63} \sim 2^{63}-1$ |
| numeric 或 decimal | 变长         | 无限制                   |
| real            | 4字节           | 6位十进制数字精度        |
| double precision | 8字节          | 15位十进制数字精度       |
| serial          | 4字节           | $1 \sim 2^{31}-1$      |
| bigserial       | 8字节           | $1 \sim 3^{63}-1$      |
#### 整数类型
PostgreSQL中没有MySQL中的tinyint(1字节)、mediumint(3字节)、unsigned类型。
#### 精确的小数类型
精确的小数类型可用numeric、numeric(m,n)、numeric(m)表示。其中，numeric与decimal是等效的，两种类型都是SQL标准，可以存储最多1000位精度的数字，并且可以进行准确的计算。这个类型特别适合用于货币金额和其他要求精确计算的场合。不过运算比浮点数慢很多。
`numeric(precision, scale)`，其中，精度preci必须为正数，标度scale可以为零或正数。如果关心移植性，最好总是声明精度和标度。
```SQL
create table t1 (id1 numeric(3), id2 numeric(3,0), id3 numeric(3,2), id4 numeric);
insert into t1 values (3.1, 3.5, 3.123, 3.123);
```
超过标度，四舍五入；超过精度，报错。

#### 浮点数类型
数据类型real和double precision是不精确的、变精确的数字类型。浮点类型的特殊值：'Infinity', '-Infinity', 'NaN'。在SQL命令里把这些数值当作常量时，必须在他们周围放上单引号，像：`update table set x = 'Infinity'`。输入时，这些值是大小写无关的。
#### 序列类型
```SQL
create table t (
  id serial
);
```
相当于：
```SQL
create sequence t_id_seq;
create table t (
  id integer not null default nextval('t_id_seq')
);
alter sequence t_id_seq owned by t.id;
```
#### 货币类型
货币类型(money)可以存储固定小数的货币数目，与浮点数不同，它是完全保证精度的。其输出格式与参数`lc_monetary`的设置有关。
```SQL
select '12.34'::money;
show lc_monetary;
set lc_monetary = 'zh_CN.UTF-8';
# set lc_monetary = 'en_US.UTF-8';
```
#### 数学函数和操作符
| 操作符/函数 | 描述     |
| :-------------: | :-------------: |
| + | 加 |
| - | 减 |
| * | 乘 |
| / | 除 |
| % | 模（求余） |
| ^ | 幂 |
| \|/ | 平方根 |
| \|\|/ | 立方根 |
| ! | 阶乘 |
| !! | 阶乘（前缀操作符） |
| @ | 绝对值 |
| & | 二进制AND |
| \| | 二进制OR |
| # | 二进制XOR |
| ~ | 二进制NOT |
| << | 二进制左移 |
| >> | 二进制右移 |
| abs(x) | 绝对值 |
| cbrt(dp) | 立方根 |
| ceil(dp/numeric) | 不小于参数的最小整数 |
| degree(dp) | 把弧度转为角度 |
| exp(dp/numeric) | 自然指数 |
| floor(dp/numeric) | 不大于参数的最大整数 |
| ln(dp/numeric) | 自然对数 |
| log(dp/numeric) | 以10为底的对数 |
| log(b numeric, x numeric) | 以b为底的对数 |
| mod(y,x) | y/x的余数（模） |
| pi() | "π"常量 |
| power(a dp, b dp) | a的b次幂 |
| power(a numeric, b numeric) | a的b次幂 |
| radians(dp) | 把度数转为弧度 |
| random() | 0.0到1.0之间的随机数 |
| round(dp/numeric) | 圆整为最近的整数（四舍五入） |
| round(v numeric, s int) | 圆整为s位小数（四舍五入） |
| setseed(dp) | 为随后的random()调用设置种子（0.0到1.0之间） |
| sign(dp/numeric) | 参数符号（-1,0,+1） |
| sqrt(dp/numeric) | 平方根 |
| trunc(dp/numeric) | 截断（向零靠近） |
| trunc(dp/numeric) | 截断为s位小数 |
| width_bucket(op numeric, b1 numeric, b2 numeric, count int) | 返回一个桶，在一个有count个桶、上界为b1、下界为b2的等深柱图中，operand将被赋予的就是这个桶 |
| acos(x) | 反余弦 |
| asin(x) | 反正弦 |
| atan(x) | 反正切 |
| atan2(x,y) | x/y的反正切 |
| cos(x) | 余弦 |
| cot(x) | 余切 |
| sin(x) | 正弦 |
| tan(x) | 正切 |

### 字符串类型
varchar(n)和char(n)分别是character varying(n)和character(n)的别名，没有声明长度的character等于character(1)，没有声明长度的character varying接受任何长度的字符串。
#### 字符串函数和操作符
Page
