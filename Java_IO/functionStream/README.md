### 节点流和过滤流 ###
#### 节点流 ####
节点流(Node Stream)是流管道两端直接连接data source和data destination上的，即为取放数据的真实载体，在流通道本身不对数据做任何加工，因而也被称为低级流。

![](https://github.com/scalad/Note/blob/master/Java_IO/functionStream/image/node_stream.png)

#### 过滤流 ####
过滤流(Filter Stream)是套在节点流或过滤上的，而且过滤流是不能够脱离节点流（低级流）存在的，因此称为高级流。过滤流的流管道本身封装方法，能够对低级流数据进行处理，因此也被称为处理流。究其本质，过滤流就是把不符合通过管道条件的数据过滤掉，而让满足条件的数据通过过滤流管道。

![](https://github.com/scalad/Note/blob/master/Java_IO/functionStream/image/filter_stream.png)