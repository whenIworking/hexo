# Timestamp时间戳边界处理

标签（空格分隔）： JAVA

---

在处理时间戳的问题的时候，首先需要了解时间戳是什么，如何计算。那么，***时间戳***实际上一个long型的数值，代表了距离**1970年1月1日上午8点0分0秒**的全部毫秒数。

这里格外需要注意的一点就是，是**上午8点0分0秒**，而不是从**0点0分0秒**算起。
如果忽略这一个细节，就会很容易犯错误。例如，判断两个时间戳是否是同一天。

如果两个时间戳所代表的时间距离1970年1月1日的天数相同，则表明两者同一天为真，否则为假。但是，考虑到时间戳为0的时间实际上表示的是**1970年1月1日上午8点0分0秒**，那么在进行运算的时候，就必须要补上这8个小时的误差。如下代码所示：

```java
static final long EIGHT_COMPLEMENT_MILLISECONDS = 8 * 60 * 60 * 1000L;
static final long MILLISECONDS_OF_DAY = 24 * 60 * 60 * 1000L;
public boolean isSameDay(long timestamp1,long timestamp2){
		return ((timestamp1+EIGHT_COMPLEMENT_MILLISECONDS) / MILLISECONDS_OF_DAY) 
		== ((timestamp2+EIGHT_COMPLEMENT_MILLISECONDS) / MILLISECONDS_OF_DAY);
}
```






