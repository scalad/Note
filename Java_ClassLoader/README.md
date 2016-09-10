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

* 引导类加载器（bootstrap class loader）：它用来加载 Java 的核心库，是用原生代码来实现的，并不继承自 java.lang.ClassLoader。

* 扩展类加载器（extensions class loader）：它用来加载 Java 的扩展库。Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类。

* 系统类加载器（system class loader）：它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它。
 
除了系统提供的类加载器以外，开发人员可以通过继承 java.lang.ClassLoader类的方式实现自己的类加载器，以满足一些特殊的需求。

除了引导类加载器之外，所有的类加载器都有一个父类加载器。通过 表 1中给出的 getParent()方法可以得到。对于系统提供的类加载器来说，系统类加载器的父类加载器是扩展类加载器，而扩展类加载器的父类加载器是引导类加载器；对于开发人员编写的类加载器来说，其父类加载器是加载此类加载器 Java 类的类加载器。因为类加载器 Java 类如同其它的 Java 类一样，也是要由类加载器来加载的。一般来说，开发人员编写的类加载器的父类加载器是系统类加载器。类加载器通过这种方式组织起来，形成树状结构。树的根节点就是引导类加载器。图 1中给出了一个典型的类加载器树状组织结构示意图，其中的箭头指向的是父类加载器。

类加载器树状组织结构示意图

![](https://github.com/silence940109/Java/blob/master/image/clasloader.jpg)

演示类加载器的树状组织结构

	public class ClassLoaderTree {   
	    public static void main(String[] args) {   
	        ClassLoader loader = ClassLoaderTree.class.getClassLoader();   
	        while (loader != null) {   
	            System.out.println(loader.toString());   
	            loader = loader.getParent();   
	        }   
	    }   
	 }  

每个 Java 类都维护着一个指向定义它的类加载器的引用，通过 getClassLoader()方法就可以获取到此引用。代码清单 1中通过递归调用getParent()方法来输出全部的父类加载器。代码清单 1的运行结果如 代码清单 2所示。

演示类加载器的树状组织结构的运行结果

	sun.misc.Launcher$AppClassLoader@9304b1   
	sun.misc.Launcher$ExtClassLoader@190d11  

如 代码清单所示，第一个输出的是 ClassLoaderTree类的类加载器，即系统类加载器。它是 sun.misc.Launcher$AppClassLoader类的实例；第二个输出的是扩展类加载器，是 sun.misc.Launcher$ExtClassLoader类的实例。需要注意的是这里并没有输出引导类加载器，这是由于有些 JDK 的实现对于父类加载器是引导类加载器的情况，getParent()方法返回 null。

在了解了类加载器的树状组织结构之后，下面介绍类加载器的代理模式。

**类加载器的代理模式**

类加载器在尝试自己去查找某个类的字节代码并定义它时，会先代理给其父类加载器，由父类加载器先去尝试加载这个类，依次类推。在介绍代理模式背后的动机之前，首先需要说明一下 Java 虚拟机是如何判定两个 Java 类是相同的。Java 虚拟机不仅要看类的全名是否相同，还要看加载此类的类加载器是否一样。只有两者都相同的情况，才认为两个类是相同的。即便是同样的字节代码，被不同的类加载器加载之后所得到的类，也是不同的。比如一个 Java 类 com.example.Sample，编译之后生成了字节代码文件 Sample.class。两个不同的类加载器ClassLoaderA和 ClassLoaderB分别读取了这个 Sample.class文件，并定义出两个 java.lang.Class类的实例来表示这个类。这两个实例是不相同的。对于 Java 虚拟机来说，它们是不同的类。试图对这两个类的对象进行相互赋值，会抛出运行时异常ClassCastException。下面通过示例来具体说明。代码清单 3中给出了 Java 类 com.example.Sample。

Sample 类

	public class Sample {   
	   private Sample instance;   
	  
	   public void setSample(Object instance) {   
	       this.instance = (Sample) instance;   
	   }   
	}

Sample类的方法 setSample接受一个 java.lang.Object类型的参数，并且会把该参数强制转换成Sample类型。测试 Java 类是否相同的代码如所示。

测试 Java 类是否相同
	
	public void testClassIdentity() {   
	    String classDataRootPath = "C:\\workspace\\Classloader\\classData";   
	    FileSystemClassLoader fscl1 = new FileSystemClassLoader(classDataRootPath);   
	    FileSystemClassLoader fscl2 = new FileSystemClassLoader(classDataRootPath);   
	    String className = "com.example.Sample";      
	    try {   
	        Class<?> class1 = fscl1.loadClass(className);   
	        Object obj1 = class1.newInstance();   
	        Class<?> class2 = fscl2.loadClass(className);   
	        Object obj2 = class2.newInstance();   
	        Method setSampleMethod = class1.getMethod("setSample", java.lang.Object.class);   
	        setSampleMethod.invoke(obj1, obj2);   
	    } catch (Exception e) {   
	        e.printStackTrace();   
	    }   
    } 

代码清单 4中使用了类 FileSystemClassLoader的两个不同实例来分别加载类 Sample，得到了两个不同的java.lang.Class的实例，接着通过 newInstance()方法分别生成了两个类的对象 obj1和 obj2，最后通过 Java 的反射 API 在对象 obj1上调用方法 setSample，试图把对象 obj2赋值给 obj1内部的 instance对象。代码清单 4的运行结果如 代码清单 5所示。

测试 Java 类是否相同的运行结果

	package com.example;   
	  
	public class Sample {   
	   private Sample instance;   
	  
	   public void setSample(Object instance) {   
	       this.instance = (Sample) instance;   
	   }   
	}  

如 代码清单 3所示，com.example.Sample类的方法 setSample接受一个 java.lang.Object类型的参数，并且会把该参数强制转换成com.example.Sample类型。测试 Java 类是否相同的代码如 代码清单 4所示。

测试 Java 类是否相同

	public void testClassIdentity() {   
	    String classDataRootPath = "C:\\workspace\\Classloader\\classData";   
	    FileSystemClassLoader fscl1 = new FileSystemClassLoader(classDataRootPath);   
	    FileSystemClassLoader fscl2 = new FileSystemClassLoader(classDataRootPath);   
	    String className = "com.example.Sample";      
	    try {   
	        Class<?> class1 = fscl1.loadClass(className);   
	        Object obj1 = class1.newInstance();   
	        Class<?> class2 = fscl2.loadClass(className);   
	        Object obj2 = class2.newInstance();   
	        Method setSampleMethod = class1.getMethod("setSample", java.lang.Object.class);   
	        setSampleMethod.invoke(obj1, obj2);   
	    } catch (Exception e) {   
	        e.printStackTrace();   
	    }   
	 }  

代码清单 4中使用了类 FileSystemClassLoader的两个不同实例来分别加载类com.example.Sample，得到了两个不同的java.lang.Class的实例，接着通过 newInstance()方法分别生成了两个类的对象 obj1和 obj2，最后通过 Java 的反射 API 在对象 obj1上调用方法 setSample，试图把对象 obj2赋值给 obj1内部的 instance对象。代码清单 4的运行结果如 代码清单 5所示。

测试 Java 类是否相同的运行结果

	java.lang.reflect.InvocationTargetException   
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)   
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)   
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)  
	at java.lang.reflect.Method.invoke(Method.java:597)   
	at classloader.ClassIdentity.testClassIdentity(ClassIdentity.java:26)   
	at classloader.ClassIdentity.main(ClassIdentity.java:9)   
	Caused by: java.lang.ClassCastException: com.example.Sample   
	cannot be cast to com.example.Sample   
	at com.example.Sample.setSample(Sample.java:7)   
	... 6 more  

