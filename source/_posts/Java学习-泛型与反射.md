---
title: Java学习 - 泛型与反射
date: 2020-08-08 16:44:16
categories: 
- 语言学习
tags:
- Java
- 泛型
- 反射
---

**知识点主要来源于Java核心技术（卷I）**
<!--more-->

# 反射
## Class类
`Class`类用于保存及获取对象的运行时类型信息。程序在运行时对自身信息（元数据）进行检测
进而修改自己的行为被称为反射。
`Object`类中的`getClass()`方法返回一个`Class`类型的实例。`Class`的静态方法`forName()`可以根据类名获得对应`Class`对象，注意要为这个方法提供异常处理。如果有一个`Class`类型的对象，可以使用`getConstructor()`方法得到一个`Constructor`类型的对象，然后使用`newInstance()`方法构造一个实例。
JVM不会一次性把所有用到的类全部加载到内存，它首先加载包含`main`方法的类，然后加载该类所需的所有类，以此类推。可以在运行期根据条件利用`Class.forName()`来手动强制加载其他类。

## 利用反射分析类

### 访问字段(Field)
首先，可以通过`Class`实例获取字段信息
* `Field getField(name)` 根据字段名称获得public的field
* `Field getDeclaredField(name)` 根据字段名称获得field
* `Field[] getFields()` 获取所有public的field
* `Field[] getDeclaredFields()` 获取所有field
`Field`对象包含了一个字段的所有信息
* `getName()` 返回字段名
* `getType()` 返回字段类型（一个`Class`实例）
* `getModifiers()` 返回字段修饰符，每个bit表示不同的涵义，可以使用`Modifier`类的一些静态方法对返回的值进行解析。
获得字段的`Field`实例后我们还可以获取或设置一个实例该字段的值。
* `get()` 返回某一对象中这个`Field`描述的字段值
* `set()` 将字段值设为一个新值
对于`private`字段进行如上的操作会得到一个`IllegalAccessException`，因此我们可以先试图查询这个字段的可访问性，如有必要可以对其进行修改
* `void setAccessible(boolean flag)`
* `boolean trySetAccessible(boolean flag)` （Java 9+）
* `boolean isAccessible()`
* `static void setAccessble(AccessibleObject[] array, boolean flag)`
  
### 调用方法(Method)
可以通过`Class`实例获取`Method`信息
* `Method getMethod(name, Class...)` 获取某个public的method
* `Method getDeclaredMethod(name, Class...)` 获取某个method
* `Method[] getMethods()`
* `Method[] getDeclaredMethods()`
`Method`对象包含一个方法的所有信息
* `getName()` 方法名
* `getReturnType()` 返回值，返回的是`Class`实例
* `getParameterTypes()` 方法的参数类型，是一个`Class`数组
* `getModifiers()` 方法的修饰符。类似于`Field`的修饰符。
可以通过`Method`对象的`invoke`方法来反射调用方法。
* `public Object invoke(Object implicitParameter, Object[] explicitParameters)` 反射调用方法，第一个参数是对象实例，即在哪个实例上调用该方法。调用静态方法时第一个参数传入`null`
和`Field`类似，对于非public方法直接调用将得到`IllegalAccessException`。查询和修改方法的可访问性的方式与`Field`类似。
使用反射调用方法时遵循多态原则，即总是调用实际类型的覆写方法（如果存在）。

### 调用构造方法(Constructor)
可以通过`Class`实例获取`Constructor`信息
* `getConstructor(Class...)`
* `getDeclaredConstructor(Class...)`
* `getConstructors()`
* `getDeclaredConstructors()`
通过`Constructor`实例可以利用`newInstance()`创建一个实例对象
* `newInstance(Object...parameters)`

### 编写泛型数组代码（Todo）

### 动态代理（Todo）

# 泛型类的定义
```java
public class ClassName<T, U> {
    private T member1;
    private U member2;

    public T method1() {...}

    public void method2(T argument1) {...}
}
```

