### ArrayList And Vector

List,就如图名字所示一样,是元素的有序列表。当我们讨论List时，将其与Set作对比是一个很好的办法,Set集合中的元素是无序且唯一的。

List接口下一共实现了三个类：ArrayList，Vector，LinkedList。LinkedList就不多说了，它一般主要用在保持数据的插入顺序的时候。ArrayList和Vector都是用数组实现的，主要有这么三个区别：

1、Vector是多线程安全的，而ArrayList不是，这个可以从源码中看出，Vector类中的方法很多有synchronized进行修饰，这样就导致了Vector在效率上无法与ArrayList相比；

2、两个都是采用的线性连续空间存储元素，但是当空间不足的时候，两个类的增加方式是不同的，很多网友说Vector增加原来空间的一倍，ArrayList增加原来空间的50%，其实也差不多是这个意思，不过还有一点点问题可以从源码中看出，一会儿从源码中分析。

3、Vector可以设置增长因子，而ArrayList不可以，最开始看这个的时候，我没理解什么是增量因子，不过通过对比一下两个源码理解了这个，先看看两个类的构造方法： ArrayList有三个构造方法：分别是

	public ArrayList(int initialCapacity)//构造一个具有指定初始容量的空列表。  
	public ArrayList()//构造一个初始容量为10的空列表。  
	public ArrayList(Collection<? extends E> c)//构造一个包含指定 collection 的元素的列表  
Vector有四个构造方法：

	//使用指定的初始容量和等于零的容量增量构造一个空向量。
	public Vector()  
	//构造一个空向量，使其内部数据数组的大小，其标准容量增量为零。
	public Vector(int initialCapacity)
	//构造一个包含指定 collection 中的元素的向量    
	public Vector(Collection<? extends E> c)
	//使用指定的初始容量和容量增量构造一个空的向量  
	public Vector(int initialCapacity,int capacityIncrement)

Vector比Arraylist多一个构造方法，没错就是
	
	public Vector(int initialCapacity,int capacityIncrement)

这个构造方法
capacityIncrement就是容量增长，即前面所说的增长因子，ArrayList中是没有的。 再贴出两个类的添加源码分析下（jdk1.7版本）：

ArrayList类：

	public boolean add(E e) {  
	    ensureCapacityInternal(size + 1);  // Increments modCount!!  
	    elementData[size++] = e;  
	    return true;  
	}  

	private void ensureCapacityInternal(int minCapacity) {  
	    modCount++;  
	    // overflow-conscious code  
	    //如果添加一个元素之后，新容器的大小大于容器的容量，那么就无法存值了，需要扩充空间  
	    if (minCapacity - elementData.length > 0)  
	        grow(minCapacity);  
	}  

	private void grow(int minCapacity) {  
	    // overflow-conscious code  
	    int oldCapacity = elementData.length;  
	    //扩充的空间增加原来的50%（即是原来的1.5倍）  
	    int newCapacity = oldCapacity + (oldCapacity >> 1);
	    //如果容器扩容之后还是不够，那么干脆直接将minCapacity设为容器的大小   
	    if (newCapacity - minCapacity < 0) 
	        newCapacity = minCapacity;  
	    //如果扩充的容器太大了的话，那么就执行hugeCapacity  
	    if (newCapacity - MAX_ARRAY_SIZE > 0) 
	        newCapacity = hugeCapacity(minCapacity);  
	    // minCapacity is usually close to size, so this is a win:  
	    elementData = Arrays.copyOf(elementData, newCapacity);  
	}  
	
jdk1.6版本是这样的：  

	 public void ensureCapacity(int minCapacity) {  
	      modCount++;  
	      int oldCapacity = elementData.length;  
	      if (minCapacity > oldCapacity) {  
	           Object oldData[] = elementData;  
	           int newCapacity = (oldCapacity * 3) / 2 + 1;  
	           if (newCapacity < minCapacity)  
	                newCapacity = minCapacity;  
	           // minCapacity is usually close to size, so this is a win:  
	           elementData = Arrays.copyOf(elementData, newCapacity);  
	      }  
	 } 

