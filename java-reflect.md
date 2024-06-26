java基础学习：java中的反射详解和使用实践

## 一、什么是java反射

什么是 java 的反射？

说到反射，写这篇文章时，我突然想到了人的”反省“，反省是什么？吾一日三省吾身，一般就是反思自身，今天做了哪些对或错的事情。

java 的反射，我觉得有同样的思想。当然 java 反射要“反思”的是 java 程序在运行时类自己的信息，它获取的信息就是它自身类的详细信息。

类的哪些详细信息呢？比如类或对象的成员变量、方法等。然后可以对这些信息加以修改，从而调整 java 的运行逻辑。

java 反射 API 提供了非常丰富的工具集，反射 API 能够获取对象的变量，方法等成员，从而可以动态的操纵 java 代码程序。文章后面会介绍这些反射 API（反射相关的类）的一些用法。

为什么反射能得到 java 程序运行时类的信息呢？这就要从 java 的虚拟机 jvm 说起。

## 二、虚拟机jvm加载文件

**Java虚拟机**（Java Virtual Machine）：用于执行编译后的 java 程序的虚拟容器。jvm 可以跨操作系统使用。

jvm 内部结构分为 3 部分：类加载器classload子系统、运行时数据区、执行引擎。



以 `.java` 结尾的文件是不能直接在 jvm 上运行，它必须通过 javac 编译为以 `.class` 为后缀结尾的字节码文件才能运行。

java 文件被编译为 .class 的文件后，java 文件中各种对象的信息就确定下来了，存在于 .class 文件里。通过 java 的反射就可以获取里面的信息。

## 三、反射原理简析

在上一小节简单了解了文件加载内容，就是 java 文件经过编译后变成 .class 文件，类的各种信息就存储在 .class 文件中了，所以反射才能获取到类的各种信息。

java 代码编译为字节码的 .class 类文件，那 .class 文件里都有什么格式是什么？

class 文件结构采用类似 c 语言的结构体来存储数据。

```c
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

它由 2 部分组成：无符号的数和表。所有的表都习惯以 `_info` 结尾。

无符号的数属于基本的数据类型，一 u1，u2，u4，u8 分别来表示 1 个字节，2 个字节，4 个字节和 8 个字节的无符号数，无符号数可以用来表示数字、索引引用、数量值。

表用于描述有层次关系的复合结构的数据，整个 class 文件本质就是一张表。



类的信息都存储在 .class 文件里，当把 .class 文件读入内存时，就会为之创建一个 Class 对象。

这里又涉及到虚拟机加载类的机制。周志明的[《深入理解java虚拟机》](https://book.douban.com/subject/34907497/)里有一节讲类加载的内容，上面 class 文件结构也来自本书，可以好好看一看此书。

简单说：java 虚拟机把描述类的数据 class 文件加载到内存，并对数据进行效验、转换解析和初始化，最终形成可以被虚拟机直接使用的类型，这就是虚拟机的类加载机制。

在 java 语言里，类的加载和连接过程都是在程序运行期间完成的，虽然有性能开销，但是为 java 应用程序提供了高度的灵活性。比如反射就是发生在 java 运行时完成的。

在类加载阶段，虚拟机会在 java 堆中生成一个代表这个类的 java.lang.Class 对象，通过这个 Class 对象就可以访问到 jvm 中的这个类。

Class 类与 class 是不同，Class 是实实在在存在于 java.lang.Class 包中的一个类。

![image-20220825033829182](https://img2022.cnblogs.com/blog/650581/202209/650581-20220905202716766-1996172084.png)

反射：获取 Class 类对象及其类内部成员信息(属性, 方法, 构造函数等)以及控制实例对象的能力

## 四、反射功能常用的类

在 java 中，要使用反射功能，主要用到下面 2 个类：

> 1. **java.lang.Class**
>
> 2. **java.lang.reflect**
>
> - java.lang.reflect.Constructor， 获取构造方法，Class 对象所表示类的构造方法
> - Java.lang.reflect.Field， 字段成员，Class 对象所表示的类的成员变量，通过它可以动态修改成员变量值，包含 private
> - Java.lang.reflect.Method，方法成员，Class 对象所表示的类的方法成员，通过它可以动态调用对象的方法
> - Java.lang.reflect.Modifier，对类和成员访问修饰符进行解码

## 五、反射功能的使用

### 5.1 获取 Class 类对象的方法

获取 class 类对象的方法，主要有 3 种：

> 1. 根据 forName 静态方法：`Class.forName(类的全限定名)`。该方法是传入一个字符串参数，该参数是某个类的全限定完整包名。
>
> 2. 根据类名直接获取：`类名.class`。调用某个类的 class 属性获取 Class 对象，比如 Student.class 返回 Student 类对应的 Class 对象。
>
> 3. 根据对象获取：`对象.getClass()`。该方法是 java.lang.Object 类中的一个方法，所有 java 对象都可以调用该方法。

![image-20220825154029798](https://img2022.cnblogs.com/blog/650581/202209/650581-20220905202716770-634175896.png)

​                                    （三种获取 Class 对象的方法）

先举一个小例子来看看这 3 种获取 Class 对象的用法。

> java v1.8

**第一步**：用 IDEA 新建 maven 项目，名字叫 JavaBasicDemos

目录如下：

```java
JavabasicDemos
  |-.idea
  |-src
    |-main
    | |-java
    | |  |-org.example
    | |   |-Main.java
    | |-resource
    |-test
