# `Spring Data JPA` 入门

标签（空格分隔）： JAVA

---

`SPring Data JPA` 提供的编程接口：

 - `Repository` ： 最顶层的接口，是一个空接口，目的是为了统一所有的 `Repository` 的类型，且能让组件扫描的时候自动识别。
 - `CrudRepository`： `Repository` 的子接口，提供`CRUD`的功能
 - `PageingAndSortingRepository` ： `CrudRepository` 的子接口，添加分页排序的功能
 - `JpaRepository` ： `PageingAndSortingRepository` 的子接口，增加批量操作等功能
 - `JpaSpecificationExecutor` ： 用来做复杂查询的接口
 

![接口的继承关系图：][1]
----------
# `JpaRepository` 接口方法：

- delete 删除或者批量删除
- findAll 查找所有
- findOne 查找单个
- save 保存单个或批量保存
- saveAndFlush 保存并保存到数据库

 示例：用户管理
 
 `persitent.xml` 配置文件
 ```xml
 <?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="1.0">
<persistence-unit name="userPU" transaction-type="RESOURCE_LOCAL">
    <!--jpa的提供者-->
    <provider>org.hibernate.ejb.HibernatePersistence</provider>
    <properties>
        <!--声明数据库连接的驱动-->
        <property name="hibernate.connection.driver_class" value="com.mysql.jdbc.Driver"/>
        <!--jdbc数据库的连接地址-->
        <property name="hibernate.connection.url" value="jdbc:mysql://localhost:3306/test?characterEncoding=UTF-8&amp;autoReconnect=true"/>
        <property name="hibernate.connection.username" value="root"/>
        <property name="hibernate.connection.password" value="zdw890926"/>
        <!--配置方言-->
        <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>
        <!--激活查询日志功能-->
        <property name="hibernate.show_sql" value="true"/>
        <!--优雅地输出Sql-->
        <property name="hibernate.format_sql" value="true"/>
        <!--添加一条解释型标注-->
        <property name="hibernate.use_sql_comments" value="false"/>
        <!--配置如何根据java模型生成数据库表结构,常用update,validate-->
        <property name="hibernate.hbm2ddl.auto" value="update"/>
    </properties>
</persistence-unit>
</persistence>
 ```
 
 `Spring` 上下文的配置文件 `applicationContext.xml` 文件：
