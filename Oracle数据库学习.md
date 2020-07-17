# Oracle数据库学习

### 关于表操作
#### 创建表

```` 创建表空间
create tablespace StudentDemo
datafile 'D:\StudentDemo.dbf'--指定位置
size 100m--设置大小
autoextend on --自动增加大小
next 10m;--每次增加大小
````

#### 删除表

````删除表空间
drop tablespace StudentDemo;
````

#### 创建用户
````创建用户
create user StudentDemo
identified by 123456
default tablespace StudentDemo;
````

#### 给表空间一个超级管理员

```` 给表空间一个超级管理员
grant dba to StudentDemo;
````

#### 创建表
````创建表
create table student(
       sid number(20),
       sname varchar(10)
);
````

#### 修改表机构
````修改表机构
--添加一列
alter table student add (gender number(1));
--修改列类型
alter table student modify gender char(1);
--修改列名称
alter table student rename column gender to sex;
--删除一列
alter table student drop column sex;
````

***

### 增删改查

#### 添加一条记录
````添加
insert into student values
(1,'小白')
commit;
````

#### 修改
````修改
update student set sname = '小黑' where sid = 1
commit;
````

#### 删除
````删除
delete from student;--删除表中全部记录
drop table stduent;--删除表
truncate table student;--先删除表,在创建表(数据量多用)
````

#### 查询
````查询
select * from student
````

***

### 序列

​	序列不属于任何一张表,但可以做逻辑的绑定

​	默认从1开始,依次递增,主要用来给主键赋值使用

​	dual:虚表,只是用来补全语法,没有任何意义

#### 创建一个序列

````创建
create sequence s_student;
````

#### 下一个

````下一个
select s_student.nextval from dual;
````

#### 当前

````当前
select s_student.currval from dual;
````

#### 有了它添加写法

```` 添加
insert into student values
(s_student.nextval,'小白')
commit;
````

***

### 用户

scott用户,密码tiger

可以模拟各种复杂的查询

解锁

```` 解锁
alter user scott account unlock;
````

解锁密码(也可用来重置密码)

```` 解锁密码
alter user scott identified by tiger;
````

***

### 复杂查询

#### 函数

##### 单行函数

作用于一行,返回一个值

###### 字符函数
```` 大小写
select upper('yes') from dual;--小写
select lower('yes') from dual;--大写
````

###### 数值函数
````数值函数
select round(11.16,1) from dual;--四舍五入,后面表示保留的位数
select trunc(11.16,1) from dual;--直接截取
select mod(11.16,1) from dual;--求余数
````

###### 日期函数
````日期函数
--查询入职几天
select sysdate-e.hiredate from emp e;
--算出明天此刻
select sysdate+1 from dual;
--查询入职到现在几月
select months_between(sysdate,e.hiredate) from emp e;
--查询入职到现在几年
select months_between(sysdate,e.hiredate)/12 from emp e;
--查询入职到现在周
select (sysdate-e.hiredate)/7 from emp e;
````

###### 转换函数
````转换函数
--日期转字符串
select to_char(sysdate,'fm yyyy-mm-dd hh:mi:ss') from dual;
--字符串转日期
select to_date('2020-5-14 11:52:51','fm yyyy-mm-dd hh:mi:ss') from dual;
````

###### 通用函数

````通用函数
--算奖金,没有奖金则为0,不显示
select e.sal*12+nvl(e.comm,0) from emp e;
````



##### 多行函数[聚合函数]

作用于多行,返回一个值

````sql
select count(1) from emp;--总数量
select sum(sal) from emp;--总合
select max(sal) from emp;--最大
select min(sal) from emp;--最小
select avg(sal) from emp;--平均
````



#### 表达式

##### 例如

###### 显示收入(起别名)[通用]
````sql
select e.sal,case
 when e.sal>3000 then '高收入'
 when e.sal>1500 then '中收入'
else '低收入'
end
from emp e
````

###### oracle专用写法(起别名)[不常用]

```` sql
select e.sal,
decode(e.sal,
       5000,'高收入',
       3000,'中收入',
            '低收入')中文收入
from emp e

````



#### 分组查询

where在查询前,having在查询后

##### 例如

​	查询每个部门平均工资高于800的,然后在查询出高于2000的
​	想查询多的数据,加上聚合函数

```` sql
select e.deptno,avg(e.sal)
from emp e
where e.sal>800
group by e.deptno
having avg(e.sal)>2000;
````



#### 分页查询

##### rownum行号

当我们在做select操作,每查询一条记录,就会加一个行号,从1开始,依次递增,不能跳着走

排序操作会影响rownum顺序

##### 例如
````sql
select rownum,t.* from
(select rownum,e.* from emp e order by e.sal desc) t;
````

emp倒序,每行五列,查第二列

##### 例如

````sql
select * from
(select rownum r,e.* from
(select * from emp order by sal desc) 
e where rownum<11)
t where r>5;
````

***

### 视图

​	视图就是提供一个查询窗口,所有数据来源于原表

##### 创建只读视图
```sql
create table emp as select * from scott.emp;
create view v_emp as select ename,job from scott.emp with read only;
select * from v_emp;
```

##### 视图的作用

1. 可以屏蔽掉一些敏感字段
2. 保证总部和分部数据统一

***

### 索引

索引就是在表的列汉构建一个二叉树

大幅度提高查询效率,但是会影响增删改的效率

##### 创建单列索引

````sql
create index idx_ename on emp(ename);
````

##### 创建复合索引
```sql
create index idx_enamejob on emp(ename,job);
```

