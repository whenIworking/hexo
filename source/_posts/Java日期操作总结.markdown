# Java日期操作总结

标签（空格分隔）： JAVA

---

Java日期的操作在日常的应用开发中使用非常普遍，然而每次使用都不得不去查文档，效率很低。工欲善其事必先利其器，所以需要好好总结一番。
Java的日期的常用操作其实主要有以下几种：

- 字符串转为日期对象
- 日期对象转换为字符串
- 时间戳转换为日期对象
- 日期对象不同的格式转换
- Calendar类的使用

`DateFormat` 类是一个用来格式化日期或者时间的抽象类。`DateFormat`提供了很多根据本地化或者指定类型 的格式化*类方法*。
`SimpleDateFormat`是`DateFormat`的子类，是一个以国别敏感的方式格式化和分析数据的具体类。 它允许格式化 (`date -> text`)、语法分析 (`text -> date`)和标准化。

`SimpleDateFormat` 允许以为日期-时间格式化选择任何用户指定的方式启动。 但是，希望用 `DateFormat` 中的 `getTimeInstance`、 `getDateInstance` 或 `getDateTimeInstance` 创建一个日期-时间格式化程序。 每个类方法返回一个以缺省格式化方式初始化的日期／时间格式化程序。 可以根据需要用 `applyPattern` 方法修改格式化方式。 

>  `SimpleDateFormat` 函数语法：
 **- G 年代标志符
 - y 年
 - M 月
 - d 日
 - h 时 在上午或下午 (1~12)
 - H 时 在一天中 (0~23)
 - m 分
 - s 秒
 - S 毫秒
 - E 星期
 - D 一年中的第几天
 - F 一月中第几个星期几
 - w 一年中第几个星期
 - W 一月中第几个星期
 - a 上午 / 下午 标记符
 - k 时 在一天中 (1~24)
 - K 时 在上午或下午 (0~11)  z 时区**
 
 
## 字符串转为日期对象
**主要方法**
```java 
 public Date parse(String source)  throws ParseException
```

> Parses text from the beginning of the given string to produce a date.
> The method may not use the entire text of the given string.

用法示例：
```java
String str = "2015-8-23 15:10:34";
//定义SimpleFormat格式
SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH-mm-ss");
Date date = format.parse(str); //这里有异常抛出，ParseException
```

## 日期对象转为字符串
**主要方法**
```java
public final String format(Date date)
```

> Formats a Date into a date/time string.

```java
Date date = new Date();
//定义SimpleFormat格式
SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH-mm-ss");
String dateStr = format.foramt(date);
```

##时间戳与日期的转换
**主要方法**

Date类：
```java
public long getTime()
```

> 返回一个时间戳：某个时间距1970年1月1日0时0分0秒的总的毫秒数

SimpleDateFormat类：
```java
public final String format(Object obj)
public Date parse(String source)  throws ParseException
```

> 首先转换为format的格式的日期字符串，然后利用parse转换为日期对象

```java
long timestamp = 445555555；
//定义SimpleFormat格式
SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH-mm-ss");
String dateStr = format.foramt(timestamp);
Date date = format.parse(str); //这里有异常抛出，ParseException
```
```java
  //Date或者String转化为时间戳
  SimpleDateFormat format =  newSimpleDateFormat("yyyy-MM-dd HH:mm:ss");
  String time="1970-01-06 11:45:55";
  Date date = format.parse(time);
  System.out.print("Format To times:"+date.getTime());
```

##小结

**注意：  定义`SimpleDateFormat时newSimpleDateFormat("yyyy-MM-dd HH:mm:ss");` 里面字符串头尾不能有空格，有空格那么用转换时对应的时间格式也要有空格（两者是对应的），如下所示：**
```java
//Date或者String转化为时间戳
      SimpleDateFormat format =  newSimpleDateFormat(" yyyy-MM-dd HH:mm:ss ");
      String time="1970-01-06 11:45:55";
      Date date = format.parse(time);
      System.out.print("Format To times:"+date.getTime());
      
```
运行结果出错：

    Exception in thread "main"java.text.ParseException: Unparseable date: "1970-01-06 11:45:55"
改正，添加对应的空格：
```java
      SimpleDateFormat format =  newSimpleDateFormat(" yyyy-MM-dd HH:mm:ss ");
      String time=" 1970-01-06 11:45:55 ";//注：改正后这里前后也加了空格
      Date date = format.parse(time);
      System.out.print("Format To times:"+date.getTime());
```
 运行结果正常：

    Format To times:445555000

#Calendar类用法

`Calendar` 类是Util包下的一个用来方便管理日期的操作类。

`Calendar` 可以用 `DateFormat`的 `getCalendar` 方法获得，也时可以通过 `Calendar`的类方法`getInstance`方法取得。

`Calendar` 类的常见用法就是时间，日期时间的累加和累减以及日期的循环操作。

示例：获取一段时间内的所有日期完成循环操作
```java
private  void findDates(String startTime,String endTime) {
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        Date dBegin= null;
        Date dEnd = null;
        try {
            dBegin = format.parse(startTime);
            dEnd = format.parse(endTime);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        Calendar calBegin = Calendar.getInstance();
        // 使用给定的 Date 设置此 Calendar 的时间
        calBegin.setTime(dBegin);
        // 测试此日期是否在指定日期之后
        while (dEnd.after(calBegin.getTime())) {
            //业务代码
            // 根据日历的规则，为给定的日历字段添加或减去指定的时间量
            calBegin.add(Calendar.DAY_OF_MONTH, 1);
        }
        //业务代码
    }
```

