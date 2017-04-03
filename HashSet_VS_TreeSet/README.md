### HashSet And TreeSet

#### Set接口

Set不允许包含相同的元素，如果试图把两个相同元素加入同一个集合中，add方法返回false。
Set判断两个对象相同不是使用==运算符，而是根据equals方法。也就是说，只要两个对象用equals方法比较返回true，Set就不 会接受这两个对象,并且最多包含一个 null 元素。

HashSet与TreeSet都是基于Set接口的实现类。其中TreeSet是Set的子接口SortedSet的实现类。Set接口及其子接口、实现类的结构如下所示：

	 	     |——SortedSet接口——TreeSet实现类
	Set接口——|——HashSet实现类
             		|——LinkedHashSet实现类

#### HashSet
* 不能保证元素的排列顺序，顺序有可能发生变化
* 不是同步的
* 集合元素可以是null,但只能放入一个null

当向HashSet结合中存入一个元素时，HashSet会调用该对象的hashCode()方法来得到该对象的hashCode值，然后根据 hashCode值来决定该对象在HashSet中存储位置。

简单的说，HashSet集合判断两个元素相等的标准是两个对象通过equals方法比较相等，并且两个对象的hashCode()方法返回值相等

注意，如果要把一个对象放入HashSet中，重写该对象对应类的equals方法，也应该重写其hashCode()方法。其规则是如果两个对 象通过equals方法比较返回true时，其hashCode也应该相同。另外，对象中用作equals比较标准的属性，都应该用来计算 hashCode的值。

例如：

#
	import java.util.HashSet;
	import java.util.Iterator;
	public class HashSetTest {
	         public static void main(String[] args){
	                 HashSet hs=new HashSet();
	                 /**//*hs.add("one");
	                 hs.add("two");
	                 hs.add("three");
	                 hs.add("four");*/
	                 hs.add(new Student(1,"zhangsan"));
	                 hs.add(new Student(2,"lishi"));
	                 hs.add(new Student(3,"wangwu"));
	                 hs.add(new Student(1,"zhangsan"));
	                
	                 Iterator it=hs.iterator();
	                 while(it.hasNext()){
	                         System.out.println(it.next());
	                 }
	         }
	}
	class Student{         //HashSet要重写hashCode和equals方法
	         int num;
	         String name;
	         Student(int num,String name){
	                 this.num=num;
	                 this.name=name;
	         }
	         public String toString(){
	                 return "num :"+num+" name:"+name;
	         }
		     public int hashCode(){
	                 return num*name.hashCode();
	         }
	         public boolean equals(Object o){
	                 Student s=(Student)o;
	                 return num==s.num && name.equals(s.name);
	         }
	}
#

#### TreeSet类
TreeSet是SortedSet接口的唯一实现类，TreeSet可以确保集合元素处于排序状态。TreeSet支持两种排序方式，自然排序 和定制排序，其中自然排序为默认的排序方式。向TreeSet中加入的应该是同一个类的对象。

TreeSet判断两个对象不相等的方式是两个对象通过equals方法返回false，或者通过CompareTo方法比较没有返回0

**1.自然排序**

自然排序使用要排序元素的CompareTo（Object obj）方法来比较元素之间大小关系，然后将元素按照升序排列。

Java提供了一个Comparable接口，该接口里定义了一个compareTo(Object obj)方法，该方法返回一个整数值，实现了该接口的对象就可以比较大小。
obj1.compareTo(obj2)方法如果返回0，则说明被比较的两个对象相等，如果返回一个正数，则表明
obj1大于obj2，如果是 负数，则表明obj1小于obj2。

如果我们将两个对象的equals方法总是返回true，则这两个对象的compareTo方法返回应该返回0


**2.定制排序**

自然排序是根据集合元素的大小，以升序排列，如果要定制排序，应该使用Comparator接口，实现 int compare(T o1,T o2)方法,我们可以构造TreeSet对象时,传递实现了Comparator接口的比较器对象.
	
	import java.util.Comparator;
	import java.util.Iterator;
	import java.util.TreeSet;
	
	public class TreeSetTest {
		public static void main(String[] args) {
	
			TreeSet<Students> ts = new TreeSet<Students>(new CompareToStudent());
			ts.add(new Students(2, "zhangshan"));
			ts.add(new Students(3, "lishi"));
			ts.add(new Students(1, "wangwu"));
			ts.add(new Students(4, "maliu"));
	
			Iterator<Students> it = ts.iterator();
			while (it.hasNext()) {
				System.out.println(it.next());
			}
		}
	}
	
	class Students implements Comparable<Students> {
		int num;
		String name;
	
		Students(int num, String name) {
			this.num = num;
			this.name = name;
		}
	
		public String toString() {
			return num + ":" + name;
		}
	
		@Override
		public int compareTo(Students o) {
			int result;
			Students s = (Students) o;
			result = num > s.num ? 1 : (num == s.num ? 0 : -1);
			if (result == 0) {
				result = name.compareTo(s.name);
			}
			return result;
		}
	}
	
	class CompareToStudent implements Comparator<Object> {
		public int compare(Object o1, Object o2) {
			Students s1 = (Students) o1;
			Students s2 = (Students) o2;
			int rulst = s1.num > s2.num ? 1 : (s1.num == s2.num ? 0 : -1);
			if (rulst == 0) {
				rulst = s1.name.compareTo(s2.name);
			}
			return rulst;
		}
	}


与HashSet相比，TreeSet还提供了几个额外的方法：

1、Comparator comparator（）：返回当前set使用的Comparator，或者返回null，表示以自然方式排序。

2、Object first（）：返回集合中的第一个元素。

3、Object last（）：返回集合中的最后一个元素。

4、Object lower（Object e）：返回集合中位于指定元素之前的一个元素。

5、Object higher（Object e）：返回集合中位于指定元素之后的一个元素。

6、SortedSet subSet（from Element，to Element）：返回此set的子集合，范围从from Element到to Element（闭包）。

7、SortedSet headSet（toElement）：返回此Set的子集，由小于toElement的元素组成。

8、SortedSet tailSet（fromElement）：返回此Set的子集，由大于等于fromElement的元素组成。


#### LinkedHashSet
HashSet还有一个子类LinkedHashSet，其集合也是根据元素hashCode值来决定元素的存储位置，但它同时用链表来维护元素的次序，这样使得元素看起来是以插入的顺序保存的，也就是说，当遍历LinkedHashSet集合元素时，它将会按元素的添加顺序来访问集合里的元素。所以LinkedHashSet的性能略低于HashSet，但在迭代访问全部元素时将有很好的性能，因为它以链表来维护内部顺序。

