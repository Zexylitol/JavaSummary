# 1. 包装类是什么？

Java有8种基本类型，每种基本类型都有一个对应的包装类。包装类是一个类，内部有一个实例变量，保存对应的基本类型的值，一般还有一些静态方法、静态变量和实例方法，以方便对数据进行操作。

| 基本类型 | 包装类  | 基本类型 | 包装类    |
| -------- | ------- | -------- | --------- |
| boolean  | Boolean | long     | Long      |
| byte     | Byte    | float    | Float     |
| short    | Short   | double   | Double    |
| int      | Integer | char     | Character |

# 2. 为什么需要包装类？

基础类型只能单纯使用数值，虽然节省内存空间，但是无法进行相关函数操作，而**Java中有很多代码（比如容器类）只能操作对象**，为了能操作基本类型，需要使用其对应的包装类；另外。包装类提供了很多有用的方法，可以方便对数据的操作。

# 3. 装箱与拆箱

各个包装类都可以与其对应的基本类型相互转换，每种包装类都有一个静态方法`valueOf()`，接受基本类型，返回引用类型，也都有一个实例方法`xxxValue()`，返回对应的基本类型。

```java
Boolean bObj = Boolean.valueOf(true);
boolean b = bObj.booleanValue();

Integer iObj = Integer.valueOf(10);
int i = iObj.intValue();

Double dObj = Double.valueOf(10.0);
double d = dObj.doubleValue();

Character cObj = Character.valueOf('a');
char c = cObj.charValue();

Byte byObj = Byte.valueOf((byte) 10);
byte by = byObj.byteValue();

Float fObj = Float.valueOf(10.0f);
float f = fObj.floatValue();

Short sObj = Short.valueOf((short) 10);
short s = sObj.shortValue();

Long lObj = Long.valueOf(10l);
long l = lObj.longValue();
```

**装箱**：将基本类型转换为包装类的过程

**拆箱**：将包装类型转换为基本类型的过程

**Java 5以后引入了自动装箱和拆箱技术**，可以直接将基本类型赋值给引用类型，反之亦可。

# 4. 装箱和拆箱是如何实现的？

```java
public class Main {
    public static void main(String[] args) {
        Integer i = 10;
        int n = i;
    }
}
```

以Java 8为例，main方法对应的字节码如下：

```java
 0 bipush 10
 2 invokestatic #2 <java/lang/Integer.valueOf>
 5 astore_1
 6 aload_1
 7 invokevirtual #3 <java/lang/Integer.intValue>
10 istore_2
11 return
```

**自动装箱/拆箱是Java编译器提供的能力**，从反编译得到的字节码内容可以看出，在装箱的时候自动调用的是`Integer`的`valueOf()`方法。而在拆箱的时候自动调用的是`Integer`的`intValue`方法。

因此可以用一句话总结装箱和拆箱的实现过程：

**装箱过程是通过调用包装器的`valueOf`方法实现的，而拆箱过程是通过调用包装器的`xxxValue`方法实现的。（xxx代表对应的基本数据类型）**。

# 5. 包装类的共同点

各个包装类的共同点包括都重写了Object类中的方法，都实现了Comparable接口，都继承了Number抽象类，都是不可变的等。

## 5.1 重写Object方法

所有包装类都重写了Object类的如下方法：

```java
boolean equals(Object obj);
int hashCode();
String toString();
```

### 5.1.1 equals

Object类的默认实现是比较地址，`equals`应该反映的是对象间的逻辑相等关系，所以所有包装类都重写了该实现，实际比较用的是其包装的基本类型值，例如，Java 8 中，Integer类的`equals`方法代码如下：

```java
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```

对于Float类，其`equals`代码实现如下：

```java
public boolean equals(Object obj) {
    return (obj instanceof Float)
        && (floatToIntBits(((Float)obj).value) == floatToIntBits(value));
}
```

**`floatToIntBits()`方法将float的二进制表示看作int**，这是因为小数计算是不精确的，数学概念上运算结果一样，但计算机运算结果可能不同，因此，**只有两个float的二进制表示完全一样时，`equals`才会返回true.**

```java
Float f1 = 0.01f;
Float f2 = 0.1f * 0.1f;
System.out.println(f1.equals(f2));
System.out.println(Float.floatToIntBits(f1));
System.out.println(Float.floatToIntBits(f2));
```

输出为：

```java
false
1008981770
1008981771
```

Double的`equals`方法与Float类似，它有一个静态方法doubleToLongBits，将double的二进制表示看作long，然后再按long比较。

### 5.1.2 hashCode

hashCode返回一个对象的哈希值。**哈希值是一个int类型的数，由对象中一般不变的属性映射得来，用于快速对对象进行区分、分组等**。一个对象的哈希值不能改变，相同对象的哈希值必须一样。不同对象的哈希值一般应不同，但这不是必需的，可以有对象不同但哈希值相同的情况。



