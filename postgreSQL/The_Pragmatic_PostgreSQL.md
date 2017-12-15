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
./configure --prefix=/usr/local/pgsql9.5.5 --with-perl --with-python --with-libxml
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
# listen_addresses='localhost'
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
| 操作符/函数 | 描述    |
| :--------: | :-----: |
| string \|\| string | 字符串连接 |
| bit_length(string) | 字符串里二进制位的个数 |
| char_length(string) 或 character_length(string) | 字符串中的字符个数 |
| convert(string using conversion_name) | 使用指定的转换名字改变编码。转换可以通过create conversion定义。当然，系统里有一些预定义的转换名字 |
| lower(string) | 把字符串转化为小写 |
| octet_length(string) | 字符串中的字节数 |
| overlay(string placing string from int [for int]) | 替换子字符串：overlap('Txxxxas' placing 'hom' from 2 for 4) $\rightarrow$ Thomas |
| position(substring in string) | 指定的子字符串的位置 |
| substring(string [from int] [for int]) | 抽取子字符串 |
| substring(string from pattern) | 抽取匹配POSIX正则表达式的子字符串：substring('Thomas' from '...\$') $\rightarrow$ mas |
| substring(string from pattern for escape) | 抽取匹配SQL正则表达式的子字符串：substring('Thomas' from '%#"o_a"_' for '#') $\rightarrow$ oma |
| trim([leading \| trailing \| both] [characters] from string) | 从字符串string的开头/结尾/两边删除只包含characters中字符（默认是一个空白）最长的字符串： trim(both 'x' from 'xThimxx') $\rightarrow$ Tom |
| upper(string) | 把字符串转化为大写 |
| ascii(string) | 参数第一个字符的ASCII码 |
| btrim(string text [, characters text]) | 从string开头和结尾删除包含在参数characters中字符，直到遇到一个不是在characters中的字符串为止，参数characters的默认值为空格：btrim('aaosdbaaa', 'aa') $\rightarrow$ osdb |
| chr(int) | 给出ASCII码的字符 |
| convert(string text, [src_encoding name], dest_encoding name) | 把原来编码为src_encoding的字符串转化为dest_encoding编码（如果省略了src_encoding，将使用数据库编码） |
| decode(string text, type text) | 把早先用encode编码的string里面的二进制数据解码 |
| encode(data bytea, type text) | 把二进制数据编码为只包含ASCII形式的数据。支持的类型有：base64、hex、escape。 |
| initcap(string) | 把每个单词的第一个字母转为大写，其余保留小写。单词是一系列字母数字组成的字符时，用非字母数字分隔。 |
| length(string) | string中字符的数目 |
| lpad(string text, length int [, fill text]) | 通过填充字符fill（默认为空白），把string填充为length长度。如果string已经比length长，则将其尾部截断。 |
| ltrim(string text [, characters text]) | 从string的开头删除包含在参数characters中的字符，值到遇到一个不是在characters中的字符为止 |
| md5(string) | 计算string的MD5散列，以十六进制返回结果 |
| pg_client_encoding() | 当前客户端编码名称 |
| quote_ident(string) | 返回使用于SQL语句的标识符形式（使用适当的引号进行界定）。只有在必要的时候会添加引号（字符串包含非标识符字符或者会转换大小写的字符）。嵌入的引号会被恰当地写双份。 |
| quote_literal(string) | 返回适用于在SQL语句里当作文本的形式。嵌入的引号和反斜杠被恰当地写了双份：quote_literal('O\\'Reilly') $\rightarrow$ 'O"Reilly' |
| regexp_replace(string text, pattern text, replacement text [, flags text]) | 替换匹配POSIX正则表达式的子字符串 |
| repeat(string text, from text, to text) | 将string重复number次 |
| replace(string text, from text, to text) |  把string里出现的所有子字符串from替换成字字符串to |
| rpad(string text, length int [, fill text]) | 通过填充字符fill（默认为空白），把string填充为length长度。如果string已经比length长，则将其尾部截断。 |
| rtrim(string text [, characters text]) | 从string的结尾删除包含在参数characters中的字符，值到遇到一个不是在characters中的字符为止 |
| split_part(string text, delimiter text, field int) | 根据delimiter分隔string返回生成的第field个子字符串（1为基） |
| strpos(string, substring) | 指定的子字符串的位置。和position(substring in string)一样，不过参数顺序相反 |
| substr(string, from [, count]) | 抽取子字符串 |
| to_ascii(string text [, encoding(text)]) | 把string从其他编码转换为ASCII（仅支持LATIN1、LATIN2、LATIN9、WIN1250编码）|
| to hex(number int/bigint) | 把number转化成16进制表现形式 |
| translate(string text, from text, to text) | 把在string中包含的任何与from中字符串匹配的字符转化为对应的在to中的字符：translate('12345', '14', 'db') $\rightarrow$ d23b5 |

### 二进制数据类型
#### 二进制数据类型解释
PostgreSQL中的bytea对应MySQL和Oracle中的blob类型。Oracle的raw类型也可以使用这个类型取代。
#### 二进制数据类型转义表示
要转义一个字节值，通常需要把它的数值转换成对应的三位八进制数，并且加两个前导反斜杠。有些八进制数值可以加一个反斜杠直接转义，比如单引号和反斜杠本身。
#### 二进制数据类型的函数 P68-P69

### 位串类型
#### 解释
位串就是一串1和0的字符串。在PostgreSQL中可以直观显式地操作二进制位。包括bit(n)、bit varying(n)，没有长度的bit等效于bit(1)，没有长度的bit varying表示没有长度限制。
如果明确地把一个位串值转换成bit(n)， 那么它的右边将被截断，或者在右边补齐0到刚好为n位，而不会抛出错误。bit varying(n) 类似。
#### 使用
```SQL
create table test (a bit(3), b bit varying(5));
insert into test values (b'101', b'00');
insert into test values (b'11110', b'101'); # 超长报错
```
#### 操作符及函数
| 操作符 | 描述 |
| :-------------: | :-------------: |
| \|\| | 连接 |
| & | 位与 |
| \| | 位或 |
| # | 异或 |
| ~ | 位非 |
| << | 左移：b'1101' << 3 $\rightarrow$ 1000 |
| >> | 右移：b'1101' >> 3 $\rightarrow$ 0001 |
函数：length, bit_length, octet_length, position, substring, overlay, get_bit, set_bit。
可以在整数和bit之间， 十进制、十六进制、二进制之间的转换：
```SQL
select 'xff'::bit(8)::int;
select to_hex(255);
```
### 日期/时间类型
#### 解释
timestamp [ (p) ] [ without time zone ]
timestamp [ (p) ] with time zone
interval [ (p) ]
date
time [ (p) ] [ without time zone ]
time [ (p) ] with time zone
p为精度值，用以指明秒域中小数部分的位数。如果没有明确的默认精度。对于timestamp和interval类型，p的范围是0~6。
timestamp数值是以双精度浮点数的方式存储的。timestamp值是以2000-01-01午夜之前或之后的秒数存储的。
#### 日期输入
如果DateStyle参数默认为"MDY"，则表示被“月 - 日 - 年”解析；如果参数设置为"DMY"，则按照“日 - 月 - 年”解析；设置为"YMD"，按照“年 - 月 - 日”解析。
```SQL
create table t (col1 date);
insert into t values (date '12-10-2010');
show datestyle;
set datestyle="YMD"
```
| 例子 | 描述 |
| :-------------: | :-------------: |
| date'April 26, 2011' | 在任何datestyle输入模式下都无歧义 |
| date'2011-01-08' | ISO 8601格式（建议格式），任何方式下都是2011年1月8号， 而不会是2011年8月1日 |
| date'1/8/2011' | 有歧义，“MDY”：2011年1月8日，“DMY”：2011年8月1日 |
| date'1/18/2011' | “MDY”：2011年1月18日，其他模式下被拒绝 |
| date'03/04/11' | “MDY”：2011年3月4日，“DMY”：2011年4月3日， “YMD”：2003年4月11日 |
| date'2011-Apr-08', date'Apr-08-2011', date'08-Apr-2011' | 2011年4月8日 |
| date'19980405' | ISO 8601格式，1998年4月5日 |
| date'110405' | ISO 8601格式， 2011年4月5日 |
| date'2011.062' | 2011年的第62天，即2011年3月3日 |
| date'J2455678' | 儒略日，即从公元前4713年1月1日起到今天过去的天数，多为天文学家使用。2455678天，即2011年4月26日 |
| date'April 26,202BC' | 公元前4月26日 |
#### 时间输入
time被认为是time without time zone的类型，这样即使字符串中有时区，也会被忽略：
```SQL
select time '04:05:06';
select time '04:05:06 PST';
select time with time zone '04:05:06 PST'
```
时间字符串可以用冒号作为分隔符，即输入格式为"hh:mm:ss"，也可以不用分隔符。
| 例子 | 描述 |
| :-------------: | :-------------: |
| time '22:55:06.789', time '22:55:06', time '22:55', time '225506' | ISO 8601 |
| time with time zone '22:55:06.789+8', '22:55:06+08:00', '225506+08' | ISO 8601 |
| time with time zone '22:55:06 CCT' | 缩写的时区 |
| select time with time zone '2003-04-12 04:05:06 Asia/Chongqing';| 用名字声明时区 |
注意，最好不要用时区缩写来表示时区。PostgreSQL中的时区缩写可以查询PostgreSQL中的时区缩写可以查询pg_timezone_abbrevs。
#### 特殊值
epoch, infinity, -infinity, now, today, tomorrow, yesterday, allballs
#### 函数和操作符
日期、时间和interval类型之间可以进行加减乘除运算。P75-76，表5-18。
函数：
| 函数 | 描述 |
| :-------------: | :-------------: |
| age(timestamp, timestamp) | 参数相减后的“符号化”结果 |
| age(timestamp) | 从current_date减去参数后的结果 |
| clock_timestamp | 实时时钟的当前时间戳 |
| current_date | 当前日期 |
| current_time | 当前时间 |
| current_timestamp | 当前事务开始时的时间戳 |
| date_part(text, timestamp/interval) | 获取子域（等效于extract）：date_part('hour', timestamp '2001-02-16 20:38:40') → 20 |
| date_trunc(text, timestamp) | 截断成指定的精度：date_trunc('hour', timestamp '2001-02-16 20:38:40') → 2001-02-16-20:00:00 |
| extract(field from timestamp/interval) | 获取子域 |
| isfinite(timestamp/interval) | 测试是否为有穷时间戳/隔 |
| justify_days(interval) | 按每月30天调整时间间隔：justify_days(interval '30 days') → 1 month |
| justify_hours(interval) | 按每天24小时调整时间间隔：justify_hours(interval '24 hours') → 1 day |
| justify_interval(interval) | 使用justify_days和justify_hours调整时间间隔的同时进行正负号调整：justify_interval(interval '1 mon - 1 hour') → 29 days 23:00:00 |
| localtime | 当日时间 |
| localtimestamp | 当前事务开始的时间戳 |
| now() | 当前事务开始时的时间戳 |
| statement_timestamp() | 实时时钟的当时时间戳 |
| timeofday() | 与clock_timestamp相同，但结果是一个text字符串 |
| transaction_timestamp() | 当前事务开始时的时间戳 |
除了这些函数以外，还支持SQL的overlaps操作符：
(start1, end1) overlaps (start2, end2)
(start1, length1) overlaps (start2, length2)
这个表达式在两个时间域重叠的时候生成真值。
#### 时间函数
current_time 和 current_timestamp 返回带有时区的值，localtime 和 localtimestamp 返回不带时区的值。这四个函数都可以接受一个精度参数，该精度会导致结果的秒数域四舍五入到指定的小数位。如果没有精度参数，将给予所能得到的全部精度。这些函数全部都是按照当前事务的开始时刻返回结果的，所以它们的值在事务运行的整个期间内都不会改变。例如在`bigin;`，`end;`事务中，这些函数的调用值都保持相同。
statement_timestamp()，clock_timestamp()，timeofday()这些函数在事务中返回实时时间值。
所有日期/时间类型还接受特殊的文本值now，用于声明当前的日期和时间（乃当前事务的开始时刻）。因此，下面三个都返回相同的结果：
```SQL
select current_timestamp;
select now();
select timestamp with time zone 'now';
```
```SQL
extract(field from source) # return double precision
```
field 的值：century、year、decade、millennium、quarter、month、week、dow[求日期是星期几，0为星期天]、day[本月的第几天]、doy[本年的第几天]、hour、minute、second、epoch[对于date和timestamp，是自1970-01-01 00:00:00以来的秒数（可为负）；对interval，它是时间间隔的总秒数]、milliseconds[秒域乘以1000]、microseconds、timezone、timezone_hour、timezone_minute。

### 枚举类型
#### 枚举类型的使用
在PostgreSQL中，使用枚举类型需要先使用create type创建一个枚举类型。
```SQL
create type week as enum ('Sun', 'Mon', 'Tues', 'Wed', 'Thur', 'Fri', 'Sat');
create table duty (person text, weekday week);
insert into duty (person, weekday) values
  ('张三', 'Sun'),
  ('李四', 'Mon'),
  ('王二', 'Tues'),
  ('赵五', 'Wed');
select * from duty where weekday = 'Sun';
# 输入的字符不在枚举类型之间，则会报错：
select * from duty where weekday = 'Sun.';
# 使用“\dT”命令查看枚举类型的定义：
\dT+ week
# 直接查询表pg_enum也可以看到枚举类型的定义：
select * from pg_enum;
```
#### 枚举类型的说明
在枚举类型中，值得顺序是创建枚举类型时定义的顺序。所有比较标准的运算符及其他相关的聚集函数都可支持枚举类型：
```SQL
select min(weekday), max(weekday) from duty;
select * from duty where weekday = (select max(weekday) from duty);
```
一个枚举值占4字节。一个枚举值的文本标签长度由namedatalen设置并编译到PostgreSQL中，且是以标准编译方式进行的，也就意味着至少是63字节。枚举类型的值是大小写敏感的。
#### 枚举类型的函数
| 函数 | 描述 |
| :-------------: | :-------------: |
| enum_first(anyenum) | 返回枚举类型的第一个值：enum_first('Mon'::week) → Sun |
| enum_last(anyenum) | 返回枚举类型的最后一个值 |
| enum_range(anyenum) | 以一个有序的数组形式返回输入枚举类型的所有值 |
| enum_range(anyenum, anyenum) | 以一个有序的数组返回在给定的两个枚举值之间的范围。若第一个为空，从第一个开始；若第二个为空，到最后一个结束：enum_range(null, 'Wed'::week) → {Sun, Mon, Tues, Wed} |
除了两个参数形式的enum_range外，其余这些函数会忽略传递给它们的具体值，使用null加上类型转换也将得到相同的结果。

### 几何类型
#### 几何类型概况
| 类型名称 | 描述 | 表现形式 |
| :----: | :----: | :----: |
| point | 平面中的点 | (x,y) |
| line | 直线 | ((x1,y1),(x2,y2)) |
| lseg | 线段 | ((x1,y1),(x2,y2)) |
| box | 矩形 | ((x1,y1),(x2,y2)) |
| path | 闭合路径 | ((x1,y1),...) |
| path | 开放路劲 | [(x1,y1),...] |
| polygon | 多边形 | ((x1,y1),...) |
| circle | 圆 | <(x,y),r> |
#### 几何类型的输入
`类型名称 '表现形式'` or `'表现形式'::类型名称`
```SQL
select '1,1'::point;
select '(1,1)'::point;
select lseg '1,1,2,2';
select lseg '(1,1),(2,2)';
select lseg '((1,1),(2,2))';
select lseg '[(1,1),(2,2)]';
select path '(1,1),(2,2)'; -- 默认为闭合
select box '1,1,2,2';
select box '(1,1),(2,2)';
select box '((1,1),(2,2))';
select circle '1,1,5';
select circle '((1,1),5)';
select circle '<(1,1),5>';
```
#### 几何类型的操作符
1. 平移运算符`+`、`-`及缩放/旋转运算符`*`、`/`
这四个运算符都是二元运算符，运算符左值可以是`point`、`box`、`path`、`circle`，运算符的右值只能是`point`：
```SQL
-- 点与点的加减乘除相当于两个复数之间的加减乘除
select point '(1,2)' + point '(10,20)'; -- (11,22)
select point '(1,2)' - point '(10,20)'; -- (-9,-18)
select point '(1,2)' * point '(10,20)'; -- (-30,40)
select point '(1,2)' / point '(10,20)'; -- (0.1,0)
-- 矩形与点之间的运算：
select box '((0,0),(1,1))' + point '(2,2)'; -- (3,3),(2,2)
select box '((0,0),(1,1))' - point '(2,2)'; -- (-1,-1),(-2,-2)
select box '((0,0),(1,1))' * point '(2,2)'; -- (0,4),(0,0)
-- 路径与点之间的运算：
select path '(0,0),(1,1),(2,2)' + point '(10,20)'; -- (10,20),(11,21),(12,22)
select path '(0,0),(1,1),(2,2)' - point '(10,20)'; -- (-10,-20),(-9,-19),(-8,-18)
-- 圆与点之间的运算：
select circle '((0,0),1)' + point '10,20'; -- <(10,20),1>
select circle '((0,0),1)' - point '10,20'; -- <(-10,-20),1>
-- 对于乘法，如果乘数的y值为0，比如point 'x,0'，则相当于集合对象缩放x 倍：
select circle '((0,0),1)' * point '(3,0)'; -- <(0,0),3>
select circle '((1,1),1)' * point '(3,0)'; -- <(3,3),3>
-- 如果乘数为point '0,1'，则相当于几何图像逆时针旋转90度，如果乘数为point '0,-1'，则表示顺时针旋转90度：
select circle '((1,1),1)' * point '(0,1)'; -- <(-1,1),1>
```
2. 运算符 `#`
对于两个线段，计算出交点；对于两个矩形，计算出相交的矩形；对于路径或多边形，计算出顶点数。
```SQL
select lseg '(0,0),(2,2)' # lseg '(0,2),(2,0)'; -- (1,1)
-- 如果两个线段没有相交，则返回空。
-- 两个矩形：
select box '(0,0),(2,2)' # box '(1,0),(3,1)'; -- (2,1),(1,0)
-- 路径或多边形
select # path '(1,1),(2,2),(3,3)'; -- 3
select # polygon '(1,1),(2,2),(3,3)'; -- 3
```
3. 运算符 `@-@`
此为一元运算符，参数只能为`lseg`、`path`。一般用于计算几何对象的长度：
```SQL
select @-@ lseg '(0,0),(1,1)'; -- 1.41421235623731
select @-@ path '(0,0),(2,2)'; -- 5.65685424949238
select @-@ path '(0,0),(1,1),(2,2)'; -- 5.65685424949238
select @-@ path '[(0,0),(1,1),(0,1)]'; -- 2.41421235623731
select @-@ path  '((0,0),(1,1),(0,1))'; -- 3.41421235623731
```
4. 运算符 `@@`
一元运算，用于计算中心点：
```SQL
select @@ circle '<(1,1),2>';
select @@ box '(0,0),(1,1)'; -- (0.5,0.5)
select @@ lseg '(0,0),(1,1)'
```
5. 运算符 `##`
二元运算符，用于计算两个几何对象上离得最近的点：
```SQL
select point '(0,0)' ## lseg '(2,0),(0,2)'; -- (1,1)
select point '(0,0)' ## box '(1,1),(2,2)'; -- (1,1)
select lseg '(1,0),(0,1.5)' ## lseg '((2,0),(0,2))'; -- (0,1.5)
```
6. 运算符 `<->`
二元运算符，用于计算两个几何对象之间的距离：
```SQL
select lseg '(0,1),(1,0)' <-> lseg '(0,2),(2,0)';
select circle '((0,0),1)' <-> circle '((3,0),1)'; -- 1
-- 对于两个矩形，实际上是中心点之间的距离：
select box '((0,0),(1,1))' <-> box '((2,0),(4,1))'; -- 2.5
```
7. 运算符 `&&`
二元运算符，用于计算两个几何对象之间是否重叠，只要有一个共同点，则为真：
```SQL
select box '((0,0),(1,1))' &&  box '((1,1),(2,2))'; -- t
select box '((0,0),(1,1))' && box '((2,2),(3,3))'; -- f
select circle '((0,0),1)' && circle '((1,1),1)'; -- t
select polygon '(0,0),(2,2),(0,2)' && polygon '(0,1),(1,1),(2,0)'; -- t
```
8. 判断两个对象相对位置的运算符
判断左右：
`<<`: 是否严格在左
`>>`: 是否严格在右
`&<`: 没有延展到右边
`&>`: 没有延展到左边
判断上下：
`<<|`: 严格在下
`|>>`: 严格在上
`&<|`: 没有延展到上面
`|&>`: 没有延展到下面
`<^`: 在下面（允许接触）
`>^`: 在上面（允许接触）
其他：
`?#`: 是否相交
`?-`: 是否水平或水平对齐
`?|`: 是否竖直或数值对齐
`?-|`: 两个对象是否垂直
`?||`: 两个对象是否平行
`@>`: 是否包含
`<@`: 包含或在其上
```SQL
select box '((0,0),(1,1))' << box '((1.1,1.1),(2,2))'; -- t
select polygon '(0,0),(0,1),(1,0)' << polygon '(0,1.1),(1.1,1),(1.1,0)'; -- f
select polygon '(0,0),(0,1),(1,0)' << polygon '(1.1,0),(1.1,1),(2,0)'; -- t
select circle '((0,0),1)' << circle '((1,1),1)'; -- f
select circle '((0,0),1)' << circle '((3,3),1)'; -- t
```
9. 判断两个几何对象是否相同的运算符 `~=`
```SQL
select polygon '((0,0),(1,1))' ~= polygon '((1,1),(0,0))'; -- t
select polygon '(0,0),(1,1),(1,0)' polygon '(1,1),(0,0),(1,0)'; -- t
select box '(0,0),(1,1)' ~= box '(1,1),(0,0)'; -- t
```
#### 几何类型的函数
| 函数 | 描述 |
| :----: | :----: |
| area(object) | 面积 |
| center(object) | 中心 |
| diameter(circle) | 直径 |
| height(box) | 高度 |
| width(box) | 宽度 |
| isclosed(path) | 是否闭合 |
| isopen(path) | 是否开放 |
| length(object) | 长度 |
| npoints(path/polygon) | 点数 |
| pclose(path) | 把路径转成闭合路径 |
| popen(path) | 把路径转成开放路径 |
| radius(circle) | 半径 |

转换函数：box(circle)、box(point,point)、box(polygon)、 circle(box)、circle(point, double precision)、circle(polygon)、lseg(box)、lseg(point,point)、path(polygon)、point(double precision, double precision)、point(box)、point(circle)、point(lseg)、point(polygon)、polygon(box)、polygon(circle)、polygon(npts,circle)、polygon(path)。

### 网络地址类型
PostgreSQL提供专门数据类型存储IPv4、IPv6和MAC地址。
| 类型 | 存储空间 | 描述 |
| :----: | :----: | :----: |
| cidr | 7或19字节 | IPv4或IPv6的网络地址 |
| inet | 7或19字节 | IPv4或IPv6的网络地址或主机地址 |
| macaddr | 6 字节 | 以太网MAC地址 |
> refer to P98-P103

### 复合类型
在PostgreSQL中可以如C语言中的结构体一样定义一个符合类型。
#### 复合类型的定义：
```SQL
create type complex as (
  r double precision,
  i double precision
);
create type person as (
  name text,
  age integer,
  sex boolean
);
-- 目前不能声明约束（比如 not null）
create table capacitance_test_data (
  test_time timestamp,
  voltage complex,
  current complex
);
-- 可使用此类型作为函数参数：
create function complex_multi(complex, complex) returns complex
  as $$ select row($1.r*$2.r - $1.i$2.i, $1.r*$2.i - $1.i*$2.r)::complex as language sql;
```
#### 复合类型的输入
`( val1, val2, ...)` 单引号加圆括号的形式。在此格式中，可以在任何字段值周围放上双引号，如果值本身包含逗号或圆括弧，则必须用双引号括起。要让一个字段值是null，那么在列表里它的位置上就不要写任何字符。要一个空字符，则写一对双引号。也可以用row表达式语法来构造复合类型值。表达式超过一个字段，关键字row可省略。
```SQL
create type person as (
  name text,
  age integer,
  sex boolean
);
create table author (
  id int,
  person_info person,
  book text
);
insert into author values (1, '("张三", 29, true)', '张三的自传');
insert into author values (2, row('李四',29, true), '自传');
insert into author values (3, ("王五", 28, true), '自传');
```
#### 访问复合类型
访问复合类型字段的一个域就如在C语言中访问结构体中的一个成员一样，即写出一个点和域的名字就可以了。
```SQL
select person_info.name from author; -- error
select (person_info).name from  author;
select (author.person_info).name from author;
select (my_func(...)).field from ...
```
#### 修改复合类型
```SQL
-- 更新整个字段：
insert into author values (('张三', 29, true), '自传');
update author set person_info = row('李四', 39, true) where id = 1;
update author set person_info = ('王二', 49, true) where id = 2;
-- 更新眸子子域：
update author set person_info.name = '王二二' where id =2;
update author set person_info.age = (person_info).age + 1 where id = 2;
-- 不能在set后面出现的字段名周围加上圆括号，但需要在等号右边的表达式中引用同一个字段则需加上圆括号，否则会报错。在insert也可以指定复合字段的子域，未指定子域用null填充：
insert into author (id, person_info.name, person_info.age) values (10, '张三', 29);
```
#### 复合类型的输入与输出
P106

### XML类型
要使用xml数据类型，在编译PostgreSQL源码时必须使用`--with-libxm`参数。
#### xml类型的输入
格式： xml '<tag>hello world</tag>' 或 '<tag>hello world</tag>'::xml类型
xml中存储的xml数据有两种：
- 由xml标准定义的“documents”。
- 由xml标准定义的“content”片段。
“content”片段可以有多个顶级元素或“character”节点，但“documents”只能有一个顶级元素。可以使用“xmlvalue is document”来判断一个特定的xml值是一个“documents”还是“content”片段。
PostgreSQL的xmloption参数用来指定输入的数据是“documents”还是“content”片段，默认情况下此值为“content”片段。修改成“document”将不能输入有多个顶级元素的内容：
```SQL
show xmloption;
select xml '<a>a</a><b>b</b>';
set xmloption to document;
select xml '<a>a</a><b>b</b>'; -- error
```
也可以由函数xmlparse通过字符串数据来产生xml类型的数据，使用xmlparse函数是SQL标准中将字符串转换成XML的唯一方式。语法如下：
xmlparse ( ( document | content) value)
其中参数“document”和“content”表示指定XML数据的类型：
```SQL
select xmlparse (document '<?xml version="1.0"?><person><name>John</name><sex>F</sex></person>');
select xmlparse (content '<person><name>John</name><sex>F</sex></person>');
```
#### 字符集的问题
提交输入到xml类型的字符串中的编码声明会被忽略掉，同时内容的字符集会被认为是当前数据库服务器的字符集。
正确处理XML字符集的方式是，先将XML数据的字符串在当前客户端中编码成当前客户端的字符集，然后再发送到服务端，这样会被转换成服务器的字符集存储。当查询xml类型的值时，数据又会被转换成客户端字符集，所以客户端收到的xml数据的字符集就是客户端的字符集。
通常xml数据都是用UTF-8编码格式处理的，因此把PostgreSQL数据库服务器端编码也设置成UTF-8将是一种比较好的选择。
#### xml类型的函数
| 函数 | 描述 | 例子 | 结果 |
| :----: | :----: | :----: | :----: |
| xmlcomment(text) | 创建一个包含XML注释的特定文本内容的值。文本中不能包含“--”或不能以“-”结束。如果参数为空，结果也为空。 | xmlcomment('hello') | \<!--hello--> |
| xmlconcat(xml[, ...]) | 把XML值列表拼接成XML“content”片段。忽略列表中的空值，只有当参数都为空时结果才是空 | xmlconcat('\<a/>', '\<os>linux\</os>') | \<a/>\<os>linux\<os/> |
| xmlelement(name, name [, xmlattributes (value [as attname][, ...])][, content, ...]) | 生成一个带有给定名称、属性和内容的xml元素 | xmlelement(name linux, xmlattributes('2.6.18' as version)) | \<linux version="2.6.18"/> |
| xmlforest(content [as name][, ...]) | 把指定的名称和内容元素生成为一个XML“森林” | xmlforest('2.6' as linux, 5.0 as vers) | \<linux>2.6\</linux>\<vers>5.0\</vers> |
| xmlpi(name target[, content]) | 创建一条XML处理指令。内容不能包含字符序列“?>” | xmlpi(name php, 'echo "hello world";') | \<php echo "hello world";?> |
| xmlroot(xml, version text \| no value[, standalone yes\|no\|no value]) | 更改root节点的属性。如果指定version，它将替换root节点的version值，如果指定一个standalone，它替换根节点的standalone值 | xmlroot(xmlparse(document '\<?xml version="1.1" standalone="no"?>\<content>abc\</content>'), version '1.0', standalone yes) |
| xmlagg(xml) | 聚合函数，把多行的xml数据聚合成一项 | create table test (i int, d xml); insert into test values (1, '\<a>a\</a>'); insert into test values (2, '\</b>'); select xmlagg(d) from test; | \<a>a\</a>\<b/> |
PostgreSQL提供了xpath函数来计算XPath1.0表达式的结果。XPath是W3C的一个标准，它最主要的目的是在XML1.0或XML1.1文档节点树中定位节点。目前有XPath1.0和XPath2.0两个版本。其中，XPath1.0是1990年成为W3C标准的，而XPath2.0标准的确立是在2007年。
xpath函数定义如下：
xapth(xpath, xml[, nsarray])
此函数的第一个参数是XPath1.0表达式，第二个参数是xml类型的值，第三参数是命名空间的数组映射。这个数组应该是一个两维数组，第二维的长度等于2，即包含2个元素。
示例：
```SQL
select xpath('/my/text()', '<my:a xmlns:my="http://example.com">test</my:a>', array [array['my', 'http://example.com']]);
select xpath('//mydefns:b/text()', '<a xmlns="http://example.com"><b>test</b></a>', array[array['mydefns', 'http://example.com']]);
```
PostgreSQL还提供了把数据库中的内容导出成XML数据的一些函数：
- table_to_xmlschema(tbl regclass, nulls boolean, tableforest boolean, targetns text)
- query_to_xmlschema(query text, nulls boolean, tableforest boolean, targetns text)
- cursor_to_xmlschema(cursor refcursor, nulls boolean, tableforest boolean, targetns text)
- table_to_xml_and_xmlschema(tbl regclass, nulls boolean, tableforest boolean, targetns text)
- query_to_xml_and_xmlschema(query text, nulls boolean, tableforest boolean, targetns text)
- schema_to_xml(schema name, nulls boolean, tableforest boolean, targetns text)
- schema_to_xmlschema(schema name, nulls boolean, tableforest boolean, targetns text)
- schema_to_xml_and_xmlschema(schema name, nulls boolean, tableforest boolean, targetns text)
- database_to_xml(nulls boolean, tableforest boolean, targetns text)
- database_to_xmlschema(nulls boolean, tableforest boolean, targetns text)
- database_to_xml_and_xmlschema(nulls boolean, tableforest boolean, targetns text)
```SQL
create table test01(id int, note text);
insert into test01 select seq, repeat(seq::text, 2) from generate_series(1,5) as t(seq);
select table_to_xmlschema('test01'::regclass, true, true, 'tangspace');
-- table_to_xmlschema把表的定义转成了xml格式
select query_to_xmlschema('select * from test01', true, true, 'tangspace');
-- query_to_xmlschema把查询结果中行的定义转成了xml
select table_to_xml_and_xmlschema('test01'::regclass, true, true, 'tangspace');
-- table_to_xml_and_xmlschema与table_to_xmlschema唯一不同的地方在于把表中的数据也导入到xml中了。
select schema_to_xml('public', true, true, 'tangspace');
-- schema_to_xml把schema中的数据导成了xml格式。
```

### JSON类型
JSON：JavaScript Object Notation
#### JSON类型简介
JSON类型是把输入的数据原封不动地存放到数据库中，使用的时候需要重新解析数据，而JSONB类型是在存放的时候就把JSON解析成二进制格式了，使用的时候就不需要再次解析了，所以JSONB在使用时性能会更高。此外，JSONB支持在其上建索引。
JSONB类型不会保留多余的空格，不会保留key的顺序，也不会保留重复的key。
可以使用\uXXXX形式的转义，从而忽视数据库的字符集编码。
#### JSON类型的输入与输出
```SQL
select '9'::json, '"osdba"'::json, 'true'::json, 'null'::json;
select json '9', json '"osdba"', json 'true', json 'null';
-- 使用jsonb的类型也一样
select '[9, true, "osdba", null]'::json, jsonb '[9, true, "osdba", null]';
select json '{"name": "osdba", "age": 40, "sex": true, "money": 250.12}';
-- 对于输入带小数点的情况：
select json '{"p": 1.6735777674525e-27}'; -- {"p": 1.6735777674525e-27}
select jsonb '{"p": 1.6735777674525e-27}'; -- {"p": 0.0000000000000000000000000016735777674525}
```
#### JSON类型的操作符
| 操作符 | 右操作符类型 | 描述 | 例子 | 结果 |
| :---: | :---: | :---: | :---: | :---: |
| -> | int | 取JSON数组的元素 | '[1,2,3]'::json->1 | 2 |
| -> | text | 通过key取JSON中的子对象 | '{"a":1, "b":2}'::json->'a' | 1 |
| ->> | int | 取JSON数组的元素，但返回的是一个text类型 | '[1,2,3]'::json->>1 | '2' |
| ->> | text | 通过key取JSON中的子对象，但返回是一个text类型 | '{"a":1, "b":2}'::json->>'a' | '1' |
| #> | text[] | 通过指定路径取JSON中的对象 | '{"a":{"b":{"c":1}}}'::json#>'{a,b}' | {"c":1} |
| #>> | text[] | 同上，但返回的是一个text类型 | '{"a":{"b":{"c":1}}}'::json#>>'{a,b,c}' | '1' |
仅可用于JSONB的操作符：
| 操作符 | 描述 |     
| :---: | :---: |
| = | 两个JSONB对象的内容是否相同 |
| @> | 左边的JSONB对象是否包含右边的JSONB对象 |
| <@ | 左边的JSONB对象是否包含于右边的JSONB对象中 |
| ? | 指定的字符串是否存在于JSON对象中的key或字符串类型元素中。注意JSON中的元素需要时字符串类型 |
| ?\| | 右值是一个数组，指定此数组中的任意一个元素是否存在于JSON对象的字符串类型的key或元素中。注意JSON中的元素需要时字符串类型 |
| ?& | 右值是一个数组，指定此数组中的任意一个元素是否存在于JSON对象的字符串类型的key或元素中。注意JSON中的元素需要时字符串类型 |
#### JSON类型的函数
P118~P121
#### JSON 类型的索引
因为JSON类型没有提供相关的比较函数。所以在JSON类型的列上无法直接建索引，但可以在JSON类型的列上建函数索引。
JSONB类型的列上可以直接建索引。除了可以建BTree索引以外，JSONB还支持建GIN索引，GIN索引可以高效地从JSONB内部的key/value对中搜索数据。
在JSONB上创建GIN索引的方式有两种：
- 使用默认的jsonb_ops操作符创建
- 使用jsonb_path_ops操作符创建

create index idx_name on table_name using gin (index_col);
create index idx_name on table_name using gin (index_col jsonb_path_ops);
关于GIN索引，jsonb_ops的与jsonb_path_ops的区别为：在json_ops的GIN索引中，JSONB数据中的每个key和value都是作为一个单独的索引项的，而jsonb_path_ops则只为每个value创建了一个索引项。
```SQL
create table jtest01 (
  id int,
  jdoc json
);
create or replace function random_string(integer) returns text as
$BODY$
select array_to_string(
  array (
    select substring(
      '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz'
      from (ceil(random()*62))::int for 1
    )
    from generate_series(1, $1)
  ),
  ''
)
$BODY$
language sql volatile;
insert into jtest01 select t.seq, ('{"a":{"a1":"a1a1", "a2":"a2a2"}, "name":"'||random_string(10)||'", "b":"bbbbb"}'):: json from generate_series(1,100000) as t(seq);
-- 用函数json_extract_path_text建立一个函数索引：
create index on jtest01 using btree (json_extract_path_text(jdoc,'name'));
analyze jtest01;
-- 看看查询没有走索引的执行计划：
explain analyze verbose select * from jtest01 where jdoc->>'name'='lnBtcJLR85'; -- Seq Scan
-- 看看走了此函数索引的执行计划：
explain analyze verbose select * from jtest01 where json_extract_path_text(jdoc, 'name') = 'lnBtcJLR85'; -- Index Scan
-- 在JSONB类型上建GIN索引：
create table jtest02 (
  id int,
  jdoc jsonb
);
create table jtest03 (
  id int,
  jdoc jsonb
);
insert into jtest02 select id, jdoc::jsonb from jtest01;
insert into jtest03 select * from jtest02;
create index idx_jtest02_jdoc on jtest02 using gin (jdoc);
create index idx_jtest03_jdoc on jtest03 using gin (jdoc jsonb_path_ops);
analyze jtest02;
analyze jtest03;
select * from jtest02 where jdoc @> '{"name":"lnBtcJLR85"}';
select * from jtest03 where jdoc @> '{"name":"lnBtcJLR85"}';
explain analyze verbose select * from jtest02 where jdoc @> '{"name":"lnBtcJLR85"}'; -- Bitmap Heap Scan
explain analyze verbose select * from jtest03 where jdoc @> '{"name":"lnBtcJLR85"}'; -- Bitmap Heap Scan
-- 这两个SQL都走到了索引上，查看索引大小
select pg_indexes_size('jtest02');
select pg_indexes_size('jtest03');
```

### Range类型
#### Rangele类型简介
用于表示范围，如一个整数的范围、一个时间的范围，而范围底下的基本类型（如整数、时间）则被称之为Range类型的subtype。
假设我们有一个需求，某个IP地址库记录了每个地区的IP地址范围，现在需要查询客户的IP地址在哪个地区。该IP地址库的定义如下：
```SQL
create table ipdb1 (
  ip_begin inet,
  ip_end inet,
  area text,
  sp text);
```
如果要查询的是IP地址115.195.180.105在哪个地区：
```SQL
select * from ipdb1 where ip_begin <= '115.195.180.105'::inet and ip_end >= '115.195.180.105'::inet;
```
因为表上没有索引，所以会进行全表扫描，故而很慢。这时可以在ip_begin和ip_end上建索引：
```SQL
create index idx_ipdb_ip_start on ipdb1(ip_begin);
create index idx_ipdb_ip_end on ipdb1(ip_end);
```
在PostgreSQL中，上面的SQL可以使用到这两个索引，但都是先分别扫描两个索引建位图，然后通过位图进行and操作：
```SQL
explain analyze verbose select * from ipdb1 where ip_begin <= '115.195.180.105'::inet and ip_end >= '115.195.180.105'::inet; -- Bitmap Heap Scan
```
使用Range类型，通过创建空间索引的方式来执行：
首先，创建类似的IP地址库表：
```SQL
create type inetrange as range (subtype = inet);
create table ipdb2 (
  ip_range inetrange,
  area text,
  sp text);
insert into ipdb2 select ('[' || ip_begin || ',' || ip_end ']') ::inetrange, area, sp from ipdb1;
```
然后，创建gist索引：
```SQL
create index id_pdb2_ip_range on ipdb2 using gist (ip_range);
select * from ipdb2 where ip_range @> '115.195.180.105':inet;
explain analyze verbose select * from ipdb2 where ip_range @> '115.195.180.105'::inet;
```
#### 创建Range类型
内置：int4range, int8range, numrange, tsrange(无时区的时间戳范围类型), tstzrange, daterange.
创建语法：
```SQL
create type name as range (
  subtype = type -- 指定子类型
  [, subtype_opclass = subtype_operator_class ] -- 指定子类的操作符
  [, collation = collation ] -- 指定排序规则
  [, canonical = canonical_function ] -- 创建一个稀疏的Range类型，而非连续的Range类型
  [, subtype_diff = subtype_diff_function ] -- 定义子类型的差别函数
)
-- 示例：
create type floatrange as range (
  subtype = float8,
  subtype_diff = float8mi
);
```
#### Range类型的输入与输出
'[value1, value2]', '[value1, value2)', '(value1, value2]', '(value1, value2)', 'empty'。如果是稀疏型的range，其内部的存储格式为“[value1, value2)”.
```SQL
select '(0,6)'::int4range; -- [1,6)
select '[0,6]'::int4range; -- [0,7)
-- int4range总是把输入格式转换成“'[value1, value2)'”格式。
-- 对于连续型的range，内部存储则是精确存储的
select '[0,6]'::numrange; -- [0,6]
select '[0,6)'::numrange; -- [0,6)
-- Range类型还可以表示极值的区间：
select '[1,)'::int4range; -- [1,)
select '[,1)'::int4range; -- (,1)
-- 使用Range类型的构造函数输入Range类型的值，Range类型的构造函数名称与类型名称相同：
select int4range(1,10,'[)'); -- [1,10)
select int4range(1,10,'()'); -- [2,10)
select int4range(1,10); -- [1,10)
```
#### Range类型的操作符
=, <>, <, >, <=, >=, @>(包含), <@(被包含), &&(重叠), <<(严格在左), >>(严格在右), &<(没有扩展到右边), &>(没有扩展到左边), -|-(连接在一起), +(union), \*(intersection), -(difference)
#### Range类型的函数
- lower(anyrange), 获得范围的起始值
lower(int4range '(11,22)') -- 12
lower(int4range '[,22)') is null -- t
lower(int4range 'empty') is null --t
- upper(anyrange), 获得范围的结束值
upper(int4range '[11,22)') -- 22
- isempty(anyrange), 是否是空范围
isempty(int4range '(,)') -- f
isempty(int4range '(1,1)') -- t
- lower_inc(anyrange), 起始值是否在范围内
lower_inc(int4range '[1,1)') -- f
- upper_inc(anyrange), 结束值是否在范围内
upper_inc(int4range '[1,2)') -- f
- lower_inf(anyrange), 起始值是否是一个无穷值
- upper_inf(anyrange), 结束值是否是一个无穷值
#### Range类型的索引和约束
在Range的列上可以创建GiST和SP-GiST索引：
```SQL
create index index_name on table_name using gist (range_column);
```
在SQL查询语句中，可以使用=, &&, <@, @>, <<, >>, -|-, &<, &>来执行索引。
在Range类型上也可以建Btree索引，但Btree索引是使用比较运算符的，通常只有在对Range的值进行排序时使用。
在Range的列上也可以建立约束，让其范围总是不重叠：
```SQL
create table rtest01 (
  idrange int4range,
  exclude using gist (idrange with &&)
);
insert into rtest01 values (int4range '[1,5)');
insert into rtest01 values (int4range '[4,5)'); -- error
```
如果是一个有两列的表，第一列是普通类型，第二列是Range类型， 若要让第一列值相等的行其第二列的值也不重叠，则需要使用另一个扩展模块btree_gist来实现了：
首先，安装btree_gist：
```bash
cd source_code_path/contrib/btree_gist
make
make install
```
使用示例：
```SQL
create extension btree_gist;
create table rtest02 (
  id int,
  idrange int4range,
  exclude using gist (id with =, idrange with &&)
);
insert into rtest02 values (1, int4range '[1,5)');
insert into rtest02 values (2, int4range '[2,5)');
insert into rtest02 values (1, int4range '[2,5)'); -- error
```

### 数组类型
#### 数组类型的声明
```SQL
create table testtab04(id int, col1 int[], col2 int[10], col3 text[][]);
```
在目前的PostgreSQL中，如果在定义数组类型中填一个数组长度的数字，并不会限制数组的长度；定义时指定数组的维度也是没有意义的，数组的维度是根据实际插入的数据来确定的。
#### 如果输入数组值
```SQL
create table testtab05 (id int, col1 int[], col2 text[], col3 box[]);
insert into testtab05 values (1, '{1,2,3}', '{how, how many, "who, what.", "It''s ok", "{os\"dba}"}', '{((1,1), (2,2)); ((3,3), (4,4)); ((1,2), (7,9))}');
-- 在PostgreSQL中，每个类型都定义的一个分隔号：
select typname, typdelim, from pg_type where typname in ('int4', 'int8', 'bool', 'char', 'box'); -- ,,,,;
-- 一个简单的数组构造器由关键字array、一个左方括号（[）、一个或多个元素值得表达式（用逗号分隔）、一个右方括号组成：
insert into testtab05 values (2, array[1,2,3], array['how', 'how many', 'who, what.', 'It''s ok', '{os"dba}'], array['((1,1),(2,2))'::box,box '(3,3),(4,4)','1,2,7,9'::box]);
-- 多维数组：
create table testtab06 (id int, col1 text[][]);
insert into testtab06 values (1, array[['os', 'dba'], ['dba', 'os']]);
-- 向多维数组中插入值时，各个维度的元素个数必须相同
insert into testtab06 values (2, '{{a,b}, {c,d,e}}'); -- error
insert into testtab06 values (2, '{{a,b,null}, {c,d,e}}');
-- PostgreSQL数据库中数组的下标是从1开始的，但也可以指定下标的开始值：
create table test02 (id int[]);
insert into test02 values ('[2:4]={1,2,3}');
select id[2], id[3], id[4] from test02;
```
#### 访问数组
```SQL
create table testtab07 (id int, col1 text[]);
insert into testtab07 values (1, '{aa, bb, cc, dd}');
select * from testtab07;
select id, col1[1] from testtab07;
select id, col1[1:2] from testtab07;
create table testtab08 (id int, col1 int[][]);
insert into testtab08 values (1, '{{1,2,3}, {4,5,6}, {7,8,9}}');
select id, col1[1][1], col1[1][2], col1[2][1], col1[2][2] from testtab08;
select id, col1[1:1] from testtab08; -- {{1,2,3}}
select id, col1[1:3][1:1] from testtab08; -- 等价于：
select id, col1[3][1:1] from testtab08;
-- PostgreSQL中规定，只要出现一个冒号，其他的单个下标将表示从1开始的一个切片，下标的数值表示该切片的结束值，“col1[3][1:2]”中的“col[3]”实际上表示的是“col[1:3]”，同理：
select id, col1[1:2][2] from testtab08;
```
#### 修改数组
```SQL
update testtab08 set col1='{{10,11,12}, {13,14,15}, {16,17,18}}' where id=1;
update testtab08 set col1[2][1]=100 where id=1;
-- 不能直接修改多维数组中某一维的值
```
#### 数组的操作符
=, <>, <, >, <=, >=, @>, <@, &&, ||([不]同维度的数组与数组连接; 元素类型必须相同), ||(元素与数组之间的连接; 元素与数组的元素类型必须相同)
#### 数组的函数
P139~P142

### 伪类型（Pseudo-Type）
是PostgreSQL中不能作为字段的数据类型，但它可以用于声明一个函数的参数或者结果类型：
any、anyelement、anyarray、anynoarray、anyenum、anyrange、cstring、internal、language_handler、fdw_handler、record、trigger、void、opaque。

### 其他类型
#### UUID类型
UUID（Universally Unique Identifiers）定义在“RFC 4122”和“ISO/IEC 9834-8:2005”中。它是一个128bit的数字。
#### pg_lsn类型
表示LSN（Log Sequence Number）的一种数据类型。LSN表示WAL日志的位置。
```SQL
\d pg_stat_replication View "pg_catalog.pg_stat_replication"
\d pg_replication_slots
```
在数据库内部，LSN是一个64bit的大整数，其输出的格式类似如下形式：
16/B374D848

## 第六章 逻辑结构管理
### 6.1 数据库逻辑结构介绍
在一个PostgreSQL数据库系统中，数据的组织结构可以分为以下三层：
1. 数据库
2. 表、索引：一般，在PostgreSQL中表的术语为“Relation”。
3. 数据行：在PostgreSQL中行的术语一般为“Tuple”。
>在PostgreSQL中，一个数据库服务(或叫实例)下可以有多个数据库，而一个数据库不能属于多个实例。在Oracle数据库中，一个实例只能有一个数据库，但一个数据库可以在多个实例中（如RAC）。

### 6.2 数据库基本操作
#### 6.2.1 创建数据库
语法：
```SQL
CREATE DATABASE name
  [ [ WITH ] [ OWNER [=] user_name ]
    [ TEMPLATE [=] template ]
    [ ENCODING [=] encoding ]
    [ LC_COLLATE [=] lc_collate ]
    [ LC_CTYPE [=] lc_ctype ]
    [ TABLESPACE [=] tablespace ] -- 指定和新数据库关联的表空间名
    [ CONNECTION LIMIT [=] connlimit ] ] -- 数据库可以接受多少并发的连接，默认为-1，表示没有限制
-- 默认模板数据库：template1, ISO-8859-1(LATIN1), template0公认不包含会受字符集编码或排序影响的数据或索引，故可以作为创建任意字符集数据库的模板。
create database testdb01 encoding 'LATIN1' template template0;
-- PostgreSQL数据库服务端并不支持通常的汉字字符集“GBK”、GB18030”，所以一般都是使用“UTF8”字符集来支持中文的。
```
#### 6.2.2 修改数据库
ALTER DATABASE name [ [ WITH ] option [ ... ] ]
option可以是：
- connection limit
- rename to new_name
- set tablespace new_tablespace
- set configuration_parameter {to|=} {value|default}
- set configuration_parameter from current
- reset configuration_parameter
- reset all
```SQL
-- 改变数据库testdb01的最大连接数为10
alter database testdb01 connection limit 10;
-- 重命名为mydb01
alter database testdb01 rename to mydb01;
-- 让用户一连接到这个数据库时，某个配置参数就设置为一个指定的值。如，关闭在数据库testdb01上的默认索引扫描：
alter database testdb01 set enable_indexscan to off;
```
#### 6.2.3 删除数据库
```SQL
drop database [ if exists ] name;
drop database mytestdb01; -- 不存在，报错
drop database if exists mytestdb01; -- 不存在，不报错
-- 如果还有人连接在这个数据库上，将不能删除该数据库
```
#### 6.2.4 常见问题及解答
- 不能在事务块中创建或删除数据库。
- 可以在事务块中修改数据库。
### 6.3 模式
#### 6.3.1 模式的定义
模式（schema）是数据库中的一个概念，可以将其理解为一个命名空间或目录。不同的模式下可以有相同名称的表、函数等对象且互相不冲突。只要有权限，每个模式的对象可以互相调用。
在MySQL中可以同时访问多个Database中的对象。
使用模式的几个主要原因：
- 允许多个用户在使用同一个数据库时彼此互不干扰。
- 把数据库对象放在不同的模式下，然后组织成逻辑组，让它们更便于管理。
- 第三方的应用可以放在不同的模式中，这样就不会和其他对象的名字冲突了。
#### 6.3.2 模式的使用
```SQL
create schema schema_name [ authorization user_name ] [ schema_element [ ... ] ]
create schema authorization user_name [ schema_element [ ... ] ]
create schema osdba;
\dn
drop schema osdba;
create schema osdba
  create table t1 (id int, title text)
  create table t2 (id int, content text)
  create view v1 as select a.id, a.title, b.content from t1 a, t2 b where a.id=b.id;
-- 切换schema：
show search_path;
set search_path to osdba,public;
-- 在模式中可以修改名称和属主：
alter schema name rename to new_name;
alter schema name owner to new_owner;
```
#### 6.3.3 公共模式
要创建或者访问模式中的对象：schema_name.table_name
#### 6.3.4 模式的搜索路径 `search_path`
在搜索路径中的第一个模式叫当前模式。它还是在create table没有声明模式名时新建表所属的模式。
#### 6.3.5 模式的权限
用户要访问模式中不属于他们的对象，需要模式的所有者在模式上赋予他们“usage”权限，创建需要“create”权限。默认情况下，每个人在public模式上都有“create”和“usage”权限。可以撤销：
```SQL
revoke create on schema public from public;
```
其中，第一个“public”是模式的名称，第二个“public”的意思是“所有用户”。前一个是标识符，后一个是关键字。
赋予权限：
```SQL
grant all on schema my_schema to my_user;
```
#### 6.3.6 模式的移植性
在SQL标准里，同一个模式里的对象是不能被不同的用户所拥有的。Oracle数据库不允许创建和它们所有者不同名的模式，模式和用户的概念几乎是一样的。如果在PostgreSQL中为每个用户都创建了一个与用户名同名的模式，那么就能与Oracle数据库相兼容了。
SQL标准里也没有public模式的概念。
MySQL通过允许跨数据库访问来提供模式功能。

### 6.4 表
#### 6.4.1 创建表
语法：
```SQL
create table [schema_name.]table_name (
  col01_name data_type,
  col02_name data_type,
  ...
);
```
主键：字段定义后面加上“primary key”。复合主键：
```SQL
constraint constraint_name primary key (col01_name, col02_name, ...)
-- 示例（约束子句是放在列定义后面的）：
create table test (id1 int, id2 int, note varchar(20), constraint pk_test primary key (id1, id2));
```
唯一键：
constraint constraint_name unique (col01_name, col02_name, ...)
check也是一种约束形式，用于定义某些字段的值必须满足某种要求：
constraint constraint_name check(expression)
如，建一张存储孩子信息的表child，其中的年龄字段（age）要求不能大于18岁：
```SQL
create table child (name varchar(20), age int, note text, constraint ck_child_age check(age<18));
```
以其他表为模板：
```SQL
create table baby (like child); -- 不复制源表列上的约束
```
复制源表列上的约束和其他信息，使用“including”：
- including defaults
- including constraints
- including indexes
- including storage
- including comments
- including all
```SQL
create table baby2 (like child including all);
```
也可以用“create table ... as ...”来创建表：
```SQL
create table baby2 as select * from child with no data;
```
#### 6.4.2 表的存储属性
P154~P156
TOAST（The Oversized-Attribute Storage Technique），用于存储一个大字段的值。
TOAST的策略：
plain：避免压缩或线外存储；extended：允许压缩和线外存储；external：允许行外存储，但是不允许压缩；main：允许压缩，但不允许行外存储。
修改：
alter table table_name alter col_name set storage external;
> 只有当数据的长度超过一个BLOCK的四分之一大小时，才会触发TOAST对数据进行压缩。

在PostgreSQL中，更新一条数据时，旧的数据行并不会被覆盖，如果块中有空闲空间，则新行直接插入这个数据块中，这时，Heap-Only Tuple技术，会在旧行与新行之间建一个链表，不需要更新索引。
#### 6.4.3 临时表
PostgreSQL支持会话级的临时表和事务级的临时表，在前者中，数据可以一直保存在整个会话的生命周期中，而后者的数据只存在于这个事务的生命周期中。
> Oracle中，只是临时表中的数据小时，而临时表还存在。

如果在两个不同的session中分别创建一个同名的临时表，实际上创建的是不同的两张表。
```SQL
create temporary table tmp_t1 (id int primary key, note text);
\d
-- 临时表实在schema下所生成的一个特殊的表，这个schema的名称为“pg_temp_xx”，其中的“xx”代表一个数字，如“2”，“3”等，不同的session这个数字是不同的。
-- 另开一个psql
psql postgres
\d pg_temp_xx.tmp_t1
-- 新的session无法访问该表
insert into tmp_t1 values (1,'1111');
insert into tmp_t1 values (2,'2222');
-- 默认情况下，创建的临时表是会话级的，事务级的临时表，要加“on commit delete rows”：
create temporary table tmp_t2(id int primary key, note text) on commit delete rows;
begin;
insert into tmp_t2 values (1,'aaaa');
insert into tmp_t2 values (2,'bbbb');
select * from tmp_t2;
end;
select * from tmp_t2; -- 空表
-- “temporary”可以缩写为“temp”，前面还可以加“global”、“local”关键字，这样可以与其他数据库创建的临时表的语句保持兼容。
```
“on commit”子句有三种形式：
- on commit preserve rows: 不带on commit时的默认情况；
- on commit delete rows: 事务一提交，数据就消失了；
- on commit drop: 事务一提交，临时表就消失了。创建临时表的语句与插入数据的语句需要放到一个事务中。
#### 6.4.4 默认值
建表时，可以为一个字段指定默认值。
```SQL
create table student (no int, name varchar(20), age int default 15);
insert into student values (1, "张三");
insert into student values (2, '李四');
-- 在使用update语句时，也可以使用关键字“default”来代表默认值：
update student set age=16;
update student set age=default where no=2;
-- 没声明默认值，那么默认为null
-- 默认值可以是一个表达式，它会在插入默认值的时候计算（不是在创建表的时候）。
create table blog (id int, title text, create_date timestamp default now());
insert into blog values (1,'PostgreSQL创建临时表');
select * from blog;
```
#### 6.4.5 约束
- 检测约束
- 非空约束
- 唯一约束
- 主键
- 外键
1. 检查约束
```SQL
-- 限制年龄在0~150之间：
create table person (
  name varchar(40),
  age int check (age >= 0 and age <= 150),
  sex boolean
);
-- 给约束加一个名字
create table person (
  name varchar(40),
  age int constraint check_age check (age >= 0 and age <= 150),
  sex boolean
);
-- 约束多个字段
create table books (
  book_no integer,
  name text,
  price numeric check (price > 0),
  discounted_price numeric check (discounted_price > 0),
  check (price > discounted_price)
); -- 前两个约束是“字段约束”，第三个约束是“表约束”。
-- 给表约束赋予名称：
create table books (
  book_no integer,
  name text,
  price numeric,
  discounted_price numeric,
  check (price > 0 and discounted_price > 0)
  constraint valid_discount check (price > discounted_price)
);
-- 当约束表达式的计算结果为null时，检查约束会被认为是满足条件的。
```
2. 非空约束
```SQL
-- 一个非空约束总被写成一个字段约束，功能上等效于创建一个检查约束：
create table books (
  book_no integer not null,
  name text,
  price numeric
);
-- 一个字段可以有多个约束，书写顺序无所谓：
create table books (
  book_no integer not null,
  name text,
  price numeric not null check (price > 0)
);
```
3. 唯一约束
```SQL
-- 唯一约束保证在一个字段或一组字段里的数据相较于表中其他行的数据是唯一的：
create table books (
  book_no integer unique,
  name text,
  price numeric
);
-- 表约束：
create table books (
  book_no integer,
  name text,
  price numeric,
  unique(book_no)
);
```
4. 外键约束
```SQL
-- 外键约束是表间关系的一种约束，用于约束本表中一个字段或多个字段的数值必须出现在另一个表的一个字段或多个字段中。也成为两个相关表之间的参照完整性约束。
create table class (
  class_no int primary key,
  class_name varchar(40)
);
create table student (
  student_no int primary key,
  student_name varchar(40),
  age int,
  class_no int references class(class_no)
);
```
#### 6.4.6 修改表
“alter table”：增加字段；删除字段；增加约束；删除约束；修改默认值；修改字段数据类型；重命名字段；重命名表。
```SQL
-- 增加字段
alter table class add column class_teacher varchar(40); -- 存在的行，默认填充null
alter table class add column class_teacher varchar(40) check (class_teacher <> '');

-- 删除字段
alter table class drop column class_teacher; -- 内容和约束都会被删除，有外键时则报错。
alter table class drop column class_no cascade; -- 会把student表中的外键“student_class_no_fkey”也删除掉。

-- 增加约束
alter table student add check (age < 16); -- 要先确定该表复符合约束条件，否则增加约束会失败
alter table class add constraint unique_class_teacher unique (class_teacher);
-- 非空约束：
alter table class alter column student_name set not null;

-- 删除约束
alter table student drop constraint constraint_name;
\d table_name; -- 查看约束的名称
-- 非空约束没有名称：
alter table student alter column student_no drop not null;

-- 修改默认值，不影响表中现有的任何数据行，只为将来的insert命令改变默认值：
alter table student alter column age set default 15;

-- 删除默认值，相当于设置默认值为null，原先没有定义默认值，也不会报错：
alter table student alter column age drop default;

-- 修改字段数据类型
alter table student alter column student_name type text;
-- 只有在字段里现有的每个项都能隐式地转换成新类型是，才能成功。
-- 改变字段numeric类型的精度时，虽然精度改小可成功，但会导致精度数据丢失
-- 数据类型转换过程中，约束都会相应的转换，可能出错，最好转前删除，转后重加。

-- 重命名字段
alter table books rename column book_no to book_id;

-- 重命名表
alter table class rename to classes;
```

#### 6.4.7 表继承
表继承是PostgreSQL特有的东西：
```SQL
create table persons (
  name text,
  age int,
  sex boolean
);
create table students (
  class_no int
) inherits (persons);
insert into students values ('张三',15,true,1);
insert into students values ('翠莲',14,false,2);
select * from students;
select * from persons;
-- 更改 “students” 表中的数据后，表 “persons” 也发生变化。反之亦然。
update students set age=13 where name='张三'; -- 与下面一句等效：
update persons set age=13 where name='张三';
select * from persons;
insert into persons values('王五',30,true);
select * from persons;
select * from students;
-- 当查询父表时，会把子表的数据也查询出来，反之则不行。只查父表本身：
select * from only persons;
-- 一个子表可以从多个父表继承。所有父表的检查约束和非空约束都会自动被子表继承。不过其他类型的约束（唯一、主键、外键）则不会被继承。
```
#### 6.4.8 分区表
PostgreSQL通过继承来实现分区表。分区表就是把逻辑上的一个大表分割成物理上的几个小块。表的大小超过了数据库服务器的物理内存大小则应该使用。
间分区表的步骤：
1） 创建“父表”，所有的分区都从它继承。
2） 创建几个“子表”，每个都从主表上继承。
3） 给分区表增加约束，定义每个分区表允许的键值。
4） 对于每个分区，在关键字字段上创建一个索引，也可创建其他你想创建的索引。
5） 定义一个规则或者触发器，把对主表的数据插入重定向到合适的分区表。
6） 确保constraint_exclusion里的配置参数postgresql.conf是打开的。打开后，如果查询中where子句的过滤条件与分区的约束条件匹配，那么这个查询会智能地只查询这个分区，而不会查询其他分区。
```SQL
-- 销售明细表（主表）：
create table sales_detail (
  product_id int not null, -- 产品编号
  price numeric(12,2), -- 单价
  amount int not null, -- 数量
  sale_date date not null, -- 销售日期
  buyer varchar(40), -- 买家名称
  buyer_contact text -- 买家的联系方式
);
-- 按销售日期进行分区（子表）：
create table sales_detail_y2014m01 (check (sale_date >= date '2014-01-01' and sale_date < date '2014-02-01')) inherits (sales_detail);
create table sales_detail_y2014m02 (check (sale_date >= date '2014-02-01' and sale_date < date '2014-03-01')) inherits (sales_detail);
create table sales_detail_y2014m03 (check (sale_date >= date '2014-03-01' and sale_date < date '2014-04-01')) inherits (sales_detail);
...
create table sales_detail_y2014m12 (check (sale_date >= date '2014-12-01' and sale_date < date '2015-01-01')) inherits (sales_detail);
-- 在分区键“sale_date”上建索引：
create index sales_detail_y2014m01_sale_date on sales_detail_y2014m01 (sale_date);
create index sales_detail_y2014m02_sale_date on sales_detail_y2014m02 (sale_date);
create index sales_detail_y2014m03_sale_date on sales_detail_y2014m03 (sale_date);
...
create index sales_detail_y2014m12_sale_date on sales_detail_y2014m12 (sale_date);
-- 触发器
create or replace function sales_detail_insert_trigger()
returns trigger as $$
begin
  if ( new.sale_date >= date '2014-01-01' and
      new.sale_date < date '2014-02-01') then
      insert into sales_detail_y2014m01 values (new.*);

  if ( new.sale_date >= date '2014-02-01' and
      new.sale_date < date '2014-03-01') then
      insert into sales_detail_y2014m02 values (new.*);

  ...

  if ( new.sale_date >= date '2014-12-01' and
      new.sale_date < date '2015-01-01') then
      insert into sales_detail_y2014m12 values (new.*);

  else
      raise exception 'Date out of range. Fix the sales_detail_insert_trigger () function!';
end;
$$
language plpgsql;

create trigger insert_sale_detail_trigger
  before insert on sales_detail
  for each row execute procedure sales_detail_insert_trigger ();

-- 删除历史表，不会使触发器失效

-- 使用规则实现分区定位：
create rule sales_detail_insert_y2014m01 as on insert to sales_detail where
    ( sale_date >= date '2014-01-01' and sale_date < date '2014-02-01' )
  do instead
    insert into sales_detail_y2014m01 values (new.*);

create rule sales_detail_insert_y2014m02 as on insert to sales_detail where
    ( sale_date >= date '2014-02-01' and sale_date < date '2014-03-01' )
  do instead
    insert into sales_detail_y2014m02 values (new.*);

...

create rule sales_detail_insert_y2014m12 as on insert to sales_detail where
    ( sale_date >= date '2014-12-01' and sale_date < date '2015-01-01' )
  do instead
    insert into sales_detail_y2014m12 values (new.*);
```
批量插入的情况下，比触发器更有优势。copy不会触发规则。超过规则设置范围之外的，直接插入主表，不报错。
**分区的优化技巧**
打开约束排除（constraint_exclusion）是一种查询优化技巧。参数 “constraint_exclusion” 默认就是 “partition” ，如果使用默认值，在SQL查询中将where语句的过滤条件与表上的check条件进行对比，就可得知不需要扫描的分区，从而跳过相应的分区表，性能也就得到了提高：
```SQL
explain select count(*) from sale_date where sale_date >= date '2014-12-01';
-- 将 “constraint_exclusion” 设置成“off”，则会扫描每张分区子表：
set constraint_exclusion='off';
explain select count(*) from sale_date where sale_date >= date '2014-12-01';
```
### 6.5 触发器
触发器（trigger）是一种由事件自动触发执行的特殊的存储过程，这些事件可以是对一个表进行insert、update、delete等操作。
#### 6.5.1 创建触发器
语法：
```SQL
create [ constraint ] trigger name { before | after | instead of} { event [or ...] }
  on table_name
  [ from referenced_table_name ]
  { not deferrable | [ defferable ] { initially immediate | initially deferred } }
  [ for [ each ] { row | statement } ]
  [ when ( condition ) ]
  execute procedure function_name (arguments)
```
步骤：
先为触发器建一个执行函数，此函数的返回类型为触发器类型，然后即可创建相应的触发器。
示例：
```SQL
create table student (
  student_no int primary key,
  student_name varchar(40),
  age int
);
create table score (
  student_no int,
  chineses_score int,
  math_score int,
  test_date date
);
-- 触发器：删除表student中的一条记录时，把表score中的成绩记录也删除掉。
-- 创建执行函数：
create or replace function student_delete_trigger()
returns trigger as $$
begin
  delete from score where student_no = old.student_no;
  return old;
end;
$$
language plpgsql;
-- 创建触发器：
create trigger delete_student_trigger
  after delete on student
  for each row execute procedure student_delete_trigger ();
-- 测试
insert into student values
  (1, '张三', 14),
  (2, '李四', 13),
  (3, '王二', 15);

insert into score values
  (1, 85, 75, date '2013-05-23'),
  (1, 80, 73, date '2013-09-18'),
  (2, 68, 83, date '2013-05-23'),
  (2, 73, 85, date '2013-09-18'),
  (3, 72, 79, date '2013-05-23'),
  (3, 78, 82, date '2013-09-18');

delete from student where student_no=3;
\d score
```
#### 6.5.1 语句级触发器与行触发器
语句级的触发器是执行每个SQL时，只执行一次，行级触发器则指每行就会执行一次。一个修改零行的操作仍然会导致合适的语句级触发器被执行。
```SQL
-- 对表student的更新情况记录log
create table log_student (
  update_time timestamp,
  db_user varchar(40),
  opr_type varchar(6)
);
-- 触发器执行函数：
create or replace function log_student_trigger ()
returns trigger as
$$
begin
  insert into log_student values (now(), user, tg_op);  -- tg_op代表DML操作类型
  return null;
end;
$$
language plpgsql;
-- 语句级触发器：
create trigger student_log_trigger
  after insert or delete or update on student
  for statement execute procedure log_student_trigger ();
-- 插入、更新操作：
insert into student values (3, '王二', 15), (4, '吴五', 16);
update student set age =15;
update student set age=16 where student_no=5; -- 触发器依然被触发
select * from log_student;

-- 删除语句级触发器，情况表student、表log_student中的记录：
drop trigger student_log_trigger on student;
delete from log_student;
delete from student;
-- 行级触发器：
create trigger student_log_trigger
  after insert or delete or update on student
  for each row execute procedure log_student_trigger ();
insert into student values (1, '张三', 14), (2, '李四', 13);
update student set age=16 where student_no=3; -- 触发器不被触发
```
#### 6.5.3 before触发器与after触发器
> 行级别的“after”触发器会在任何语句级别的“after”触发器被触发之前触发。

“before”触发器可以直接修改“new”值以改变实际更新的值：
```SQL
create function student_use_new_name_trigger ()
returns trigger as '
begin
  new.student_name = new.student_name || new.student_no;
  return new;
end;'
language plpgsql;

create trigger user_new_name_student_trigger
  before insert or update on student
  for each row execute procedure student_use_new_name_trigger ();
```
#### 删除触发器
语法：
drop trigger [ if exists ] name on table [ cascade | restrict ];
> restrict是默认值，如果有任何依赖对象存在，那么拒绝删除。在PostgreSQL中，要在删除触发器的语法中指定“on table”，其他数据库可能不需要。

#### 触发器的行为
触发器执行函数有返回值，语句级触发器应该总是返回null，即必须显式地在函数中写上“return null”，否则触发时报错。对于“before”和“instead of”这类行级别触发器来说，如果返回的是null，则表示忽略当前行的操作。对于“after”行级触发器，返回值会被忽略。
如果同一事件上有多个触发器，则按照触发器名字的顺序来触发。如果是“before”和“instead of”行级触发器，每个触发器返回的行将成为下个触发器的输入。若返回为空，那么该行上的其他行级触发器也不会被触发。
#### 触发器函数中的特殊变量
当把一个PL/pgSQL函数当作触发器函数调用的时候，系统会在顶层的声明段里自动创建几个特殊的变量：
- new：该变量为INSERT/UPDATE操作触发的行级触发器中存在的新的数据行，数据类型是“RECORD”。在语句级别的触发器里此变量没有分配，DELETE操作触发的行级别触发器中此变量也没有分配。
- old：该变量为UPDATE/DELETE操作触发的行级触发器中存储的旧数据行，数据类型是“RECORD”。在语句级别的触发器里此变量没有分配，INSERT操作触发的行级触发器中此变量也没有分配。
- tg_name：数据类型是name，该变量包含实际触发的触发器名。
- tg_when：内容为“BEFORE”或“AFTER”的字符串用于指定是BEFORE触发器还是AFTER触发器。
- tg_level：内容为“ROW”或“STATEMENT”的字符串。
- tg_op：内容为“INSERT”、“UPDATE”、“DELETE”、“TRUNCATE”之一的字符串，用于指定DML语句的类型。
- tg_relid：触发器所在表的OID。
- tg_relname：触发器所在表的名称（废弃）。
- tg_table_name：触发器所在表的名称。
- tg_table_schema：触发器所在表的模式。
- tg_nargs：在create trigger语句里赋予触发器过程的参数个数。
- tg_argv[]：为text类型的一个数组，是create trigger语句里的参数。

### 6.6 事件触发器
弥补PostgreSQL以前版本不支持DDL触发器的中能：
- ddl_command_start：一个DDL开始执行前被触发。
- ddl_command_end：一个DDL执行完成后被触发。
- sql_stop：删除一个数据库对象前被触发。
> 只有超级用户才能创建和修改时间触发器。触发事件见P181~183

#### 6.6.1 创建事件触发器
语法：
```SQL
create event trigger name
  on event
  [ with filter_variable in (filter_value [, ... ]) [ and ... ] ]
  execute procedure function_name ();

-- 触发器执行函数的返回类型为event_trigger
-- 官方手册中一个禁止所有DDL语句的例子：
create or replace function abort_any_command ()
returns event_trigger
language plpgsql
as $$
begin
  raise exception 'command % is disabled', tg_tag;
end;
$$;

create event trigger abort_DDL on ddl_command_start
  execute procedure abort_any_command ();

-- truncate table还是可以执行的。事件触发器本身的操作不会再触发时间触发器。
-- 禁止事件触发器：
alter event trigger abort_DDL disable;
```
对于“aql_drop”事件触发器中的函数，可以调用一个函数pg_event_trigger_dropped_object()获得删除数据库对象的信息，其返回的结果包括：
classid：数据库对象的类型（catalog）的OID
objid：数据库对象的OID
objsubid：数据库对象的子对象（如列）
object_type：数据库对象的类型
schema_name：数据库对象的模式名
object_name：数据库对象的名称
object_identity：数据库对象的标识符
查询系统视图pg_event_trgger可以看到已有的所有事件触发器：
`select * from pg_event_trigger`

```SQL
-- 创建一个事件触发器，用于记录数据库对象删除的审计日志：
create table log_drop_objects (
  op_time timestamp,
  ddl_tag text,
  classid oid,
  objid oid,
  objsubid oid,
  object_type text,
  schema_name text,
  object_name text,
  object_identity text
);

create function event_trigger_log_drop ()
returns event_trigger language plpgsql as $$
declare
  obj record;
begin
  insert into log_drop_objects select now(), tg_tag, classid, objid, objsubid, object_type,
    schema_name, object_name, object_identity from pg_event_trigger_dropped_objects ();
end
$$;

create event trigger drop_log_event_trigger
  on sql_drop
  execute procedure event_trigger_log_drop ();

-- 测试：
create table test01 (id int primary key, note varchar(20));
alter table test01 drop column note;
select * from log_drop_objects;
drop table test01;
select * from log_drop_objects;
```
#### 修改事件触发器
语法：
```SQL
alter event trigger name disable
alter event trigger name enable [ replica | always ]
alter event trigger name OWNER to new_owner
alter event trigger name rename to new_name
```
### 6.7 表空间
#### 6.7.1 表空间的定义
在PostgreSQL中，表空间实际上是为表指定一个存储目录。在创建数据库时可以为数据库指定默认的表空间。
#### 表空间的使用
```SQL
-- 语法
create tablespace tablespace_name [ owner user_name ] location 'directory'
-- 示例：
create tablespace tbs_data location '/data/pgdata';
create database db01 tablespace tbs_data;
alter database db01 set tablespace tbs_data; -- 必须确保没人同时连接到这个数据库上
-- 改变数据库的默认表空间时，数据库中已有表的表空间并不会改变
create table test01 (id int, note text) tablespace tbs_data;
create index idx_test01_id on testdb01 (id) tablespace tbs_data;
alter table test01 add constraint unique_test01_id unique (id) using index tablespace tbs_data;
alter table test01 add constraint pk_test01_id primary key (id) using index tablespace tbs_data;
-- 把一个表从一表空间移动到另一表空间：
alter table test01 set tablespace pg_default; -- 在移动表的时候会锁表，此时对该表的所有操作都将要被阻塞。
```
### 6.8 视图
#### 6.8.1 视图的定义
通俗的说，视图就是由查询语句定义的虚拟表。PostgreSQL中提供的视图默认是只读的，但可以使用规则系统做出一张可以更新的视图。
#### 6.8.2 创建视图
```SQL
-- 语法：
create [ or replace ] [ temp | temporary ] view name [ ( column_name [, ... ] ) ] as query
-- 示例：
-- 建视图把原表敏感字段“password”排除掉：
create table users (
  id int,
  user_name varchar(40),
  password varchar(256),
  user_email text,
  user_mark text
);
create view vw_users as select id, user_name, user_email, user_mark from users;
create view vw_users (no, name, email, mark) as select id, user_name, user_email, user_mark from users;
```
#### 6.8.3 可更新视图
```SQL
insert into users values (1, '张三', '123456', 'zhangsan@yahoo.com.cn', 'hello');
-- 定义规则(<9.3)：
create rule vw_users_upd as
  on update to vw_users do instead update users set user_email=new.user_email;
update vw_users set user_email='zhangsan@163.com' where id=1;
-- 9.5.5 可以直接操作视图
-- 也可以使用“INSTEAD OF”的触发器更新视图
```
### 6.9 索引
#### 6.9.2 索引的分类
PostgreSQL中，支持以下几类索引：
- B-tree：适合处理等值查询和范围查询。
- Hash：只能处理简单的等值查询。
- GiST：是一种架构，可以在这种架构上实现很多不同的索引策略。GiST索引定义的特定操作符可以用于特定索引策略。
- SP-GiST：“apace-partitioned GiST”的缩写，即空间区分索引。
- GIN：反转索引，可以处理包含多个建的值，如数组等。
#### 6.9.3 创建索引
```SQL
-- 语法
create [ unique ] index [ concurrently ] [ name ] on table_name [ using method ]
  ( { column_name | ( expression ) } [ collate collation ] [ opclass ] [ asc | sesc ] [ nulls { first | last } ] [, ... ] )
  [ with ( storage_parameter = value [, ... ] ) ]
  [ where predicate ]
-- 示例：
create table contacts (
  id int primary key,
  name varchar(40),
  phone varchar(32)[],
  address text
);
create index idx_contacts_name on contacts (name); -- B-tree
create index idx_contacts_phine on contacts using gin (phone);
-- Hash索引的更新不会记录到WAL日志中。
-- 指定存储参数“with （storage_paramet=value）”，常使用的存储参数为fillfactor：
create index idx_contacts_name on contacts (name) with (fillfactor=50);
-- 按降序建索引：
create index idx_contacts_name on contacts (name desc);
-- 如果是字段“name”中有空值，则可以在建索引时，指定空值排在非空值前面或后面：
create index idx_contacts_name on contacts (name desc nulls first);
create index idx_contacts_name on contacts (name desc nulls last);
```
#### 6.9.4 并发创建索引
通常，在创建索引的时候PostgreSQL会锁定表以防写入，然后对表做全表扫描，从而完成创建索引操作。PostgreSQL支持不长时间阻塞更新的情况下创建索引，通过在create index中加concurrently（并发创建索引）选项来实现。
```SQL
create table testtab01 (id int primary key, note text);
insert into testtab01 select generate_series(1,5000000), generate_series(1,5000000);
create index concurrently idx_testtab01_note on testtab01 (note);
-- 重建索引：使用“concurrently”选项建一个新的索引，然后把旧的索引删除。
```
并发创建索引的时候要注意，如果索引在创建过程中被强行取消，可能会留下一个无效的索引，这个索引会导致更新变慢，如果所创建的是一个唯一的索引，这个无效索引还会导致插入重复值失败。需要手动删除该索引。
#### 6.9.5 修改索引
```SQL
alter index name rename to new_name
alter index name set tablespace tablespace_name
alter index name set ( storage_parameter = value [, ... ] )
alter index name reset ( storage_parameter [, ... ] )
-- 查看索引的信息：
\d+ index_name
```
#### 6.9.6 删除索引
```SQL
drop index [ if exists ] name [, ...] [ cascade | restrict ]
```
### 6.10 用户及权限管理
#### 6.10.1 用户和角色
PostgreSQL使用角色的概念管理数据库访问权限。角色是一系列相关权限的集合。为了管理方便，通常会把一些列相关的数据库权限赋给一个角色，如果哪个用户需要这些权限，就把角色赋给相应的用户。在PostgreSQL中，不区分角色与用户。
用户和角色在整个数据库实例中都是全局的，且在同一个实例中的不同数据库中，看到的用户也都是相同的。在初始化数据库系统时，预定义的超级用户的名称与初始化该数据库的操作系统用户相同。
#### 6.10.2 创建用户和角色
```SQL
-- 语法：
create role name [ [ with ] option [ ... ] ]
create user name [ [ with ] option [ ... ] ]
-- "create user" 默认创建出来的用户有 "login" 权限，而 "create role" 没有。
```
option可以是:
- superuser | nosuperuser：表示创建出来的用户是否为超级用户。只有超级用户才能创建超级用户。
- createdb | nocreatedb
- createrole | nocreaterole
- createuser | nocreateuser
- inherit | noinherit：如果创建的一个用户拥有某一个或某几个角色，这时若指定inherit，则表示用户自动拥有相应角色的权限，否则这个用户没有改角色的权限。
- login | nologin
- connection limit connlimit：指定该用户可以使用的并发连接数量，默认是-1，表示没有限制。
- [ encrypted | unencrypted ] password 'password'：用于控制存储在系统表里面的口令是否加密。
- valid until 'timestamp'：密码失效时间，如果不指定这个子句，那么口令将永远有效。
- in role role_name [, ...]：指定用户成为哪些角色的成员，请注意没有任何选项可以把新角色添加为管理员，必须使用独立的grant命令来做这件事情。
- in group role_name [, ...]：与in role相同，是已过时的语法。
- admin role_name [, ...]：role_name将有这个新建角色的with admin option权限。
- user role_name [, ...]：与ROLE子句相同，但已过时。
- sysid uid：此子句主要是为了SQL向下兼容，实际没什么用处。

#### 6.10.3 权限的管理
在PostgreSQL中，删除一个对象及任意修改它的权力都不能赋予别人，它是所有者固有的，不能被赋予或撤销。所有者也隐含地拥有把操作该对象的权限赋给别人的权限。
一个用户的权限分为两类，一类是在创建用户时就指定的权限，后面可以使用alter role命令来修改：
- 超级用户的权限
- 创建数据库的权限
- 是否允许LOGIN的权限
另一类权限是由命令grant和revoke来管理的：
- 在数据库中创建模式（schema）
- 允许在指定数据库中创建临时表
- 连接某个数据库
- 在模式中创建数据库对象，如创建表、视图、函数等
- 在一些表中做select、update、insert、delete操作
- 对序列进行查询（执行序列的currval函数）、使用（执行序列的currval函数和nextval函数）、更新等操作
- 在声明表上创建触发器
- 可以把表、索引等建到指定的表空间
如果要给用户赋予创建数据库的权限，则需要使用“alter role”命令，而要给用户赋予创建模式的权限时，需要使用“grant”命令。
```SQL
alter role name [ [ with ] option [ ... ] ]
grant role_name [, ...] to role_name [, ...] [ with admin option ] -- 命令格式见P200
grant some_privileges on database_object_type object_name to role_name;
```
some_privileges: select; insert; update; delete; truncate; references; trigger; create; connect; temporary或temp; execute; usage; all previleges
#### 6.10.4 函数和触发器的权限
PostgreSQL只允许超级用户使用PL语言写函数。
#### 6.10.5 权限的总结
管理的层次：
- 首先管理赋在用户特殊属性上的权限
- 在数据库中创建模式的权限
- 模式中创建数据库对象的权限
- 表查询、插入、更新、删除表内数据的权限
- 操作表中某些字段的权限
#### 6.10.6 权限的示例
```SQL
-- 创建一个只读用户
revoke create on schema public from public; -- PostgreSQL中默认任何用户都可以在名称为public的shema中创建列表，此处收回这个权限
create user readonly with password 'query';
grant select on all tables in schema public to readonly; -- select现有表的权限
alter default previleges in schema public grant select on tables to readonly; -- select新建表的权限
```

### 6.11 事务、并发、锁
#### 6.11.1 ACID
在一个事务中，多个插入、修改、删除操作要么全部成功，要么全部失败，这称为“原子性”。一个事务还需要其他三个特性：一致性、隔离性、持久性。这四者统称ACID（atomicity、consistency、isolation、duration）
在PostgreSQL中，可使用多版本并发控制（MVCC）来维护数据的一致性，优点在于MVCC里对检索（读）数据的锁请求与写数据的锁请求不冲突。
#### 6.11.2 DDL事务
相比其他数据库，PostgreSQL中大多数DDL可以包含在一个事务中，而且可以回滚，适合把PostgreSQL作为Sharding的分布式数据系统的底层数据库。比如，在Sharding中，常常需要在多个节点中建相同的表，这时可以考虑把建表语句放在同一个事务中，这样就可以在各个节点上先启动一个事务，然后执行建表语句了。如果中建某个节点失败，可以回滚前面的建表操作。
#### 6.11.3 事务的使用
```SQL
create table testtab01 (id int);
-- 关闭自动提交：
\set AUTOCOMMIT off
\echo :AUTOCOMMIT
insert into testtab01 values (1);
insert into testtab01 values (2);
select * from testtab01;
rollback;
select * from testtab01;

-- 使用begin来启动一个事务：
begin;
insert into testtab01 values (1);
insert into testtab01 values (2);
select * from testtab01;
rollback;
select * from testtab01;
```
#### 6.11.4 SAVEPOINT
PostgreSQL支持保存点（savepoint）的功能，在一个大的事务中，可以把操作过程分成几个部分，第一部分成功后，可以建一个保存点，若后面的执行失败，则回滚到这个保存点。
```SQL
begin;
insert into testtab01 values(1);
insert into testtab01 values(2);
savepoint my_savepoint01;
insert into testtab02 values(1); -- error
rollback to  savepoint my_savepoint01;
select * from testtab01;
create table testtab02 (id int);
insert into testtab02 values(1);
commit;
```
#### 6.11.5 事务隔离级别
- READ UNCOMMITED：读未提交
- READ COMMITED：已读提交
- REPEATABLE READ：重复读
- SERIALIZABLE：串行化
并发事务的级别：脏读、不可重复读、幻读。
读已提交的隔离级别中，select查询看到的是在查询开始运行瞬间的一个快照。
#### 6.11.6 两阶段提交
两阶段提交用以保证分布式事务中多台数据库之间的原子性。
两阶段提交的步骤：
1） 应用程序先调用各台数据库做一些操作，但不提交事务；然后调用事务协调器中的提交方法。
2） 事务协调器将联络事务中涉及的每台数据库，并通知它们准备提交事务，这是第一阶段的开始。在PostgreSQL一般是调用“prepare transaction”命令。
3） 每台数据库接收到“prepare transaction”命令后，如果返回成功，则数据库必须将自己置于以下状态：确保后续能在被要求提交事务时提交任务，或者在被要求回滚时能回滚任务。所以PostgreSQL会将已准备好提交的信息写入持久存储区中，如果数据库无法完成此事务，它会直接返回失败给事务协调器。
4） 事务协调器接收到所有数据库的响应。
5） 在第二阶段，如果任一数据库在第一阶段返回失败，则事务协调器将会发一个回滚命令（rollback prepared）给各台数据库。如果所有数据库的响应都是成功的，则向各台数据库发送“commit prepared”命令，通知各台数据库事务成功。
示例：
```SQL
-- 先将参数 max_prepared_transactions 设置成一个大于0的数字，否则有错误
set max_prepared_transactions=10; -- 无法成功
-- 直接修改 postgresql.conf 文件，重启数据库
-- 建一张测试表
create table testtab01 (id int primary key);
begin;
insert into testtab01 values (1);
prepare transaction 'osdba_global_trans_0001'; -- 该事务会被持久化，即使重启，也不会被回滚或丢失
-- 至此，停止数据库，然后重启：
pg_ctl stop -D $PGDATA
pg_ctl start -D $PGDATA
psql postgres
commit prepared 'osdba_global_trans_0001';
```
#### 6.11.7 锁机制
两类：表级锁和行级锁。
1. 表级锁模式
share, exclusive, access share, acess exclusive, row share, row exclusive, share update exclusive, share row exclusive
2. 行级锁模式
共享锁和排它锁。
#### 6.11.8 死锁防范
死锁是指两个或两个以上的事务在执行过程中互相持有对方期待的锁。PostgreSQL能够自动侦测到死锁，然后退出其中一个事务。
死锁发生的四个必要条件：互斥条件、请求和保持条件、不剥夺条件、环路等待条件。
#### 6.11.9 表级锁命令lock table
语法：
```SQL
lock [ table ] [ only ] name [, ...] [ in lockmode mode ] [ nowait ] -- lockmode 就是前边的表级锁模式
```
#### 6.11.10 行级锁命令
语法
```SQL
-- 显式的行级锁命令是由select命令后面加子句来完成的：
select ... for { update | share } [ of table_name [, ...] [ nowait ] [...] ]
```
#### 6.11.11 锁的查看
pg_locks 视图：
locktype, database, relation, page, tuple, virtualxid, transactionid, classid, objid, objsubid, virtualtransaction, pid, mode, granted
描述事务ID的字段：virtualxid, transactionid, virtualtransaction
首先“transactionid”代表事务ID，简写为“xid”；“virtualxid”代表虚拟事务ID，简写为“vxid”，比如只读事务或空事务等。
“virtualtransaction”之前的字段（不包括其本身），称为第一部分；之后（包括其本身）的字段称为第二部分。第一部分用于描述锁定对象的信息，第二部分字段用于描述的是持有锁或等待锁session的信息。
```SQL
-- 第1个psql窗口：
select pg_backedn_pid();
begin;
lock table testtab01;
-- 第2个psql窗口：
begin;
lock table testtab01;
-- 第2个psql窗口：
select locktype, relation::regclass as rel, virtualxid as vxid, transactionid as xid, virtualtransaction as vxid2, pid, mode, granted from pg_locks;
```
行级锁不仅会在表上加意向锁，也会在相应的主键上加意向锁。`pg_locks`并不能显示每个行级锁的信息，因为PostgreSQL在内存中并不记录行级锁的信息。想看哪个进程被阻塞了，只需要看“granted”字段为false的pid即可。查询因行级锁被阻塞的进程信息，只需查询视图`pg_locks`中类型为transactionid的锁信息就可以了。哪一行被阻塞可以通过查看pg_locks的“page”和“tuple”字段来了解。
```SQL
-- 窗口1：
update testtab01 set note='aaaa' while id=1;
-- 窗口2：
begin;
update testtab01 set note='aaaa' while id=1;
-- 窗口3：
select locktype, relation::regclass as rel, page || ',' || tuple as ctid, virtualxid as vxid, transactionid as vxid, virtualtransaction as vxid2, pid, mode, granted from pg_locks;
select * from testdb01 where ctid='(0,1)';
```

## 第7章 PostgreSQL的核心架构
### 7.1 应用程序的访问接口
如果应用程序与数据库在一台机器上，也可以使用UNIX domain sockets连接到PostgreSQL数据库。
Python、Perl等使用C语言接口库libpq驱动连接到PostgreSQL数据库上。
### 7.2 进程及内存结构 
PostgreSQL数据库启动时，会先启动一个叫Postmaster的主进程，还会fork出一些辅助子进程：
- SysLogger（系统日志）进程
> `postgresql.conf`中参数`logging_collect`设置为`on`时，主进程才会启动SysLogger辅助进程。
- BgWriter（后台写）进程
> 把共享内存中的脏页写到磁盘上的进程。由“bgwriter_”开头的配置参数来控制。
- WalWriter（预写式日志）进程
> WAL是Write Ahead Log的缩写，即预写式日志。预写式日志就是在修改之前，必须要把这些修改的操作记录到磁盘中。WAL日志保存在`pg_xlog`下。未持久化的数据都可以通过WAL日志来恢复。
- PgArch（归档）进程
> PgArch归档进程会在覆盖前把WAL日志备份出来。
- AutoVAcuum（系统自动清理）进程
> 只有在没有并发的其他事务读到旧数据时，它们才会被清除掉，由AutoVAcuum完成。
- PgStat（统计收集）进程
> 收集的数据统计信息主要用于查询优化时的代价估算。系统表`pg_statistic`中存储了PgStat收集的各类统计信息。
#### 7.2.2 主进程Postmaster
postmaster命令是一个指向postgres的链接。查询`pg_stat_activity`时看到的pid，就是服务进程的pid。PostgreSQL数据库是进程架构模型，MySQL数据库是线程架构模型。
#### 共享内存与本地内存
## 7.3 目录结构
#### 安装目录结构
bin、include、lib、share（文档和配置模板文件，一些扩展包的sql文件在此目录的子目录extension下）
#### 数据目录的结构
$PGDATA：
- 配置文件：`postgresql.conf`、`pg_hha.conf`（认证配置文件，配置了允许哪些IP的主机访问数据库，认证的方法是什么等信息）、`pg_ident.conf`（“ident”认证方式的用户映射文件）
- 子目录：`base`（默认表空间的目录）、`global`（一些共享系统表的目录）、`pg_clog`（commit log的目录）、`pg_log`（系统日志目录）、`pg_stat_tmp`（统计信息的存储目录）、`pg_tblsp`（存储了指向各个用户自建表空间实际目录的链接文件）、`pg_twophase`（使用两阶段提交功能时分布式事务的存储目录）、`pg_xlog`（WAL日志的目录）

## 第8章 服务管理
### 8.1 服务的启停与创建
#### 8.1.1 启停方法
```shell
## 启动：
postgres -D $PGDATA [&]
pg_ctl -D $PGDATA start
## 停止：
pg_ctl stop -D $PGDATA -m smart|fast|immediate
```
#### 8.1.2 pg_ctl
帮助文档：`man pg_ctl`
初始化PostgreSQL数据库实例的命令：
`pg_ctl init [db] [-s] [-D datadir] [-o options]`
- -s：只打印错误和警告信息，不打印提示性信息。

启动PostgreSQL数据库的命令：
`pg_ctl start [-w] [-t seconds] [-s] [-D datadir] [-l filename] [-o options] [-p path] [-c]`
- -w：等待启动完成
- -t：等待启动完成的秒数，默认为60s
- -s：只打印错误和警告信息，不打印提示性信息。
- -l：把服务器日志输出附加在“filename”文件上，不存在时创建。
- -o options：声明要直接传递给postgres的选项，具体见命令帮助。
- -p：指定postgres可执行文件的位置。默认情况下postgres可执行文件来自和pg_ctl相同的目录。
- -c：提高服务器的软限制（ulimit -c），尝试允许数据库实例在右异常时产生一个coredump文件，以便于问题定位和故障分析。

停止PostgreSQL数据库的命令：
`pg_ctl stop [-W] [-t seconds] [-s] [-D datadir] [-m s[mart] | f[ast] | i[mmediate] ]`
- -W：不等待数据库停下来，命令就返回。
- -m：停止模式。

重启PostgreSQL数据库的命令：
`pg_ctl restart [-w] [-t seconds] [-s] [-D datadir] [-c] [-m s[mart] | f[ast] | i[mmediate] ] -o options`
重新读取配置文件的命令：
`pg_ctl reload [-s] [-D datadir]`
查询数据库实例的命令：
`pg_ctl status [-D datadir]`
Windows平台下：
`pg_ctl kill [signal_name] [process_id]`
`select pg_backend_pid(); select pg_sleep(600); pg_ctl kill int pid`
`pg_cancle_backend(pid int)`
Windows平台下注册服务和取消服务的命令：
`pg_ctl register [-N servicename] [-U username] [-P password] [-D datadir] [-w] [-t seconds] [-o options]`
`pg_ctl unregister [-N servicename]`
- -U：用于启动服务的用户名。如果是域用户，需要使用“DOMAIN\username”的格式。
- -N：要注册的系统服务的名字。这个名字将用于服务名和显示名。
#### 8.1.3 信号
- sigterm：smart shutdown关机模式
- sigint：fast shutdown关机模式
- sigquit：immediate shutdown关机模式
#### 8.1.4 Postgres及单用户模式
postgres单用户模式就是在启动postgres程序时加上“--single”参数，这时postgres不会进入到后台服务模式，而是进入到一个交互式的命令行模式下
```shell
postgres --single -D $PGDATA
```
在此交互式模式下，可以执行一些命令，如一些SQL语句等。

### 8.2 服务器配置
#### 8.2.1 配置参数
`postgresql.conf`文件：
配置项的参数名都是大小写不敏感的，参数值有下面的类型：
- 布尔：大小写无关，可以是on、off、true、false、yes、no、1、0。
- 整数：数值可以指定单位（KB、MB、GB等）。
- 浮点数
- 字符串
- 枚举

使用`include 'filename'`包含其他文件中的配置内容。
所有的配置参数都在系统视图`pg_settings`中。
```SQL
-- 当不知道枚举类型的配置参数`client_min_message`可以取哪些值时，可以用下面的语句查询：
select enumvals from pg_settings where name='client_min_message';
-- 查询`autoacuum_vacuum_cost_delay`的单位：
select unit from pg_settings where name='autoacuum_vacuum_cost_delay';
-- 查询`autoacuum_vacuum_cost_delay`的描述：
select short_desc, extra_desc, from pg_settings where name='autoacuum_vacuum_cost_delay';
```
参数的分类：internal（只读参数，由postgres程序在初始化实例时写死的）、postmaster（改变这些参数的值要重启PostgreSQL实例）、sighup、backend、superuser（由超级用户用set来改变）、user（普通用户用set改变）
可以通过查询`pg_settings`表中的context字段值知道改变参数在`postgresql.conf`中的配置时，是否需要重启数据库：
```SQL
select name,context from pg_settings where name like 'wal_buffers'; -- postmaster，需要重启
select name,context from pg_settings where name like 'local_preload_libraries'; -- backed，不需要重启，使用pg_ctl reload
```
#### 8.2.2 连接配置项
- listen_addresses
- port
- max_connections
- superuser_reserved_connections
- unix_socket_group
- unix_socket_permissions
- bonjour
- bonjour_name
- tcp_keepalives_interval
- tcp_keepalives_count
#### 8.2.3 内存配置项
- shared_buffers（物理内存的25%）
- temp_buffers
- work_mem
- maintenance_work_mem
- max_stack_deep
#### 8.2.4 预写式日志的配置项
- wal_level
- fsync
- synchronous_commit
- wal_sync_method
- full_page_writes
- wal_buffers
- wal_writer_delay
- commit_delay
- commit_siblings
#### 错误报告和日志项
`logging_collector = on`
PostgreSQL也可以把日志发送到操作系统的syslog中，或者多生成一个csv格式的日志：
`log_destination = 'syslog'`
`log_destination = 'evwntlog'` Windows平台下。
生成csv格式日志：
`log_destination = 'csvlog'`
其他参数：
- log_directory
- log_filename
- log_rotation_age
- log_rotation_size
- log_truncate_on_rotation
- syslog_facility
- syslog_ident
- debug_print_parse
- debug_print_rewritten
- debug_print_plan
- debug_pretty_print
- log_checkpoints
- log_connections
- log_disconnections
- log_duration
- log_hostname
- log_lock_waits
### 8.3 访问控制配置文件
`pg_hba.conf`文件：
由多条记录组成，每条记录占一行。每条记录声明一种连接类型、一个客户端IP地址范围、一个数据库名、一个用户名，以及匹配这些参数的连接所使用的认证方式。
#### 8.3.1 `pg_hba.conf`文件
格式：
local dbname user auth-method [auth-options]
host dbname user ip/masklen auth-method [auth-options]
hostssl dbname user ip/masklen auth-method [auth-options]
hostnossl dbname user ip/masklen auth-method [auth-options]
hostssl dbname user ip mask auth-method [auth-options]
hostnossl dbname user ip mask auth-method [auth-options]
- dbname：用于设置一个数据库名称，如果设置为all，表示可以匹配任何数据库；如果设置为replication，表示允许流复制连接
- ip/masklen：掩码为32，完全匹配这个IP；192.168.1.0/24表示IP地址前缀为192.168.1.X的主机都允许访问数据库服务器
- auth-method：表示验证方法（trust、reject、md5、ident等）
#### 8.3.2 认证方法介绍
- trust
- reject
- md5
- gss
- sspi
- password
- krb5
- ident：允许客户端上的特定用户连接到数据库。
- ldap
- radius
- cert
- pam
### 8.4 备份和还原
#### 8.4.1 逻辑备份
PostgreSQL提供了`pg_dump`、`pg_dunmpall`命令进行数据库的逻辑备份。
`pg_dump`并不阻塞其他用户对数据库的访问（读或写）。
`pg_dump`生成的备份文件可以是一个SQL脚本文件或归档文件。
#### 8.4.2 `pg_dump`命令
格式：
`pg_dump [connection-option...] [option...] [dbname]`
连接选项：
- -h host 或 --host=host：默认为$PGGHOST环境变量
- -p port 或 --port=port：$PGPORT
- -U username 或 --username=username
- -w 或 --no-password：从不提示密码。
- -W 或 --password：从不提示密码。
- --role=rolename：该选项导致pg_dump在连接到数据库时发布一个set role rolename命令。这相当于切换到另一个角色。
- dbname
专有参数，用来控制备份哪些表的数据及输出数据的格式：
- -a 或 --data-only：只输出数据，不输出数据定义的SQL语句。
- -b 或 --blobs：在转储中是否包含大对象。除非指定了选择性转储的选项--schema、--table、--schema-only开关，否则默认会转储大对象。
- -c 或 --clean：指定输出的脚本中是否含有生成清理该数据库对象语句（如drop table命令）。
- -C 或 --create：指定脚本中是否输出一条create database语句和连接到该数据看的语句。
- -E encoding 或 --encoding=encoding：以指定的字符集编码创建转储。（$PGCLINTENCODING）
- -f file 或 --file=file：输出到指定的文件中，否则输出到标准输出中。
- -F format 或 --format=format：可以是p（plain，纯文本SQL脚本文件格式）、c（custom，以一个适合`pg_restore`使用的自定义格式输出并归档，默认是压缩的）、t（以一个适合输入`pg_restore`的tar格式输出并归档，不支持压缩，独立表的大小限制为8GB）
- -n schema 或 --schema=schema：可以使用多个-n指定多个模式。
- -N schema 或 --exclude-schema=schema
- -o 或 --oids
- -O 或 --no-owner
- -s 或 --schema-only：只输出对象定义（模式），不输出数据。
- -S username 或 --superuser=username
- -t table 或 --table=table：使用-t之后，-n和-N选项就失效了。并不会转储表所依赖的其他数据库对象。
- -T table 或 --exclude-table=table
- -v 或 --verbose：输出到标准错误上
- -V 或 --version
- -x 或 -- no-privileges 或 --no-acl：禁止转储访问权限（grant/revoke命令）
- -Z 0...9 或 --compress=0...9
- --binary-upgrade
- --insert：将为每一行生成一个单独的insert命令
- --column-inserts 或 --attribute-inserts：显式列名的INSERT命令
- --disable-dollor-quoting：用于关闭使用美元符界定函数体。