# JPA规范

标签（空格分隔）： JAVA

---
原文正版参考：[一直在等 的寂寞：JPA博客][1]
#什么是JPA

JPA(Java Persistence API)是Sun官方提出的Java持久化规范。它为Java开发人员提供了一种对象/关联映射工具来管理Java应用中的关系数据。他的出现主要是为了简化现有的持久化开发工作和整合ORM技术，结束现在Hibernate，TopLink，JDO等ORM框架各自为营的局面。值得注意的是，JPA是在充分吸收了现有Hibernate，TopLink，JDO等ORM框架的基础上发展而来的，具有易于使用，伸缩性强等优点。从目前的开发社区的反应上看，JPA受到了极大的支持和赞扬，其中就包括了Spring与EJB3.0的开发团队。

总的来说，JPA包括以下3方面的技术：

* ORM映射元数据
* * JPA支持XML和JDK5.0注释(也可译作注解)两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持久化到数据库表中。
* Java持久化API
* * 用来操作实体对象，执行CRUD操作，框架在后台替我们完成所有的事情，开发者可以从繁琐的JDBC和SQL代码中解脱出来。
* 查询语言（JPQL）
* * 这是持久化操作中很重要的一个方面，通过面向对象而非面向数据库的查询语言查询数据，避免程序的SQL语句紧密耦合。

JPA不是一种新的ORM框架，它的出现只是用于规范现有的ORM技术，他不能取代现有的Hibernate，TopLink等ORM框架。相反，在采用JPA开发时，我们仍将使用到这些ORM框架，只是此时开发出来的应用不再依赖于某个持久化提供商。应用可以在不修改代码的情况下在任何JPA环境下运行，真正做到低耦合，可扩展的程序设计。

#JPA的配置
JPA规范要求在类路径的`META-INF`目录下放置persistence.xml，文件的名称是固定的，配置模板如下：
```xml
<?xml version="1.0"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-xsi:schemaLocation="http://java.sun.com/xml/ns/persistence/persistence_1_0.xsd" version="1.0">
　　<persistence-unit name="sample" transaction-type="RESOURCE_LOCAL">
　　　　<properties>
　　　　　　<property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>
　　　　　　<property name="hibernate.connection.driver_class" value="org.gjt.mm.mysql.Driver"/>
　　　　　 <property name="hibernate.connection.username" value="root"/>
　　　　　 <property name="hibernate.connection.password" value="123456"/>
　　　　　　<property name="hibernate.connection.url" value="jdbc:mysql://localhost:3306/hibernate"/>
　　　　　 <property name="hibernate.show_sql" value="true"/>
　　　　　　<property name="hibernate.hbm2ddl.auto" value="update"/>
　　　 </properties>
　　</persistence-unit>
</persistence>
```

> 其实这个*`hibernate.hbm2ddl.auto`*参数的作用主要用于：自动创建|更新|验证数据库表结构。如果不是此方面的需求建议`set value="none"`。里面可以设置的几个参数：
• `validate` 每次加载`hibernate`时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。
• `create` 每次加载`hibernate`时都会删除上一次的生成的表，然后根据你的model类再重新来生成新表，哪怕两次没有任何改变也要这样执行，这就是导致数据库表数据丢失的一个重要原因。
• `create-drop` 每次加载`hibernate`时根据`model`类生成表，但是`sessionFactory`一关闭,表就自动删除。
• `update` 最常用的属性，第一次加载`hibernate`时根据`model`类会自动建立起表的结构（前提是先建立好数据库），以后加载`hibernate`时根据 `model`类自动更新表结构，即使表结构改变了但表中的行仍然存在不会删除以前的行。要注意的是当部署到服务器后，表结构是不会被马上建立起来的，是要等应用第一次运行起来后才会。
注意：请慎重使用此参数，没必要就不要随便用。如果发现数据库表丢失，请检查`hibernate.hbm2ddl.auto`的配置。

-------

>  通常在企业开发中，有两种做法：
> 
> * 先建表，后再根据表来编写配置文件和实体`bean`。使用这种方案的开发人员受到了传统数据库建模的影响。
> * 先编写配置文件和实体`bean`，然后再生成表，使用这种方案的开发人员采用的是领域建模思想，这种思想相对前一种思想更加`OOP`。

