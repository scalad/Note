#### Java中Comparable和Comparator比较

#### Comparable 简介

Comparable 是排序接口。

若一个类实现了Comparable接口，就意味着“该类支持排序”。  即然实现Comparable接口的类支持排序，假设现在存在“实现Comparable接口的类的对象的List列表(或数组)”，则该List列表(或数组)可以通过 Collections.sort（或 Arrays.sort）进行排序。

此外，“实现Comparable接口的类的对象”可以用作“有序映射(如TreeMap)”中的键或“有序集合(TreeSet)”中的元素，而不需要指定比较器。

Comparable 定义

Comparable 接口仅仅只包括一个函数，它的定义如下：

	package java.lang;
	import java.util.*;
	
	public interface Comparable<T> {
	    public int compareTo(T o);
	}

说明：
假设我们通过 x.compareTo(y) 来“比较x和y的大小”。若返回“负数”，意味着“x比y小”；返回“零”，意味着“x等于y”；返回“正数”，意味着“x大于y”。

#### Comparator 简介

Comparator 是比较器接口。

我们若需要控制某个类的次序，而该类本身不支持排序(即没有实现Comparable接口)；那么，我们可以建立一个“该类的比较器”来进行排序。这个“比较器”只需要实现Comparator接口即可。

也就是说，我们可以通过“实现Comparator类来新建一个比较器”，然后通过该比较器对类进行排序。

Comparator 定义

Comparator 接口仅仅只包括两个个函数，它的定义如下：

	package java.util;

	public interface Comparator<T> {
	
	    int compare(T o1, T o2);
	
	    boolean equals(Object obj);
	}


说明：
(01) 若一个类要实现Comparator接口：它一定要实现compareTo(T o1, T o2) 函数，但可以不实现 equals(Object obj) 函数。

为什么可以不实现 equals(Object obj) 函数呢？ 因为任何类，默认都是已经实现了equals(Object obj)的。 Java中的一切类都是继承于java.lang.Object，在Object.java中实现了equals(Object obj)函数；所以，其它所有的类也相当于都实现了该函数。

(02) int compare(T o1, T o2) 是“比较o1和o2的大小”。返回“负数”，意味着“o1比o2小”；返回“零”，意味着“o1等于o2”；返回“正数”，意味着“o1大于o2”。

#### Comparator 和 Comparable 比较
Comparable是排序接口；若一个类实现了Comparable接口，就意味着“该类支持排序”。
而Comparator是比较器；我们若需要控制某个类的次序，可以建立一个“该类的比较器”来进行排序。

我们不难发现：Comparable相当于“内部比较器”，而Comparator相当于“外部比较器”。