```

把上面的 org.example 改成 org.basicdemo。然后在 org.basicdemo 下新建 reflect/reflectdemo1.java,student.java 文件，目录如下：

![image-20220904203411041](https://img2022.cnblogs.com/blog/650581/202209/650581-20220905202716761-1115440410.png)

**第二步**：编写 student.java 代码

```java
package org.basicdemo.reflect;

public class student {
    private String name = "Tom";
    private int age;

    public  student(){}

    public student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    private void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

**第三步**：编写 reflectdemo1.java

```java
package org.basicdemo.reflect;

public class reflectdemo1 {
    public static void main(String[] args) {
        try {
            Class<?> stuclz1 = Class.forName("org.basicdemo.reflect.student");
            System.out.println("Class.forName: " + stuclz1);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

        student stu = new student();
        Class stuclz2 = stu.getClass();
        System.out.println("对象.getClass(): " + stuclz2);

        Class stuclz3 = student.class;
        System.out.println("类名.class: " + stuclz3);
    }
}

```

点击IDEA上运行程序的绿色三角形按钮，输出如下：

```java
Class.forName: class org.basicdemo.reflect.student
对象.getClass(): class org.basicdemo.reflect.student
类名.class: class org.basicdemo.reflect.student
```

>此demo完整详细代码在 github 上：[reflect demo](https://github.com/jiujuan/learning-java-demos/tree/main/JavaBasicDemos/src/main/java/org/basicdemo/reflect)



**其他一些常用写法：**

> Class<?> clazz = Class.forName("Student");
>
> System.out.println(clazz);
>
> // or
>
> Class cls = Class.forName("Student"); 	
>
> System.out.println(cls);
>
> // 以前最常用反射获取jdbc
>
>
> Class.forName("com.mysql.jdbc.Driver.class").



**Class 类的其它一些方法：**

- getName() - 获取完整的类名，包括包名
- getSimpleName() - 获取类名，不包括包名
- isInterface() - 判断 Class 对象是否表示一个接口
- getInterfaces() - 表示 Class 对象所引用的类所实现的所有接口
- isInstance() - 判断是否为某个类的实例

更多方法请查看 Class 的文档：https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html

### 5.2 通过反射创建类实例方法

1. newInstance() 方法，Class 对象提供的 newInstance() 方法，来创建 Class 对象对应类的实例

```java
Class<Student> clz = Studnt.class
Student stu = clz.newInstance()
// 或者
Student stu = Student.class.newInstance()

//==========
Class<?> c = String.class;
Object str = c.newInstance();
```

2. 通过 Class 对象的构造器

通过 Class 对象获取 Constructor，再调用 Constructor 对象的 newInstance() 方法来创建对象

```java
// 获取构造方法 Integer(int)
Construct construct1 = Integer.class.getConstructor(int.class)
Integer n1 = construct1.newInstance(123) // 调用构造方法
System.out.println(n1);

// 获取构造方法 Integer(String)
Constructor construct2 = Integer.class.getConstructor(String.class);
Integer n2 = (Integer) construct2.newInstance("567");
System.out.println(n2);
```

两种方法区别：

> newInstance() 的局限是，它只能调用该类的 public 无参数构造方法。如果构造方法带有参数，或者不是 public，就无法直接通过Class.newInstance() 来调用。
>
> 为了调用任意的构造方法，反射 API 提供了 Constructor 对象，它包含一个构造方法的所有信息，可以创建一个实例。

### 5.3 反射获取构造方法

通过反射来获取构造方法，然后使用。

java 反射里的 [Constructor](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Constructor.html) 类，Class Constructor<T>，这个 Constructor 类表示的是 Class 对象所表示的类的构造方法。

https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Constructor.html。

```java
java.lang.Object
     java.lang.reflect.AccessibleObject
          java.lang.reflect.Executable
               java.lang.reflect.Constructor<T>
```



通过 Class 类来获取类的构造方法主要有 4 个，分别为获取单个构造方法和获取多个构造方法。

- 方法如下：

| 方法                                                         | 使用说明                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| public Constructor<?>[] [getConstructors()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getConstructors--) | 返回所有public权限的构造函数的对象数组。                     |
| public Constructor<?>[] [getDeclaredConstructors()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredConstructors--) | 获取所有构造函数方法，包括所有权限public，private，protected，default权限。 |
| public Constructor<T> [getConstructor(Class<?>... parameterTypes)](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getConstructor-java.lang.Class...-) | 获取单个public权限的构造方法。                               |
| public Constructor<T> [getDeclaredConstructor(Class<?>... parameterTypes)](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredConstructor-java.lang.Class...-) | 获取单个构造方法，包括所有权限public，private，protected，default权限。 |

- 调用构造方法：

> newInstance(Object... initargs) ，使用此 Constructor 对象表示的构造方法，用它来创建类的新实例，并用指定的初始化参数初始化该实例。上面也有讲到过该方法使用。



**写一个 demo 例子**：

> 此demo完整详细代码在 github 上：[github-reflect construct](https://github.com/jiujuan/learning-java-demos/tree/main/JavaBasicDemos/src/main/java/org/basicdemo/reflectconstructor)，下面说说编写代码步骤。

**第一步**：新建一个 reflectconstructor 包，然后新建 2 个 java 文件，reflectconstructdemo1.java 和 student.java，如下图

![image-20220904215527983](https://img2022.cnblogs.com/blog/650581/202209/650581-20220905202716764-268648228.png)

**第二步**：把上一小节的 student.java 代码复制过来，然后增加几个构造函数

```java
student(String name) {
   System.out.println("(student)private construct-age: "+age);
}
    
public student(){
    System.out.println("no args");
}

public student(String name, int age) {
    this.name = name;
    this.age = age;
}

private student(int age) {
    System.out.println("private construct-age: "+age);
}
```

**第三步**：在 reflectconstructdemo1.java 里编写获取构造函数方法

首先获取 Class 对象：

```java
Class stuclz = Class.forName("org.basicdemo.reflectconstructor.student");
```



获取所有公有构造函数方法 getConstructors()

```java
// 获取所有公有(public)构造方法
System.out.println("===========获取所有公有构造方法=========");
Constructor[] consarr = stuclz.getConstructors();
for(Constructor c : consarr) {
    System.out.println(c);
}
```



获取所有的构造函数方法 getDeclaredConstructors()

```java
// 获取所有(public,protected,private,default)的构造方法
System.out.println("===========获取所有的构造方法=========");
Constructor[] consall = stuclz.getDeclaredConstructors();
for(Constructor c : consall) {
    System.out.println(c);
}
```



获取单个构造函数方法(公有、无参的方法)

```java
// 获取单个构造方法，公有无参的构造方法
System.out.println("===========获取单个公有、无参数的构造方法=========");
try {
    Constructor con = stuclz.getConstructor(null);
    System.out.println("con: " + con);
} catch (NoSuchMethodException e) {
    e.printStackTrace();
}
```



获取单个私有private构造方法

```java
System.out.println("===========获取单个私有private构造方法=========");
Constructor con = stuclz.getDeclaredConstructor(int.class);
System.out.println(con);
// 调用设置能访问
con.setAccessible(true); // 因为是私有，所以必须设置能访问
// 创建 student 对象
student stu = (student) con.newInstance(12);
stu.setAge(13);
System.out.println("age: "+stu.getAge());
```

> 此demo完整详细代码在 github 上：[github-reflect constructor](https://github.com/jiujuan/learning-java-demos/tree/main/JavaBasicDemos/src/main/java/org/basicdemo/reflectconstructor)

### 5.4 获取成员字段 Field

通过 Class 类提供的方法来获取成员字段信息，主要方法有 4 种，也分为获取单个和多个成员，是公有还是私有，还是所有权限都能获取。

- 方法如下：

| 方法                                                         | 使用说明                                              |
| ------------------------------------------------------------ | ----------------------------------------------------- |
| public Field [getField(String name)](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getField-java.lang.String-) | 根据名字获取单个public权限的字段                      |
| public Field [getDeclaredField](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredField-java.lang.String-)(String name) | 根据名字获取某个字段，字段权限可以是所有，包括private |
| public Field[] [getFields](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getFields--)() | 获取所有public权限字段                                |
| public Field[] [getDeclaredFields](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredFields--)() | 获取所有字段，字段权限可以是所有，包括private         |

- 写一个 demo 例子

**第一步**：创建一个 reflectfield 的包，然后新建 2 个 java 文件，reflectfielddemo1.java 和 student.java，如下图

![image-20220905133010915](https://img2022.cnblogs.com/blog/650581/202209/650581-20220905202716763-1435117244.png)

**第二步**：把上一小节 reflectconstructor 包里的 student 类代码复制到这里的 student.java 里，然后添加几个字段

```java
public String address;

public int grade;

protected String email;

String phone;
```

**第三步**：编写获取字段的方法

获取 Class 对象

```java
// 获取 Class 对象
Class stuClz = Class.forName("org.basicdemo.reflectfield.student");
```

获取所有字段的方法

```java
// 获取所有 public 权限的字段
System.out.println("==========获取所有 public 权限的字段===========");
Field[] fieldArr = stuClz.getFields();
for(Field f : fieldArr) {
    System.out.println(f+" - ("+f.getDeclaringClass() +") - ("+f.getName()+":"+f.getType()+")");
}

// 获取所有权限的字段，包括private
System.out.println("==========获取所有权限的字段，包括private===========");
Field[] fieldsArr = stuClz.getDeclaredFields();
for(Field f : fieldsArr) {
    System.out.println(f);
}
```

获取单个字段方法

```java
// 根据名字获取单个public字段
System.out.println("===========根据名字获取public字段============");
Field addressField = stuClz.getField("address");
System.out.println(addressField);
// 根据反射来设置下这个字段
Object obj = stuClz.getConstructor().newInstance();
// 用 set 方法来设置字段的值
addressField.set(obj, "setTestValue");
// 打印设置的值
student stu = (student) obj;
System.out.println("print address value: " + stu.address);

// 根据名字获取某个字段，字段权限包括所有，也包括private
System.out.println("=========根据名字获取某个字段，字段权限包括所有，也包括private=======");
// 来获取一个 private 字段
Field nameField = stuClz.getDeclaredField("name");
System.out.println(nameField);
// 没有设置前的name值
System.out.println("name value before setting: "+stu.getName());
// 来设置值
nameField.setAccessible(true); // 因为是private，所以先要设置可访问。相当于打开一个开关，原本是不可以写的。
nameField.set(obj, "jimmy");
System.out.println("name value after setting: " + stu.getName());
```

IDEA 上代码运行输出：

```shell
==========获取所有 public 权限的字段===========
public java.lang.String org.basicdemo.reflectfield.student.address - (class org.basicdemo.reflectfield.student) - (address:class java.lang.String)
public int org.basicdemo.reflectfield.student.grade - (class org.basicdemo.reflectfield.student) - (grade:int)
==========获取所有权限的字段，包括private===========
private java.lang.String org.basicdemo.reflectfield.student.name
private int org.basicdemo.reflectfield.student.age
public java.lang.String org.basicdemo.reflectfield.student.address
public int org.basicdemo.reflectfield.student.grade
protected java.lang.String org.basicdemo.reflectfield.student.email
java.lang.String org.basicdemo.reflectfield.student.phone
===========根据名字获取public字段============
public java.lang.String org.basicdemo.reflectfield.student.address
no args
print address value: setTestValue
=========根据名字获取某个字段，字段权限包括所有，也包括private=======
private java.lang.String org.basicdemo.reflectfield.student.name
name value before setting: Tom
print name value after setting: jimmy
```



- 关于 [Field](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Field.html) 的一些其他常用操作方法：

| 方法名                             | 使用说明                                                     |
| ---------------------------------- | ------------------------------------------------------------ |
| Object get(Object obj)             | 返回指定对象上字段值                                         |
| void set(Object obj, Object value) | 指定对象上为field字段设置新值                                |
| Class<?> getType()                 | field字段表示的声明的字段类型                                |
| Class<?> getDeclaringClass()       | field字段所在类的Class对象                                   |
| String getName()                   | 返回字段名称                                                 |
| void setAccessible(boolean flag)   | 对象的 accessible 标志设置，可以设置字段的访问性。比如设置为true表示其可写 |

更多方法可以查看 [Field](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Field.html) 类的 API: https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Field.html

> 此demo完整详细代码在 github 上：[github-reflect field](https://github.com/jiujuan/learning-java-demos/tree/main/JavaBasicDemos/src/main/java/org/basicdemo/reflectfield)

### 5.5 获取成员方法 Method

通过 Class 类获取 Method 对象的方法，与上面获取字段field方法相似，也有 4 种，分为获取单个和获取多个方法。

| 方法                                                         | 使用说明                             |
| ------------------------------------------------------------ | ------------------------------------ |
| public Method[] [getMethods](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getMethods--)() | 获取所有public的方法，包含父类的方法 |
| public Method [getDeclaredMethod](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredMethod-java.lang.String-java.lang.Class...-)(String name, Class<?> ...parameterTypes) | 获取所有的方法，包括private          |
| public Method [getMethod](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getMethod-java.lang.String-java.lang.Class...-)(String name, Class<?>... parameterTypes) | 获取单个public的方法                 |
| public Method [getDeclaredMethod](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredMethod-java.lang.String-java.lang.Class...-)(String name, Class<?>... parameterTypes) | 根据名字获取所有的方法               |

- demo 例子

**第一步**：与上一小节filed一样，先新建一个 reflectmethod 包，然后在里面新建 2 个 java 文件，reflectmethoddemo1.java 和 student.java，如下图：

![image-20220905190723904](https://img2022.cnblogs.com/blog/650581/202209/650581-20220905202716770-814577147.png)

**第二步**：把上一小节的 student.java 代码复制到这里

并增加一个基础 student.java 的class TomStudent，

```java
class TomStudent extends student {
    private void printName() {
        System.out.println("a student name: Tom");
    }

    public int getTomAge() {
        return 12;
    }

    public void setTomeAge(int age) {
        this.setAge(age);
    }
}
```

**第三步**：编写获取方法的代码

获取多个的方法

```java
// 获取所有public method方法
System.out.println("=============获取所有public method方法，包括继承父类的===============");
Method[] methodArr = stuClz.getMethods();
for(Method m:methodArr) {
    System.out.println(m); // 不仅打印出了 TomStudent 所有 public 方法，它继承的方法也打印出来
}
```

获取单个的方法：

```java
// 根据参数获取public的方法，包含继承自父类的方法
System.out.println("=======根据参数获取public的方法，包含继承自父类的方法======");
Method method = stuClz.getMethod("setAge", int.class);
System.out.println(method);
```

详细的demo代码查看：[github reflect method](https://github.com/jiujuan/learning-java-demos/tree/main/JavaBasicDemos/src/main/java/org/basicdemo/reflectmethod)

- 方法调用 invoke

```java
// 根据参数获取public的方法，包含继承自父类的方法
System.out.println("=======根据参数获取public的方法，包含继承自父类的方法======");
Method method = stuClz.getMethod("setAge", int.class);
System.out.println(method);
// 反射调用方法
Object obj = stuClz.newInstance();
method.invoke(obj, 12);
student stu = (student)obj;
System.out.println(stu.getAge());
```

如果是调用private方法，一定要设置 `setAccessible(true)`。

> 说明：上面所有的代码以 github 上的为准：[reflect demo系列](https://github.com/jiujuan/learning-java-demos/tree/main/JavaBasicDemos/src/main/java/org/basicdemo)

## 六、反射有哪些用途

- **动态代理**：动态代理可以通过反射来实现。
- **注解**：注解也是用反射来实现。注解利用反射机制来调用注解的解释器。
- **开发框架**：比如 Spring 框架。Spring 中的 XML 配置 Bean 等。

等用途。

## 七、参考

- [《深入理解java虚拟机》](https://book.douban.com/subject/34907497/) 类加载机制一节 作者: 周志明
- https://www.liaoxuefeng.com/wiki/1252599548343744/1255945147512512 反射的各种方法
- https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Member.html reflect member
- https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html Class api
- https://docs.oracle.com/javase/tutorial/reflect/TOC.html
- https://www.liaoxuefeng.com/wiki/1252599548343744/1260454185794944 构造方法
- https://github.com/jiujuan/learning-java-demos demo 代码地址
