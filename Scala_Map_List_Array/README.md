### scala map/list/array/的常用内置遍历操作总结

Scala 是面向函数的，所以在集合函数里，它很轻易地提供了非常丰富遍历操作，数组变换操作。这对于我们数据挖掘，爬虫，文本处理等都非常有帮助。有了这些内置的遍历变换操作，我们再也不用像java那样写一个笨笨的for循环来迭代，然后还要定义一些规范的函数来迎合需求。而scala不同，随时随地就可以写一个你自己想要的函数，而不需要严格地定义它，规范它。（注意，scala里一切事物皆函数，一切函数皆对象）

下面将提供一些集合内置遍历方法用法，熟练运用，常常使得代码精简整洁。

### 一、常用遍历变换操作
#### 1.map 遍历
map[B](f: (A) ⇒ B): List[B]

定义一个变换,把该变换应用到列表的每个元素中,原列表不变，返回一个新的列表数据

Example1 平方变换
	val nums = List(1,2,3)  
	val square = (x: Int) => x*x     
	val squareNums1 = nums.map(num => num*num)    //List(1,4,9)  
	val squareNums2 = nums.map(math.pow(_,2))    //List(1,4,9)  
	val squareNums3 = nums.map(square)            //List(1,4,9)  

Example2 保存文本数据中的某几列

	val text = List("Homeway,25,Male","XSDYM,23,Female")  
	val usersList = text.map(_.split(",")(0))      
	val usersWithAgeList = text.map(line => {  
	    val fields = line.split(",")  
	    val user = fields(0)  
	    val age = fields(1).toInt  
	    (user,age)  
	})  

#### 2.flatMap，flatten扁平化
flatten: flatten[B]: List[B] 对列表的列表进行平坦化操作 flatMap: flatMap[B](f: (A) ⇒ GenTraversableOnce[B]): List[B] map之后对结果进行flatten

定义一个变换f, 把f应用列表的每个元素中，每个f返回一个列表，最终把所有列表连结起来。

	val text = List("A,B,C","D,E,F")  
	val textMapped = text.map(_.split(",").toList) // List(List("A","B","C"),List("D","E","F"))  
	val textFlattened = textMapped.flatten          // List("A","B","C","D","E","F")  
	val textFlatMapped = text.flatMap(_.split(",").toList) // List("A","B","C","D","E","F")  

#### 3.reduce遍历简化
定义一个变换f, f把两个列表的元素合成一个，遍历列表，最终把列表合并成单一元素

Example 列表求和

	val nums = List(1,2,3)  
	val sum1 = nums.reduce((a,b) => a+b)   //6  
	val sum2 = nums.reduce(_+_)            //6  
	val sum3 = nums.sum                 //6  

	reduceLeft: reduceLeft[B >: A](f: (B, A) ⇒ B): B

	reduceRight: reduceRight[B >: A](op: (A, B) ⇒ B): B

reduceLeft从列表的左边往右边应用reduce函数，reduceRight从列表的右边往左边应用reduce函数

Example
	
	val nums = List(2.0,2.0,3.0)  
	val resultLeftReduce = nums.reduceLeft(math.pow)  // = pow( pow(2.0,2.0) , 3.0) = 64.0  
	val resultRightReduce = nums.reduceRight(math.pow) // = pow(2.0, pow(2.0,3.0)) = 256.0

#### 4.Fold折叠
fold: fold[A1 >: A](z: A1)(op: (A1, A1) ⇒ A1): A1 带有初始值的reduce,从一个初始值开始，从左向右将两个元素合并成一个，最终把列表合并成单一元素。

foldLeft: foldLeft[B](z: B)(f: (B, A) ⇒ B): B 带有初始值的reduceLeft

foldRight: foldRight[B](z: B)(op: (A, B) ⇒ B): B 带有初始值的reduceRight

	val nums = List(2,3,4)  
	val sum = nums.fold(1)(_+_)  // = 1+2+3+4 = 9  
	  
	val nums = List(2.0,3.0)  
	val result1 = nums.foldLeft(4.0)(math.pow) // = pow(pow(4.0,2.0),3.0) = 4096  
	val result2 = nums.foldRight(1.0)(math.pow) // = pow(1.0,pow(2.0,3.0)) = 8.0    

#### 5.sort排序
sortBy: sortBy[B](f: (A) ⇒ B)(implicit ord: math.Ordering[B]): List[A] 按照应用函数f之后产生的元素进行排序

sorted： sorted[B >: A](implicit ord: math.Ordering[B]): List[A] 按照元素自身进行排序

sortWith： sortWith(lt: (A, A) ⇒ Boolean): List[A] 使用自定义的比较函数进行排序

	val nums = List(1,3,2,4)  
	val sorted = nums.sorted  //List(1,2,3,4)  
	  
	val users = List(("HomeWay",25),("XSDYM",23))  
	val sortedByAge = users.sortBy{case(user,age) => age}  //List(("XSDYM",23),("HomeWay",25))  
	val sortedWith = users.sortWith{case(user1,user2) => user1._2 < user2._2} //List(("XSDYM",23),("HomeWay",25)) 

