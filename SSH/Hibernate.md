# hibernate学习笔记

## 简介

Hibernate的创始人Gavin King
EJB 3.0专家委员会成员
JBoss核心成员之一
《Hibernate in Action》的作者
优秀的Java持久化层解决方案
主流的对象-关系映射工具

## Hibernate的优缺点

简化了JDBC 繁琐的编码

```java
Session session = HiberanteUtil.currentSession();
Query query = session.createQuery("from User");
List<User> users = (List<User>) query.list();
```

对面向对象特性支持良好
可移植性好
Hibernate的缺点
不适合需要使用数据库的特定优化机制的情况
不适合大规模的批量数据处理

## 与MyBatis的比较

相对于MyBatis的“SQL-Mapping”的ORM实现，Hibernate的ORM实现更加完善，提供了对象状态管理、级联操作等功能
完全面向对象，语句与数据库无关，开发者无需关注SQL的生成，开发简单，便于修改，数据库移植性好
由于直接使用SQL，MyBatis使用自由度较高

## 准备Hibernate

编写Hibernate配置文件
用于配置数据库连接
运行时所需的各种特性
一般命名为“hibernate.cfg.xml”

```xml
<property name="connection.url">						jdbc:oracle:thin:@127.0.0.1:1521:orcl</property>
<property name="connection.username">scott</property>	
<property name="connection.password">tiger</property>	
<property name="connection.driver_class">					oracle.jdbc.OracleDriver</property>
<property name="dialect">
		org.hibernate.dialect.Oracle10gDialect</property>
<property name="show_sql">true</property>
<property name="format_sql">true</property>
<property name="current_session_context_class">thread</property>

```

创建持久化类和映射文件
定义持久化类（也称实体类），实现java.io.Serializable 接口，添加默认构造方法
配置映射文件（*.hbm.xml）
向hibernate.cfg.xml文件中配置映射文件

```java
<session-factory>
    <!-- 省略其他配置 -->
    <mapping  resource="cn/hibernatedemo/entity/Dept.hbm.xml" />
</session-factory>
```

```java
public class Dept implements Serializable {
    private Byte deptNo;
    private String deptName;
    private String location;

    public Dept() {
    }
    //省略getter&setter 方法
}

```

```xml
<hibernate-mapping>
    <class  name="cn.hibernatedemo.entity.Dept"  table="`DEPT`">
        <id  name="deptNo"  column="`DEPTNO`"  type="java.lang.Byte">
            <generator  class="assigned" />
        </id>
        <property  name="deptName"  type="java.lang.String">
        	<column  name="`DNAME`"></column>
        </property>
        <property name="location" type="java.lang.String" column="`LOC`"/>
    </class>
</hibernate-mapping>

```

搭建Hibernate环境的步骤
引入所需的jar文件
配置hibernate.cfg.xml
创建持久化类并配置相关hbm.xml映射文件
在hibernate.cfg.xml中引入hbm.xml映射文件

## Hibernate三种状态之间的转换

