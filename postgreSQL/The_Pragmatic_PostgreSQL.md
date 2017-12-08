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
