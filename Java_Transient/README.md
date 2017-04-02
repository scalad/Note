#### Java中Transient关键字
虽然自己最熟的是Java，但很多Java基础知识都不知道，比如transient关键字以前都没用到过，所以不知道它的作用是什么，今天做笔试题时发现有一题是关于这个的，于是花个时间整理下transient关键字的使用，涨下姿势~~~好了，废话不多说，下面开始：

#### 1. transient的作用及使用方法
我们都知道一个对象只要实现了Serilizable接口，这个对象就可以被序列化，java的这种序列化模式为开发者提供了很多便利，我们可以不必关系具体序列化的过程，只要这个类实现了Serilizable接口，这个类的所有属性和方法都会自动序列化。

然而在实际开发过程中，我们常常会遇到这样的问题，这个类的有些属性需要序列化，而其他属性不需要被序列化，打个比方，如果一个用户有一些敏感信息（如密码，银行卡号等），为了安全起见，不希望在网络操作（主要涉及到序列化操作，本地序列化缓存也适用）中被传输，这些信息对应的变量就可以加上transient关键字。换句话说，这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。

总之，java 的transient关键字为我们提供了便利，你只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。

**示例code如下:**

	import java.io.FileInputStream;
	import java.io.FileNotFoundException;
	import java.io.FileOutputStream;
	import java.io.IOException;
	import java.io.ObjectInputStream;
	import java.io.ObjectOutputStream;
	import java.io.Serializable;
	
	/**
	 * @description 使用transient关键字不序列化某个变量
	 *        注意读取的时候，读取数据的顺序一定要和存放数据的顺序保持一致
	 */
	public class TransientTest {
	    
	    public static void main(String[] args) {
	        
	        User user = new User();
	        user.setUsername("Alexia");
	        user.setPasswd("123456");
	        
	        System.out.println("read before Serializable: ");
	        System.out.println("username: " + user.getUsername());
	        System.err.println("password: " + user.getPasswd());
	        
	        try {
	            ObjectOutputStream os = new ObjectOutputStream(
	                    new FileOutputStream("C:/user.txt"));
	            os.writeObject(user); // 将User对象写进文件
	            os.flush();
	            os.close();
	        } catch (FileNotFoundException e) {
	            e.printStackTrace();
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	        try {
	            ObjectInputStream is = new ObjectInputStream(new FileInputStream(
	                    "C:/user.txt"));
	            user = (User) is.readObject(); // 从流中读取User的数据
	            is.close();
	            
	            System.out.println("\nread after Serializable: ");
	            System.out.println("username: " + user.getUsername());
	            System.err.println("password: " + user.getPasswd());
	            
	        } catch (FileNotFoundException e) {
	            e.printStackTrace();
	        } catch (IOException e) {
	            e.printStackTrace();
	        } catch (ClassNotFoundException e) {
	            e.printStackTrace();
	        }
	    }
	}
	
	class User implements Serializable {
	    private static final long serialVersionUID = 8294180014912103005L;  
	    
	    private String username;
	    private transient String passwd;
	    
	    public String getUsername() {
	        return username;
	    }
	    
	    public void setUsername(String username) {
	        this.username = username;
	    }
	    
	    public String getPasswd() {
	        return passwd;
	    }
	    
	    public void setPasswd(String passwd) {
	        this.passwd = passwd;
	    }
	
	}

输出为：

	read before Serializable: 
	username: Alexia
	password: 123456
	
	read after Serializable: 
	username: Alexia
	password: null

密码字段为null，说明反序列化时根本没有从文件中获取到信息。

#### 2. transient使用小结

1）一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

2）transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口。

3）被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。

第三点可能有些人很迷惑，因为发现在User类中的username字段前加上static关键字后，程序运行结果依然不变，即static类型的username也读出来为“Alexia”了，这不与第三点说的矛盾吗？实际上是这样的：第三点确实没错（一个静态变量不管是否被transient修饰，均不能被序列化），反序列化后类中static型变量username的值为当前JVM中对应static变量的值，这个值是JVM中的不是反序列化得出的，不相信？好吧，下面我来证明：

	import java.io.FileInputStream;
	import java.io.FileNotFoundException;
	import java.io.FileOutputStream;
	import java.io.IOException;
	import java.io.ObjectInputStream;
	import java.io.ObjectOutputStream;
	import java.io.Serializable;
	
	/**
	 * @description 使用transient关键字不序列化某个变量
	 *        注意读取的时候，读取数据的顺序一定要和存放数据的顺序保持一致
	 */
	public class TransientTest {
	    
	    public static void main(String[] args) {
	        
	        User user = new User();
	        user.setUsername("Alexia");
	        user.setPasswd("123456");
	        
	        System.out.println("read before Serializable: ");
	        System.out.println("username: " + user.getUsername());
	        System.err.println("password: " + user.getPasswd());
	        
	        try {
	            ObjectOutputStream os = new ObjectOutputStream(
	                    new FileOutputStream("C:/user.txt"));
	            os.writeObject(user); // 将User对象写进文件
	            os.flush();
	            os.close();
	        } catch (FileNotFoundException e) {
	            e.printStackTrace();
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	        try {
	            // 在反序列化之前改变username的值
	            User.username = "jmwang";
	            
	            ObjectInputStream is = new ObjectInputStream(new FileInputStream(
	                    "C:/user.txt"));
	            user = (User) is.readObject(); // 从流中读取User的数据
	            is.close();
	            
	            System.out.println("\nread after Serializable: ");
	            System.out.println("username: " + user.getUsername());
	            System.err.println("password: " + user.getPasswd());
	            
	        } catch (FileNotFoundException e) {
	            e.printStackTrace();
	        } catch (IOException e) {
	            e.printStackTrace();
	        } catch (ClassNotFoundException e) {
	            e.printStackTrace();
	        }
	    }
	}
	
	class User implements Serializable {
	    private static final long serialVersionUID = 8294180014912103005L;  
	    
	    public static String username;
	    private transient String passwd;
	    
	    public String getUsername() {
	        return username;
	    }
	    
	    public void setUsername(String username) {
	        this.username = username;
	    }
	    
	    public String getPasswd() {
	        return passwd;
	    }
	    
	    public void setPasswd(String passwd) {
	        this.passwd = passwd;
	    }
	
	}

运行结果为：

	read before Serializable: 
	username: Alexia
	password: 123456
	
	read after Serializable: 
	username: jmwang
	password: null

这说明反序列化后类中static型变量username的值为当前JVM中对应static变量的值，为修改后jmwang，而不是序列化时的值Alexia。

#### 3. transient使用细节——被transient关键字修饰的变量真的不能被序列化吗？

思考下面的例子：

	import java.io.Externalizable;
	import java.io.File;
	import java.io.FileInputStream;
	import java.io.FileOutputStream;
	import java.io.IOException;
	import java.io.ObjectInput;
	import java.io.ObjectInputStream;
	import java.io.ObjectOutput;
	import java.io.ObjectOutputStream;
	
	/**
	 * @descripiton Externalizable接口的使用
	 */
	public class ExternalizableTest implements Externalizable {
	
	    private transient String content = "是的，我将会被序列化，不管我是否被transient关键字修饰";
	
	    @Override
	    public void writeExternal(ObjectOutput out) throws IOException {
	        out.writeObject(content);
	    }
	
	    @Override
	    public void readExternal(ObjectInput in) throws IOException,
	            ClassNotFoundException {
	        content = (String) in.readObject();
	    }
	
	    public static void main(String[] args) throws Exception {
	        
	        ExternalizableTest et = new ExternalizableTest();
	        ObjectOutput out = new ObjectOutputStream(new FileOutputStream(
	                new File("test")));
	        out.writeObject(et);
	
	        ObjectInput in = new ObjectInputStream(new FileInputStream(new File(
	                "test")));
	        et = (ExternalizableTest) in.readObject();
	        System.out.println(et.content);
	
	        out.close();
	        in.close();
	    }
	}

content变量会被序列化吗？好吧，我把答案都输出来了，是的，运行结果就是：
	
	是的，我将会被序列化，不管我是否被transient关键字修饰

这是为什么呢，不是说类的变量被transient关键字修饰以后将不能序列化了吗？

我们知道在Java中，对象的序列化可以通过实现两种接口来实现，若实现的是Serializable接口，则所有的序列化将会自动进行，若实现的是Externalizable接口，则没有任何东西可以自动序列化，需要在writeExternal方法中进行手工指定所要序列化的变量，这与是否被transient修饰无关。因此第二个例子输出的是变量content初始化的内容，而不是null。

#### 关于java.io.Serializable序列化
Java API中java.io.Serializable接口源码：

     public interface Serializable {

     }

类通过实现java.io.Serializable接口可以启用其序列化功能。未实现次接口的类无法使其任何状态序列化或反序列化。可序列化类的所有子类型本身都是可序列化的。序列化接口没有方法或字段，仅用于标识可序列化的语义。

Java的"对象序列化"能让你将一个实现了Serializable接口的对象转换成byte流，这样日后要用这个对象时候，你就能把这些byte数据恢复出来，并据此重新构建那个对象了。

要想序列化对象，你必须先创建一个OutputStream，然后把它嵌进ObjectOutputStream。这时，你就能用writeObject()方法把对象写入OutputStream了。

writeObject()方法负责写入特定类的对象的状态，以便相应的 readObject()方法可以还原它。通过调用 out.defaultWriteObject 可以调用保存 Object 的字段的默认机制。该方法本身不需要涉及属于其超类或子类的状态。状态是通过使用 writeObject 方法或使用 DataOutput 支持的用于基本数据类型的方法将各个字段写入 ObjectOutputStream 来保存的。

读的时候，你得把InputStream嵌到ObjectInputStream里面，然后再调用readObject()方法。不过这样读出来的，只是一个Object的reference，因此在用之前，还得先下传。readObject() 方法负责从流中读取并还原类字段。它可以调用 in.defaultReadObject 来调用默认机制，以还原对象的非静态和非瞬态字段。　 defaultReadObject()方法使用流中的信息来分配流中通过当前对象中相应命名字段保存的对象的字段。这用于处理类发展后需要添加新字段的情形。该方法本身不需要涉及属于其超类或子类的状态。状态是通过使用 writeObject 方法或使用 DataOutput 支持的用于基本数据类型的方法将各个字段写入 ObjectOutputStream 来保存的。

**在序列化时，有几点要注意的：**

* 当一个对象被序列化时，只保存对象的非静态成员变量（包括声明为private的变量），不能保存任何的成员方法和静态的成员变量。
* 如果一个对象的成员变量是一个对象，那么这个对象的数据成员也会被序列化。
* 如果一个可序列化的对象包含对某个不可序列化的对象的引用，那么整个序列化操作将会失败，并且会抛出一个NotSerializableException。我们可以将这个引用标记为transient，那么对象仍然可以序列化。


**1、序列化是干什么的？**

简单说就是为了保存在内存中的各种对象的状态，并且可以把保存的对象状态再读出来。虽然你可以用你自己的各种各样的方法来保存Object States，但是Java给你提供一种应该比你自己好的保存对象状态的机制,那就是序列化。

**2、什么情况下需要序列化**

a）当你想把的内存中的对象保存到一个文件中或者数据库中时候；

b）当你想用套接字在网络上传送对象的时候；

c）当你想通过RMI传输对象的时候；


**3、当对一个对象实现序列化时，究竟发生了什么？**
在没有序列化前，每个保存在堆（Heap）中的对象都有相应的状态（state），即实例变量（instance ariable）比如：

	 Foo myFoo = new Foo();
	 myFoo .setWidth(37);
	 myFoo.setHeight(70);

当通过下面的代码序列化之后，MyFoo对象中的width和Height实例变量的值（37，70）都被保存到foo.ser文件中，这样以后又可以把它从文件中读出来，重新在堆中创建原来的对象。当然保存时候不仅仅是保存对象的实例变量的值，JVM还要保存一些小量信息，比如类的类型等以便恢复原来的对象。

	FileOutputStream fs = new FileOutputStream("foo.ser");
	ObjectOutputStream os = new ObjectOutputStream(fs);
	os.writeObject(myFoo);

**4、实现序列化（保存到一个文件）的步骤**
	
	FileOutputStream fs = new FileOutputStream("foo.ser");
	ObjectOutputStream os = new ObjectOutputStream(fs);
	os.writeObject(myObject1);
	os.writeObject(myObject2);
	os.writeObject(myObject3);
	os.close();

**6、相关注意事项**

a）当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；

b）当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；

c）并非所有的对象都可以序列化，至于为什么不可以，有很多原因了,比如：

* 安全方面的原因，比如一个对象拥有private，public等field，对于一个要传输的对象，比如写到文件，或者进行rmi传输 等等，在序列化进行传输的过程中，这个对象的private等域是不受保护的。

* 资源分配方面的原因，比如socket，thread类，如果可以序列化，进行传输或者保存，也无法对他们进行重新的资源分配，而且，也是没有必要这样实现。


