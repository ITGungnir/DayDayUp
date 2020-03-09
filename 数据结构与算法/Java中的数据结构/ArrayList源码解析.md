## Java中的数据结构-ArrayList源码解析

> &emsp;&emsp;`ArrayList`是Java集合框架中的一员，是`List`接口的一种变长数组的实现。ArrayList中实现了线性表的所有操作，且允许存放一切数据，包括`null`。除了实现List接口之外，此类还提供一些方法来操纵内部用于存储列表的数组的大小。ArrayList类与`Vector`类大致相似，区别只是ArrayList是线程不同步的，而Vector是线程同步的。<br/>
> &emsp;&emsp;`size()`、`isEmpty()`、`get()`、`set()`、`iterator()`、`listIterator()`等方法具有常量级的运行时间，而`add()`方法具有`摊销固定时间(amortized constant time)`，即向ArrayList中添加n个元素`平均`需要`O(n)`的时间复杂度，其他操作所需的时间都是线性的（大致而言）。相比于`LinkedList`的实现，ArrayList中的常量因子相对较小。<br/>
> &emsp;&emsp;每个ArrayList对象都有一个`capacity`属性，这个属性用于标识其中用以存储数据元素的数组的长度，其值至少要与ArrayList中存储的元素数量相等。随着不断有元素添加到ArrayList中，它的capacity属性也跟着自动增长。<br/>
> &emsp;&emsp;ArrayList在应用程序添加大量元素之前会先调用`ensureCapacity()`方法，这个方法可以减少增量重新分配的数量。<br/>
> &emsp;&emsp;**注意：ArrayList不是线程同步的。** 如果多个线程并发地访问同一个ArrayList对象，且至少有一个线程修改了列表的结构（如增加、删除元素，或者明确修改数组的长度。仅仅对一个元素重新赋值不算修改列表结构），则必须在外部处理线程同步问题，通常做法是在封装了本列表的对象上做同步操作。如果是直接使用ArrayList，外层没有包裹对象，则可以使用`List list = Collections.synchronizedList(new ArrayList(...));`的方法。<br/>
> &emsp;&emsp;ArrayList中的`iterator()`、`listIterator()`方法返回的迭代器是`fail-fast`的：当此列表上的迭代器被创建之后，当不调用迭代器自身的`remove()`和`add()`方法对列表进行结构化修改时，都会抛出`ConcurrentModificationException`异常。因此，面对并发修改，迭代器会快速干净地失败，而不会在未来的不确定时间内冒任意，不确定的行为的风险。

### 0、ArrayList的继承关系
```java
public class ArrayList<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```
从继承关系来看，`ArrayList`继承自`AbstractList`类，实现了`List`，且支持`快速随机访问`、`克隆`、`序列化`。

### 1、ArrayList构造方法
```java
private static final Object[] EMPTY_ELEMENTDATA = {};
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

transient Object[] elementData;

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    }
}

public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```
ArrayList中维护了一个Object数组`elementData`，它被`transient`关键字修饰，表示这个数组不会被序列化（序列化时需要通过`readObject()`和`writeObject()`方法操作，后续会介绍）。

ArrayList有三种构造方式：
* `ArrayList()`方法构造，将一个空数组赋值给elementData（这个数组会在添加第一个元素时扩容至默认长度10，后续会介绍）；
* `ArrayList(int initialCapacity)`方法构造，传入一个初始长度initialCapacity，创建数组；如果长度为0，创建一个空数组；如果长度小于0，则抛出异常；
* `ArrayList(Collection<? extends E> c)`方法构造，传入一个其他集合对象c，创建数组。

### 2、add(E e)
```java
private int size;
protected transient int modCount = 0;

private static final int DEFAULT_CAPACITY = 10;
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

public boolean add(E e) {
    ensureCapacityInternal(size + 1);
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```
上述代码是向ArrayList中添加一个元素时所涉及到的代码，其主要流程是：先判断列表是否需要扩容，如果需要扩容则将列表扩容到当前长度的1.5倍之后与目标长度做比较，如果仍比目标长度小，则直接使用目标长度作为扩容后的长度；如果新的扩容长度大于Integer.MAX_VALUE，会发生OOM，此时会抛出`OutOfMemoryError`错误。扩容之后再在当前最后一个元素后面添加目标元素即可。

### 3、modCount
`modCount`是`AbstractList`类中的一个变量，其定义如下：
```java
protected transient int modCount = 0;
```
modCount用于标识列表被`结构化修改`的次数，在使用通过`iterator()`、`listIterator()`方法得到的迭代器，或使用`forEach()`方法对列表中的元素进行迭代时，会校验这个标识，如果发现遍历过程中这个标识发生了变化，则会抛出`ConcurrentModificationException`异常。以forEach方法为例，代码如下：
```java
public void forEach(Consumer<? super E> action) {
    Objects.requireNonNull(action);
    final int expectedModCount = modCount;
    @SuppressWarnings("unchecked")
    final E[] elementData = (E[]) this.elementData;
    final int size = this.size;
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        action.accept(elementData[i]);
    }
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

### 总结
* ArrayList是基于数组的线性存储结构，线程不同步，与之相似的Vector是线程同步的；
* ArrayList的默认初始长度是10，这个值在默认创建ArrayList时不会体现，会在添加第一个元素时体现；可以通过调用不同的构造方法来修改这个值；
* 向ArrayList中添加元素时，如果需要扩容，则每次扩容为原来的1.5倍，但如果仍小于目标长度，则直接使用目标长度作为扩容后的长度；
* ArrayList中数据的长度最大不能超过Integer.MAX_VALUE，否则会造成OOM错误；
