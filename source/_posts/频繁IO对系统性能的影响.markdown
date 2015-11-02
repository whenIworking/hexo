# 频繁IO对系统性能的影响

标签（空格分隔）： JAVA

---
最近在维护系统的时候，发现有个页面加载速度明显比其他页面的加载速度慢一些，遂逐行进行跟踪排查，最后发现一段操作数据库的代码，其形式如下：
```java
List<T> ts = **Dao.findBy**(query);
for(T t : ts){
    E e = *****Dao.findBy***(t.get**());
    ...
}
```
这里在循环中出现了频繁的数据库操作，共访问数据库`ts.size()+1`次。虽然，现在各种ORM框架都提供了对数据池的支持与较好的管理，省去了很多的打开连接关闭连接的操作，但是访问数据库的IO开销依然不得不纳入思考的范围。

*为了验证频繁的数据库操作所带来的IO开销是否会对系统的性能造成一定的影响，下面通过一个例子进行测试比较：*

**首先，假设有一个学校，学校有多个班级，每个班级都有一定数量的学生。现在学校领导想要知道某个年级某几个班级今年的学生的数量有多少。**

接下来，我们将通过编码来实现该功能。

| 开发环境|开发技术|
|:---:|:----:|
|IDEA,MySQL|hibernate,junit|


-------
##搭建项目的框架
首先，添加Hibernate配置文件hibernate.cfg.xml
```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="connection.url">
            jdbc:mysql://localhost:3306/hibernate?characterEncoding=utf-8
        </property>
        <property name="connection.driver_class">
            com.mysql.jdbc.Driver
        </property>
        <property name="connection.username">root </property>
        <property name="connection.password">123456</property>
        <!-- DB schema will be updated if needed -->
        <property name="hbm2ddl.auto">update</property>
        <property name="dialect">org.hibernate.dialect.MySQLDialect</property>
        <property name="show_sql">true</property>
        <property name="current_session_context_class">thread</property>

        <mapping class="com.zdw.hibernate.entity.ClassRoom"/>
        <mapping class="com.zdw.hibernate.entity.Student"/>
    </session-factory>
</hibernate-configuration>
```
定义实体类：`ClassRoom` 和 `Student`

```java
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

/**
 * @author zhoudongwei
 * @version 1.0
 * @since 1.0
 */

@Entity
@Table(name = "t_classroom")
public class ClassRoom {

	@Id
	@GeneratedValue(strategy= GenerationType.AUTO)
	private Integer id;

	@Column(length=255,nullable=true,name="class_name")
	private String className;

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public String getClassName() {
		return className;
	}

	public void setClassName(String className) {
		this.className = className;
	}
}
```
```java

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

/**
 * @author zhoudongwei
 * @version 1.0
 * @since 1.0
 */
@Entity
@Table(name="t_student")
public class Student {
	@Id
	@GeneratedValue(strategy= GenerationType.AUTO)
	private Integer id;
	@Column(length=255,nullable=true,name="name")
	private String name;
	@Column(length=255,nullable=true,name="class_id")
	private Integer classId;

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Integer getClassId() {
		return classId;
	}

	public void setClassId(Integer classId) {
		this.classId = classId;
	}
}
```
编写Hibernate工具类HibernateUtil
```java
package com.zdw.hibernate.utils;

import org.hibernate.SessionFactory;
import org.hibernate.boot.registry.StandardServiceRegistry;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.cfg.Configuration;
import org.hibernate.service.Service;
import org.hibernate.service.ServiceRegistry;
import org.hibernate.service.ServiceRegistryBuilder;

import java.io.Serializable;


/**
 * @author zhoudongwei
 * @version 1.0
 * @since 1.0
 */
public class HibernateUtil{

	private static final SessionFactory sessionFactory = buildSessionFactory();

	private static SessionFactory buildSessionFactory() {
		try {
			// Create the SessionFactory from hibernate.cfg.xml
			Configuration configuration = new Configuration().configure();
			StandardServiceRegistry serviceRegistry = new StandardServiceRegistryBuilder().applySettings(configuration.getProperties()).build();
			SessionFactory sessionFactory = configuration.buildSessionFactory(serviceRegistry);
			return sessionFactory;
		}
		catch (Throwable ex) {
			// Make sure you log the exception, as it might be swallowed
			System.err.println("Initial SessionFactory creation failed." + ex);
			throw new ExceptionInInitializerError(ex);
		}
	}

	public static SessionFactory getSessionFactory() {
		return sessionFactory;
	}
}
```
##编写测试类
为了测试我们的想法，首先需要先向数据库中添加数据。
编写测试类`ClassRoomAndStudentGenerator`，在数据库中随机添加一定量的数据，由于在实体类中没有定义ClassRoom和Student的关系映射，所以在设置某个学生的班级id的时候，要保证不要设置一个不存在的班级id。
```java
package com.zdw.hibernate.dao.test;

import com.seewo.hibernate.entity.ClassRoom;
import com.seewo.hibernate.entity.HibernateSessionFactory;
import com.seewo.hibernate.entity.HibernateUtil;
import com.seewo.hibernate.entity.Student;
import org.hibernate.Session;
import org.junit.Test;

import org.hibernate.Query;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import org.junit.runners.MethodSorters;
import org.junit.FixMethodOrder;

/**
 * @author zhoudongwei
 * @version 1.0
 * @since 1.0
 */
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class ClassRoomAndStudentGenerator {

    private static final int CLASS_NUM = 10;
    private static final int STUDENT_NUM = 100;


	@Test
	public void addClassroom(){

		Session session = HibernateUtil.getSessionFactory().openSession();
		session.beginTransaction();
		for (int i = 0; i < CLASS_NUM ; i++) {
			ClassRoom classRoom = new ClassRoom();
			classRoom.setClassName("实验"+i+"班");
			session.persist(classRoom);
			System.out.println(classRoom.getId());

		}
		session.getTransaction().commit();
		session.close();

	}

	@Test
	public void addStudent(){
		Random random = new Random();
		Session session = HibernateUtil.getSessionFactory().openSession();
		session.beginTransaction();

		int classId = 0;
		for (int i = 0; i < STUDENT_NUM; i++) {
			Student student = new Student();
			classId = random.nextInt(CLASS_NUM);
			student.setClassId(classId);
			student.setName("student_"+classId+"_"+i);
			session.persist(student);
		}
		session.getTransaction().commit();
		session.close();
	}

}
```
首先我们编写本文一开始提到的`loop`方法，这种方法的想法就是：先查询出这些班级的ids，然后遍历这些id，查询班级id为特定值的学生个数。思路很简单，代码也很好编写，但是需要访问`n+1`次数据库。
```java
@Test
@Test
	public void loopCount(){
		long start = System.currentTimeMillis();
		Session session = HibernateUtil.getSessionFactory().openSession();
		session.beginTransaction();
		List<ClassRoom> classRooms = session.createQuery("from ClassRoom ").list();
		for (ClassRoom classRoom : classRooms) {
			Integer classId = classRoom.getId();
			Query query = session.createQuery("select count(*) from Student where classId = :classId");
			query.setParameter("classId", classId);
			List res = query.list();
		}
		session.getTransaction().commit();
		session.close();
		long end = System.currentTimeMillis();
		System.out.println(end - start);

	}
```

    我在方法的前后统计其执行的时间，来与不同的实现方式进行对比。

