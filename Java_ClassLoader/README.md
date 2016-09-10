### Java CLassLoader

类加载器（class loader）是 Java™中的一个很重要的概念。类加载器负责加载 Java 类的字节代码到 Java 虚拟机中。本文首先详细介绍了 Java 类加载器的基本概念，包括代理模式、加载类的具体过程和线程上下文类加载器等，接着介绍如何开发自己的类加载器，最后介绍了类加载器在 Web 容器和 OSGi™中的应用。

类加载器是 Java 语言的一个创新，也是 Java 语言流行的重要原因之一。它使得 Java 类可以被动态加载到 Java 虚拟机中并执行。类加载器从 JDK 1.0 就出现了，最初是为了满足 Java Applet 的需要而开发出来的。Java Applet 需要从远程下载 Java 类文件到浏览器中并执行。现在类加载器在 Web 容器和 OSGi 中得到了广泛的使用。一般来说，Java 应用的开发人员不需要直接同类加载器进行交互。Java 虚拟机默认的行为就已经足够满足大多数情况的需求了。不过如果遇到了需要与类加载器进行交互的情况，而对类加载器的机制又不是很了解的话，就很容易花大量的时间去调试 ClassNotFoundException和NoClassDefFoundError等异常。本文将详细介绍 Java 的类加载器，帮助读者深刻理解 Java 语言中的这个重要概念。下面首先介绍一些相关的基本概念。

**类的加载过程**

JVM将类加载过程分为三个步骤：装载（Load），链接（Link）和初始化(Initialize)链接又分为三个步骤，如下图所示：

![](https://github.com/silence940109/Java/blob/master/image/load.gif)

1) 装载：查找并加载类的二进制数据；

2)链接：

	验证：确保被加载类的正确性；
	准备：为类的静态变量分配内存，并将其初始化为默认值；
	解析：把类中的符号引用转换为直接引用；

3)初始化：为类的静态变量赋予正确的初始值；

那为什么我要有验证这一步骤呢？首先如果由编译器生成的class文件，它肯定是符合JVM字节码格式的，但是万一有高手自己写一个class文件，让JVM加载并运行，用于恶意用途，就不妙了，因此这个class文件要先过验证这一关，不符合的话不会让它继续执行的，也是为了安全考虑吧。

准备阶段和初始化阶段看似有点牟盾，其实是不牟盾的，如果类中有语句：private static int a = 10，它的执行过程是这样的，首先字节码文件被加载到内存后，先进行链接的验证这一步骤，验证通过后准备阶段，给a分配内存，因为变量a是static的，所以此时a等于int类型的默认初始值0，即a=0,然后到解析（后面在说），到初始化这一步骤时，才把a的真正的值10赋给a,此时a=10。

**类的初始化**

类什么时候才被初始化：

* 1）创建类的实例，也就是new一个对象

* 2）访问某个类或接口的静态变量，或者对该静态变量赋值

* 3）调用类的静态方法

* 4）反射（Class.forName("com.lyj.load")）

* 5）初始化一个类的子类（会首先初始化子类的父类）

* 6）JVM启动时标明的启动类，即文件名和类名相同的那个类

只有这6中情况才会导致类的类的初始化。

类的初始化步骤：

1）如果这个类还没有被加载和链接，那先进行加载和链接

2）假如这个类存在直接父类，并且这个类还没有被初始化（注意：在一个类加载器中，类只能初始化一次），那就初始化直接的父类（不适用于接口）

3)加入类中存在初始化语句（如static变量和static块），那就依次执行这些初始化语句。

**类的加载**

类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个这个类的java.lang.Class对象，用来封装类在方法区类的对象。看下面图

