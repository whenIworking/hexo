title: Annotation的工作原理
date: 2015-09-10
tags:
- JAVA
categories:
- JAVA

---

Annotation的出现真的是非常惊艳，能把许多繁琐的工作化整为零。那么Annotation到底是怎么工作的呢，那么就亲自编写一个Annotation程序尝试一下就比较清楚了。

下面就演示一个简单的Annotation到底是怎么起作用的:

先定义一个叫Hello的Annotation注解



下面就演示一个简单的Annotation到底是怎么起作用的:

先定义一个叫Hello的Annotation注解
```java
 @Retention(RetentionPolicy.RUNTIME)//表示运行时能获取参数
 @Target({ElementType.METHOD})//表示只能作用在方法上面
 public @interface Hello {
     String name() default "hello world!";//能设置默认值
 }
 ```
但是仅仅是这个注解是没什么作用的，还必须有解析器来解析她，下面再来写一个解析器Parser.java:

```java
public class Parser {
    public static void parser(Object obj){
        Method[] ms=obj.getClass().getMethods();//取得对象的所有方法
        for(Method m:ms){//遍历方法
            if(m.isAnnotationPresent(Hello.class)){//判断方法是否有Hello注解
                Hello hel=m.getAnnotation(Hello.class);//取得Hello注解中属性的值
                System.out.println("before……"+hel.name());
                try {
                    m.invoke(obj, null);//唤醒函数
                } catch (Exception e) {
                    e.printStackTrace();
                } 
                System.out.println("after……"+hel.name());
            }
        }
    }
}
```
这样一个完整的annotation注解就完成了，下面来测试一下，先写一个目标类Sample.java：

```java
public class Sample {
        private String something;
        public String getSomething() {
            return something;
        }
        public void setSomething(String something) {
            this.something = something;
        }
        @Hello //在此处使用注解
        public void func(){
            System.out.println(something);
        }
}
```
最后运行测试：

```java
public class Test {
    public static void main(String[] args) {
        Sample sample=new Sample();
        sample.setSomething("some functions");
        Parser.parser(sample); //调用解析器进行解析分析
    }
    }
```
控制台打印结果：
```java
before……hello world!
some functions
after……hello world!
```
Spring中运用Annotation控制事务其实是一样的道理，所以Java的反射机制真的是很强大啊