# 泛型方法的定义
```java
class ClassName {
    public <T> T method(T argument) {...}
}
```
泛型方法的类型推导是可能出错的（？）

# 对类型变量进行限定
可以限制传入的类型必须是某个类型的子类或必须实现某些接口。
```java
class ClassName {
    public <T extends SuperClass & Interface1 & Interface2> T method() {...}
}
```
可以根据需要声明多个限定，但最多有一个限定可以是类，且类作为限定时必须是限定列表中的第一个限定。

# 泛型代码与虚拟机

## 类型擦除
定义一个泛型类型时，都会自动提供一个相应的原始类型(raw type)。这个原始类型的名字就是去掉类型参数之后的泛型类型名。类型变量会被擦除(erased)，并替换为限定类型。对于无限定的变量则替换为Object。对于泛型类
```java
public class ClassName<T> {
    private T member1;
    
    public T method1() {...}

    public void method2(T argument1) {...}
}
```
其原始类型为
```java
public class ClassName {
    private Object member1;

    public Object method1() {...}

    public void method2(Object argument1) {...}
}
```
程序中包含不同类型的`ClassName`，被擦除类型后都会变成`ClassName`的原始类型。与C++的模板相比，C++会为每个模板的实例化产生不同的类型。

而对于给出限定的泛型类，其原始类型会使用第一个限定来替换类型变量。对于泛型类
```java
public class ClassName<T extends Interface1 & Interface2> {
    private T member;

    public ClassName(T argument) {...}
}
```
其原始类型将为
```java
public class ClassName {
    private Interface1 member;

    public ClassName(Interface1 argument) {...}
}
```
如果有必要，编译器可能会在此时插入强制类型转换。为提高效率，应将没有方法的接口放在限定列表的末尾。

## 转换泛型表达式
调用泛型方法时，如果擦除了返回类型，编译器会插入强制类型转换。例如，对于如下的泛型类
```java
public class ClassName<T> {
    private T member;
    public T getMember() {
        return this.member;
    }
}
```
考虑以下语句序列
```java
ClassName<ClassName2> a = ...;
ClassName2 b = a.getMember();
```
原始类型中的`getMember()`方法返回值类型被擦除，返回类型为`Object`。此时第二行语句会发生从`Object`到`ClassName2`的强制类型转换（在结果字节码中）。


## 转换泛型方法
继承传入类型参数的泛型类得到的子类，在重写父类的同名方法时可能会遇到类型擦除与多态性相冲突的情况。编译器会合成桥方法(bridge method)来保证多态性。（有待补充）

# Java泛型的限制与局限性
## 不能用基本类型实例化类型参数（无法类型擦除）

## 运行时类型查询只适用于原始类型
对于所有泛型类，所有的类型查询只会产生原始类型。对于`ClassName<T>`，所有类型查询(`instanceof`, 强制类型转换)只能判断变量是否为任意类型的一个`ClassName`。同时，`getClass`方法也总是返回原始类型。
```java
ClassName1<ClassName2> a = ...;
// 只返回ClassName1.class
a.getClass();
```