***

### pl/sql编程语言
​	pl/sql编程语言是对sql语言的扩展，是的sql语言具有过程化编程的特性
​	pl/sql编程语言比一般的过程化编程语言，更加灵活高效
​	pl/sql编程语言主要用来编写存储过程和存储函数等。

​	声明方法,定义变量
​	赋值操作可以用 := 也可以使用 into 查询语句赋值

```sql
declare
    i number(2):=10;        --数值型变量
    s varchar2(10):='小明'; --字符型变量
    ena emp.ename%type;   --引用型变量，直接取出emp表中ename的类型给ena
    emprow emp%rowtype;   --记录型变量，可以理解为可以存一行记录
begin
    dbms_output.put_line(i); --输出语句
    dbms_output.put_line(s);
    select ename into ena from emp where empno=7788;
    dbms_output.put_line(ena);
    select * into emprow from emp where empno=7788;
    dbms_output.put_line(emprow.ename||'的工作为：'||emprow.job);
end;
```

#### pl/sql中的loop循环

​	用三种方式输出1到10十个数字 --while循环 

```sql
declare
   i number(2):=1;
begin
  while i<11 loop
    dbms_output.put_line(i);
    i:=i+1;
  end loop;
end;
```

####  exit循环 

````sql
declare
  i number(2):=1;
begin
  loop
    exit when i>10;
    dbms_output.put_line(i);
    i:=i+1;
  end loop;
end;
````

####  for循环 

````sql
declare

begin
  for i in 1..10 loop
     dbms_output.put_line(i);   
  end loop;
end;
````

***

#### 游标

游标：可以存放多个对象，多行记录

输出emp表中的所有员工的姓名 

````sql
declare
  cursor c1 is select * from emp;
  emprow emp%rowtype;
begin
  open c1;
       loop
         fetch c1 into emprow;
         exit when c1%notfound;
         dbms_output.put_line(emprow.ename);
       end loop;
  close c1;
end;
````

给指定部门员工涨工资,用到带参数的游标 

````sql
declare
  cursor c2(eno emp.deptno%type)
  is select empno from emp where deptno=eno;
  en emp.empno%type;
begin
  open c2(10);
       loop
         fetch c2 into en;
         exit when c2%notfound;
         update emp set sal=sal+100 where empno=en;
         commit;
       end loop;  
  close c2;  
end;
````

***

#### 存储过程

存储过程：存储过程就是提前编译好的一段pl/sql语言，放置在数据库端 ---可以直接被调用。这一段pl/sql一般都是固定步骤的业务。 

给指定员工涨100块钱 

````sql
create or replace procedure p1(eno emp.empno%type)
is

begin
  update emp set sal=sal+100 where empno=eno;
  commit;
end;
````

***

####  存储函数 

通过存储函数计算指定员工的年薪

存储过程和存储函数的参数都不能带长度

存储函数的返回值类型不能带长度 

````sql
create or replace function f_yearsal(eno emp.empno%type) return number
is
  s number(10);
begin
  select sal*12+nvl(comm,0) into s from emp where empno=eno;
  return s;
end;
````

 测试f_yearsal 

存储函数在调用的时候，返回值需要接收 

````sql
declare
  s number(10);
begin
  s:=f_yearsal(7788);
  dbms_output.put_line(s);
end;
````

***

### out类型参数如何使用 

使用存储过程来算年薪 

````sql
create or replace procedure p_yearsal(eno emp.empno%type,yearsal out number)
is
  s number(10);
  c emp.comm%type;
begin
  select sal*12,nvl(comm,0) into s,c from emp where empno=eno;
  yearsal:=s+c;
end;
````

#### in和out类型参数的区别是什么？
凡是涉及到into查询语句复制或者 := 复制操作的参数，都必须用out来修饰。

#### 存储过程和存储函数的区别
##### 语法区别：关键字不一样
存储函数比存储过程多了两个return。
##### 本质区别：存储函数有返回值，而存储过程没有返回值。
如果存储过程实现有返回值的业务，我们就必须使用out类型的参数
即便是存储过程使用了out类型的参数，其本质也不是真的有了返回值
而是在存储过程内部给out类型的参数赋值，在执行完毕后，我们直接拿到输出类型参数的值。



我们可以使用存储函数有返回值的特性，来自定义函数。
而存储过程不能用来自定义函数。
案例需求：查询出员工姓名，员工所在部门名称。
案例准备工作：把scott用户下的dept表复制到当前用户下

````sql
create table dept as select * from scott.dept;
````

使用传统方式来实现需求 

````sql
select e.ename,d.dname 
from emp e,dept d 
where e.deptno=d.deptno; 
````

***

### 触发器

触发器，就是制定一个规则，在我们做增删改操作得时候
只需要满足该规则，就自动触发，无须调用
语句级触发器：不包含for each row的触发器
行级触发器：包含for each row的及时行级触发器
加for each row是为了使用 :old 或者 :new 对象或者一行记录

插入一条记录，输出一个新员工入职

````sql
create or replace trigger t1
after
insert
on person
declare

begin
  dbms_output.put_line('一个新员工入职');
end;
````

触发器实现主键自增【行级触发器】
分析：在用户做插入操作之前，拿到即将插入的数据
给给该数据中的主键列赋值。

````sql
create or replace trigger auid
before
insert
on person
for each row
declare

begin
  select s_person.nextval into :new.pid from dual;
end;
````

 使用auid实现主键自增 

````sql
insert into person (pname) values ('a'); commit; 
````

