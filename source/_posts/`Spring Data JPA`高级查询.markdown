# `Spring Data JPA`高级查询

标签（空格分隔）： JAVA

---

- 使用`Criteria`查询
- `JpaSpecificationExecutor` 接口的使用
- `@Query` 和 `@NamedQuery` 的使用

# 使用 `Criteria` 查询

`Criteria` 对不同的函数进行了切分，然后通过对这些函数进行组合来完成一些较为复杂的查询。

    JPQL ： select u from User u where u.old > 20
    
```java
public List<User> findAll(){
    CriteriaBuilder cb = entirymanager.getCriteriaBuilder();
    CriteriaQuery cq = cb.createQuery();
    Root<User> root = cq.from(User.class);
    cq.select(root);
    Predicate pre = cb.greaterThan(root.get("old").as(Integer.class),20);
    cq.where(pre);
    
    Query query = entityManager.createQuery(cq);
    List<User> users = query.getResultList();
    return users;
}
```
    JPQL ： select u.old,u.name from User u where u.old > 20
```java
//如果只查询某几个字段，那么返回的结果就不能是对象而是一个对象数组
public List<Object[]> findAll(){
    CriteriaBuilder cb = entirymanager.getCriteriaBuilder();
    CriteriaQuery cq = cb.createQuery();
    Root<User> root = cq.from(User.class);
    //cq.select(root);
    //cq.select(root.get("old"); 如果只查询一个字段，可以使用select
    cq.multiselect(root.get("old"),root.get("name"));//查询多个字段使用multiselect
    Predicate pre = cb.greaterThan(root.get("old").as(Integer.class),cb.parameter(Integer.class,"old"));
    cq.where(pre);
    
    Query query = entityManager.createQuery(cq);
    query.setParameter("old",20);
    List<Object[]> users = query.getResultList();
    return users;
}

//如果想要查询多个字段，但是又想返回对象那就需要使用construct进行构造
public List<User> findAll(){
    CriteriaBuilder cb = entirymanager.getCriteriaBuilder();
    CriteriaQuery cq = cb.createQuery();
    Root<User> root = cq.from(User.class);
    cq.select(cb.construct(User.class,root.get("old"),root.get("name)));
    cq.multiselect(root.get("old"),root.get("name"));
    Predicate pre = cb.greaterThan(root.get("old").as(Integer.class),cb.parameter(Integer.class,"old"));
    cq.where(pre);
    
    Query query = entityManager.createQuery(cq);
    query.setParameter("old",20);
    List<User> users = query.getResultList();
    return users;
}
```

如果要进行`join`操作，就需要使用 `Root` 的 `join` 操作