## 不能创建参数化类型的数组
Java的数组不是泛型容器，在运行时会持有它的元素类型信息，如果试图存储其它类型的元素会先尝试类型转换，失败则会抛出一个ArrayStoreException异常。
对于示例类
```java
// 这是一个后续各个例子都可能使用的类
class Pair<T> {
	private T first;
	private T second;
	
	public Pair(T f, T s) {
		this.first = f;
		this.second = s;
	}
	
	public T getFirst() {
		return this.first;
	}
	
	public T getSecond() {
		return this.second;
	}
	
	public void setFirst(T f) {
		this.first = f;
	}
	
	public void setSecond(T s) {
		this.second = s;
	}
}
```
尝试定义这样一个数组
```java
var table = new Pair<String>[10];
// 事实上无法编译通过
```
如果能够编译通过，`table`的类型应为`Pair[]`。对于泛型类型，类型擦除会让数组的元素类型检查失效。
```java
table[0] = new Pair<OtherType>();
```
因此不允许创建参数化类型的数组。但是声明类型为`Pair<String>[]`的变量依然是合法的。
声明通配类型的数组是可以编译通过的。声明后可以对其进行强制类型转换，但这是不安全的。
```java
var table1 = (Pair<String>[]) new Pair<?>[10]; // 可以通过
table1[0] = new Pair<Integer>(1, 1); // 按Java Core I中的大意写出的代码，但是直接报了类型转换错误，无法编译通过

var table2 = new Pair<?>[10];
table2[0] = new Pair<Integer(1, 1)>;
System.out.println(((Pair<String>[]) table2)[0].getFirst()); // 在调用getFirst()前对table2进行强制类型转换，可以编译通过，但是运行时会得到一个ClassCastException异常
// 问题: 上述table2的例子是否算是heap pollution的一个例子?
```
总之，如果有收集参数化类型对象的需求，使用泛型容器（如用ArrayList代替数组）更简单有效。

## Varargs警告
Java不支持泛型类型的数组，但是在某种场合下这个限制会得到放松：向参数个数可变的方法传递泛型类型的实例。考虑以下代码
```java
public class Main {
    public static void main(String[] args) {
		Collection<Pair<String>> table = new ArrayList<Pair<String>>();
		Pair<String> pair1 = new Pair<String>("1", "2");
		Pair<String> pair2 = new Pair<String>("3", "4");
		ClassName.addAll(table, pair1, pair2);
	}
}

class ClassName {
    public static <T> void addAll(Collection<T> coll, T...ts) {
        // var type = ts.getClass();
        for(T t: ts) coll.add(t);
    }
}
```
为了调用`addAll()`，JVM必须建立一个`Pair<String>`数组。实际上使用debug可以发现参数ts的类型为`Pair[]`（已经经历了类型擦除）。这种情况下，不会得到错误，而只会在调用`addAll()`的地方得到警告`Type safety: A generic array of Pair<String> is created for a varargs parameter`。可以使用`@SafeVarargs`注解来消除创建泛型数组的有关限制，但要注意这始终是一个危险的操作，下面的代码给出了一个例子。
```java
public class Main {
	public static void main(String[] args) {
		Collection<Pair<String>> table = new ArrayList<Pair<String>>();
		Pair<String> pair1 = new Pair<String>("1", "2");
		Pair<String> pair2 = new Pair<String>("3", "4");
		Pair<String> pair3 = (Pair<String>)(Object)new Pair<Integer>(5, 6);
		ClassName.addAll(table, pair1, pair2, pair3);
		try {
			// BOOM!
			String omg = ((ArrayList<Pair<String>>) table).get(2).getFirst();
		} catch (ClassCastException e) {
			System.out.println("BOOM!");
		}
	}
}
```
上述代码还会在定义`addAll()`的地方得到警告`Type safety: Potential heap pollution via varargs parameter ts`。Heap pollution指的是一种特殊的引用（references），这种引用持有的类型并不是它所指向的对象的超类。

## 不能实例化类型变量
即对于`Pair<T>`，不能实例化`T`。换言之这样的构造器是非法的
```java
class Pair<T> {
    ...
    public Pair() {
        first = new T();
        second = new T();
    }
} // ERROR!
```
解决方法：
1. 让调用者提供一个构造器表达式（Java 8+）
   ```java
   public static <T> Pair<T> makePair(Supplier<T> constr) {
       return new Pair<>(constr.get(), constr.get());
   }
   ```
   其中`Supplier<T>`为一个函数式接口，表示一个无参数且返回类型为`T`的函数。
2. 通过反射调用`Constructor.newInstance`方法来构造泛型对象（？）
   ```java
   public static <T> Pair<T> makePair(Class<T> cl) {
       try {
           return new Pair<>(cl.getConstructor().newInstance(), cl.getConstructor().newInstance());
       } catch (Exception e) {
           return null;
       }
   }
   ```

