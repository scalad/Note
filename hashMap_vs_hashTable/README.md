### HashMap And HashTable
HashMap和Hashtable两个类都实现了Map接口，二者保存K-V对（key-value对）；HashSet则实现了Set接口，性质类似于集合。Hashtable的应用非常广泛，HashMap是新框架中用来代替Hashtable的类，也就是说建议使用HashMap，不要使用Hashtable。可能你觉得Hashtable很好用，为什么不用呢？这里简单分析他们的区别。

一、继承的父类不同

Hashtable继承自Dictionary类，而HashMap继承自AbstractMap类，HashMap是Java1.2引进的Map
interface 的一个实现。但二者都实现了Map接口。

	public class Hashtable<K,V>
	    extends Dictionary<K,V>
	    implements Map<K,V>, Cloneable, java.io.Serializable {

	public class HashMap<K,V> extends AbstractMap<K,V>
    	implements Map<K,V>, Cloneable, Serializable {

二、线程安全性不同

Hashtable 中的方法是Synchronize的，而HashMap中的方法在缺省情况下是非Synchronize的。在多线程并发的环境下，可以直接使用Hashtable，不需要自己为它的方法实现同步，但使用HashMap时就必须要自己增加同步处理。

HashTable

	public synchronized boolean isEmpty() {
        return count == 0;
    }
	
HashMap

	public boolean isEmpty() {
        return size == 0;
    }

三、是否提供contains方法

HashMap把Hashtable的contains方法去掉了，改成containsValue和containsKey，因为contains方法容易让人引起误解。
Hashtable则保留了contains，containsValue和containsKey三个方法，其中contains和containsValue功能相同，实际上containsValue调用的是contains方法。
	
	public synchronized boolean contains(Object value) {
        if (value == null) {
            throw new NullPointerException();
        }

        Entry<?,?> tab[] = table;
        for (int i = tab.length ; i-- > 0 ;) {
            for (Entry<?,?> e = tab[i] ; e != null ; e = e.next) {
                if (e.value.equals(value)) {
                    return true;
                }
            }
        }
        return false;
    }

	public boolean containsValue(Object value) {
        return contains(value);
    }


四、key和value是否允许null值

其中key和value都是对象，并且不能包含重复key，但可以包含重复的value。
Hashtable中，key和value都不允许出现null值。HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。当get()方法返回null值时，可能是 HashMap中没有该键，也可能使该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断。

HashTable
	  
	  public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }

HashMap

    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

	final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }


五、两个遍历方式的内部实现上不同

Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。

六、hash值不同

哈希值的使用不同，HashTable直接使用对象的hashCode。而HashMap重新计算hash值。

七、内部实现使用的数组初始化和扩容方式不同

Hashtable和HashMap它们两个内部实现方式的数组的初始大小和扩容的方式。HashTable中hash数组默认大小是11，增加的方式是 old*2+1。HashMap中hash数组的默认大小是16，而且一定是2的指数。

Java 5提供了ConcurrentHashMap，它是HashTable的替代，比HashTable的扩展性更好。

### 我们能否让HashMap同步？
HashMap可以通过下面的语句进行同步：

	Map m = Collections.synchronizeMap(hashMap);

### 关于ConcurrentHashMap

	public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
	    implements ConcurrentMap<K,V>, Serializable {

Hashtable和HashMap有几个主要的不同：线程安全以及速度。仅在你需要完全的线程安全的时候使用Hashtable，而如果你使用Java 5或以上的话，请使用ConcurrentHashMap吧。