>建议使用第二种（领域建模思想），从软件开发来想，这种思想比第一种思想更加面向对象。 领域建模思想也是目前比较新的一门建模思想，第一种是传统的建模思想，已经有10来年的发展历程了，而领域建模思想是近几年才兴起的，这种思想更加的面向对象。

## 基本配置解析
*`<persistence-unit><persistence-unit/>`*持久化单元，简单说，就是代表一堆实体`bean`的集合，那么这堆实体`bean`，我们叫他们做实体`bean`单元。我们在学*`Hibernate`*就已经知道，他们就是专门用于跟数据库映射的普通Java对象，在我们JPA里面，这些对象叫做实体`bean`。持久化单元就是一堆实体`bean`的集合，我们为这堆集合取个名称，*`<persistence-unit name="……"><persistence-unit/>`*

- ***全局事务***：资源管理器管理和协调的事务，可以跨越多个数据库和进程。资源管理器一般使用二次提交协议与数据库进行交互。
- ***本地事务***：在单个数据库里进行的事务。本地数据源不涉及多个数据来源。

在`<persistence-unit>`中使用`transaction-type="RESOURCE_LOCAL / JTA"`来控制本地和全局事务的开启。

> ***二次提交协议***简单说就这样：如果你先执行第一条语句，执行的结果先预提交到数据库，预提交到数据库了，数据库会执行这条语句，然后返回一个执行的结果，这个结果假如我们用布尔值表示的话，成功就是true，失败就是false.然后把执行的结果放入一个（假设是List）对象里面去，接下来再执行第二条语句，执行完第二条语句之后（也是预处理，数据库不会真正实现数据的提交，只是说这条语句送到数据库里面，它模拟下执行，给你返回个执行的结果），假如这两条语句的执行结果在List里面都是true的话，那么这个事务就认为语句是成功的，这时候全局事务就会提交。 二次提交协议，数据库在第一次提交这个语句时，只会做预处理，不会发生真正的数据改变，当我们在全局事务提交的时候，这时候发生了第二次提交，那么第二次提交的时候才会真正的发生数据的改动。如果说在执行这两条语句中，有一个出错了，那么List集合里就有个元素为false，那么全局事务就认为你这个事务是失败的，它就会进行回滚，回滚的时候，哪怕你的第二条语句在第一次提交的时候是成功的，它在第二次提交的时候也会回滚，那么第一次的更改也会恢复到之前的状态，这就是二次提交协议。（可以查看一下数据库方面的文档来了解二次提交协议）

回到persistence.xml的配置里面去，事务类型有两种，什么时候该用全局事务(JTA)?什么时候改用本地事务(RESOURCE_LOCAL)?应有你的业务应用需求来定，我们的大部分应用只是需要本地事务。全局事务通常是在应用服务器里使用，比如weblogic,JBoss等。

#JPA的主键生成策略
每个实体bean都要有个实体标识属性，这个实体标识属性主要用于在内存里判断对象。通过`@Id`就可以定义实体标识。可以标识在属性的get方法前面，也可以标识在字段上面。
如果希望采用数据库的id自增长的方式来生成主键的话，就需要`@GeneratedValue`注解，该注解中有一些属性，其中之一是策略`strategy`，JPA为用户提供了四种生成的策略：
> * AUTO  ： JPA自动选择合适的策略，是默认选项
> * IDENTITY ： 采用数据库ID自增长的方式来生成主键值，但是ORacle不支持这种方式
> * SEQUENCE：通过序列产生主键，通过`@SequenceGenerator`注解来指定序列名，MySql不支持这种方式
> * TABLE：采用表生成方式来生成主键值，这种生成方式效率不高

如果在开发的过程中，我们无法预测用户到底使用哪种数据库，那么就应该设置为`AUTO`,这就意味着持久化实现产品，来根据你使用的数据库的方言来决定它采用的主键值的生成方式，到底是`IDENTITY`、`SEQUENCE`、`TABLE`。如果想把声场策略设置成`@GeneratedValue(strategy=GenerationType.AUTO)的话，AUTO本身就是策略的默认值，就可以省略掉，简单写成@GenerateValue`
如果想要使用更加丰富的策略，可以使用`Hibernate`，它提供了更多的方案。

