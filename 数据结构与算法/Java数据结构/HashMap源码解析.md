## Java数据结构-HashMap源码解析

> &emsp;&emsp;HashMap是Map接口的一种基于哈希表的实现，其中的元素以`key-value`的键值对形式存储。HashMap中的key和value都允许为null（所有key中只能有一个null）。HashMap不能保证其中存储的元素的有序。<br/>
> &emsp;&emsp;只要HashMap中每个Bucket中的元素分散足够均匀，那么对HashMap的存取操作都将达到常量级的时间复杂度；HashMap的遍历操作的时间复杂度是与HashMap的容量成正比的，因此如果对HashMap的遍历效率要求较高，那么不应该为其初始容量设置过大的值。<br/>
> &emsp;&emsp;影响HashMap性能的参数有两个：`initial capacity`和`load factor`。capacity（容量）是指HashMap中bucket的数量，初始容量就是HashMap在被初始化时的容量。loadFactor（加载因子）是决定HashMap扩容时机的参数，当HashMap中的元素数量超过加载因子允许的范围后，HashMap就会扩容一倍并进行重新哈希。默认的加载因子值为0.75，为了提升HashMap的性能，需要在初始化HashMap时设置适合自己业务场景的值。<br/>
> &emsp;&emsp;HashMap中是根据key的`hashCode`划分bucket的，因此如果多个元素具有相同的hashCode，它们就会被分配到同一个bucket中，这样会影响HashMap的存取效率。<br/>
> &emsp;&emsp;**HashMap是线程不同步的。**<br/>
> &emsp;&emsp;与ArrayList、LinkedList相似，我们可以通过`Collections.synchronizedMap(new HashMap(...))`方法创建一个线程安全的Map对象。<br/>
> &emsp;&emsp;每个bucket中的元素默认是存储为链表结构，但如果bucket中的元素超过一定数量，就会将这个链表转化为一棵红黑树，以提高查询效率。

### 1、HashMap中的重要属性
* `transient Node<K,V>[] table`：存储HashMap中所有元素的数组；
* `int threshold`：下一次扩容阈值，即当HashMap中存储的元素数量超过这个值时，HashMap就会扩容；
* `final float loadFactor`：加载因子，默认是`0.75f`，当HashMap中元素数量占总数量的比例超过加载因子时，就会进行扩容；
* `static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;`：HashMap的默认容量，即16；
* `static final float DEFAULT_LOAD_FACTOR = 0.75f;`：HashMap的默认加载因子，即0.75；

### 2、构造方法
#### `public HashMap()`
```java
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
}
```

#### `public HashMap(int initialCapacity)`
```java
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

#### `public HashMap(int initialCapacity, float loadFactor)`
```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

#### `public HashMap(Map<? extends K, ? extends V> m)`
```java
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}

final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        if (table == null) { // pre-size
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ? (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        else if (s > threshold)
            resize();
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

上面四个构造方法中，前三个都是构建新的HashMap，
并为initialCapacity、loadFactor和threshold三个属性设置初始值；最后一个构造方法是通过一个已有的Map来构建HashMap，然后将Map中的元素通过遍历添加到HashMap中。

### 3、HashMap的结构
从HashMap中table属性的定义来看，HashMap是一个由链表组成的数组（后续链表可能会变为红黑树，后续会介绍）：
```java
transient Node<K,V>[] table;

static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```
Node类是`Map.Entry`类的子类，代表HashMap中单个元素所在的节点，其中存储着当前节点的哈希值、key、value以及指向下一个节点的指针。

### 4、HashMap的长度
在创建HashMap时，如果没有设置默认容量，则会具备一个默认容量16；如果设置了默认容量，则会自动将HashMap的容量设置为最相近的2的幂次：
```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

### 5、扩容

### **总结**
* HashMap中的key和value都允许为null，其中最多只能有一个key为null；
* HashMap是线程不同步的；
* HashMap通过扩容因子loadFactor和扩容阈值threshold来决定扩容时机；