![状态转换]( https://img-blog.csdn.net/20170719112118578?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSGFpdGFvMDgyMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast )

1. 如果对象刚创建，没有被持久化的话，那么这样对象的状态就是临时状态。
   瞬时对象特点:
   -不和Session 实例关联
   -在数数据库中 没有和瞬时对象关联的记录
2. 持久状态:持久化对象就是已经被保存进数据库的实体对象，并且这个实体对
   象现在还处于Hibernate的缓存管理之中。这时对该实体对象的任何修改，都会
   在清理缓存时同步到数据库中。
   其特点:
   -持久的实例在数据库中有对应的记录，并拥有一一个持久化标识(如ID) 。
   和Session相关联的对象
   瞬时对象转为持久对象:
   -通过Session对 象的save或者saveorupdate方法把-个瞬时对象与数据库相关联，这个瞬时对象就
   成为持久化对象。
3. 托管状态:托管状态的最大特征就是对象脱离了Hibernate缓存管理状态。
   游离状态与自由状态的区别是:
   自由状态数据库没有与其对应的记录，
   -托管 状态数据库有- -个对应的记录，但是脱离了Hibernate缓存管理状态，比如Session.close()

## 脏检查与刷新缓存

​	当刷新缓存（调用Session的flush()方法）时，Hiberante会对Session中持久状态的对象进行检测，判断对象的数据是否发生了改变
commit()方法会首先刷新缓存

​	刷新缓存就是将数据库同步为与Session缓存一致
刷新缓存时会执行脏检查
Session会在以下时间点刷新缓存
调用Session的flush()方法
调用Transaction的commit()方法

## 更新数据的方法

update()方法
saveOrUpdate()方法
merge()方法

## Hibernate支持的查询方式

### HQL查询

HQL是Hibernate查询语言（Hibernate Query Language）

```java
from  cn.hibernatdemo.entity.Dept
from  Dept

from  Dept  where  deptName = 'SALES' 
from  Dept  dept  where  dept.location  is not null

from  Emp  order by  hireDate, salary desc

select  deptNo, deptName  from  Dept
```

执行HQL语句的步骤
获取Session对象
编写HQL语句
创建Query对象
执行查询，得到查询结果

list()方法

```java
session = sessionFactory.getCurrentSession();
String hql = "from Emp";
Query query = session.createQuery( hql );
List<Emp> empList = query.list();

```

iterate()方法

```java
session = sessionFactory.getCurrentSession();
String hql = "from Emp";
Query query = session.createQuery( hql );
Iterator<Emp> empIterator = query.iterate();

```



Criteria查询
原生SQL（Native SQL）查询

### 在HQL查询语句中绑定参数

使用字符串拼接查询条件存在各种弊端"from User where name = '" + name + "'"
性能低
不安全



使用占位符
按参数位置绑定
from User where name = ?

[下标从0开始]



按参数名称绑定
from User where name = :name

[可读性好，易维护，推荐使用]

### 为参数赋值

setXXX()：针对具体数据类型
setXXX( int position,  XXX value)
setXXX( String name, XXX value)
setParameter()：任意类型参数
setParameter( int position,  Object value)
setParameter( String name, Object value)
setProperties()：专为命名参数定制

### 分页查询

查询第一页的雇员信息，按员工编号升序排列，每页显示2条记录
Query接口的相关方法
uniqueResult()  ：获取唯一对象
setFirstResult() ：设置从第几条开始
setMaxResults()：设置读取最大记录数

### 投影

HQL投影查询是查询一个持久化类的一个或多个属性值，或者是通过表达式或聚合函数得到的值
投影查询需要使用HQL的select子句
查询结果的封装主要分三种情况
封装成Object对象
封装成Object数组
通过构造方法封装成对象
对象不是持久化状态，仅用于封装结果

**经验**:若查询结果仅用于展示，不需要保持持久化状态，应尽量使用投影查询以减少开销，提高效率

## 关联映射

类与类之间最普遍的关系就是关联关系
单向的关联
双向的关联

### 多对一关联关系

配置 Emp 到 Dept 的多对一关联
Emp 持久化类

```java
public class Emp implements Serializable {
	……
	private Dept dept;	// 省略其他属性及getter、setter访问器
}
```

Emp.hbm.xml

```java
<class  name="cn.hibernatedemo.entity.Emp"  table="`EMP`">
	......
	<many-to-one  name="dept"  column="`DEPTNO`" 				          class="cn.hibernatedemo.entity.Dept"  />
</class>
```

关联关系下的持久化操作

### 一对多关联关系

配置 Dept 到 Emp 的一对多关联
Dept 持久化类

```java
public class Dept implements Serializable {
	private Set<Emp> emps = new HashSet<Emp>();
	// 省略其他属性及getter、setter访问器
}
```

Dept.hbm.xml

```java
<class name="cn.hibernatedemo.entity.Dept" table="`DEPT`">
       ......
       <set name="emps">
              <key column="`DEPTNO`"></key>
              <one-to-many class="cn.hibernatedemo.entity.Emp"/>
       </set>
</class>
```

#### cascade属性

| **cascade**** 属性值** | **描      述**                                               |
| ---------------------- | ------------------------------------------------------------ |
| **none**               | **当**Session操纵当前对象时，忽略其他关联的对象。它是cascade属性的默认值 |
| **save-update**        | **当通过**Session的save()、update()及saveOrUpdate()方法来保存或更新当前对象时，级联保存所有关联的新建的瞬时状态的对象，并且级联更新所有关联的游离状态的对象 |
| **merge**              | **当通Session的**merge()方法来保存或更新当前对象时，对其关联对象也执行merge()方法 |
| **delete**             | **当通过**Session的delete()方法删除当前对象时，会级联删除所有关联的对象 |
| **all**                | **包含所有的级联行为**                                       |

#### <set>元素的inverse属性

inverse属性指定了关联关系中的方向
inverse设置为false，则为主动方，由主动方负责维护关联关系，默认是false 
inverse设置为true，不负责维护关联关系

经验:

1. 在建立两个对象的双向关联时，应该同时修改两个关联对象的相关属性

2. 建议inverse设置为true

### 多对多关联关系

配置Project和Employee的双向多对多关联
Project、Employee持久化类
Project.hbm.xml、Employee.hbm.xml

```java
<class name="cn.hibernatedemo.entity.Employee" table="EMPLOYEE">
    <id name="empid" type="java.lang.Integer">
        <column name="EMPID" precision="6" scale="0" />
        <generator class="assigned" />
    </id>
    ......
    <set name="projects" inverse="true" table="PROEMP">
        <key column="REMPID"/>
        <many-to-many class="cn.hibernatedemo.entity.Project" 
                                    column="RPROID" />
    </set>
</class>
```

小结

请按照多对多关联，写出Project和Employee的映射文件

```java
<!-- Project.hbm.xml中的关键配置 -->
<set name="employees" table="PROEMP" cascade="save-update">
            <key column="RPROID" />
            <many-to-many class="cn.hibernatedemo.entity.Employee"
                                        column="REMPID" />
</set>
<!-- Employee.hbm.xml中的关键配置-->
<set name="projects" inverse="true" table="PROEMP">
            <key column="REMPID"/>
            <many-to-many class="cn.hibernatedemo.entity.Project" 
                                        column="RPROID" />
</set>
```

请按照一对多关联，写出Project和Employee的映射文件

```java
<!-- Project.hbm.xml中的关键配置 -->
<set name="proemps" inverse="true">
    <key><column name="RPROID" not-null="true" /></key>
    <one-to-many class="cn.hibernatedemo.entity.Proemp" />
</set>
<!-- Employee.hbm.xml中的关键配置-->
<set name="proemps" inverse="true">
    <key><column name="REMPID" not-null="true" /></key>
    <one-to-many class="cn.hibernatedemo.entity.Proemp" />
</set>
<!-- 关系表PROEMP的持久化类映射Proemp.hbm.xml中的关键配置-->
<many-to-one name="project" class="cn.hibernatedemo.entity.Project"
    column="RPROID" />
<many-to-one name="employee" class="cn.hibernatedemo.entity.Employee" 
    column="REMPID" />
```

### 延迟加载

延迟加载（lazy load懒加载）是在真正需要数据时才执行SQL语句进行查询，避免了无谓的性能开销
延迟加载策略的设置分为
类级别的查询策略
一对多和多对多关联的查询策略
多对一关联的查询策略

### lazy属性

一对多和多对多关联的查询策略

| **lazy 属性值**  | **加载策略**     |
| -------------------- | ---------------- |
| **true（默认）** | **延迟加载**     |
| **false**            | **立即加载**     |
| **extra**            | **增强延迟加载** |

多对一关联的查询策略

| **lazy 属性值**   | **加载策略**       |
| --------------------- | ------------------ |
| **proxy（默认）** | **延迟加载**       |
| **no-proxy**          | **无代理延迟加载** |
| **false**             | **立即加载**       |

### Open Session In View模式

在用户的每一次请求过程始终保持一个Session对象打开着

![1592442349624](C:\Users\XXX\AppData\Roaming\Typora\typora-user-images\1592442349624.png)

## HQL连接查询和注解

###  HQL的连接查询

和SQL查询一样，HQL也支持各种各样的连接查询，如内连接、外连接

| **连接类型**     | **HQL**语法                                          |
| ---------------- | ---------------------------------------------------- |
| **内连接**       | **inner join**         **或** **join**               |
| **迫切内连接**   | **inner join fetch    或 join fetch**                |
| **左外连接**     | **left outer join      或 left join**                |
| **迫切左外连接** | **left outer join fetch** **或** **left join fetch** |
| **右外连接**     | **right outer join    或 right join**                |

### 等值连接

适用于两个类之间没有定义任何关联时
在where子句中通过属性作为筛选条件

```java
from Dept d , Emp e where d = e.dept
```

1. 没有where条件，直接执行“from Dept d , Emp e”会返回笛卡尔积，结果没有实际意义
2. 语句 from Dept d , Emp e where d.emps = e 是错误的，Set类型不能与Emp类型划等号

### 隐式内连接

```java
from  Emp e  where  e.dept.dname  = 'SALES'  -- 用于where子句
select empno, ename, dept.dname from Emp   -- 用于select子句
```

Hibernate会根据关联关系自动使用等值连接（等效于内连接）查询
允许以更加面向对象的方式编写HQL语句，更多地依据对象间的关系，而不必考虑数据库结构

## HQL连接查询和注解

###  HQL的连接查询

和SQL查询一样，HQL也支持各种各样的连接查询，如内连接、外连接

| **连接类型**     | **HQL**语法                                          |
| ---------------- | ---------------------------------------------------- |
| **内连接**       | **inner join**         **或** **join**               |
| **迫切内连接**   | **inner join fetch    或 join fetch**                |
| **左外连接**     | **left outer join      或 left join**                |
| **迫切左外连接** | **left outer join fetch** **或** **left join fetch** |
| **右外连接**     | **right outer join    或 right join**                |

### 等值连接

适用于两个类之间没有定义任何关联时
在where子句中通过属性作为筛选条件

```sql
from Dept d , Emp e where d = e.dept
```

1. 没有where条件，直接执行“from Dept d , Emp e”会返回笛卡尔积，结果没有实际意义

2. 语句 from Dept d , Emp e where d.emps = e 是错误的，Set类型不能与Emp类型划等号

### 隐式内连接

```sql
from  Emp e  where  e.dept.dname  = 'SALES'  -- 用于where子句
select empno, ename, dept.dname from Emp   -- 用于select子句
```

Hibernate会根据关联关系自动使用等值连接（等效于内连接）查询
允许以更加面向对象的方式编写HQL语句，更多地依据对象间的关系，而不必考虑数据库结构

### 分组统计数据

聚合函数

| **函数名称** | **说明**         |
| ------------ | ---------------- |
| **count()**  | **统计记录条数** |
| **sum()**    | **求和**         |
| **max()**    | **求最大值**     |
| **min()**    | **求最小值**     |
| **avg**()    | **求平均值**     |

### 子查询

子查询语句应用在HQL查询语句的where子句中

| **关键字** | **说明**               |
| ---------- | ---------------------- |
| **all**    | **返回的所有记录**     |
| **any**    | **返回的任意一条记录** |
| **some**   | **和“**any”意思相同    |
| **in**     | **与“**=any”意思相同   |
| **exists** | **至少返回一条记录**   |

| **函数或属性**               | **说    明**                                   |
| ---------------------------- | ---------------------------------------------- |
| **size()** **或**size        | **获取集合中元素的数目**                       |
| **minIndex**()或minIndex     | **对于建立了索引的集合，获得最小的索引**       |
| **maxIndex**()或maxIndex     | **对于建立了索引的集合，获得最大的索引**       |
| **minElement**()或minElement | **对于包含基本类型元素的集合，获取最小值元素** |
| **maxElement**()或maxElement | **对于包含基本类型元素的集合，获取最大值元素** |
| **elements()**               | **获取集合中的所有元素**                       |

### 查询性能优化

Hibernate查询优化策略
使用延迟加载等方式避免加载多余数据
通过使用连接查询，配置二级缓存、查询缓存等方式减少select语句数目
结合缓存机制，使用iterate()方法减少查询字段数及数据库访问次数
对比list()方法和iterate()方法
HQL优化
注意避免or、not、like使用不当导致的索引失效
注意避免having子句、distinct导致的开销
注意避免对索引字段使用函数或进行计算导致的索引失效

### 在Hibernate中使用注解

代替hbm.xml文件完成对象-关系映射
使用步骤如下：
使用注解配置持久化类以及对象关联关系
在Hibernate配置文件（hibernate.cfg.xml）中声明持久化类
<mapping  class="持久化类完整限定名"  />

### 注解配置持久化类

| **注  解**             | **含义和作用**                 |
| ---------------------- | ------------------------------ |
| **@Entity**            | **将一个类声明为一个持久化类** |
| **@Table**             | **为持久化类映射指定表**       |
| **@Id**                | **声明了持久化类的标识属性**   |
| **@GeneratedValue**    | **定义标识属性值的生成策略**   |
| **@SequenceGenerator** | **定义序列生产器**             |
| **@Column**            | **将属性映射到列（字段）**     |
| **@Transient**         | **将忽略这些属性**             |