```java
@Entity
public class Person{
    @Id
    @GeneratedValue
    private Integer id;
    @Column(length=10,nullable=false,name="personname")
    private String name;
    //省略getter和setter
    ....
}
```
测试代码
```java
@Test
public void save(){
    EntityManagerFactory factory = Persistence.createEntityManagerFactory("Test");
    EntityManager em = factory.createEntityManager();
    em.getTransaction().begin();
    em.persist(new Person());
    em.getTransaction().commit();
    em.close();
    factory.close();
}
```

在`Hibernate`内同时存在持久化对象的`save()`和`persist()`两种方法，但是`Hibernate`的作者已经不太推荐大家使用`save`方法，而是推荐`persist`。
在本例中，目前数据库表示不存在的，在采用实体（领域）建模的思想，让它根据实体bean来生成数据库表，在persisence.xml中，`<property name="hibernate.hbm2ddl.auto" value="update" />`，生成策略是`update`说明表不存在的时候，它会根据实体对象创建对应的数据库表。
**问题是它什么时候创建表，创建表的时机是在什么时候创建？**
**答案是得到`SessionFactory`的时候，在JPA里也是一样，我们得到`EntityManagerFactory`的时候创建表**

#JPA日期以及枚举类型等字段的映射

```java
public enum Gender{
    MAN,WOMAN
}

import java.util.Date;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.EnumType;
import javax.persistence.Enumerated;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.persistence.Temporal;
import javax.persistence.TemporalType;
    
@Entity
@Table(name="PersonTable")
public class Person {
    @Id
    @GeneratedValue
    private Integer id;
    @Column(length=10,nullable=false,name="personname")
    private String name;
    @Temporal(TemporalType.DATE)
    private Date birthday;
    @Enumerated(EnumType.STRING) 
    @Column(length = 5, nullable = false)
    private Gender gender = Gender.MAN;
    public Person(){}
    public Person(String name) {
        this.name = name;
    }
   //省略getter和setter
   ....
}
```

 - @Table(name = "PersonTable") //改变数据库中映射表名 private Gender gender =
 - Gender.MAN; //这里可以设置默认值， Gender是一个枚举类型。
 - @Enumerated(EnumType.STRING)//说明这个属性是个枚举类型，括号内的表示存入数据库的枚举字符串而不是枚举索引 
 - @Temporal(TemporalType.DATE)   //说明这个属性映射到数据库中是一个日期类型，括号中的是日期格式
**对象是由Hibernate为我们创建的，当我们通过ID来获取某个实体的时候，这个实体给我们返回了这个对象的创建是由Hibernate内部通过反射技术来创建的，反射的时候用到了默认的构造函数，所以这时候必须给它提供一个public的无参构造函数。**

#大字段的映射*@Lob*与字段延迟加载*@Basic(fetch = FetchType.LAZY)*
```java
@Entity
@Table(name="PersonTable")
public class Person {
    @Id
    @GeneratedValue
    private Integer id;
    @Column(length=10,nullable=false,name="personname")
    private String name;
    @Temporal(TemporalType.DATE)
    private Date birthday;
    @Enumerated(EnumType.STRING) 
    @Column(length = 5, nullable = false)
    private Gender gender = Gender.MAN;
    @Lob //申明属性对应的数据库字段为一个大文本类，文件属性也是用这个声明映射。
    private String info;
    @Lob //声明属性对应的是一个大文件数据字段
    @Basic(fetch = FetchType.LAZY) //设置为延迟加载，当我们在数据库中取这条记录的时候，不会去取这个字段
    private Byte[] file;
    @Transient//这个注解用来标注imagePath这个属性不作为可持久化字段，就是说不跟数据库的字段做任何关联
    private String imagePath;
        省略get set方法……
}
```

> 假如`private byte[] file`保存的是一个文件，那么当要获取一个`Person`对象时候，会把file这个字段保存的内容取出来，并且放在内存中里。但是，如果这个文件有100M，那么每次获取`Person`的对象的时候，都会取得这个file文件，在内存中占用100M内存，太耗费资源了。所以在file上添加`@Basic(fetch=FetchType.LAZY)`这个注解。
如果设置了延迟加载，那么当我们调用`Hibernate`的`get`方法的时候得到`Person`的对象的记录的时候，如果没有访问`file`的属性的`get`方法的时候，就不会从数据库中帮我找到这个`file`，只有需要访问`file`的时候，才会把文件的数据加载上来。
所以，`@Basic`标签一般用在大数据上，如果存放的数据大小比较大，大概数据超过1M，就应该使用`@Basic`标签，把属性做延迟初始化。
一定要注意，在第一次访问file属性的时候，必须要确保`**EntityManager**`对象处于打开状态，否则会出现延迟加载的异常。