## 不能构造泛型数组
与“不能创建参数化类型的数组”相似。解决方法：
1. 让用户提供一个数组构造器表达式（Java 8+）
   ```java
   public static <T extends Comparable> T[] minmax(IntFunction<T[]> constr, T...a) {
       T[] result = constr.apply(2);
       ...
   }
   ...
   //调用
   String[] res = ArrayAlg.minmax(String[]::new, "abc", "def", "ghi");
   ```
2. 利用反射调用`Array.newInstance`
   ```java
   public static <T extends Comparable> T[] minmax(T...a) {
       var result = (T[]) Array.newInstance(a.getClass.getComponentType(), 2);
   }
   ```

## 泛型类的静态上下文中类型变量无效
不能在静态字段或方法中引用类型变量。
```java
public class Singleton<T> {
    private static T singleInstance; // ERROR (这个类以不同类型实例化两次不就出事了吗)
    
    public static T getSingleInstance() {
        ...
    } // ERROR
}
```

## 不能抛出或捕获泛型类的实例
既不能抛出也不能捕获泛型类的对象，实际上泛型类无法扩展`Throwable`。`catch`子句中不能使用类型变量。

## 可以取消对检查型异常的检查（Todo）

## 注意类型擦除后的冲突（Todo）

# 泛型类型的继承规则
考虑一个类`Class`及其子类`SubClass`。有如下的继承规则：
1. `ArrayList<Class>`继承了`ArrayList`（原始类型），实现了`List<Class>`
2. `ArrayList<SubClass>`继承了`ArrayList`（原始类型），实现了`List<SubClass>`
3. `List<Class>`继承了`List`（原始类型）
4. `List<SubClass>`继承了`List`（原始类型）
5. `ArrayList`（原始类型）实现了`List`（原始类型）
6. `ArrayList<Class>`和`ArrayList<SubClass>`没有关系
7. `List<Class>`和`List<SubClass>`没有关系

# 通配符类型
首先给出三个类及其继承关系：
```java
class SuperClass {...}
class SubClass extends SuperClass {...}
class SubSubClass extends SubClass {...}
```
## 概念
通配符类型允许类型参数发生变化。`Pair<? extends SuperClass>`表示任何泛型`Pair`类型，其类型参数是`SuperClass`的子类。若`SubClass`是`SuperClass`的子类，那么`Pair<SuperClass>`与`Pair<SubClass>`都是`Pair<? extends SuperClass`的子类，而`Pair<? extends SuperClass>`是`Pair`（原始类型）的子类。

## 超类型限定
通配符`? super ClassName`限制为`ClassName`的所有超类型。稍微有点绕，举个例子进行说明:对于`Pair<? super SubClass>`而言，有如下方法：
```java
void setFirst(? super SubClass) {...}
? super SubClass getFirst() {...}
```
由此可见，编译器不知道`setFirst`的具体类型（参数类型是`SubClass`的某个超类，这个超类及其子类才能作为参数传入，那么这个超类是哪一个呢？是`SuperClass`吗？还是`Object`？），因此无法接受`SuperClass`类型或`Object`类型的方法调用。而`SubClass`的子类型（如`SubSubClass`）对象或是`SubClass`类型的对象则可以作为参数传入。另外，如果调用`getFirst`，则不能保证返回值的类型（返回值类型是`SubClass`的某个超类，类型为这个超类及其超类的引用才能托管`getFirst`的返回值，然而这个超类具体是哪个同样无从得知），只能把返回值赋给一个`Object`。
对于超类型限定的继承关系，`Pair<SuperClass>`以及`Pair<Object>`是`Pair<? super SubClass>`的子类，而`Pair<? super Manager>`为`Pair<?>`的子类。

## 无限定通配符
例如`Pair<?>`，它有如下方法
```java
? getFirst()
void setFirst(?)
```
`getFirst`的返回值只能赋给一个`Object`，`setFirst`无法被调用。

## 通配符捕获（Todo）

## 通过反射分析泛型类信息（Todo）