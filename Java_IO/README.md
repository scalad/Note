### JAVA IO 流 ###

#### 一、Java IO 体系结构 ####

#### (1). 流的概念 ####
流是对数据传输的总称或抽象，流的本质是数据传输，是数据的有序排列,根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作。

![](https://github.com/scalad/Note/blob/master/Java_IO/image/io_stream.jpg)

在C++中，我们将数据从一个对象到另一个对象的流动抽象为"流"。Java继承C++的流机制，不过在具体实现上有别，Java中的"流"就是指把数据从一个对象移动到另一个对象的流动模式的抽象。(专业术语就是拿来装逼的)![](https://github.com/scalad/Note/blob/master/Java_IO/image/emoji1.png)

James Gosling的Java流模式图与水流模式图概念映射。数据源（data source）即水库，数据目的地(data destination)就是脸盆，数据（data）就是水，流（stream）实例化就是在管子中流动的水流。输入流（input stream）就是用水泵从水库中抽出来要到水管中的水，输出流（output stream）经过水龙头将要达到脸盆中的水，计算机内存（memory）就是上图中的水流管道，关闭输入流（close input stream）就是关闭水泵开关，关闭输出流（close output stream）就是关闭关闭水龙头开关.

![](https://github.com/scalad/Note/blob/master/Java_IO/image/io_stream1.png)

#### (2). IO 流的分类 ####
* 根据处理数据类型的不同分为：[(3). 字符流和字节流的区别](#字符流和字节流的区别)
* 根据数据流向不同分为：[输入流和输出流](https://github.com/scalad/Note/tree/master/Java_IO/inputOutputStream)
* 根据流的功能来分：[节点流（又称低级流）、过滤流（又称高级流、处理流、包装流）](https://github.com/scalad/Note/tree/master/Java_IO/functionStream)

![](https://github.com/scalad/Note/blob/master/Java_IO/image/Java_IO.png)

#### (3). 字符流和字节流的区别 ####

* 读写单位不同：字节流以字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节,在Java流的处理上，字符流处理的单元为 2 个字节的 Unicode 字符，分别操作字符、字符数组或字符串，而字节流处理单元为 1 个字节，操作字节和字节数组
* 处理对象不同：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型的数据
* 字节流默认是不带缓冲区的，而字符流默认是带缓冲区的
* 字节流是底层数据流，是数据有意义的最小单位。字符流是字节流的包装，底层实现是字节流

> 提醒：如果只是处理纯文本数据的话优先使用字符流，其他的使用字符流

字节流和字符流在物理层面的实现都是比特流，二进制数据流可以认为是字节流，而字符流是遵循unicode编码规则的字节流。因此计算机中的"流"概念实际上就是指字节数据（bytes data）从源对象对按顺序流向目标对象的一种流动形式

![](https://github.com/scalad/Note/blob/master/Java_IO/image/Java_IO_Detail.png)

![](https://github.com/scalad/Note/blob/master/Java_IO/image/input.png)

![](https://github.com/scalad/Note/blob/master/Java_IO/image/output.png)