#使用JPA加载、更新、删除对象

![【JPA实体对象的生命周期】][2]

目前我们使用的是 `Hibernate` ，实际上我们操作 `EntityManager` ， 内部是操作了 `Hibernate` 的 `session` 对象， 只是对 `session` 对象进行了封装。

```java
@Test public void getPerson(){
        EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
        EntityManager em=factory.createEntityManager();
        Person p=em.find(Person.class, 1);//相当于Hibernate的get方法
        System.out.println(p.getName());
        em.close();
        factory.close();
    }
```
如果不存在id为1的Person，则返回null。
```java
@Test 
public void getPerson(){
        EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
        EntityManager em=factory.createEntityManager();
        Person p=em.getReference(Person.class, 1);//相当于Hibernate的get方法
        System.out.println(p.getName());
        em.close();
        factory.close();
}
@Test
public void update(){
    EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
    EntityManager em=factory.createEntityManager();
    em.getTransaction().begin();
    Person p=em.find(Person.class, 1);
    p.setName("jame");//p处于持久态，所以直接更改，在事务提交时会与数据库进行同步
    em.getTransaction().commit();
    em.close();
    factory.close();
}
@Test
public void update2(){
    EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
    EntityManager em=factory.createEntityManager();
    em.getTransaction().begin();
    Person p=em.find(Person.class, 1);
    //在clear之后，p变成了游离状态，这时候对游离状态的实体进行更新的话（p.setName("jim");）,更新的数据是不能同步到数据库的。可以采用方法em.merge(p);这方法是用于把在游离状态时候的更新同步到数据库。
    
    em.clear();////把实体管理器中的所有实体变成游离状态。
    p.setName("jim");
    em.merge(p);
    em.getTransaction().commit();
    em.close();
    factory.close();
}
@Test
public void delete(){
    EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
    EntityManager em=factory.createEntityManager();
    em.getTransaction().begin();
    Person p=em.find(Person.class, 2);
    em.remove(p);//删除的bean对象也必须是处于托管状态的对象才能被删除成功。
    em.getTransaction().commit();
    em.close();
    factory.close();
}
```

#JPA的JPQL查询语言
`JPQL`是`Java Persistence Query Language`的简称。`JPQL` 是一种可移植的查询语言，旨在以面向对象表达式语言的方式，将`SQL`和简单语义绑定在一起。
JPQL的特征与原生的`SQL`类似，但是完全的面向对象，通过类名和属性名进行访问，而不是表名和表的属性

```java
@Test 
public void query(){
    EntityManagerFactory factory = Persistence.createEntityManagerFactory("sample");
    EntityManager entityManager = factory.createEntiryManager();
    Query query = entityManager.createQuery("select p from Person p where p.id = ?1");
    query.setParameter(1,3);
    Person p = (Person)query.getSingleResult();
    System.out.println(p);
    entityManager.close();
    factory.close();
}

@Test
public void deleteQuery(){
 EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
        EntityManager em=factory.createEntityManager();
        em.getTransaction().begin();
        Query query=em.createQuery("delete from Person o where o.id=?1");
        query.setParameter(1, 3);
        query.executeUpdate();
        em.getTransaction().commit();
        em.close();
        factory.close();
}

@Test
public void updateQuery(){
    EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
    EntityManager em=factory.createEntityManager();
    em.getTransaction().begin();
    Query query=em.createQuery("update Person o set o.name=:name where o.id=:id");
    query.setParameter("name", "brian");
    query.setParameter("id", 1);
    query.executeUpdate();
    em.getTransaction().commit();
    em.close();
    factory.close();
}
```
#JPA的一对多关系
通过一个例子来说明

    Order.java