#### 6.filter过滤
count(p: (A) ⇒ Boolean): Int

计算列表中所有满足条件p的元素的个数，等价于 filter(p).length

val nums = List(-1,-2,0,1,2) val plusCnt1 = nums.count( > 0) val plusCnt2 = nums.filter( > 0).length 

#### 8.diff，union，intersect两个集合的交集联结

diff:diff(that: collection.Seq[A]): List[A] 保存列表中那些不在另外一个列表中的元素，即从集合中减去与另外一个集合的交集

union : union(that: collection.Seq[A]): List[A] 与另外一个列表进行连结

intersect: intersect(that: collection.Seq[A]): List[A] 与另外一个集合的交集
	
	val nums1 = List(1,2,3)  
	val nums2 = List(2,3,4)  
	val diff1 = nums1 diff nums2   // List(1)  
	val diff2 = nums2.diff(num1)   // List(4)  
	val union1 = nums1 union nums2  // List(1,2,3,2,3,4)  
	val union2 = nums2 ++ nums1        // List(2,3,4,1,2,3)  
	val intersection = nums1 intersect nums2  //List(2,3)  

#### 9.distinct去重

distinct: List[A] 保留列表中非重复的元素，相同的元素只会被保留一次

	val list = List("A","B","C","A","B") val distincted = list.distinct // List("A","B","C")

#### 10.group分组

groupBy : groupBy[K](f: (A) ⇒ K): Map[K, List[A]] 将列表进行分组，分组的依据是应用f在元素上后产生的新元素 
grouped: grouped(size: Int): Iterator[List[A]] 按列表按照固定的大小进行分组  

	val data = List(("HomeWay","Male"),("XSDYM","Femail"),("Mr.Wang","Male"))  
	val group1 = data.groupBy(_._2) // = Map("Male" -> List(("HomeWay","Male"),("Mr.Wang","Male")),"Female" -> List(("XSDYM","Femail")))  
	val group2 = data.groupBy{case (name,sex) => sex} // = Map("Male" -> List(("HomeWay","Male"),("Mr.Wang","Male")),"Female" -> List(("XSDYM","Femail")))  
	val fixSizeGroup = data.grouped(2).toList // = Map("Male" -> List(("HomeWay","Male"),("XSDYM","Femail")),"Female" -> List(("Mr.Wang","Male")))  

#### 11.scan扫描

scan[B >: A, That](z: B)(op: (B, B) ⇒ B)(implicit cbf: CanBuildFrom[List[A], B, That]): That

由一个初始值开始，从左向右，进行积累的op操作，这个比较难解释，具体的看例子吧。
	
	val nums = List(1,2,3)  
	val result = nums.scan(10)(_+_)   // List(10,10+1,10+1+2,10+1+2+3) = List(10,11,12,13)  

scanLeft: scanLeft[B, That](z: B)(op: (B, A) ⇒ B)(implicit bf: CanBuildFrom[List[A], B, That]): That

scanRight: scanRight[B, That](z: B)(op: (A, B) ⇒ B)(implicit bf: CanBuildFrom[List[A], B, That]): That

scanLeft: 从左向右进行scan函数的操作，scanRight：从右向左进行scan函数的操作

	val nums = List(1.0,2.0,3.0)  
	val result = nums.scanLeft(2.0)(math.pow)   // List(2.0,pow(2.0,1.0), pow(pow(2.0,1.0),2.0),pow(pow(pow(2.0,1.0),2.0),3.0) = List(2.0,2.0,4.0,64.0)  
	val result = nums.scanRight(2.0)(math.pow)  // List(2.0,pow(3.0,2.0), pow(2.0,pow(3.0,2.0)), pow(1.0,pow(2.0,pow(3.0,2.0))) = List(1.0,512.0,9.0,2.0)

#### 12.take截取

take : takeRight(n: Int): List[A] 提取列表的前n个元素 takeRight: takeRight(n: Int): List[A] 提取列表的最后n个元素 takeWhile: takeWhile(p: (A) ⇒ Boolean): List[A] 从左向右提取列表的元素，直到条件p不成立
	
	val nums = List(1,1,1,1,4,4,4,4)  
	val left = nums.take(4)   // List(1,1,1,1)  
	val right = nums.takeRight(4) // List(4,4,4,4)  
	val headNums = nums.takeWhile( _ == nums.head)  // List(1,1,1,1)    

#### 13.drop丢弃

drop: drop(n: Int): List[A] 丢弃前n个元素，返回剩下的元素 dropRight: dropRight(n: Int): List[A] 丢弃最后n个元素，返回剩下的元素 dropWhile: dropWhile(p: (A) ⇒ Boolean): List[A] 从左向右丢弃元素，直到条件p不成立
	
	val nums = List(1,1,1,1,4,4,4,4)  
	val left = nums.drop(4)   // List(4,4,4,4)  
	val right = nums.dropRight(4) // List(1,1,1,1)  
	val tailNums = nums.dropWhile( _ == nums.head)  // List(4,4,4,4)  