`hashCode`和`equals`方法联系密切，**对两个对象，如果`equals`方法返回true，则`hashCode`也必须一样**。反之不要求，`equals`方法返回false时，`hashCode`可以一样，也可以不一样，但应该尽量不一样。**hashCode的默认实现一般是将对象地址转换为整数**，子类如果重写了`equals`方法，也必须重写`hashCode`。之所以有这个规定，是因为Java API中有很多类依赖于这个行为，尤其是容器中的一些类。



包装类都重写了`hashCode`，根据包装的基本类型值计算`hashCode`，对于Byte、Short、Integer、Character，**hashCode就是其内部值**；以Byte类型为例，其代码为：（Java 8）

```java
@Override
public int hashCode() {
    return Byte.hashCode(value);
}

public static int hashCode(byte value) {
    return (int)value;
}
```

对于Boolean类型，其hashCode代码为：

```java
@Override
public int hashCode() {
    return Boolean.hashCode(value);
}

/**
     * Returns a hash code for a {@code boolean} value; compatible with
     * {@code Boolean.hashCode()}.
     *
     * @param value the value to hash
     * @return a hash code value for a {@code boolean} value.
     * @since 1.8
     */
public static int hashCode(boolean value) {
    return value ? 1231 : 1237;
}
```

根据基类类型**返回了两个不同的质数**（即只能被1和自己整除的数），**质数用于哈希时不易冲突**。



对于Long类型，hashCode代码为：

```java
@Override
public int hashCode() {
    return Long.hashCode(value);
}

/**
     * Returns a hash code for a {@code long} value; compatible with
     * {@code Long.hashCode()}.
     *
     * @param value the value to hash
     * @return a hash code value for a {@code long} value.
     * @since 1.8
     */
public static int hashCode(long value) {
    return (int)(value ^ (value >>> 32));
}
```

Long类型的哈希值是高32位和低32位进行异或操作后的结果。



对于Float类型，hashCode代码为：

```java
@Override
public int hashCode() {
    return Float.hashCode(value);
}

/**
     * Returns a hash code for a {@code float} value; compatible with
     * {@code Float.hashCode()}.
     *
     * @param value the value to hash
     * @return a hash code value for a {@code float} value.
     * @since 1.8
     */
public static int hashCode(float value) {
    return floatToIntBits(value);
}
```

Float是将float的二进制表示看作int作为哈希值。



对于Double类型，hashCode代码为：

```java
@Override
public int hashCode() {
    return Double.hashCode(value);
}

/**
     * Returns a hash code for a {@code double} value; compatible with
     * {@code Double.hashCode()}.
     *
     * @param value the value to hash
     * @return a hash code value for a {@code double} value.
     * @since 1.8
     */
public static int hashCode(double value) {
    long bits = doubleToLongBits(value);
    return (int)(bits ^ (bits >>> 32));
}
```

Double类型首相将double的二进制表示看作long，然后再按long计算hashCode.

### 5.1.3 toString

每个包装类也都重写了`toString`方法，返回对象的字符串表示。

## 5.2 Comparable

每个包装类都实现了Comparable接口，Comparable接口代码如下：

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

接口只有一个方法`CompareTo`，当前对象与参数对象进行比较，在小于、等于、大于参数时，应分别返回-1、0、1.

各个包装类的实现基本都是根据其基本类型值进行比较，对于Boolean，false小于true。对于Float和Double，存在和equals方法一样的问题，0.01和0.1*0.1相比的结果并不为0.

## 5.4 Number

6种数值类型包装类（Character、Boolean除外）有一个共同的父类。Number是一个抽象类，通过这些方法，包装类实例可以返回任意的基本数值类型。

```java
public abstract class Number implements java.io.Serializable {

    public abstract int intValue();
    public abstract long longValue();
    public abstract float floatValue();
    public abstract double doubleValue();
    public byte byteValue() {
        return (byte)intValue();
    }
    public short shortValue() {
        return (short)intValue();
    }
    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -8742448824652078965L;
}
```

## 5.5 不可变性

**包装类都是不可变类**。所谓不可变是指实例对象一旦创建，就没有办法修改了，这是通过如下方式强制实现的：

- 所有包装类都声明为了`final`，不能被继承
- 内部基本类型值是私有的，且声明为了`final`。
- 没有定义`setter`方法

**不可变使得程序更为简单安全，因为不用操心数据被意外改写的可能，可以安全地共享数据，尤其是在多线程的环境下。**

# Reference
- [《Java编程的逻辑》](https://www.cnblogs.com/swiftma/p/5631311.html)