```java
@Entity
@Table(name="orders") //这里非常讨厌，order是sql的关键字，所以一定不能用关键字来作为表名，否则会创建失败
public class Order {
	@Id
	private String orderId;
	@Column(nullable=false)
	private Float amount = 0f;
	@OneToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY, mappedBy = "order") //此处mappedBy指的是对应表中的哪个字段，这里是orderItem中的order字段
	private Set<OrderItem> items = new HashSet<OrderItem>();

	public String getOrderId() {
		return orderId;
	}

	public void setOrderId(String orderId) {
		this.orderId = orderId;
	}

	public Float getAmount() {
		return amount;
	}

	public void setAmount(Float amount) {
		this.amount = amount;
	}

	public Set<OrderItem> getItems() {
		return items;
	}

	public void setItems(Set<OrderItem> items) {
		this.items = items;
	}
}
```

    OrderItem.java

```java
@Entity
@Table(name = "order_item")
public class OrderItem {
	@Id
	@GeneratedValue(strategy= GenerationType.AUTO)
	private Integer id;
	@Column(length=255,nullable=false)
	private String  productName;
	@Column(nullable=false)
	private Float price = 0f;
	@ManyToOne(cascade={CascadeType.MERGE, CascadeType.REFRESH},optional=false)
	@JoinColumn(name="order_id")
	private Order order; //在Order.java中被用来做映射的字段

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public String getProductName() {
		return productName;
	}

	public void setProductName(String productName) {
		this.productName = productName;
	}

	public Float getPrice() {
		return price;
	}

	public void setPrice(Float price) {
		this.price = price;
	}

	public Order getOrder() {
		return order;
	}

	public void setOrder(Order order) {
		this.order = order;
	}
}
```

> 在JPA里面，一对多关系(1-n)：多的一方为关系维护端，关系维护端负责外键记录的更新（如果是条字段就负责字段的更新，如果是多对多关系中的中间表就负责中间表记录的更新），关系被维护端是没有权力更新外键记录（外键字段）的。

- CascadeType.REFRESH：级联刷新，也就是说，当你刚开始获取到了这条记录，那么在你处理业务过程中，这条记录被另一个业务程序修改了（数据库这条记录被修改了），那么你获取的这条数据就不是最新的数据，那你就要调用实体管理器里面的refresh方法来刷新实体，所谓刷新，大家一定要记住方向，它是获取数据，相当于执行select语句的（但不能用select，select方法返回的是EntityManager缓存中的数据，不是数据库里面最新的数据），也就是重新获取数据。
- CascadeType.PERSIST：级联持久化，也就是级联保存。保存order的时候也保存orderItem，如果在数据库里已经存在与需要保存的orderItem相同的id记录，则级联保存出错。
- CascadeType.MERGE： 级联更新，也可以叫级联合并；当对象Order处于游离状态时，对对象Order里面的属性作修改，也修改了Order里面的orderItems。
- CascadeType.REMOVE：当对Order进行删除操作的时候，也要对orderItems对象进行级联删除操作。

如果在应用中，要同时使用这四项的话，可以改成cascade = CascadeType.ALL

应用场合问题：这四种级联操作，并不是对所有的操作都起作用，只有当我们调用实体管理器的persist方法的时候，CascadeType.PERSIST才会起作用；同样道理，只有当我们调用实体管理器的merge方法的时候，CascadeType.MERGE才会起作用,其他方法不起作用。同样道理，只有当我们调用实体管理器的remove方法的时候，CascadeType.REMOVE才会起作用。

注意： Query query = em.createQuery("delete from Person o where o.id=?1");这种删除会不会起作用呢？是不会起作用的，因为配置里那四项都是针对实体管理器的对应的方法。

#JPA的多对多关系
在多对多的关系当中，两个实体的关系是对等的，所以无论是哪个实体作为关系的维护端都可以，在实际的业务中可以根据需要自行选择。
下面通过一个老师和学生的例子进行简单的说明:一个老师可以教多名学生，一个学生同时有多个老师，那么关系的维护端既可以是学生，也可以是老师。在本例子中，将维护段交给学生。

    Student.java

```java
@Entity
@Table(name="student")
public class Student {
	@Id
	@GeneratedValue(strategy= GenerationType.AUTO)
	private Integer id;
	private String name;
	
	//创建中间表，student对应到中间表的连接列studentId的是student的id，中间表teacherId对应到teacher的连接列是teacher的id字段，可以有多个连接列
	@ManyToMany
	@JoinTable(name="student_teacher",joinColumns = @JoinColumn(name="studentId",referencedColumnName = "id"),inverseJoinColumns = @JoinColumn(name="teacherId",referencedColumnName = "id"))
	private Set<Teacher> teachers = new HashSet<Teacher>();
	public Student() {
	}
	public Student(String name) {
		this.name = name;
	}
	//省略getter和setter
}
```

    Teacher.java