我们通过一个测试程序来对这两个接口进行说明。源码如下：
	   import java.util.*;
	   import java.lang.Comparable;
	   
	   /**
	    * @desc "Comparator"和“Comparable”的比较程序。
	    *   (01) "Comparable"
	    *   它是一个排序接口，只包含一个函数compareTo()。
	    *   一个类实现了Comparable接口，就意味着“该类本身支持排序”，它可以直接通过Arrays.sort() 或 Collections.sort()进行排序。
	    *   (02) "Comparator"
	   *   它是一个比较器接口，包括两个函数：compare() 和 equals()。
	   *   一个类实现了Comparator接口，那么它就是一个“比较器”。其它的类，可以根据该比较器去排序。
	   *
	   *   综上所述：Comparable是内部比较器，而Comparator是外部比较器。
	   *   一个类本身实现了Comparable比较器，就意味着它本身支持排序；若它本身没实现Comparable，也可以通过外部比较器Comparator进行排序。
	   */
	  public class CompareComparatorAndComparableTest{
	  
	      public static void main(String[] args) {
	          // 新建ArrayList(动态数组)
	          ArrayList<Person> list = new ArrayList<Person>();
	          // 添加对象到ArrayList中
	          list.add(new Person("ccc", 20));
	          list.add(new Person("AAA", 30));
	          list.add(new Person("bbb", 10));
	          list.add(new Person("ddd", 40));
	  
	          // 打印list的原始序列
	          System.out.printf("Original  sort, list:%s\n", list);
	  
	          // 对list进行排序
	          // 这里会根据“Person实现的Comparable<String>接口”进行排序，即会根据“name”进行排序
	          Collections.sort(list);
	          System.out.printf("Name      sort, list:%s\n", list);
	  
	          // 通过“比较器(AscAgeComparator)”，对list进行排序
	          // AscAgeComparator的排序方式是：根据“age”的升序排序
	          Collections.sort(list, new AscAgeComparator());
	          System.out.printf("Asc(age)  sort, list:%s\n", list);
	  
	          // 通过“比较器(DescAgeComparator)”，对list进行排序
	          // DescAgeComparator的排序方式是：根据“age”的降序排序
	          Collections.sort(list, new DescAgeComparator());
	          System.out.printf("Desc(age) sort, list:%s\n", list);
	  
	          // 判断两个person是否相等
	          testEquals();
	      }
	  
	      /**
	       * @desc 测试两个Person比较是否相等。
	       *   由于Person实现了equals()函数：若两person的age、name都相等，则认为这两个person相等。
	       *   所以，这里的p1和p2相等。
	       *
	       *   TODO：若去掉Person中的equals()函数，则p1不等于p2
	       */
	      private static void testEquals() {
	          Person p1 = new Person("eee", 100);
	          Person p2 = new Person("eee", 100);
	          if (p1.equals(p2)) {
	              System.out.printf("%s EQUAL %s\n", p1, p2);
	          } else {
	              System.out.printf("%s NOT EQUAL %s\n", p1, p2);
	          }
	      }
	  
	      /**
	       * @desc Person类。
	       *       Person实现了Comparable接口，这意味着Person本身支持排序
	       */
	      private static class Person implements Comparable<Person>{
	          int age;
	          String name;
	  
	          public Person(String name, int age) {
	              this.name = name;
	              this.age = age;
	          }
	  
	          public String getName() {
	              return name;
	          }
	  
	          public int getAge() {
	              return age;
	          }
	  
	          public String toString() {
	              return name + " - " +age;
	          }
	  
	          /**
	           * 比较两个Person是否相等：若它们的name和age都相等，则认为它们相等
	           */
	          boolean equals(Person person) {
	              if (this.age == person.age && this.name == person.name)
	                  return true;
	              return false;
	          }
	  
	         /**
	          * @desc 实现 “Comparable<String>” 的接口，即重写compareTo<T t>函数。
	          *  这里是通过“person的名字”进行比较的
	          */
	         @Override
	         public int compareTo(Person person) {
	             return name.compareTo(person.name);
	             //return this.name - person.name;
	         }
	     }
	 
	     /**
	      * @desc AscAgeComparator比较器
	      *       它是“Person的age的升序比较器”
	      */
	     private static class AscAgeComparator implements Comparator<Person> {
	         
	         @Override 
	         public int compare(Person p1, Person p2) {
	             return p1.getAge() - p2.getAge();
	         }
	     }
	 
	     /**
	      * @desc DescAgeComparator比较器
	      *       它是“Person的age的升序比较器”
	      */
	     private static class DescAgeComparator implements Comparator<Person> {
	         
	         @Override 
	         public int compare(Person p1, Person p2) {
	             return p2.getAge() - p1.getAge();
	         }
	     }
	 
	 }


下面对这个程序进行说明。


a) Person类定义。如下：

	private static class Person implements Comparable<Person>{
	    int age;
	    String name;
        
        ...

	    /** 
	     * @desc 实现 “Comparable<String>” 的接口，即重写compareTo<T t>函数。
	     *  这里是通过“person的名字”进行比较的
	     */
	    @Override
	    public int compareTo(Person person) {
	        return name.compareTo(person.name);
	        //return this.name - person.name;
	    }   
	} 

说明：
(01) Person类代表一个人，Persong类中有两个属性：age(年纪) 和 name“人名”。
(02) Person类实现了Comparable接口，因此它能被排序。


