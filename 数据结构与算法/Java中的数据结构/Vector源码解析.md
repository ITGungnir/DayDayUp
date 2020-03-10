## Java中的数据结构-Vector源码解析

### 1、Vector是线程安全的
**Vector与ArrayList大致相似，不同点是Vector是线程安全的而ArrayList不是**

Vector实现线程安全的方法是在一些方法上加了`synchronized`关键字。以`add()`方法为例：
```java
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}
```

### 2、扩容增量
Vector相较于ArrayList新增了一个构造方法，可以设置一个扩容增量`capacityIncrement`：
```java
protected int capacityIncrement;

public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}
```
在扩容时，如果设置了这个值，则每次都会扩容capacityIncrement个元素，否则扩容为当前长度的两倍：
```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### 3、序列化
与ArrayList相似，Vector序列化的方式也是重写了`writeObject()`和`readObject()`方法，但方法内部的实现方式不同，ArrayList是将其中存储的元素逐个写入到`ObjectOutputStream`中，反序列化时再从`ObjectInputStream`中将元素逐个读取出来。Vector中的实现方式略有不同，它将所有元素、元素个数和扩容增量通过`key-value`的方式进行存取：
```java
private void readObject(ObjectInputStream in)
        throws IOException, ClassNotFoundException {
    ObjectInputStream.GetField gfields = in.readFields();
    int count = gfields.get("elementCount", 0);
    Object[] data = (Object[])gfields.get("elementData", null);
    if (count < 0 || data == null || count > data.length) {
        throw new StreamCorruptedException("Inconsistent vector internals");
    }
    elementCount = count;
    elementData = data.clone();
}

private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
    final java.io.ObjectOutputStream.PutField fields = s.putFields();
    final Object[] data;
    synchronized (this) {
        fields.put("capacityIncrement", capacityIncrement);
        fields.put("elementCount", elementCount);
        data = elementData.clone();
    }
    fields.put("elementData", data);
    s.writeFields();
}
```
