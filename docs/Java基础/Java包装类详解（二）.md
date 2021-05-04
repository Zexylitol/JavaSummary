
# 面试相关问题

## 1. 下面这段代码的输出结果是什么？

```java
public class Main {
    public static void main(String[] args) {
         
        Integer i1 = 100;
        Integer i2 = 100;
        Integer i3 = 200;
        Integer i4 = 200;
         
        System.out.println(i1==i2);  
        System.out.println(i3==i4);  
    }
}
```
答案：

```java
true
false
```
输出结果表明`i1`和`i2`指向的是同一个对象，而`i3`和`i4`指向的是不同的对象。Java 8中`Integer`的`valueOf()`方法的具体实现如下：

```java
// IntegerCache.low = -128
// IntegerCache.high 可通过JVM参数 -XX:AutoBoxCacheMax=<size> 设置，必须大于等于127，默认为127
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```
其中`IntegerCache`是一个私有静态内部类，其实现如下：

```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```
`IntegerCache`表示`Integer`缓存，其中的`cache`变量是一个静态`Integer`数组，在静态初始化块中被初始化，默认情况下，保存了`-128~127`共256个整数对应的`Integer`对象。


在`valueOf`代码中，**如果数值位于被缓存的范围，则直接从`IntegerCache`中获取已预先创建的`Integer`对象，只有不在缓存范围时，才通过`new`创建对象。**

**通过共享常用对象，可以节省内存空间，由于`Integer`是不可变的，所以缓存的对象可以安全的被共享。**`Boolean、Byte、Short、Long、Character`都有类似的实现。这种共享常用对象的思路，叫**享元模式**，英文叫Flyweight，即共享的轻量级元素。

## 2. 下面这段代码的输出结果是什么？

```java
public class Main {
    public static void main(String[] args) {
         
        Double i1 = 100.0;
        Double i2 = 100.0;
        Double i3 = 200.0;
        Double i4 = 200.0;
         
        System.out.println(i1==i2); 
        System.out.println(i3==i4);  
    }
}
```
答案：

```java
false
false
```

Java 8中`Double`的`valueOf()`方法的具体实现如下，不难理解答案为何都是`false`。

```java
public static Double valueOf(double d) {
    return new Double(d);
}
```

为什么`Double`类的`valueOf()`方法会采用与`Integer`类的`valueOf()`方法不同的实现。很简单：**在某个范围内的整型数值的个数是有限的，而浮点数却不是**。

需要注意的是，`Integer、Short、Byte、Character、Long`这几个类的`valueOf()`方法的实现是类似的:

```java
public static Byte valueOf(byte b) {
    final int offset = 128;
    return ByteCache.cache[(int)b + offset];
}

public static Long valueOf(long l) {
    final int offset = 128;
    if (l >= -128 && l <= 127) { // will cache
        return LongCache.cache[(int)l + offset];
    }
    return new Long(l);
}

public static Character valueOf(char c) {
    if (c <= 127) { // must cache
        return CharacterCache.cache[(int)c];
    }
    return new Character(c);
}

public static Short valueOf(short s) {
    final int offset = 128;
    int sAsInt = s;
    if (sAsInt >= -128 && sAsInt <= 127) { // must cache
        return ShortCache.cache[sAsInt + offset];
    }
    return new Short(s);
}
```

`Double、Float`的`valueOf()`方法的实现是类似的:

```java
public static Float valueOf(float f) {
    return new Float(f);
}
```

## 3. 下面这段代码的输出结果是什么？

```java
public class Main {
    public static void main(String[] args) {
         
        Boolean i1 = false;
        Boolean i2 = false;
        Boolean i3 = true;
        Boolean i4 = true;
         
        System.out.println(i1==i2); 
        System.out.println(i3==i4); 
    }
}
```
答案：

```java
true
true
```

看完`Boolean`类的`valueOf`方法的实现后，答案同样一目了然！

