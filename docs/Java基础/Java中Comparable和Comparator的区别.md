<!-- GFM-TOC -->

- [引言](#引言)
- [自然排序：java.lang.Comparable](#自然排序：javalangComparable)
- [定制排序：java.util.Comparator](#定制排序：javautilComparator)
- [总结](#总结)
- [Reference](#Reference)

<!-- GFM-TOC -->

# 引言

- 在Java中经常会涉及到对象数组的排序问题，那么就涉及到对象之间的比较问题  
- Java中的对象，正常情况下，只能进行比较：`==` 或 `!=`，不能使用 `>` 或 `< `，但是在开发场景中，需要对多个对象进行排序，就需要比较对象的大小

<center><img src="https://i.loli.net/2021/05/03/OPjK8vRFZ5J2Ex4.png"/></center>

- `Comparable`与`Comparator`是Java提供的两种比较机制，二者都是用来实现对象的比较、排序
  - <font color="red">自然排序：java.lang.Comparable</font>
  - <font color="red">定制排序：java.util.Comparator</font>

# 自然排序：java.lang.Comparable

- `Comparable`接口强行对实现它的每个类的对象进行整体排序。这种排序被称为类的自然排序。

- 实现`Comparable`的类必须实现`compareTo(Object obj)`方法，两个对象即通过`compareTo(Object obj)`方法的返回值来比较大小。
  - **如果当前对象this大于形参对象obj，则返回正整数**
  - **如果当前对象this小于形参对象obj，则返回负整数**
  - **如果当前对象this等于形参对象obj，则返回零**

- **实现`Comparable`接口的对象列表(和数组)可以通过`Collections.sort`或`Arrays.sort`进行自动排序。实现此接口的对象可以用作有序映射中的键或有序集合中的元素，无需指定比较器。**

- 对于类`C`的每一个`e1`和`e2`来说，当且仅当`e1.compareTo(e2)==0`与`e1.equals(e2)`具有相同的`boolean`值时，类`C`的自然排序才叫做与`equals`一致。建议(虽然不是必需的)**最好使自然排序与equals一致。**

- `Comparable`的典型实现: (<font color="red">默认都是从小到大排列的</font>)
  
  - `String`:按照字符串中字符的Unicode值进行比较
  
    - ```java
      public int compareTo(String anotherString) {
          int len1 = value.length;
          int len2 = anotherString.value.length;
          int lim = Math.min(len1, len2);
          char v1[] = value;
          char v2[] = anotherString.value;
      
          int k = 0;
          while (k < lim) {
              char c1 = v1[k];
              char c2 = v2[k];
              if (c1 != c2) {
                  return c1 - c2;
              }
              k++;
          }
          return len1 - len2;
      }
      ```
  
  - `Character`:按照字符的Unicode值来进行比较
  
    - ```java
          public int compareTo(Character anotherCharacter) {
              return compare(this.value, anotherCharacter.value);
          }
      
          /**
           * Compares two {@code char} values numerically.
           * The value returned is identical to what would be returned by:
           * <pre>
           *    Character.valueOf(x).compareTo(Character.valueOf(y))
           * </pre>
           *
           * @param  x the first {@code char} to compare
           * @param  y the second {@code char} to compare
           * @return the value {@code 0} if {@code x == y};
           *         a value less than {@code 0} if {@code x < y}; and
           *         a value greater than {@code 0} if {@code x > y}
           * @since 1.7
           */
          public static int compare(char x, char y) {
              return x - y;
          }
      ```
  
  - 数值类型对应的包装类以及`BigInteger`、`BigDecimal`: 按照它们对应的数值大小进行比较
  
  - `Boolean`:true对应的包装类实例大于false对应的包装类实例
  
  - `Date`、`Time`等:后面的日期时间比前面的日期时间大

```java
public class Domain implements Comparable<Domain> {
   private String str;
   public Domain(String str) {
       this.str = str;
   }

   public int compareTo(Object o) {
       if (o instanceof Domain) {
           Domain domain = (Domain) o;
           if (this.str.compareTo(domain.str) > 0)
               return 1;
           else if (this.str.compareTo(domain.str) == 0)
               return 0;
           else 
               return -1;
       }
       throw new RuntimeException("输入的数据类型不一致");
   }

   public String getStr() {
       return str;
   }
}

public static void main(String[] args) {
       Domain d1 = new Domain("c");
       Domain d2 = new Domain("c");
       Domain d3 = new Domain("b");
       Domain d4 = new Domain("d");
       System.out.println(d1.compareTo(d2));
       System.out.println(d1.compareTo(d3));
       System.out.println(d1.compareTo(d4));
}
```

# 定制排序：java.util.Comparator

- <font color="blue">当元素的类型没有实现`java.lang.Comparable`接口而又不方便修改代码，或者实现了`java.lang.Comparable`接口的排序规则不适合当前的操作，那么可以考虑使用`Comparator`的对象来排序</font>，强行对多个对象进行整体排序的比较。

- 重写`compare(Object o1 ,Object o2)`方法，比较o1和o2的大小:
  - **方法返回正整数，则表示o1大于o2**
  - **返回0，表示相等**
  - **返回负整数，表示o1小于o2**

- 可以将`Comparator`传递给`sort` 方法( 如`Collections. sort`或`Arrays .sort`)从而允许在排序顺序上实现精确控制。

- 还可以使用Comparator来控制某些数据结构(如有序set或有序映射)的顺序，或者为那些没有自然顺序的对象collection提供排序。

```java
public static void main(String[] args) {
    Comparator<Integer> comparator = new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            return o1.compareTo(o2);
        }
    };

    Integer[] integers = {-1,1,2,0,-2,3};
    Arrays.sort(integers, comparator);
    List<Integer> list = new ArrayList<>();
    Collections.sort(list, comparator);
    TreeSet<Integer> treeSet = new TreeSet<>(comparator);
}
```

# 总结

- **如果实现类没有实现`Comparable`接口**，又想对两个类进行比较（或者实现类实现了`Comparable`接口，但是对`compareTo`方法内的比较算法不满意），**那么可以实现`Comparator`接口，自定义一个比较器，写比较算法**

```java
class myObject {
    String str = "";
    public myObject(String str) {
        this.str = str;
    }
}
public static void main(String[] args) {
    List<myObject> list = new ArrayList<>();
    list.add(new myObject("1"));
    list.add(new myObject("2"));
    Collections.sort(list, new Comparator<myObject>() {
        @Override
        public int compare(myObject o1, myObject o2) {
            return o1.str.compareTo(o2.str);
        }
    });
}
```

- **实现`Comparable`接口的方式比实现`Comparator`接口的耦合性要强一些**，如果要修改比较算法，要修改`Comparable`接口的实现类，而实现`Comparator`的类是在外部进行比较的，不需要对实现类有任何修改

- 对于一些普通的数据类型（比如` String, Integer, Double…`），它们默认实现了`Comparable `接口，实现了 `compareTo` 方法，可以直接使用。
- 而对于一些自定义类，它们可能在不同情况下需要实现不同的比较策略，可以新创建 `Comparator` 接口，然后使用特定的`Comparator` 实现进行比较

- `Comparator`位于`java.util`包下，而`Comparable`位于`java.lang`包下

- `Comparable`接口保证`Comparable`接口实现类的对象在任何位置都可以比较大小，`Comparator`接口属于临时性的比较

# Reference

- [尚硅谷-Java基础教程](https://www.bilibili.com/video/BV1Kb411W75N?p=490)

- [Java中Comparable和Comparator的区别](https://mp.weixin.qq.com/s/SvPOOQBMzdBlgrYLWkl38g)