#### 14.span，spliAt，partition拆分

span : span(p: (A) ⇒ Boolean): (List[A], List[A]) 从左向右应用条件p进行判断，直到条件p不成立，此时将列表分为两个列表

splitAt: splitAt(n: Int): (List[A], List[A]) 将列表分为前n个，与，剩下的部分

partition: partition(p: (A) ⇒ Boolean): (List[A], List[A]) 将列表分为两部分，第一部分为满足条件p的元素，第二部分为不满足条件p的元素

	val nums = List(1,1,1,2,3,2,1)  
	val (prefix,suffix) = nums.span( _ == 1) // prefix = List(1,1,1), suffix = List(2,3,2,1)  
	val (prefix,suffix) = nums.splitAt(3)  // prefix = List(1,1,1), suffix = List(2,3,2,1)  
	val (prefix,suffix) = nums.partition( _ == 1) // prefix = List(1,1,1,1), suffix = List(2,3,2  

#### 15.padTo批量扩展

padTo(len: Int, elem: A): List[A]

将列表扩展到指定长度，长度不够的时候，使用elem进行填充，否则不做任何操作。
	
	val nums = List(1,1,1)  
	val padded = nums.padTo(6,2)   // List(1,1,1,2,2,2)  

#### 16.combinations,permutations随机组合

combinations: combinations(n: Int): Iterator[List[A]] 取列表中的n个元素进行组合，返回不重复的组合列表，结果一个迭代器

permutations: permutations: Iterator[List[A]] 对列表中的元素进行排列，返回不重得的排列列表，结果是一个迭代器

	val nums = List(1,1,3)  
	val combinations = nums.combinations(2).toList //List(List(1,1),List(1,3))  
	val permutations = nums.permutations.toList        // List(List(1,1,3),List(1,3,1),List(3,1,1)) 

#### 17.zip打包

zip: zip[B](that: GenIterable[B]): List[(A, B)] 与另外一个列表进行拉链操作，将对应位置的元素组成一个pair，返回的列表长度为两个列表中短的那个

zipAll: zipAll[B](that: collection.Iterable[B], thisElem: A, thatElem: B): List[(A, B)] 与另外一个列表进行拉链操作，将对应位置的元素组成一个pair，若列表长度不一致，自身列表比较短的话使用thisElem进行填充，对方列表较短的话使用thatElem进行填充

zipWithIndex：zipWithIndex: List[(A, Int)] 将列表元素与其索引进行拉链操作，组成一个pair

unzip: unzip[A1, A2](implicit asPair: (A) ⇒ (A1, A2)): (List[A1], List[A2]) 解开拉链操作

unzip3: unzip3[A1, A2, A3](implicit asTriple: (A) ⇒ (A1, A2, A3)): (List[A1], List[A2], List[A3]) 3个元素的解拉链操作
	
	val alphabet = List("A",B","C")  
	val nums = List(1,2)  
	val zipped = alphabet zip nums   // List(("A",1),("B",2))  
	val zippedAll = alphabet.zipAll(nums,"*",-1)   // List(("A",1),("B",2),("C",-1))  
	val zippedIndex = alphabet.zipWithIndex  // List(("A",0),("B",1),("C",3))  
	val (list1,list2) = zipped.unzip        // list1 = List("A","B"), list2 = List(1,2)  
	val (l1,l2,l3) = List((1, "one", '1'),(2, "two", '2'),(3, "three", '3')).unzip3   // l1=List(1,2,3),l2=List("one","two","three"),l3=List('1','2','3')  

#### 18.slice提取

slice(from: Int, until: Int): List[A] 提取列表中从位置from到位置until(不含该位置)的元素列表
	
	val nums = List(1,2,3,4,5)  
	val sliced = nums.slice(2,4)  //List(3,4)  

#### 19.sliding按步长分组（不同于group）

sliding(size: Int, step: Int): Iterator[List[A]] 将列表按照固定大小size进行分组，步进为step，step默认为1,返回结果为迭代器

	val nums = List(1,1,2,2,3,3,4,4)  
	val groupStep2 = nums.sliding(2,2).toList  //List(List(1,1),List(2,2),List(3,3),List(4,4))  
	val groupStep1 = nums.sliding(2).toList //List(List(1,1),List(1,2),List(2,2),List(2,3),List(3,3),List(3,4),List(4,4))  

#### 20.updte更新（对于List产生新对象）

updated(index: Int, elem: A): List[A] 对列表中的某个元素进行更新操作
	
	val nums = List(1,2,3,3)
	val fixed = nums.updated(3,4) // List(1,2,3,4)

#### 21.contains，exits包含存在

	List(1,2,3) contains 1
	Set(1,2,3).exists(x=>x==1);    

  
