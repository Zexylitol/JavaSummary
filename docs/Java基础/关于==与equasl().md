# ==

== : 判断两个对象的地址是否相等

- 对于基本数据类型，比较的是值
- 对于引用数据类型，比较的是内存地址，即判断是不是指向同一个对象

# equals()

equals()：也是判断两个对象是否相等，**是需要开发者覆写的，让开发者自己定义满足什么条件的两个对象是相等的**，分两种情况：

- 类没有覆盖equals方法，此时，等价于"=="运算符，比较的是两个对象的地址
- 类覆盖了equals方法，此时，开发者自己定义满足什么条件的两个对象是相等的，一般来比较两个对象的内容是否相等

# String

- 对于**字符串变量**来说，使用“==”和“equals()”方法比较字符串时，其比较方法不同:

1. 	“==”比较两个变量本身的值，即两个对象在内存中的首地址

2. “equals()”比较字符串中所包含的内容是否相同(**String类重写了equals方法**)

   ```java
   // Java 8
   public boolean equals(Object anObject) {
           if (this == anObject) {
               return true;
           }
           if (anObject instanceof String) {
               String anotherString = (String)anObject;
               int n = value.length;
               if (n == anotherString.value.length) {
                   char v1[] = value;
                   char v2[] = anotherString.value;
                   int i = 0;
                   while (n-- != 0) {
                       if (v1[i] != v2[i])
                           return false;
                       i++;
                   }
                   return true;
               }
           }
           return false;
       }
   ```

- Object类中的equals方法是用来比较“地址”的

  ```java
  public boolean equals(Object obj) {
      return (this == obj);
  }
  ```

  