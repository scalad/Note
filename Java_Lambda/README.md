###Java8 Lambda表达式
**1. 什么是λ表达式**

λ表达式本质上是一个匿名方法。让我们来看下面这个例子：

    public int add(int x, int y) {
        return x + y;
    }

转成λ表达式后是这个样子：

    (int x, int y) -> x + y;

参数类型也可以省略，Java编译器会根据上下文推断出来：

    (x, y) -> x + y; //返回两数之和

或者

    (x, y) -> { return x + y; } //显式指明返回值

可见λ表达式有三部分组成：参数列表，箭头（->），以及一个表达式或语句块。

下面这个例子里的λ表达式没有参数，也没有返回值（相当于一个方法接受0个参数，返回void，其实就是Runnable里run方法的一个实现）：

    () -> { System.out.println("Hello Lambda!"); }

如果只有一个参数且可以被Java推断出类型，那么参数列表的括号也可以省略：

    c -> { return c.size(); }

**2. λ表达式的类型（它是Object吗？）**

λ表达式可以被当做是一个Object（注意措辞）。λ表达式的类型，叫做“目标类型（target type）”。λ表达式的目标类型是“函数接口（functional interface）”，这是Java8新引入的概念。它的定义是：一个接口，如果只有一个显式声明的抽象方法，那么它就是一个函数接口。一般用@FunctionalInterface标注出来（也可以不标）。举例如下：

    @FunctionalInterface
    public interface Runnable { void run(); }

    public interface Callable<V> { V call() throws Exception; }

    public interface ActionListener { void actionPerformed(ActionEvent e); }

    public interface Comparator<T> { int compare(T o1, T o2); boolean equals(Object obj); }

注意最后这个Comparator接口。它里面声明了两个方法，貌似不符合函数接口的定义，但它的确是函数接口。这是因为equals方法是Object的，所有的接口都会声明Object的public方法——虽然大多是隐式的。所以，Comparator显式的声明了equals不影响它依然是个函数接口。

你可以用一个λ表达式为一个函数接口赋值：

    Runnable r1 = () -> {System.out.println("Hello Lambda!");};

然后再赋值给一个Object：

    Object obj = r1;

但却不能这样干：

    Object obj = () -> {System.out.println("Hello Lambda!");}; 
	// ERROR! Object is not a functional interface!

必须显式的转型成一个函数接口才可以：

    Object o = (Runnable) () -> { System.out.println("hi"); }; // correct

