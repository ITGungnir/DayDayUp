## Java数据结构-HashMap源码解析

> &emsp;&emsp;HashMap是Map接口的一种基于哈希表的实现，其中的元素以`key-value`的键值对形式存储。HashMap中的key和value都允许为null（所有key中只能有一个null）。HashMap不能保证其中存储的元素的有序。<br/>
> &emsp;&emsp;只要HashMap中每个Bucket中的元素分散足够均匀，那么对HashMap的存取操作都将达到常量级的时间复杂度；HashMap的遍历操作的时间复杂度是与HashMap的容量成正比的，因此如果对HashMap的遍历效率要求较高，那么不应该为其初始容量设置过大的值。<br/>
> &emsp;&emsp;影响HashMap性能的参数有两个：`initial capacity`和`load factor`。capacity（容量）是指HashMap中bucket的数量，初始容量就是HashMap在被初始化时的容量。loadFactor（加载因子）是决定HashMap扩容时机的参数，当HashMap中的元素数量超过加载因子允许的范围后，HashMap就会扩容一倍并进行重新哈希。默认的加载因子值为0.75，为了提升HashMap的性能，需要在初始化HashMap时设置适合自己业务场景的值。<br/>
> &emsp;&emsp;HashMap中是根据key的`hashCode`划分bucket的，因此如果多个元素具有相同的hashCode，它们就会被分配到同一个bucket中，这样会影响HashMap的存取效率。<br/>
> &emsp;&emsp;**HashMap是线程不同步的。**<br/>
> &emsp;&emsp;与ArrayList、LinkedList相似，我们可以通过`Collections.synchronizedMap(new HashMap(...))`方法创建一个线程安全的Map对象。<br/>
> &emsp;&emsp;每个bucket中的元素默认是存储为链表结构，但如果bucket中的元素超过一定数量，就会将这个链表转化为一棵红黑树，以提高查询效率。

### **总结**
* HashMap中的key和value都允许为null，其中最多只能有一个key为null；
* HashMap是线程不同步的；
* HashMap通过扩容因子loadFactor来决定扩容时机；