从 代码清单 5给出的运行结果可以看到，运行时抛出了 java.lang.ClassCastException异常。虽然两个对象 obj1和 obj2的类的名字相同，但是这两个类是由不同的类加载器实例来加载的，因此不被 Java 虚拟机认为是相同的。

了解了这一点之后，就可以理解代理模式的设计动机了。代理模式是为了保证 Java 核心库的类型安全。所有 Java 应用都至少需要引用java.lang.Object类，也就是说在运行的时候，java.lang.Object这个类需要被加载到 Java 虚拟机中。如果这个加载过程由 Java 应用自己的类加载器来完成的话，很可能就存在多个版本的 java.lang.Object类，而且这些类之间是不兼容的。通过代理模式，对于 Java 核心库的类的加载工作由引导类加载器来统一完成，保证了 Java 应用所使用的都是同一个版本的 Java 核心库的类，是互相兼容的。

不同的类加载器为相同名称的类创建了额外的名称空间。相同名称的类可以并存在 Java 虚拟机中，只需要用不同的类加载器来加载它们即可。不同类加载器加载的类之间是不兼容的，这就相当于在 Java 虚拟机内部创建了一个个相互隔离的 Java 类空间。这种技术在许多框架中都被用到，后面会详细介绍。

**文件系统类加载器**

第一个类加载器用来加载存储在文件系统上的 Java 字节代码。完整的实现如 代码清单 6所示。
	
	public class FileSystemClassLoader extends ClassLoader {   
	  
	    private String rootDir;   
	  
	    public FileSystemClassLoader(String rootDir) {   
	        this.rootDir = rootDir;   
	    }   
	  
	    protected Class<?> findClass(String name) throws ClassNotFoundException {   
	        byte[] classData = getClassData(name);   
	        if (classData == null) {   
	            throw new ClassNotFoundException();   
	        }   
	        else {   
	            return defineClass(name, classData, 0, classData.length);   
	        }   
	    }   
	  
	    private byte[] getClassData(String className) {   
	        String path = classNameToPath(className);   
	        try {   
	            InputStream ins = new FileInputStream(path);   
	            ByteArrayOutputStream baos = new ByteArrayOutputStream();   
	            int bufferSize = 4096;   
	            byte[] buffer = new byte[bufferSize];   
	            int bytesNumRead = 0;   
	            while ((bytesNumRead = ins.read(buffer)) != -1) {   
	                baos.write(buffer, 0, bytesNumRead);   
	            }   
	            return baos.toByteArray();   
	        } catch (IOException e) {   
	            e.printStackTrace();   
	        }   
	        return null;   
	    }   
	  
	    private String classNameToPath(String className) {   
	        return rootDir + File.separatorChar   
	                + className.replace('.', File.separatorChar) + ".class";   
	    }   
	 }  

如 代码清单 6所示，类 FileSystemClassLoader继承自类 java.lang.ClassLoader。在 表 1中列出的 java.lang.ClassLoader类的常用方法中，一般来说，自己开发的类加载器只需要覆写 findClass(String name)方法即可。java.lang.ClassLoader类的方法loadClass()封装了前面提到的代理模式的实现。该方法会首先调用 findLoadedClass()方法来检查该类是否已经被加载过；如果没有加载过的话，会调用父类加载器的 loadClass()方法来尝试加载该类；如果父类加载器无法加载该类的话，就调用 findClass()方法来查找该类。因此，为了保证类加载器都正确实现代理模式，在开发自己的类加载器时，最好不要覆写 loadClass()方法，而是覆写 findClass()方法。

类 FileSystemClassLoader的 findClass()方法首先根据类的全名在硬盘上查找类的字节代码文件（.class 文件），然后读取该文件内容，最后通过 defineClass()方法来把这些字节代码转换成 java.lang.Class类的实例。

