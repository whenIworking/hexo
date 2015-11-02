# NULL值处理哪家强之 net.sf.json.JSONObject 和 com.alibaba.fastjson.JSONObject 

标签（空格分隔）： JAVA

---


JSON作为一个轻量级的文本数据交换格式常常用于web后台的数据封装和传输。JSON及其工具包给开发带来非常多的好处，提高了开发效率。然而，世间总是免不了存在一些坑，吾辈需笑看人生路，潜心研码。NULL值作为一个特殊情况，在处理的时候尤其需要小心处理。不幸的是，随着我们使用的工具类的作者的不同，对NULL的处理也有不同。此处，就扒一扒如题两家JSONObject工具类对NULL的处理。

问题来了，这两家对NULL是如何处理的，示例代码如下：
```java
String resp = "{" +
				"    \"id\": \"123456fdae23\",\n" +
				"    \"userName\": \"dodoro\",\n" +
				"    \"cnName\": \"清风明月多多爱工作\",\n" +
				"    \"anull\": null,\n" +
				"    \"anothernull\": null,\n" +
				"}";

		JSONObject jsonObject = JSONObject.fromObject(resp);
		if(jsonObject.getString("anull").equals("null")){
			System.out.println("null became a string");
		}

		com.alibaba.fastjson.JSONObject userInfo = com.alibaba.fastjson.JSONObject.parseObject(resp);
		if(userInfo.getString("anull") == null){
			System.out.println("alibaba NB, null is a null");
		}
		System.out.println(jsonObject);
```
运行的测试结果如下：
```
null became a string
alibaba NB, null is a null
```
结果很明显，`net.sf.json.JSONObject`将null转换成了一个字符串`"null"`，`com.alibaba.fastjson`则是将null转换为了`null`，两者是有不同的。

首先，`net.sf.json.JSONObject` 提供了一个将其他类型的数据转换为转换为`JSONObject`的方法`fromObject()`

```java
public static JSONObject fromObject(Object object) {
        return fromObject(object, new JsonConfig());
}
public static JSONObject fromObject(Object object, JsonConfig jsonConfig) {
        if(object != null && !JSONUtils.isNull(object)) {
            if(object instanceof Enum) { //是否是枚举类型
                throw new JSONException("\'object\' is an Enum. Use JSONArray instead");
            } else if(!(object instanceof Annotation) && (object == null || !object.getClass().isAnnotation())) { //是否是注解类型
                if(object instanceof JSONObject) { //是不是JSONObject
                    return _fromJSONObject((JSONObject)object, jsonConfig);
                } else if(object instanceof DynaBean) {
                    return _fromDynaBean((DynaBean)object, jsonConfig);
                } else if(object instanceof JSONTokener) {
                    return _fromJSONTokener((JSONTokener)object, jsonConfig);
                } else if(object instanceof JSONString) { //是不是符合JSON格式的字符串
                    return _fromJSONString((JSONString)object, jsonConfig);
                } else if(object instanceof Map) { //是不是Map
                    return _fromMap((Map)object, jsonConfig);
                } else if(object instanceof String) { //是不是字符串
                    return _fromString((String)object, jsonConfig);
                } else if(!JSONUtils.isNumber(object) && !JSONUtils.isBoolean(object) && !JSONUtils.isString(object)) {
                    if(JSONUtils.isArray(object)) { //是不是数组
                        throw new JSONException("\'object\' is an array. Use JSONArray instead");
                    } else {
                        return _fromBean(object, jsonConfig); //从对象转换为JSONObject，需要满足Bean的getter和setter
                    }
                } else {
                    return new JSONObject();
                }
            } else {
                throw new JSONException("\'object\' is an Annotation.");
            }
        } else {
            return new JSONObject(true);
        }
    }
```


