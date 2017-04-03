### Java异常处理机制

#### 1. 引子

 try…catch…finally恐怕是大家再熟悉不过的语句了，而且感觉用起来也是很简单，逻辑上似乎也是很容易理解。不过，我亲自体验的“教训”告诉我，这个东西可不是想象中的那么简单、听话。不信？那你看看下面的代码，“猜猜”它执行后的结果会是什么？不要往后看答案、也不许执行代码看真正答案哦。如果你的答案是正确，那么这篇文章你就不用浪费时间看啦。

	package Test;  
	  
	public class TestException {  
	    public TestException() {  
	    }  
	  
	    boolean testEx() throws Exception {  
	        boolean ret = true;  
	        try {  
	            ret = testEx1();  
	        } catch (Exception e) {  
	            System.out.println("testEx, catch exception");  
	            ret = false;  
	            throw e;  
	        } finally {  
	            System.out.println("testEx, finally; return value=" + ret);  
	            return ret;  
	        }  
	    }  
	  
	    boolean testEx1() throws Exception {  
	        boolean ret = true;  
	        try {  
	            ret = testEx2();  
	            if (!ret) {  
	                return false;  
	            }  
	            System.out.println("testEx1, at the end of try");  
	            return ret;  
	        } catch (Exception e) {  
	            System.out.println("testEx1, catch exception");  
	            ret = false;  
	            throw e;  
	        } finally {  
	            System.out.println("testEx1, finally; return value=" + ret);  
	            return ret;  
	        }  
	    }  
	  
	    boolean testEx2() throws Exception {  
	        boolean ret = true;  
	        try {  
	            int b = 12;  
	            int c;  
	            for (int i = 2; i >= -2; i--) {  
	                c = b / i;  
	                System.out.println("i=" + i);  
	            }  
	            return true;  
	        } catch (Exception e) {  
	            System.out.println("testEx2, catch exception");  
	            ret = false;  
	            throw e;  
	        } finally {  
	            System.out.println("testEx2, finally; return value=" + ret);  
	            return ret;  
	        }  
	    }  
	  
	    public static void main(String[] args) {  
	        TestException testException1 = new TestException();  
	        try {  
	            testException1.testEx();  
	        } catch (Exception e) {  
	            e.printStackTrace();  
	        }  
	    }  
	}  

你的答案是什么？是下面的答案吗？

	i=2
	i=1
	testEx2, catch exception
	testEx2, finally; return value=false
	testEx1, catch exception
	testEx1, finally; return value=false
	testEx, catch exception
	testEx, finally; return value=false

如果你的答案真的如上面所说，那么你错啦。^_^，那就建议你仔细看一看这篇文章或者拿上面的代码按各种不同的情况修改、执行、测试，你会发现有很多事情不是原来想象中的那么简单的。现在公布正确答案：

	i=2
	i=1
	testEx2, catch exception
	testEx2, finally; return value=false
	testEx1, finally; return value=false
	testEx, finally; return value=false

注意说明：

finally语句块不应该出现 应该出现return。上面的return ret最好是其他语句来处理相关逻辑。

#### 2.JAVA异常

异常指不期而至的各种状况，如：文件找不到、网络连接失败、非法参数等。异常是一个事件，它发生在程序运行期间，干扰了正常的指令流程。Java通 过API中Throwable类的众多子类描述各种不同的异常。因而，Java异常都是对象，是Throwable子类的实例，描述了出现在一段编码中的 错误条件。当条件生成时，错误将引发异常。

Java异常类层次结构图：

![](https://github.com/silence940109/Java/blob/master/image/java_exception.jpg)

在 Java 中，所有的异常都有一个共同的祖先 Throwable（可抛出）。Throwable 指定代码中可用异常传播机制通过 Java 应用程序传输的任何问题的共性。

**Throwable**： 有两个重要的子类：Exception（异常）和 Error（错误），二者都是 Java 异常处理的重要子类，各自都包含大量子类。

 **Error（错误）**:是程序无法处理的错误，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟机）出现的问题。例如，Java虚拟机运行错误（Virtual MachineError），当 JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。

这些错误表示故障发生于虚拟机自身、或者发生在虚拟机试图执行应用时，如Java虚拟机运行错误（Virtual MachineError）、类定义错误（NoClassDefFoundError）等。这些错误是不可查的，因为它们在应用程序的控制和处理能力之 外，而且绝大多数是程序运行时不允许出现的状况。对于设计合理的应用程序来说，即使确实发生了错误，本质上也不应该试图去处理它所引起的异常状况。在 Java中，错误通过Error的子类描述。

**Exception（异常）**:是程序本身可以处理的异常。

 Exception 类有一个重要的子类 RuntimeException。RuntimeException 类及其子类表示“JVM 常用操作”引发的错误。例如，若试图使用空值对象引用、除数为零或数组越界，则分别引发运行时异常（NullPointerException、ArithmeticException）和 ArrayIndexOutOfBoundException。

>注意：异常和错误的区别：异常能被程序本身可以处理，错误是无法处理。

通常，Java的异常(包括Exception和Error)分为可查的异常（checked exceptions）和不可查的异常（unchecked exceptions）。

**可查异常（编译器要求必须处置的异常）**：正确的程序在运行中，很容易出现的、情理可容的异常状况。可查异常虽然是异常状况，但在一定程度上它的发生是可以预计的，而且一旦发生这种异常状况，就必须采取某种方式进行处理。

除了RuntimeException及其子类以外，其他的Exception类及其子类都属于可查异常。这种异常的特点是Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。

**不可查异常(编译器不要求强制处置的异常)**:包括运行时异常（RuntimeException与其子类）和错误（Error）。
 
Exception 这种异常分两大类运行时异常和非运行时异常(编译异常)。程序中应当尽可能去处理这些异常。

**运行时异常**：都是RuntimeException类及其子类异常，如NullPointerException(空指针异常)、IndexOutOfBoundsException(下标越界异常)等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。

运行时异常的特点是Java编译器不会检查它，也就是说，当程序中可能出现这类异常，即使没有用try-catch语句捕获它，也没有用throws子句声明抛出它，也会编译通过。

**非运行时异常 （编译异常）**：是RuntimeException以外的异常，类型上都属于Exception类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如IOException、SQLException等以及用户自定义的Exception异常，一般情况下不自定义检查异常。

#### 4.处理异常机制
在 Java 应用程序中，异常处理机制为：抛出异常，捕捉异常。

**抛出异常**：当一个方法出现错误引发异常时，方法创建异常对象并交付运行时系统，异常对象中包含了异常类型和异常出现时的程序状态等异常信息。运行时系统负责寻找处置异常的代码并执行。

