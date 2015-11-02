# Java反射：字符串轻松变对象

标签（空格分隔）： JAVA

---

需求场景：
 *一个由键值对组成的字符串，需要将该字符串转换为对象。字符串形式如下：*
```js
    "username:dodoro|sex:male|address:KeZhu Road|phone:153864422"
```
首先，声明一个对应字段的POJO类，User如下：
```java
public class User{
    String username;
    String sex;
    String address;
    String phone;
    //省略getter和setter
}   
```

直观的印象是在转换的过程中，首先将字符串拆分出来，然后逐一判断，调用setter设置，代码片段如下:
```java
User user = new User();
String[] keyValues = str.split("\\|");
for(String keyValue : keyValues){
    String[] field = keyValue.split(":");
    if(field.length == 2){
        if(field[0].equals("username"){ user.setUsername(field[1]) ;}   
        //....类推，省略
    }
}
```
那么在这里就需要重复的多写很多的判断语句，如果POJO类的属性很多，有几十个，那就需要写几十个`if`逐一判断，那么在这里可以利用`JAVA`的反射技巧轻松的完成该任务。

 - 首先，根据字符串名字获取POJO类的属性名
 - 其次，设置属性的Access属性为true 
 - 最后，调用setter方法

代码如下：
```java
User user = new User();
String[] fields = str.split("\\|");
for(String field : fields){
	keyValue = field.split(":");
	if(keyValue.length == 2) {
		try {
			Field f = UnitCheckVo.class.getDeclaredField(keyValue[0]);
			f.setAccessible(true);
			f.set(unitCheckVo, keyValue[1]);
		} catch (NoSuchFieldException e) {
			log.info("No field " + keyValue[0] + " exists!");
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			log.info("Set property for " + keyValue[0] + " denied!");
			e.printStackTrace();
		}
	}

}

```