```xml
 <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:context="http://www.springframework.org/schema/context"   
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"    
    xmlns:jee="http://www.springframework.org/schema/jee"   
    xmlns:tx="http://www.springframework.org/schema/tx"  
    xmlns:jpa="http://www.springframework.org/schema/data/jpa"  
    xsi:schemaLocation="  
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd  
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd  
        http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-3.2.xsd  
        http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-3.2.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.2.xsd  
        http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd"  
    default-lazy-init="true">

    <!--第一步-->
    <!--定义服务层代码存放的包扫描路径-->
	<context:component-scan base-package="javaskills.jpa.usermanage.service" />

    <!--第二步-->
    <!--定义实体的工厂bean-->
    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="persistenceUnitName" value="userPU" />
        <property name="persistenceXmlLocation" value="classpath:persistent.xml"></property>
    </bean>

    <!--第三步-->
    <!--定义事务管理器-->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"/>
    </bean>

    <!--第四步-->
    <!--定义repository接口的存放目录-->
    <!--定义接口实现的后缀，通常用Impl-->
    <!--定义实体工厂的引用-->
    <!--定义事务管理器的引用-->
   	<jpa:repositories base-package="javaskills.jpa.usermanage.repository"
   					  repository-impl-postfix="Impl" 
   					  entity-manager-factory-ref="entityManagerFactory" 
   					  transaction-manager-ref="transactionManager"/>
    <!--第五步-->
    <!--声明采用注解的方式申明事务-->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

查询操作的基本实现 --- 方法名构造方法

    find + 全局修饰 + By + 实体的属性名称 + 限定词 + 连接词 + ... （其他实体属性）+ OrderBy + 排序属性 + 排序方向
例如
`findDistinctByFirstNameIgnoreCaseAndLastNameOrderByAgeDesc(String firstName,String lastName)`

> 其中：
        `Distinct` 是全局修饰 非必须
        `FirstName` 和 `LastName` 是实体的属性名
        `And` 是连接词
        `IgnoreCase` 是限定词
        `Age` 是排序属性
        `Desc` 是排序方向
        **限定词和连接词统称为“关键词”**


> 
关键词分类：
全局修饰： `Distinct` , `Top` , `First`
关键词： `IsNull`,`IsNotNull`,`Like`,`NotLike`,`Containing`,`In`,`NotIn`,`IgnoreCase`,`Between`,`Equals`,`LessThan`,`GreaterThan`,`After`,`Before`......
排序方向：`Asc`,`Desc`
连接词：`And`,`Or`


Table 2.3. Supported keywords inside method names

|Keyword|Sample|JPQL |
| :----- | :----- | :-----  |   
|`And`|	`findByLastnameAndFirstname`|	`… where x.lastname = ?1 and x.firstname = ?2`|
|`Or`	|`findByLastnameOrFirstname`|	`… where x.lastname = ?1 or x.firstname = ?2`|
|`Is`,`Equals`	|`findByFirstname`, `findByFirstnameIs`, `findByFirstnameEquals`	|`… where x.firstname = 1?`|
|`Between`|	`findByStartDateBetween`|	`… where x.startDate between 1? and ?2`|
|`LessThan`|	`findByAgeLessThan`|	`… where x.age < ?1`|
|`LessThanEqual`|	`findByAgeLessThanEqual`|`	… where x.age <= ?1`|
|`GreaterThan`	|`findByAgeGreaterThan`|	`… where x.age > ?1`|
|`GreaterThanEqual`|	`findByAgeGreaterThanEqual`|	`… where x.age >= ?1`|
|`After`|	`findByStartDateAfter`|`	… where x.startDate > ?1`|
|`Before`|	`findByStartDateBefore`|	`… where x.startDate < ?1`|
|`IsNull`|	`findByAgeIsNull`	|`… where x.age is null`|
|`IsNotNull,NotNull`|	`findByAge(Is)NotNull`|	`… where x.age not null`|
|`Like`|	`findByFirstnameLike`	|`… where x.firstname like ?1`|
|`NotLike`|	`findByFirstnameNotLike`|	`… where x.firstname not like ?1`|
|`StartingWith`|`findByFirstnameStartingWith`	|`… where x.firstname like ?1` \ `(parameter bound with appended %)`|
|`EndingWith`	|`findByFirstnameEndingWith`|`… where x.firstname like ?1` \ `(parameter bound with prepended %)`|
|`Containing`|`findByFirstnameContaining`|`… where x.firstname like ?1 ` \ `(parameter bound wrapped in %)`|
|`OrderBy`|`findByAgeOrderByLastnameDesc`|	`… where x.age = ?1 order by x.lastname desc`|
|`Not`	|`findByLastnameNot`|	`… where x.lastname <> ?1`|
|`In`|`findByAgeIn(Collection<Age> ages)`|	`… where x.age in ?1`|
|`NotIn`|`findByAgeNotIn(Collection<Age> age)`|	`… where x.age not in ?1`|
|`True`|`findByActiveTrue()`|	`… where x.active = true`|
|`False`|`findByActiveFalse()`	|`… where x.active = false`|
|`IgnoreCase`|`findByFirstnameIgnoreCase`|`	… where UPPER(x.firstame`|

```java
@Entity
@Table(name = "user")
public class User {

    @Id
    @GeneratedValue
    private Integer id;
    private String name;
    private String address;
    private String phone;
    //省略getter和setter
}

```





















  [1]: http://d.pcs.baidu.com/thumbnail/0d3a57298038ca2dd772a2fb758e1f80?fid=2266416092-250528-93475818706227&time=1440748800&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-tfdz%2FT%2BweDtIW0Qzac6BBjUiyTM%3D&rt=sh&expires=2h&r=390105949&sharesign=unknown&size=c710_u500&quality=100