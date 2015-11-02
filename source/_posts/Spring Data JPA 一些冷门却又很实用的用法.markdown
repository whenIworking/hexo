# Spring Data JPA 一些冷门却又很实用的用法

标签（空格分隔）： JAVA

---

最近工作中频繁的使用 `SPring Data JPA` ，一开始对`SPring Data JPA`不甚了解，就感觉在利用`SPring Data JPA`进行复杂查询的时候感觉非常滞涩。事实证明，`SPring Data JPA`虽然存在一些局限性，但是只要能够了解其提供的接口就可以用来完成非常复杂的功能。本文就从实用的角度，对`SPring Data JPA`的一些较为冷门而又实用的用法进行总结。

## 利用`JPQL`进行`@OneToMany`的关联查询
首先，举一个一对多的例子，球队和球员，一个球队有多个球员，那么具体的映射代码如下：
```java
@Entity
@Table(name="Team")
public class Team{
   //....省略
   @OneToMany(mappedBy="team",cascade=CascadeType.ALL,fetch=FetchType.LAZY)
   private List<Player> players = new ArrayList<Player>();
   //....省略
}
```


```java
@Entity
@Table(name="Player")
public class Player{
    private String name;
    private int age;
   @ManyToOne(cascade={CascadeType.MERGE,CascadeType.REFRESH})  
   @JoinColumn(name="team_id")  
   private Team team;
   //....省略

}
```
现在假设要查询队员有年龄小于20的队伍有哪些，有两种方法。一种是在继承 `JpaRepository` 的接口中定义查询方法，但是当查询条件复杂的时候，查询的方法名就会非常的长，不利于阅读；于是就有了第二种方法，在自定义的查询方法上写`JPQL`查询语句
```java
    findByNamePlayersAgeLessThan(String name , int age)
    //虽然可以直接在定义的查询方法中直接写 players+Age，但是这种类推到JPQL中是错误的
    //例如，不可以写成：SELECT DISTINCT t FROM Team t  WHERE t.players.name LIKE :name 
    
    @Query("select t from Team t, IN(t.players) p where p.name like ?1 and age <= ?2")
    findTeamsAccordingAge(String name , int age)
    //或者
    @Query("select t from Team t JOIN t.players p where p.name like ?1 and age <= ?2")
    findTeamsAccordingAge(String name , int age)
    
    
```

以上是从One到Many的查询，那么如何从Many到One进行关联查询呢？
例如查出某个球队下的所有球员，可以使用如下的查询语句：

```SQL
SELECT p FROM Player p JOIN p.team t WHERE t.id = :id 
//或者
SELECT p FROM Player p, IN(p.team) t WHERE t.id = :id 
//或者再简洁一些
SELECT p FROM Player p WHERE p.team.id = :id 
```
以上的三条查询实际上会生成两条查询`SQL`，如果只想生成一条查询SQL，则需要使用`fetch join`,如下：
```SQL
SELECT p FROM Player p JOIN FETCH p.team t WHERE t.id = :id 

//实际产生的SQL
Hibernate:  
    select 
        player0_.id as id1_0_,  
        team1_.id as id2_1_,  
        player0_.name as name1_0_,  
        player0_.team_id as team3_1_0_,  
        team1_.name as name2_1_  
    from 
        player player0_  
    inner join 
        team team1_  
            on player0_.team_id=team1_.id  
    where 
        team1_.id=? 
```

## @Query（）中的Top k查找
为了能够处理更复杂的条件查询，常常使用`@Query`注解，自己写`JPQL`语句，但是这个时候在自定义方法查询中的`Top`关键字就不能使用了，在这种情况下如果还想要查询`Top k`记录就需要利用`Pageable`的分页功能。如下所示

```java
@Query("select i.nName,sum(i.number) as num from Item i where i.type = ?1 group by i.id order by num desc ")
List<Object[]> findTopk(String productType,Integer status, Pageable pageable);

//使用方法
Pageable pageable = new PageRequest(0,k);
List<Object[]> ress = findTopk("type",pageable); //返回Top k条记录
```

在`Pageable` 中还可以对排序结果进行排序和分页，使用如下注解

    @PageDefault(size= 10 , sort = "attributeTobeSort" , direction = Sort.Directin.DESC)
    
其中： `size` 表示每一页的数据条数
    `sort` 表示按照哪一个字段进行排序
    `direction` 表示排序是按照升序还是降序进行排列
    
注解`@PageDefault`主要在`Controller`中进行使用，所以要想使用`Pageable`的功能，就需要为其传递如下的几个参数
```json
{
    "size":10, //每页的数据数量
    "page":0, //第几页
    "sort":attributeTobeSort,asc(desc) //按照哪一个属性进行排序，是降序还是升序
}
```
##Spring Data Jpa 查询返回自定义对象
```java
@Entity
@Table(name="t_user")
public class User {
    private Integer id;
    private String name;
    private String address;
    //省略getter和setter
}

package com.project.user.vo
//查询返回对象
public class UserVo{
    private Integer id;
    private String name;
    //省略getter和setter

}
```

查询方法如下
```java
//查询方法
@Query("select id,name from User u ")  
public List<UserDto> find();
```
此时返回的结果是`List<Object[]>`，如果想要返回自定义的`UserVo`对象，则需要在类中添加构造函数如下
```java
 public UserVo(Integer id, String name) {
        this.id = id;
        this.name = name;
    }
```

查询方法如下
```java
//查询方法,要加上包名,如果使用了nativeQuery则这种方法是不起作用的
@Query("select new com.project.user.vo.UserVo(u.id,u.name) from User u ")  
public List<UserDto> find();
```
【不定期持续更新，未完待续】