![](https://github.com/silence940109/Java/blob/master/image/classLoader.gif)

![](https://github.com/silence940109/Java/blob/master/image/classLoader1.gif)

类的加载的最终产品是位于堆区中的Class对象

Class对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口
加载类的方式有以下几种：

1）从本地系统直接加载

2）通过网络下载.class文件

3）从zip，jar等归档文件中加载.class文件

4）从专有数据库中提取.class文件

5）将Java源文件动态编译为.class文件（服务器）

**类加载器基本概念**

顾名思义，类加载器（class loader）用来加载 Java 类到 Java 虚拟机中。一般来说，Java 虚拟机使用 Java 类的方式如下：Java 源程序（.java 文件）在经过 Java 编译器编译之后就被转换成 Java 字节代码（.class 文件）。类加载器负责读取 Java 字节代码，并转换成java.lang.Class类的一个实例。每个这样的实例用来表示一个 Java 类。通过此实例的 newInstance()方法就可以创建出该类的一个对象。实际的情况可能更加复杂，比如 Java 字节代码可能是通过工具动态生成的，也可能是通过网络下载的。

**加载器**

来自[http://blog.csdn.net/cutesource/article/details/5904501](http://blog.csdn.net/cutesource/article/details/5904501)

JVM的类加载是通过ClassLoader及其子类来完成的，类的层次关系和加载顺序可以由下图来描述：

![](https://github.com/silence940109/Java/blob/master/image/loader.gif)

1）Bootstrap ClassLoader
负责加载$JAVA_HOME中jre/lib/rt.jar里所有的class，由C++实现，不是ClassLoader子类

2）Extension ClassLoader
负责加载java平台中扩展功能的一些jar包，包括$JAVA_HOME中jre/lib/*.jar或-Djava.ext.dirs指定目录下的jar包

3）App ClassLoader
负责记载classpath中指定的jar包及目录中class

4）Custom ClassLoader

属于应用程序根据自身需要自定义的ClassLoader，如tomcat、jboss都会根据j2ee规范自行实现ClassLoader

加载过程中会先检查类是否被已加载，检查顺序是自底向上，从Custom ClassLoader到BootStrap 
ClassLoader逐层检查，只要某个classloader已加载就视为已加载此类，保证此类只所有
ClassLoader加载一次。而加载的顺序是自顶向下，也就是由上层来逐层尝试加载此类。

基本上所有的类加载器都是 java.lang.ClassLoader类的一个实例。下面详细介绍这个 Java 类。

**java.lang.ClassLoader类介绍**

java.lang.ClassLoader类的基本职责就是根据一个指定的类的名称，找到或者生成其对应的字节代码，然后从这些字节代码中定义出一个 Java 类，即 java.lang.Class类的一个实例。除此之外，ClassLoader还负责加载 Java 应用所需的资源，如图像文件和配置文件等。不过本文只讨论其加载类的功能。为了完成加载类的这个职责，ClassLoader提供了一系列的方法，比较重要的方法如 表 1所示。关于这些方法的细节会在下面进行介绍。


* getParent()									返回该类加载器的父类加载器。

* loadClass(String name)	加载名称为 name的类，返回的结果是 java.lang.Class类的实例。

* findClass(String name)	查找名称为 name的类，返回的结果是 java.lang.Class类的实例。

* findLoadedClass(String name)	查找名称为 name的已经被加载过的类，返回的结果是 java.lang.Class类的实例。

* defineClass(String name, byte[] b, int off, int len)	把字节数组 b中的内容转换成 Java 类，返回的结果是 java.lang.Class类的实例。这个方法被声明为 final的。

* resolveClass(Class<?> c)	链接指定的 Java 类。

对于 表 1中给出的方法，表示类名称的 name参数的值是类的二进制名称。需要注意的是内部类的表示，如 com.example.Sample$1和com.example.Sample$Inner等表示方式。这些方法会在下面介绍类加载器的工作机制时，做进一步的说明。下面介绍类加载器的树状组织结构。

**类加载器的树状组织结构**

Java 中的类加载器大致可以分成两类，一类是系统提供的，另外一类则是由 Java 应用开发人员编写的。系统提供的类加载器主要有下面三个：

