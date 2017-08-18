### Java IO 流 ###

#### 一、Java IO 体系结构 ####

#### IO 流的分类 ####
* 根据处理数据类型的不同分为：字符流和字节流
* 根据数据流向不同分为：输入流和输出流

![](https://github.com/scalad/Note/blob/master/Java_IO/image/Java_IO.png)

#### 流的概念 ####
流是对数据传输的总称或抽象，流的本质是数据传输，根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作。

#### 字符流和字节流的区别 ####

* 读写单位不同：字节流以字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节
* 处理对象不同：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型的数据



![](https://github.com/scalad/Note/blob/master/Java_IO/image/Java_IO_Detail.png)

![](https://github.com/scalad/Note/blob/master/Java_IO/image/input.png)

![](https://github.com/scalad/Note/blob/master/Java_IO/image/output.png)