```java
@Entity
@Table(name="teacher")
public class Teacher {
	@Id
	@GeneratedValue(strategy= GenerationType.AUTO)
	private Integer id;
	private String name;

	@ManyToMany(mappedBy = "teachers")
	private Set<Student> students = new HashSet<Student>();

	public Teacher() {
	}

	public Teacher(String name) {
		this.name = name;
	}
	//省略getter和setter
}
```
测试代码
```java
@Test
	public void save2(){//存储实体
		EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
		EntityManager em=factory.createEntityManager();
		em.getTransaction().begin();
		em.persist(new Teacher("teacher"));
		em.persist(new Student("student"));
		em.getTransaction().commit();
		em.close();
		factory.close();
	}
	@Test
	public void buildRelation(){//建立实体间关系
		EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
		EntityManager em=factory.createEntityManager();
		em.getTransaction().begin();
		Student student=em.find(Student.class, 3);
		Teacher teacher=em.getReference(Teacher.class, 3);//getReference延迟加载，可提高性能
		student.getTeachers().add(teacher);
		em.getTransaction().commit();
		em.close();
		factory.close();
	}
	@Test
	public void deleteRelation(){//解除实体间关系
		EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
		EntityManager em=factory.createEntityManager();
		em.getTransaction().begin();
		Student student=em.find(Student.class, 1);
		Teacher teacher=em.getReference(Teacher.class, 1);//getReference延迟加载，可提高性能
		student.getTeachers().remove(teacher);
		em.getTransaction().commit();
		em.close();
		factory.close();
	}
	@Test
	public void deleteTeacher(){//删除老师，必须先删除中间表中得关系，才能删除老师
		EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
		EntityManager em=factory.createEntityManager();
		em.getTransaction().begin();
		Student student=em.find(Student.class, 1);
		Teacher teacher=em.getReference(Teacher.class, 1);//getReference延迟加载，可提高性能
		student.getTeachers().remove(teacher);
		em.remove(teacher);
		em.getTransaction().commit();
		em.close();
		factory.close();
	}
	@Test
	public void deleteStudent(){//删除学生，因为学生端是关系维护端，不用手动删除中间表的关系，会自动删除
		EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
		EntityManager em=factory.createEntityManager();
		em.getTransaction().begin();
		Student student=em.find(Student.class, 3);
		em.remove(student);
		em.getTransaction().commit();
		em.close();
		factory.close();
	}

```
#JPA一对一关系
一对一的关系是指：一个A事物对应一个B事物，一个B事物对应且仅对应一个A事物。在生活中这样的例子非常常见：一个人只有一个身份证，一个身份证只能被一个人拥有。
```java
@Entity
public class IDCard {
	@Id
	@GeneratedValue(strategy= GenerationType.AUTO)
	private Integer id;
	private String cardNo;
	@OneToOne(mappedBy = "idCard",cascade = {CascadeType.PERSIST,CascadeType.MERGE, CascadeType.REFRESH})
	private People people;

	public IDCard(){}
	public IDCard(String cardNo){
		this.cardNo = cardNo;
	}
		//省略getter和setter
}
```
```java
@Entity
public class People {
	@Id
	@GeneratedValue(strategy= GenerationType.AUTO)
	private Integer id;
	private String name;
	@OneToOne(cascade = CascadeType.ALL,optional = false)
	@JoinColumn(name = "idcard_id")
	private IDCard idCard;
	public People(){}
	public People(String name){
		this.name = name;
	}
		//省略getter和setter
}
```

测试代码
```java
@Test
public void save4(){
 EntityManagerFactory factory=Persistence.createEntityManagerFactory("sample");
  EntityManager em=factory.createEntityManager();
  em.getTransaction().begin();
  People people = new People("tom");

people.setIdCard(new IDCard("411121198909106536"));
em.persist(people);
  em.getTransaction().commit();
  em.close();
  factory.close();
}
```

#JPA联合主键
关于联合主键类，要遵守以下几点JPA规范：

