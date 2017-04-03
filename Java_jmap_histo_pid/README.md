### jmap -histo pid 输出结果样式

因为在Linux环境，用jdk自带的jmap工具(Linux/Unix环境特有的)，可以对进程中的内存对象监视，然后就运行命令jmap -histo [pid]，找内存中的对象数目变化,如下图所示：

	 num     #instances         #bytes  class name
	----------------------------------------------
	   1:       1169837      131659368  [C
	   2:         25945       38337824  [I
	   3:         31548       29407968  [B
	   4:       1164546       27949104  java.lang.String
	   6:         91313       12829072  <constMethodKlass>
	   7:         12395       12404880  [S
	   8:         91313       11700288  <methodKlass>
	   9:          7525        9303112  <constantPoolKlass>
	  10:          7525        5606808  <instanceKlassKlass>
	  11:          6043        5028288  <constantPoolCacheKlass>
	  12:         10048        2007888  [Ljava.lang.Object;
	  14:          3507        1707048  <methodDataKlass>
	  15:          8132         980616  java.lang.Class
	  16:         26854         859328  java.util.HashMap$Entry
	  17:         12368         699296  [[I
	  18:         14135         452320  java.util.concurrent.ConcurrentHashMap$HashEntry
	  19:         20883         334128  java.lang.Object
	  20:           590         316240  <objArrayKlassKlass>
	  21:          1757         305904  [Ljava.util.HashMap$Entry;
	  22:          2809         224720  net.sf.ehcache.Element
	  23:          1992         223104  java.net.SocksSocketImpl
	  24:          2668         213440  java.lang.reflect.Method
	  26:          5932         183928  [Ljava.lang.String;
	  27:          7588         182112  java.util.concurrent.ConcurrentSkipListMap$Node
	  28:          7317         175608  java.lang.Long
	  29:          5303         169696  java.util.Hashtable$Entry
	  30:          6778         162672  java.util.ArrayList
	  31:          3931         157240  java.lang.ref.SoftReference
	  32:          2972         118880  java.util.LinkedHashMap$Entry
	  33:          1565         112680  org.apache.commons.pool2.impl.DefaultPooledObject
	  34:          2817         112680  net.sf.ehcache.store.chm.SelectableConcurrentHashMap$HashEntry
	  35:          2243         107664  java.util.HashMap
	  36:          2592         103680  java.util.TreeMap$Entry
	  37:          3214         102848  java.lang.ref.WeakReference
	  38:          1565         100160  redis.clients.jedis.Client
	  39:          4155          99720  java.util.LinkedList$Node
	  40:          1986          95328  java.net.SocketInputStream
	  41:           414          92952  [Ljava.util.concurrent.ConcurrentHashMap$HashEntry;
	  42:          2275          91000  java.lang.ref.Finalizer
	  43:          1161          83592  java.lang.reflect.Constructor
	  44:           757          78728  java.io.ObjectStreamClass
	  45:          1587          76176  java.net.SocketOutputStream
	  46:          1189          66584  java.beans.MethodDescriptor
	  47:          2770          66480  org.apache.commons.pool2.impl.LinkedBlockingDeque$Node
	  48:           388          66368  [Ljava.util.Hashtable$Entry;
	  49:          1989          63648  java.net.Socket
	  50:           749          53928  java.lang.reflect.Field
	  ...
	  ...
	2947:             1             16  sun.misc.Launcher
	2948:             1             16  org.codehaus.jackson.map.ser.std.DateSerializer
	2949:             1             16  org.apache.phoenix.schema.types.PDataType$2
	2950:             1             16  org.springframework.data.redis.connection.convert.StringToRedisClientInfoConverter
	Total       3090439      316004152

#### 输出结果说明

	[C is a char[]
	[S is a short[]
	[I is a int[]
	[B is a byte[]
	[[I is a int[][]
	
上面的输出中[C对象占用Heap这么多，往往跟String有关，String其内部使用final char[]数组来保存数据的

constMethodKlass/ methodKlass/ constantPoolKlass/ constantPoolCacheKlass/instanceKlassKlass/ methodDataKlass与Classloader相关，常驻与Perm区。其中最后一行(total行），分别记录了实例总数、程序占用总内存数，本例显示的程序总占用内存约300M

