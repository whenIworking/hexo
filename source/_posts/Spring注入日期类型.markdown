# Spring注入日期类型

标签（空格分隔）： JAVA

---
```java
import java.util.Date;
 
public class Employee {
 
    private int empId;
    private String name;
    private String role;
    private Date doj;
     
    public int getEmpId() {
        return empId;
    }
    public void setEmpId(int empId) {
        this.empId = empId;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getRole() {
        return role;
    }
    public void setRole(String role) {
        this.role = role;
    }
    public Date getDoj() {
        return doj;
    }
    public void setDoj(Date doj) {
        this.doj = doj;
    }
    public void testMe(){
        System.out.println("Employee: Doj: "+this.doj);
    }
     
}
```
直接使用已经存在的SimpleDateFormat作为日期类型的格式化工具，对日期类型数据进行注入操作，配置如下
```xml
 <bean id="dateFormater" class="java.text.SimpleDateFormat">
        <constructor-arg value="dd-MM-yyyy" />
</bean>
<bean id="myEmployee" class="com.zdw.entity.Employee">
    <property name="doj">
        <bean factory-bean="dateFormater" factory-method="parse">
            <constructor-arg value="23-03-1982" />
        </bean>
    </property>
</bean>
```
使用`CustomDateEditor`自定义日期处理方式，然后将其注册到`CustomEditorConfigurer`上，在注入日期类型的时候，Spring就会按照我们自定义的方式去注入日期类型。
```xml
<bean id="dateEditor" class="org.springframework.beans.propertyeditors.CustomDateEditor">             <constructor-arg> 
        <bean class="java.text.SimpleDateFormat"> 
            <constructor-arg value="yyyy-MM-dd" /> 
        </bean> 
    </constructor-arg> 
    <constructor-arg value="true" />
</bean> 
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer"> 
    <property name="customEditors"> 
        <map> 
            <entry key="java.util.Date"> 
                <ref local="dateEditor" /> 
            </entry> 
        </map> 
    </property> 
</bean>
<bean id="myEmployee" class="com.zdw.entity.Employee"> 
    <property name="doj" value="23-03-1982" /> 
</bean> 
```