- 必须提供一个public的无参数构造函数。
- 必须实现序列化接口。
- 必须重写hashCode()和equals()这两个方法。这两个方法应该采用复合主键的字段作为判断这个对象是否相等的。
- 联合主键类的类名结尾一般要加上PK两个字母代表一个主键类，不是要求而是一种命名风格。

```java
@Embeddable
public class TrainLinePK implements Serializable {

	private String startStation;
	private String endStation;

	public TrainLinePK() {
	}

	public String getStartStation() {
		return startStation;
	}

	public void setStartStation(String startStation) {
		this.startStation = startStation;
	}

	public String getEndStation() {
		return endStation;
	}

	public void setEndStation(String endStation) {
		this.endStation = endStation;
	}
    //重新equal和hashCode方法
	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (o == null || getClass() != o.getClass()) return false;

		TrainLinePK that = (TrainLinePK) o;

		if (!endStation.equals(that.endStation)) return false;
		if (!startStation.equals(that.startStation)) return false;

		return true;
	}

	@Override
	public int hashCode() {
		int result = startStation.hashCode();
		result = 31 * result + endStation.hashCode();
		return result;
	}
}
```

```java
@Entity
@Table(name="trainLine")
public class TrainLine {
	@EmbeddedId
	private TrainLinePK id;
	private String name;

	public TrainLinePK getId() {
		return id;
	}

	public void setId(TrainLinePK id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}

```

    Test Code:

```java
@Test
	public void saveTrainLine() {
		EntityManagerFactory factory = Persistence.createEntityManagerFactory("sample");
		EntityManager em = factory.createEntityManager();
		em.getTransaction().begin();
		TrainLinePK id = new TrainLinePK();
		id.setStartStation("广州");
		id.setEndStation("漯河");
		TrainLine trainline = new TrainLine();
		trainline.setId(id);
		trainline.setName("回家的列车");
		em.persist(trainline);

		em.getTransaction().commit();
		em.close();
		factory.close();
	}
```
#【完】
----------------------------
附表： JPA的常见注解

|注解|	描述|
|:---|:----|
|`@Entity`|	声明类为实体或表。|
|`@Table`|	声明表名。|
|`@Basic`|	指定非约束明确的各个字段。|
|`@Embedded`|	指定类或它的值是一个可嵌入的类的实例的实体的属性。|
|`@Id`	|指定的类的属性，用于识别（一个表中的主键）|
|`@GeneratedValue`	|指定如何标识属性可以被初始化，例如自动，手动，或从序列表中获得的值。|
|`@Transient`	|指定的属性，它是不持久的，即，该值永远不会存储在数据库中。|
|`@Column`	|指定持久属性栏属性。|
|`@SequenceGenerator`|	指定在@GeneratedValue注解中指定的属性的值。它创建了一个序列。|
|`@TableGenerator`	|指定在@GeneratedValue批注指定属性的值发生器。它创造了的值生成的表。|
|`@AccessType`	|这种类型的注释用于设置访问类型。如果设置@AccessType（FIELD），然后进入FIELD明智的。如果设置@AccessType（PROPERTY），然后进入属性发生明智的。|
|`@JoinColumn`	|指定一个实体组织或实体的集合。这是用在多对一和一对多关联。|
|`@UniqueConstraint`|	指定的字段和用于主要或辅助表的唯一约束。|
|`@ColumnResult`|	参考使用select子句的SQL查询中的列名。|
|`@ManyToMany`	|定义了连接表之间的多对多一对多的关系。|
|`@ManyToOne`|	定义了连接表之间的多对一的关系。|
|`@OneToMany`	|定义了连接表之间存在一个一对多的关系。|
|`@OneToOne`|	定义了连接表之间有一个一对一的关系。|
|`@NamedQueries`|	指定命名查询的列表。|
|`@NamedQuery`|	指定使用静态名称的查询。|


  [1]: http://www.cnblogs.com/lich/tag/JPA/
  [2]: http://d.pcs.baidu.com/thumbnail/b02c7740f61876db017936b8f880272c?fid=2266416092-250528-52980154889024&time=1441508400&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-ssBnflshQ1tgamGljCOMdDJsvGM%3D&rt=sh&expires=2h&r=181469924&sharesign=unknown&size=c710_u500&quality=100