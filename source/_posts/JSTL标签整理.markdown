# JSTL标签整理

标签（空格分隔）： JAVA

---

为了在JSP页面中使用`JSTL`标签，那么需要在JSP页面中引入 `JSTL` 声明和启用`EL`的声明，如下所示：
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```
###`c:out`
out.jsp：
```jsp
action parameters is : <c:out value="${param.action}"></c:out>
```
访问：
`localhost：8080/out.jsp?action=add`
结果：
`action parameters is : add`

###`c:if`
if.jsp:
```jsp
<c:if test="${param.action=='add'}">
    <table>
        <tr>
            <td>账号</td>
            <td><input type="text" name="login"/></td>
        </tr>
        <tr>
            <td>真实姓名</td>
            <td><input type="text" name="name"/></td>
        </tr>
    </table>
</c:if>
```
访问：
`localhost：8080/out.jsp?action=add`

但是 `if` 标签没有 `else` 的功能，如果要实现类似于 `java` 中 `if...else` 的功能，需要使用 `choose` 标签。

choose.jsp:
```jsp
<c:choose>
    <c:when test="${param.action}" >
        when 标签的输出
    </c:when>
    <c:otherwise>
        otherwise 标签的输出
    </c:otherwise>
</c:choose>
```