b) 在main()中，我们创建了Person的List数组(list)。如下：

	// 新建ArrayList(动态数组)
	ArrayList<Person> list = new ArrayList<Person>();
	// 添加对象到ArrayList中
	list.add(new Person("ccc", 20));
	list.add(new Person("AAA", 30));
	list.add(new Person("bbb", 10));
	list.add(new Person("ddd", 40));

c) 接着，我们打印出list的全部元素。如下：

	// 打印list的原始序列
	System.out.printf("Original sort, list:%s\n", list);

d) 然后，我们通过Collections的sort()函数，对list进行排序。

由于Person实现了Comparable接口，因此通过sort()排序时，会根据Person支持的排序方式，即 compareTo(Person person) 所定义的规则进行排序。如下：

	// 对list进行排序
	// 这里会根据“Person实现的Comparable<String>接口”进行排序，即会根据“name”进行排序
	Collections.sort(list);
	System.out.printf("Name sort, list:%s\n", list);

e) 对比Comparable和Comparator

我们定义了两个比较器 AscAgeComparator 和 DescAgeComparator，来分别对Person进行 升序 和 降低 排序。

e.1) AscAgeComparator比较器

它是将Person按照age进行升序排序。代码如下：

	/**
	 * @desc AscAgeComparator比较器
	 *       它是“Person的age的升序比较器”
	 */
	private static class AscAgeComparator implements Comparator<Person> {
	
	    @Override
	    public int compare(Person p1, Person p2) {
	        return p1.getAge() - p2.getAge();
	    }
	}

e.2) DescAgeComparator比较器

它是将Person按照age进行降序排序。代码如下：

	/**
	 * @desc DescAgeComparator比较器
	 *       它是“Person的age的升序比较器”
	 */
	private static class DescAgeComparator implements Comparator<Person> {
	
	    @Override
	    public int compare(Person p1, Person p2) {
	        return p2.getAge() - p1.getAge();
	    }
	}

f) 运行结果
运行程序，输出如下：

	Original  sort, list:[ccc - 20, AAA - 30, bbb - 10, ddd - 40]
	Name      sort, list:[AAA - 30, bbb - 10, ccc - 20, ddd - 40]
	Asc(age)  sort, list:[bbb - 10, ccc - 20, AAA - 30, ddd - 40]
	Desc(age) sort, list:[ddd - 40, AAA - 30, ccc - 20, bbb - 10]
	eee - 100 EQUAL eee - 100

### Difference between Comparator and Comparable in Java