Vector类：

	 public synchronized boolean add(E e) {  
	     modCount++;  
	     ensureCapacityHelper(elementCount + 1);  
	     elementData[elementCount++] = e;  
	     return true;  
	 }  

	 private void ensureCapacityHelper(int minCapacity) {  
	     // overflow-conscious code  
	     if (minCapacity - elementData.length > 0)  
	         grow(minCapacity);  
	 }  

	 private void grow(int minCapacity) {  
	     // overflow-conscious code  
	     int oldCapacity = elementData.length;  
	     int newCapacity = oldCapacity + ((capacityIncrement > 0) ?  
	                                      capacityIncrement : oldCapacity);  
	
	     /** 
	     这个扩容需要做个判断：如果容量增量初始化的不是0，即使用的public Vector(int initialCapacity,int capacityIncrement) 
	     构造方法进行的初始化，那么扩容的容量是(oldCapacity+capacityIncrement)，就是原来的容量加上容量增量的值； 
	     如果没有设置容量增量，那么扩容后的容量就是(oldCapacity+oldCapacity)，就是原来容量的二倍。 
	     **/  
	     if (newCapacity - minCapacity < 0)  
	         newCapacity = minCapacity;  
	     if (newCapacity - MAX_ARRAY_SIZE > 0)  
	         newCapacity = hugeCapacity(minCapacity);  
	     elementData = Arrays.copyOf(elementData, newCapacity);  
	 }  
	
jdk1.6版本是这样的：       

	private void ensureCapacityHelper(int minCapacity) {  
	       int oldCapacity = elementData.length;  
	       if (minCapacity > oldCapacity) {  
	            Object[] oldData = elementData;  
	            int newCapacity = (capacityIncrement > 0) ? (oldCapacity + capacityIncrement)  
	                      : (oldCapacity * 2);//方式与jdk1.7一样  
	            if (newCapacity < minCapacity) {  
	                 newCapacity = minCapacity;  
	            }  
	            elementData = Arrays.copyOf(elementData, newCapacity);  
	       }  
	  }
  
ArrayList和Vector都是使用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，它们都允许直接按序号索引元素，但是插入元素要涉及数组元素移动等内存操作，所以索引数据快而插入数据慢，Vector由于使用了synchronized方法(线程安全)，通常性能上较ArrayList差。

**关于LinkedList**

LinkedList 是一个双链表,在添加和删除元素时具有比ArrayList更好的性能.但在get与set方面弱于ArrayList.当然,这些对比都是指数据量很大或者操作很频繁的情况下的对比,如果数据和运算量很小,那么对比将失去意义.

而 LinkedList 还实现了 Queue 接口,该接口比List提供了更多的方法,包括 offer(),peek(),poll()等.注意: 默认情况下ArrayList的初始容量非常小,所以如果可以预估数据量的话,分配一个较大的初始值属于最佳实践,这样可以减少调整大小的开销。

**什么时候使用ArrayList或Vector**

在什么时候使用ArrayList或Vector，它完全取决于你的需求，如果你需要执行一个线程安全的操作，那么Vector是最好的选择，它保证你同一个时刻只有一个线程访问你的集合。

性能：同步操作相比没有同步操作消耗更多的时间，所以如果你不需要线程安全的操作，ArrayList将会是更好的选择，它将会因为并发进程提高性能。

**怎么让ArrayList同步**

使用Collecions.synzhonizedList

	List list = Collections.synchronizedList(new ArrayList());
	... 
	//If you wanna use iterator on the synchronized list, use it
	//like this. It should be in synchronized block.
	synchronized (list) {
	  Iterator iterator = list.iterator();
	  while (iterator.hasNext())
	      ...
	      iterator.next();
	      ...
	}

**ArrayList,Vector, LinkedList的存储性能和特性**

ArrayList 和 Vector 都是使用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素， 它们都允许直接按序号索引元素，但是插入元素要涉及数组元素移动等内存操作，所以索引数据快而插入数据慢，Vector 由于使用了 synchronized 方法（线程安全） ，通常性能上较 ArrayList 差，而 LinkedList 使用双向链表实现存储，按序号索引数据需要进行前向或后向遍历， 但是插入数据时只需要记录本项的前后项即可，所以插入速度较快。LinkedList 也是线程不安全的，LinkedList 提供了一些方法，使得LinkedList 可以被当作堆栈和队列来使用。