从其源代码可以看出`net.sf.json`功能强大，接口简洁，然而主要看的并不是这里,而是它的`_fromJSONString((JSONString)object, jsonConfig);` 方法。
```java
 private static JSONObject _fromString(String str, JsonConfig jsonConfig) {
        if(str != null && !"null".equals(str)) {
            return _fromJSONTokener(new JSONTokener(str), jsonConfig);
        } else {
            fireObjectStartEvent(jsonConfig);
            fireObjectEndEvent(jsonConfig);
            return new JSONObject(true);
        }
    }
    
    private static JSONObject _fromJSONTokener(JSONTokener tokener, JsonConfig jsonConfig) {
        try {
            if(tokener.matches("null.*")) {
                fireObjectStartEvent(jsonConfig);
                fireObjectEndEvent(jsonConfig);
                return new JSONObject(true);
            } else if(tokener.nextClean() != 123) {
                throw tokener.syntaxError("A JSONObject text must begin with \'{\'");
            } else {
                fireObjectStartEvent(jsonConfig);
                Collection exclusions = jsonConfig.getMergedExcludes();
                PropertyFilter jsonPropertyFilter = jsonConfig.getJsonPropertyFilter();
                JSONObject jsonObject = new JSONObject();

                while(true) {
                    char jsone = tokener.nextClean();
                    switch(jsone) {
                    case '\u0000':
                        throw tokener.syntaxError("A JSONObject text must end with \'}\'");
                    case '}':
                        fireObjectEndEvent(jsonConfig);
                        return jsonObject;
                    default:
                        tokener.back();
                        String key = tokener.nextValue(jsonConfig).toString(); //此处取得一个token
                        jsone = tokener.nextClean();
                        if(jsone == 61) {
                            if(tokener.next() != 62) {
                                tokener.back();
                            }
                        } else if(jsone != 58) {
                            throw tokener.syntaxError("Expected a \':\' after a key");
                        }

                        char peek = tokener.peek();
                        boolean quoted = peek == 34 || peek == 39;
                        Object v = tokener.nextValue(jsonConfig); //获取当前key的value
                        if(!quoted && JSONUtils.isFunctionHeader(v)) {
                            String params = JSONUtils.getFunctionParams((String)v); //转换为String袅
                        //省略.....
    }
```

 `nextValue`方法是`JSONTokener`类的方法，代码如下：

```java
     public Object nextValue(JsonConfig jsonConfig) {
        char c = this.nextClean();
        switch(c) {
        case '\"':
        case '\'':
            return this.nextString(c);
        case '[':
            this.back();
            return JSONArray.fromObject(this, jsonConfig);
        case '{':
            this.back();
            return JSONObject.fromObject(this, jsonConfig);
        default:
            StringBuffer sb = new StringBuffer();

            char b;
            for(b = c; c >= 32 && ",:]}/\\\"[{;=#".indexOf(c) < 0; c = this.next()) {
                sb.append(c);
            }

            this.back();
            String s = sb.toString().trim();
            if(s.equals("")) {
                throw this.syntaxError("Missing value.");
            } else if(s.equalsIgnoreCase("true")) {
                return Boolean.TRUE;
            } else if(s.equalsIgnoreCase("false")) {
                return Boolean.FALSE;
            } else if(!s.equals("null") && (!jsonConfig.isJavascriptCompliant() || !s.equals("undefined"))) {
                if((b < 48 || b > 57) && b != 46 && b != 45 && b != 43) {
                    if(!JSONUtils.isFunctionHeader(s) && !JSONUtils.isFunction(s)) {
                        switch(this.peek()) {
                        case ',':
                        case '[':
                        case ']':
                        case '{':
                        case '}':
                            throw new JSONException("Unquotted string \'" + s + "\'");
                        default:
                            return s;
                        }
                    } else {
                        return s;
                    }
                } else {
                    if(b == 48) {
                        if(s.length() > 2 && (s.charAt(1) == 120 || s.charAt(1) == 88)) {
                            try {
                                return new Integer(Integer.parseInt(s.substring(2), 16));
                            } catch (Exception var13) {
                                ;
                            }
                        } else {
                            try {
                                return new Integer(Integer.parseInt(s, 8));
                            } catch (Exception var12) {
                                ;
                            }
                        }
                    }

                    try {
                        return new Integer(s);
                    } catch (Exception var11) {
                        try {
                            return new Long(s);
                        } catch (Exception var10) {
                            try {
                                return new Double(s);
                            } catch (Exception var9) {
                                return s;
                            }
                        }
                    }
                }
            } else {
                return JSONNull.getInstance(); //返回一个JSONNUll对象
            }
        }
    }
    
    
    public final class JSONNull implements JSON {

    public String toString() {
        return "null"; //返回一个“null”字符串
    }
    public String toString(int indentFactor) {
        return this.toString();
    }
}
```
至此可以看到`net.sf.json`将NULL转换为一个“null” 字符串处理。如果直接对存在null值得JSON对象使用`getString`进行操作，那么就容易出现使用`getString("anull") == null`的操作，而`fastjson`则是将null直接转换为一个`null`对象，所以可以直接使用`getString("anull") == null`进行比较。

万幸的是，如果我们通过JSON的方法将JSON对象转换为封装相同字段的`JAVA`对象，无论是哪种`JSON`工具包，最后都会把NULL转换为`null`。

**所以建议后台获取的JSON对象，如果想要使用，最好先转换为JAVABean来避免不同工具包之间的差异所带来的异常故障。**