```java
/**
* The {@code Boolean} object corresponding to the primitive
* value {@code true}.
*/
public static final Boolean TRUE = new Boolean(true);

/**
* The {@code Boolean} object corresponding to the primitive
* value {@code false}.
*/
public static final Boolean FALSE = new Boolean(false);

public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

## 4. `Integer i = new Integer(10)`和`Integer i = 10`这两种方式的区别

   - 第一种方式不会触发自动装箱；第二种方式会；
   - 从执行效率和资源占用上来讲，`new`每次都会创建一个新对象，而除了`Float`和`Double`外的其他包装类，都会缓存包装类对象，减少需要创建对象的次数，节省空间，提升性能。
   - 从Java 9开始，这些构造方法已经被标记为过时了，推荐使用静态的`valueOf`方法。

## 5. 下面程序的输出结果是什么？
   ```java
   public class Main {
       public static void main(String[] args) {         
           Integer a = 1;
           Integer b = 2;
           Integer c = 3;
           Integer d = 3;
           Integer e = 321;
           Integer f = 321;
           Long g = 3L;
           Long h = 2L;         
           System.out.println(c==d);
           System.out.println(e==f);
           System.out.println(c==(a+b));
           System.out.println(c.equals(a+b));
           System.out.println(g==(a+b));
           System.out.println(g.equals(a+b));
           System.out.println(g.equals(a+h));
       }
   }
   ```

   答案：

   ```java
   System.out.println(c==d);               // true
   System.out.println(e==f);               // false
   System.out.println(c==(a+b));           // true
   System.out.println(c.equals(a+b));      // true
   System.out.println(g==(a+b));           // true
   System.out.println(g.equals(a+b));      // false
   System.out.println(g.equals(a+h));      // true
   ```

  -  `c==d` 和 `e == f` 的结果不多解释了。

  -  `System.out.println(c==(a+b))`对应的字节码如下：

	   ```java
	    91 aload_3         // 将c压入操作数栈
	    92 invokevirtual #10 <java/lang/Integer.intValue>    // 自动拆箱
	    95 aload_1         // 将a压入操作数栈
	    96 invokevirtual #10 <java/lang/Integer.intValue>    // 自动拆箱
	    99 aload_2         // 将b压入操作数栈
	   100 invokevirtual #10 <java/lang/Integer.intValue>    // 自动拆箱
	   103 iadd            // 计算a+b的值并将结果压入操作数栈
	   104 if_icmpne 111 (+7)   // 比较 c 和 a+b 的值
	   107 iconst_1
	   108 goto 112 (+4)
	   111 iconst_0
	   112 invokevirtual #9 <java/io/PrintStream.println>
	   115 getstatic #8 <java/lang/System.out>
	```

	   当`==`运算符的两个操作数都是包装器类型的引用时，则**比较的是引用地址**，而如果其中有一个操作数是表达式（即包含算术运算符）则**比较的是数值**（即会触发自动拆箱过程），从对应的字节码也可以看出调用了`intValue()`方法，触发了自动拆箱，比较它们的数值是否相等，因此`c==(a+b)`的结果为`true`。

  -  `System.out.println(c.equals(a+b))`对应的字节码如下：

	   ```java
	   118 aload_3     // 将c压入操作数栈
	   119 aload_1     // 将a压入操作数栈
	   120 invokevirtual #10 <java/lang/Integer.intValue> // 自动拆箱
	   123 aload_2     // 将b压入操作数栈
	   124 invokevirtual #10 <java/lang/Integer.intValue> // 自动拆箱 
	   127 iadd        // 计算a+b的值并将结果压入操作数栈
	   128 invokestatic #2 <java/lang/Integer.valueOf>    // 将a+b的值的自动装箱
	   131 invokevirtual #11 <java/lang/Integer.equals>   // 调用equals方法
	   134 invokevirtual #9 <java/io/PrintStream.println>
	   137 getstatic #8 <java/lang/System.out>
	```

	   从字节码可以看出，`c.equals(a+b)`首先触发自动拆箱，又触发了自动装箱，再调用`equals`方法，**而所有的包装类都重写了Object类中的`equals`方法**，equals用于判断当前对象和参数传入的对象是否相同，**Object类的默认实现是比较地址**，它和比较运算符（`==`）的结果是一样的。

	  ` equals`应该反映的是对象间的**逻辑相等**关系，**所以包装类都重写了该实现，实际比较用的是其包装的基本类型值**，对于`Integer`类，其`equals`方法代码如下（Java 8）：

	   ```java
	   public boolean equals(Object obj) {
	       if (obj instanceof Integer) {
	           return value == ((Integer)obj).intValue();
	       }
	       return false;
	   }
	   ```

	   	因此，`c.equals(a+b)`的结果为`true`.

 -  `System.out.println(g==(a+b))`对应的字节码如下：

	   ```java
	   140 aload 7       // 将g压入操作数栈
	   142 invokevirtual #12 <java/lang/Long.longValue>    // 自动拆箱
	   145 aload_1       // 将a压入操作数栈 
	   146 invokevirtual #10 <java/lang/Integer.intValue>  // 自动拆箱
	   149 aload_2       // 将b压入操作数栈
	   150 invokevirtual #10 <java/lang/Integer.intValue>  // 自动拆箱
	   153 iadd          // 计算a+b的值并将结果压入操作数栈
	   154 i2l           // 将 a+b 的结果从 int 转换为 long
	   155 lcmp          // 比较 g 和 a+b 的值
	   156 ifne 163 (+7) 
	   159 iconst_1
	   160 goto 164 (+4)
	   163 iconst_0
	   164 invokevirtual #9 <java/io/PrintStream.println>
	   167 getstatic #8 <java/lang/System.out>
	   ```

  	 `g==(a+b)`的比较过程和`c==(a+b)`类似，多了一个类型转换，都是比较数值，因此结果为`true`.

 -   `System.out.println(g.equals(a+b))`对应的字节码如下：

	   ```java
	   170 aload 7        // 将g压入操作数栈
	   172 aload_1        // 将a压入操作数栈
	   173 invokevirtual #10 <java/lang/Integer.intValue>  // 自动拆箱
	   176 aload_2        // 将b压入操作数栈
	   177 invokevirtual #10 <java/lang/Integer.intValue>  // 自动拆箱
	   180 iadd           // 计算a+b的值并将结果压入操作数栈
	   181 invokestatic #2 <java/lang/Integer.valueOf>     // 自动装箱
	   184 invokevirtual #13 <java/lang/Long.equals>       // 调用equals比较
	   187 invokevirtual #9 <java/io/PrintStream.println>
	   190 getstatic #8 <java/lang/System.out>
	   ```

  	 对于Long类，其`equals`方法代码如下（Java 8）：

	   ```java
	   public boolean equals(Object obj) {
	       if (obj instanceof Long) {
	       	return value == ((Long)obj).longValue();
	       }
	       return false;
	   }
	   ```
	
	   从字节码可以看出，`g.equals(a+b)`也进行了自动拆箱、装箱过程，但是`a+b`装箱后为`Integer`类型，而`g`是`Long`类型，因此调用`equasl`会输出`false`.

  -  `System.out.println(g.equals(a+h))`对应的字节码如下：

	   ```java
	   193 aload 7           // 将g压入操作数栈
	   195 aload_1           // 将a压入操作数栈
	   196 invokevirtual #10 <java/lang/Integer.intValue> // 自动拆箱
	   199 i2l               // 从a的数值从 int 转换成 long
	   200 aload 8           // 将h压入操作数栈
	   202 invokevirtual #12 <java/lang/Long.longValue>  // 自动拆箱
	   205 ladd              // 计算a+h的值并将结果压入操作数栈
	   206 invokestatic #5 <java/lang/Long.valueOf>       // 自动装箱
	   209 invokevirtual #13 <java/lang/Long.equals>      // 调用equals比较
	   212 invokevirtual #9 <java/io/PrintStream.println>
	```

  	 `g.equals(a+h)`相比于`g.equals(a+b)`多了一步类型转换，`a+h`装箱后的类型为`Long`，因此结果为`true`.

  	- main方法对应的字节码如下：

	   ```java
	    0 iconst_1
	     1 invokestatic #2 <java/lang/Integer.valueOf>
	     4 astore_1
	     5 iconst_2
	     6 invokestatic #2 <java/lang/Integer.valueOf>
	     9 astore_2
	    10 iconst_3
	    11 invokestatic #2 <java/lang/Integer.valueOf>
	    14 astore_3
	    15 iconst_3
	    16 invokestatic #2 <java/lang/Integer.valueOf>
	    19 astore 4
	    21 sipush 321
	    24 invokestatic #2 <java/lang/Integer.valueOf>
	    27 astore 5
	    29 sipush 321
	    32 invokestatic #2 <java/lang/Integer.valueOf>
	    35 astore 6
	    37 ldc2_w #3 <3>
	    40 invokestatic #5 <java/lang/Long.valueOf>
	    43 astore 7
	    45 ldc2_w #6 <2>
	    48 invokestatic #5 <java/lang/Long.valueOf>
	    51 astore 8
	    53 getstatic #8 <java/lang/System.out>
	    56 aload_3
	    57 aload 4
	    59 if_acmpne 66 (+7)
	    62 iconst_1
	    63 goto 67 (+4)
	    66 iconst_0
	    67 invokevirtual #9 <java/io/PrintStream.println>
	    70 getstatic #8 <java/lang/System.out>
	    73 aload 5
	    75 aload 6
	    77 if_acmpne 84 (+7)
	    80 iconst_1
	    81 goto 85 (+4)
	    84 iconst_0
	    85 invokevirtual #9 <java/io/PrintStream.println>
	    88 getstatic #8 <java/lang/System.out>
	    91 aload_3
	    92 invokevirtual #10 <java/lang/Integer.intValue>
	    95 aload_1
	    96 invokevirtual #10 <java/lang/Integer.intValue>
	    99 aload_2
	   100 invokevirtual #10 <java/lang/Integer.intValue>
	   103 iadd
	   104 if_icmpne 111 (+7)
	   107 iconst_1
	   108 goto 112 (+4)
	   111 iconst_0
	   112 invokevirtual #9 <java/io/PrintStream.println>
	   115 getstatic #8 <java/lang/System.out>
	   118 aload_3
	   119 aload_1
	   120 invokevirtual #10 <java/lang/Integer.intValue>
	   123 aload_2
	   124 invokevirtual #10 <java/lang/Integer.intValue>
	   127 iadd
	   128 invokestatic #2 <java/lang/Integer.valueOf>
	   131 invokevirtual #11 <java/lang/Integer.equals>
	   134 invokevirtual #9 <java/io/PrintStream.println>
	   137 getstatic #8 <java/lang/System.out>
	   140 aload 7
	   142 invokevirtual #12 <java/lang/Long.longValue>
	   145 aload_1
	   146 invokevirtual #10 <java/lang/Integer.intValue>
	   149 aload_2
	   150 invokevirtual #10 <java/lang/Integer.intValue>
	   153 iadd
	   154 i2l
	   155 lcmp
	   156 ifne 163 (+7)
	   159 iconst_1
	   160 goto 164 (+4)
	   163 iconst_0
	   164 invokevirtual #9 <java/io/PrintStream.println>
	   167 getstatic #8 <java/lang/System.out>
	   170 aload 7
	   172 aload_1
	   173 invokevirtual #10 <java/lang/Integer.intValue>
	   176 aload_2
	   177 invokevirtual #10 <java/lang/Integer.intValue>
	   180 iadd
	   181 invokestatic #2 <java/lang/Integer.valueOf>
	   184 invokevirtual #13 <java/lang/Long.equals>
	   187 invokevirtual #9 <java/io/PrintStream.println>
	   190 getstatic #8 <java/lang/System.out>
	   193 aload 7
	   195 aload_1
	   196 invokevirtual #10 <java/lang/Integer.intValue>
	   199 i2l
	   200 aload 8
	   202 invokevirtual #12 <java/lang/Long.longValue>
	   205 ladd
	   206 invokestatic #5 <java/lang/Long.valueOf>
	   209 invokevirtual #13 <java/lang/Long.equals>
	   212 invokevirtual #9 <java/io/PrintStream.println>
	   215 return
	```

基础类型是为了单纯使用数值时节省内存空间但是无法进行相关函数操作，而包装类型是为了方便进行相关操作但有占用更多的内存空间，而有时需要用基础类型，有时又需要用包装类型，为了避免写这种重复转换的代码，才提供了自动拆箱/装箱方便操作。


# Reference
- [深入剖析Java中的装箱和拆箱](https://www.cnblogs.com/dolphin0520/p/3780005.html)
- [《Java编程的逻辑》](https://www.cnblogs.com/swiftma/p/5631311.html)