以下根据[https://www.javacodegeeks.com/2013/03/difference-between-comparator-and-comparable-in-java.html](https://www.javacodegeeks.com/2013/03/difference-between-comparator-and-comparable-in-java.html)翻译而来

Comparable接口：如果一个对象需要实现排序那么它将实现该接口，此外，我们还需要实现compareTo(Object)这个方法。

例如：

	public class Country implements Comparable{
       @Override
	    public int compareTo(Object arg0) {
	        Country country=(Country) arg0;
	        return (this.countryId < country.countryId ) ? -1: (this.countryId > country.countryId ) ? 1:0 ;
		}
    }
 
若任何实现了comparable接口的类集合对象将可以使用Collection.sort()或者Arrays.sort()方法根据compareTo方法对其进行排序。

在java中，实现了Comparable接口的对象可以在SortedMap，像TreeMap或者SortedSet或者TreeSet 作为关键字而不用实现任何接口。

Comparator接口：实现可排序的类的对象不需要实现该接口，其他一个类可以实现该接口，例如下面的CountrySortByIdComparator 实现了Comparator接口来完成对集合对象Country根据id进行排序：

	public class CountrySortByIdComparator implements Comparator<Country>{
    @Override
    public int compare(Country country1, Country country2) {
        return (country1.getCountryId() < country2.getCountryId() ) ? -1: (country1.getCountryId() > country2.getCountryId() ) ? 1:0 ;
    }

使用Comparator接口，我们可以根据不同的属性来写不同的排序规则，你可以使用匿名的comparator来对代码进行对比：

    Country indiaCountry=new Country(1, 'India');
    Country chinaCountry=new Country(4, 'China');
    Country nepalCountry=new Country(3, 'Nepal');
    Country bhutanCountry=new Country(2, 'Bhutan');
    List<Country> listOfCountries = new ArrayList<Country>();
    listOfCountries.add(indiaCountry);
    listOfCountries.add(chinaCountry);
    listOfCountries.add(nepalCountry);
    listOfCountries.add(bhutanCountry);
	//Sort by countryName
    Collections.sort(listOfCountries,new Comparator<Country>() {
       @Override
       public int compare(Country o1, Country o2) {
           return o1.getCountryName().compareTo(o2.getCountryName());
       }
    });

### Comparator vs Comparable

![](https://github.com/silence940109/Java/blob/master/image/comparableAndcomparator.png)

java代码：

对于Comparable我们创建了包含id和name属性的country类，它实现了Comparable接口并实现了CompareTo方法并根据id对该集合进行排序。

	package org.arpit.javapostsforlearning;
	//If this.cuntryId < country.countryId:then compare method will return -1
	//If this.countryId > country.countryId:then compare method will return 1
	//If this.countryId==country.countryId:then compare method will return 0
	public class Country implements Comparable{
	    int countryId;
	    String countryName;
	    public Country(int countryId, String countryName) {
	        super();
	        this.countryId = countryId;
	        this.countryName = countryName;
	    }
	    @Override
	    public int compareTo(Object arg0) {
	        Country country=(Country) arg0;
	        return (this.countryId < country.countryId ) ? -1: (this.countryId > country.countryId ) ? 1:0 ;
	    }
	    public int getCountryId() {
	        return countryId;
	    }
	    public void setCountryId(int countryId) {
	        this.countryId = countryId;
	    }
	    public String getCountryName() {
	        return countryName;
	    }
	    public void setCountryName(String countryName) {
	        this.countryName = countryName;
	    }
	}
 

2.ComparatorMain.java

	package org.arpit.javapostsforlearning;	
	import java.util.ArrayList;
	import java.util.Collections;
	import java.util.List;
	public class ComparatorMain {
		/**
		 * @author Arpit Mandliya
		 */
		public static void main(String[] args) {
			 Country indiaCountry=new Country(1, 'India');
			 Country chinaCountry=new Country(4, 'China');
			 Country nepalCountry=new Country(3, 'Nepal');
			 Country bhutanCountry=new Country(2, 'Bhutan');
	
		        List&lt;Country&gt; listOfCountries = new ArrayList&lt;Country&gt;();
		        listOfCountries.add(indiaCountry);
		        listOfCountries.add(chinaCountry);
		        listOfCountries.add(nepalCountry);
		        listOfCountries.add(bhutanCountry);
	
		        System.out.println('Before Sort  : ');
		        for (int i = 0; i &lt; listOfCountries.size(); i++) {
					Country country=(Country) listOfCountries.get(i);
					System.out.println('Country Id: '+country.getCountryId()+'||'+'Country name: '+country.getCountryName());
				}
		        Collections.sort(listOfCountries);
	
		        System.out.println('After Sort  : ');
		        for (int i = 0; i &lt; listOfCountries.size(); i++) {
					Country country=(Country) listOfCountries.get(i);
					System.out.println('Country Id: '+country.getCountryId()+'|| '+'Country name: '+country.getCountryName());
				}
		}	
	}

Output：

	Before Sort  : 
	Country Id: 1||Country name: India
	Country Id: 4||Country name: China
	Country Id: 3||Country name: Nepal
	Country Id: 2||Country name: Bhutan
	After Sort  : 
	Country Id: 1|| Country name: India
	Country Id: 2|| Country name: Bhutan
	Country Id: 3|| Country name: Nepal
	Country Id: 4|| Country name: China

Comparator：我们创建了有id和name属性的Country类，并且，我们将创建另一个实现了Comparator接口和compare方法来根据country对象的id进行排序的CountrySortByIdComparator类，我们也将回看见如何使用匿名的compator来进行排序。

1.Country.java

	package org.arpit.javapostsforlearning;
	
	public class Country{
	    int countryId;
	    String countryName;
	
	    public Country(int countryId, String countryName) {
	        super();
	        this.countryId = countryId;
	        this.countryName = countryName;
	    }
	
	    public int getCountryId() {
	        return countryId;
	    }
	
	    public void setCountryId(int countryId) {
	        this.countryId = countryId;
	    }
	
	    public String getCountryName() {
	        return countryName;
	    }
	
	    public void setCountryName(String countryName) {
	        this.countryName = countryName;
	    }
	
	}

2.CountrySortbyIdComparator.java

	package org.arpit.javapostsforlearning;
	
	import java.util.Comparator;
	//If country1.getCountryId()&lt;country2.getCountryId():then compare method will return -1
	//If country1.getCountryId()&gt;country2.getCountryId():then compare method will return 1
	//If country1.getCountryId()==country2.getCountryId():then compare method will return 0
	 public class CountrySortByIdComparator implements Comparator&lt;Country&gt;{
	
	    @Override
	    public int compare(Country country1, Country country2) {
	
	        return (country1.getCountryId() &lt; country2.getCountryId() ) ? -1: (country1.getCountryId() &gt; country2.getCountryId() ) ? 1:0 ;
	    }
	
	}

3.ComparatorMain.java

	package org.arpit.javapostsforlearning;
	
	import java.util.ArrayList;
	import java.util.Collections;
	import java.util.Comparator;
	import java.util.List;
	
	public class ComparatorMain {
	
		/**
		 * @author Arpit Mandliya
		 */
		public static void main(String[] args) {
			 Country indiaCountry=new Country(1, 'India');
			 Country chinaCountry=new Country(4, 'China');
			 Country nepalCountry=new Country(3, 'Nepal');
			 Country bhutanCountry=new Country(2, 'Bhutan');
	
		        List&lt;Country&gt; listOfCountries = new ArrayList&lt;Country&gt;();
		        listOfCountries.add(indiaCountry);
		        listOfCountries.add(chinaCountry);
		        listOfCountries.add(nepalCountry);
		        listOfCountries.add(bhutanCountry);
	
		        System.out.println('Before Sort by id : ');
		        for (int i = 0; i &lt; listOfCountries.size(); i++) {
					Country country=(Country) listOfCountries.get(i);
					System.out.println('Country Id: '+country.getCountryId()+'||'+'Country name: '+country.getCountryName());
				}
		        Collections.sort(listOfCountries,new CountrySortByIdComparator());
	
		        System.out.println('After Sort by id: ');
		        for (int i = 0; i &lt; listOfCountries.size(); i++) {
					Country country=(Country) listOfCountries.get(i);
					System.out.println('Country Id: '+country.getCountryId()+'|| '+'Country name: '+country.getCountryName());
				}
	
		        //Sort by countryName
		        Collections.sort(listOfCountries,new Comparator&lt;Country&gt;() {
	
					@Override
					public int compare(Country o1, Country o2) {
						return o1.getCountryName().compareTo(o2.getCountryName());
					}
				});
	
				System.out.println('After Sort by name: ');
		        for (int i = 0; i &lt; listOfCountries.size(); i++) {
					Country country=(Country) listOfCountries.get(i);
					System.out.println('Country Id: '+country.getCountryId()+'|| '+'Country name: '+country.getCountryName());
				}
		}
	
	}

Output:

	Before Sort by id : 
	Country Id: 1||Country name: India
	Country Id: 4||Country name: China
	Country Id: 3||Country name: Nepal
	Country Id: 2||Country name: Bhutan
	After Sort by id: 
	Country Id: 1|| Country name: India
	Country Id: 2|| Country name: Bhutan
	Country Id: 3|| Country name: Nepal
	Country Id: 4|| Country name: China
	After Sort by name: 
	Country Id: 2|| Country name: Bhutan
	Country Id: 4|| Country name: China
	Country Id: 1|| Country name: India
	Country Id: 3|| Country name: Nepal



或者可以参考[http://my.oschina.net/itblog/blog/221468](http://my.oschina.net/itblog/blog/221468)