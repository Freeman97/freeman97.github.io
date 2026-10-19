---
title: 重新认识Java SE - Java SE随学随记系列
tags: Java
---

# `Object.equals()`, `==`运算符与`Object.hashCode()`
## `Object.equals()`与`==`
对于基本类型而言，`==`运算符就是比较值是否相等，没啥好说的。而对于对象，`==`判断的是左右两边是否为同一个对象，也就是左右两边的对象的内存地址是否相同。
而`Object.equals()`是真正用于比较两个对象是否相等的方法（相等和"同一个"是两个概念）。`Object`提供的默认实现是使用`==`进行判断的，所有继承了`Object`的类都可以对`equals()`方法进行重写，有时候会需要根据具体的逻辑来重写自定义类的`equals()`方法
需要注意的是，Java的8种基本类型的包装类存在对象缓存池，数据在一定范围内时调用`valueOf`方法也不会创建新对象（而Java的自动装箱恰是调用`valueOf`来完成的）。在这个时候用`==`来比较两个对象是会返回`true`的。Java对于`String`类也维护了一个字符串池，因此使用`String a = "blablabla"`的方式对变量进行初始化也会发生类似的事情。然而，使用`new`关键字新建对象是一定会在堆上创建新对象的。
```java
Integer a = Integer.valueOf(256);
Integer b = Integer.valueOf(256);
System.out.println(a == b);
// false
System.out.println(a.equals(b));
// true

Integer c = Integer.valueOf(100);
Integer d = Integer.valueOf(100);
System.out.println(c == d);
// true
```

## `Object.hashCode()`
一个常见的问题是"为什么重写`equals`方法时必须重写`hashCode`方法"。`Object.hashCode()`提供一个对象的哈希码(`int`类型)，这个哈希码主要提供给与哈希算法有关的场景使用(例如集合`HashMap`)。**而根据Oracle的JDK[文档](https://docs.oracle.com/javase/10/docs/api/java/lang/Object.html#hashCode())，这个方法返回的并不一定是对象的内存地址（The hashCode may or may not be implemented as some function of an object's memory address at some point in time.）。**
生成`hashCode`通常只需要遵守这些规则：
1. 对于**同一个**对象，无论`hashCode()`在什么时候被调用，被调用了几次，都应该返回相同的整数，除非该对象在`equals()`方法里用于比较的信息被修改了。
2. 一个程序在被多次运行时，每次调用`hashCode()`的整数可以是不同的。因此不要用`hashCode()`返回的整数进行序列化。
3. 如果通过`equals()`方法能够判断两个对象相等，那在这两个对象上调用`hashCode()`必须返回相同的整数。
4. 如果通过`equals()`方法判断两个对象不相等，此时**并不要求**在这两个对象上调用`hashCode()`返回的整数必须不相等。这说明hash collision是允许发生的，即使是`Object`的默认实现也不例外，因此可以断定`==`运算符肯定不是比较`Object`产生的哈希码，因为只要堆内对象足够多就会有可观的概率产生哈希冲突（birthday paradox）。当然程序员在重写`hashCode()`时应该注意hash collision带来的性能问题。
那么`hashCode()`会被用来干什么，为什么重写了`equals()`一定要重写它？如果能够保证这个对象一定不会参加任何与哈希表有关的算法（一定不会被调用`hashCode()`，一定不会被放进`HashMap`之类的集合），不重写它也可以。`hashCode()`在与哈希表相关的算法中可以减少`equals()`的调用次数，进而提高性能。根据`hashCode()`的性质，如果两个对象的哈希码不相等，说明不需要调用`equals()`也能确定两个对象不相等。

# 值传递和引用传递
首先明确一点，Java中只存在值传递。在方法中传入对象时实际上传入了对象的地址，而修改这个的地址并不会对方法外的变量造成影响。但是，对这个地址上的对象本身做出修改会直接反应到对象上，这样的修改对方法外也是可见的。
举个例子更能体现C++"引用传递"和Java"按值传递对象的地址"的区别。Java的`swap`函数无法生效，而C++的`swap`函数的确能交换两个变量。
```java
public static void swap(Object x, Object y) {
    Object temp = x;
    x = y;
    y = x;
}
```
```cpp
void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}
```

# Java 异常处理
* Throwable
  * Errors(也属于Unchecked Excpetions)，无法被处理
    * StackOverflowError
    * VirtualMachineError
    * OutOfMemoryError
  * Exceptions(异常)
    * 检查型异常(Checked Exceptions): 在编译期由编译器检查出的可以通过执行方法或构造函数产生的异常。必须处理，否则无法通过编译。
    * 非检查型异常(Unchecked Exceptions): `RuntimeException`及其子类、`Error`及其子类。可以不处理。
* try-catch-finally块中，如果在`try`或`catch`块中遇到`return`，`finally`块会在返回前被执行。如果`finally`中也有`return`，`finally`中的返回值会覆盖原始的返回值。多个资源可以在括号中用分号分隔。
* `try-with-resources`: 实现`java.lang.AutoCloseable`或`java.io.Closeable`的对象可以作为资源。`catch`和`finally`会在资源关闭后运行。

# Java I/O
## I/O流的概念
用流来表示数据的序列，输入流表示输入数据的数据源，输出流表示输出数据的目的地。不同的I/O流的数据单位不一样，可以是字节(byte stream)，可以是字符(character)，也可以是对象。程序用输入流从数据源中读取数据，每次读取一个项（一个单元），用输出流向目的地输出数据，每次输出一个项。

## 字节流
* `InputStream`和`OutputStream`的子类都是字节流。

## 字符流
* 自动完成输入输出数据和字符之间的转换。

## 缓冲流
* 普通的I/O流每次独写请求都直接交给操作系统完成。缓冲流维护内存中的缓冲区，缓冲区满时才调用本地的输入输出API。

## 数据流
* 能直接对基本数据类型和字符串进行输入输出。

## 对象流
* 支持对实现了`Serializable`接口的对象进行输入输出。
* 在一个对象流中一个对象只会保存一份副本，显式地在一个对象流中写入同一对象多次，读取的时候会读到指向同一个对象的多个引用。

## 使用NIO.2的文件I/O