第二种思路：全连接。从两个表中选择`班级id=学生班级id`然后`group by 班级id`，最后`count(*)`返回结果。只需要访问一次数据库。
```java
@Test
	public void innerJoinCount(){
		long start = System.currentTimeMillis();
		Session session = HibernateUtil.getSessionFactory().openSession();
		session.beginTransaction();

		Query query = session.createQuery("select t.id,t.className,count(t.id) from ClassRoom t ,Student s where t.id = s.classId group by t.id");
		List<Object[]> res = query.list();

		session.getTransaction().commit();
		session.close();
		long end = System.currentTimeMillis();
		System.out.println(end - start);
}
```

第三种思路：利用`in`进行查询。首先，查询出指定的班级ids,利用`in`字句进行查询，返回结果。访问数据库两次。
```java
@Test
	public void batchCount(){
		long start = System.currentTimeMillis();
		Session session = HibernateUtil.getSessionFactory().openSession();
		session.beginTransaction();

		List<ClassRoom> classRooms = session.createQuery("from ClassRoom ").list();
		List<Integer> classIds = new ArrayList<Integer>(classRooms.size());
		for (ClassRoom classRoom : classRooms) {
			classIds.add(classRoom.getId());
		}

		Query query = session.createQuery("select classId,count(*) from Student where classId in (:classIds) group by classId");
		query.setParameterList("classIds", classIds);
		List<Object> res =query.list();

		session.getTransaction().commit();
		session.close();
		long end = System.currentTimeMillis();
		System.out.println(end - start);

	}
```

当改变

     private static final int CLASS_NUM = 10; //100,1000
     private static final int STUDENT_NUM = 100; //1000,10000
的值就会有不同的结果出现。

当`CLASS_NUM=10，STUDENT_NUM=100`时，

    loopCount : 1352 ms
    innberJoinCount : 1345 ms
    batchCount : 1368 ms
    
当`CLASS_NUM=100，STUDENT_NUM=10000`时，

    loopCount : 1780ms
    innberJoinCount : 1346ms
    batchCount : 1434ms

当`CLASS_NUM=1000，STUDENT_NUM=100000`时，

    loopCount : 25615 ms
    innberJoinCount : 1472 ms
    batchCount : 1605 ms

当数据量增大的时候，可以明显的看到，每多一次数据库访问，就会额外的多带来一定的开销。如果一次查询就能完成的工作不建议分为多次查询。

