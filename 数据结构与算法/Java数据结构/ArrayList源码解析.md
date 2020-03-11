## Java数据结构-ArrayList源码解析

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
也就是说，在使用迭代器或forEach方法对ArrayList进行遍历的过程中，不能调用ArrayList本身的add、remove等可能导致其结构发生变化的方法，如果要做这些操作，可以使用迭代器中提供的相关方法。

### 4、迭代器
ArrayList中有两个方法可以获取到迭代器对象，分别是`iterator()`和`listIterator()`，这两个方法分别返回一个`Itr`和`ListItr`对象，其方法定义如下：
```java
public ListIterator<E> listIterator() {
    return new ListItr(0);
}

public Iterator<E> iterator() {
    return new Itr();
}
```
梳理一下这几个类的关系：
* iterator()方法返回Itr对象，Itr类是Iterator接口的实现类；
* listIterator()方法返回ListItr对象，ListItr类是Itr类的子类，同时实现了ListIterator接口；
* ListIterator接口是Iterator接口的子接口，前者可以双向遍历，而后者只能从前向后遍历。

```java
public interface Iterator<E> {
    boolean hasNext();

    E next();

    default void remove() {
        throw new UnsupportedOperationException("remove");
    }

    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}

public interface ListIterator<E> extends Iterator<E> {
    boolean hasNext();

    E next();

    boolean hasPrevious();

    E previous();

    int nextIndex();

    int previousIndex();

    void remove();

    void set(E e);

    void add(E e);
}
```
前面提到了modCount，这里以Itr类中`remove()`方法的代码为例解释为什么通过迭代器删除元素不会抛异常：
```java
private class Itr implements Iterator<E> {
    int expectedModCount = modCount;

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();
        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```
可以看到，这个方法只是手动为expectedModCount变量赋了值，使这次结构化修改变成了合理的修改而已。

### 5、SubList
ArrayList提供了一个`subList()`方法，用于获取整个列表中的一部分，返回一个`SubList`对象，但这个对象仍然引用着ArrayList对象本身，因此，对这个SubList进行的操作都将最终反应到ArrayList本身。
```java
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);
    return new SubList(this, 0, fromIndex, toIndex);
}

private class SubList extends AbstractList<E> implements RandomAccess {
    private final AbstractList<E> parent;

    SubList(AbstractList<E> parent, int offset, int fromIndex, int toIndex) {
        this.parent = parent;
        // ...
    }

    public void add(int index, E e) {
        rangeCheckForAdd(index);
        checkForComodification();
        parent.add(parentOffset + index, e);
        this.modCount = parent.modCount;
        this.size++;
    }
}
```
可以看到，在创建SubList时传入了ArrayList本身`this`作为SubList对象的`parent`属性，而SubList类中提供的方法最终都调用了`parent.xxx()`方法，因此其修改会最终反应到ArrayList本身。

### 6、序列化
ArrayList中使用`transient`关键字来修改其存储元素的数组`elementData`，表示这个数组在ArrayList被序列化时不会参加序列化，这样做的原因是elementData中可能含有空元素，如果这个数组参与序列化，那么可能会对很多空元素进行序列化，这样做无论从时间、空间还是性能上都是不友好的。

为了解决这个问题，ArrayList实现了`java.io.Serializable`接口并重写了其`writeObject()`、`readObject()`方法：
```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();
    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);
    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}

private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;
    // Read in size, and any hidden stuff
    s.defaultReadObject();
    // Read in capacity
    s.readInt(); // ignored
    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);
        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```
这两个方法是Serializable接口中提供的两个签名，任何实现了Serializable接口的对象，如果需要对其待序列化的属性进行特殊处理，则可以实现这两个方法。从上述代码可以看出，这两个方法实际上就是对elementData中有意义的元素进行了写入和读出操作，去掉无意义的元素，节省时间和空间。

### 7、Collections.synchronizedList
前面提到过，让ArrayList线程安全的一种方法是使用`List<String> list = Collections.synchronizedList(new ArrayList<String>());`的方法，接下来看一下其中的原理：
```java
// Collections # synchronizedList()
public static <T> List<T> synchronizedList(List<T> list) {
    return (list instanceof RandomAccess ?
            new SynchronizedRandomAccessList<>(list) :
            new SynchronizedList<>(list));
}
```
`SynchronizedRandomAccessList`、`SynchronizedList`类共同的父类是`SynchronizedCollection`：
```java
static class SynchronizedCollection<E> implements Collection<E>, Serializable {
    final Object mutex;     // Object on which to synchronize

    public boolean add(E e) {
        synchronized (mutex) {return c.add(e);}
    }
    // ...
}
```
可以发现，实际上就是在非线程安全的类的操作外层套了一层`synchronized`关键字而已。

这里有一个细节，synchronizedList()方法传入的参数是List类型的，也就是说这个方法不仅适用于ArrayList，而是适用于一切List的子类。此外，Collections类还提供了诸如`synchronizedSet()`、`synchronizedMap()`等方法，用来处理Set、Map等数据结构。

### **总结**
* ArrayList是基于数组的线性存储结构，线程不同步，与之相似的Vector是线程同步的；
* ArrayList的默认初始长度是10，这个值在默认创建ArrayList时不会体现，会在添加第一个元素时体现；可以通过调用不同的构造方法来修改这个值；
* 向ArrayList中添加元素时，如果需要扩容，则每次扩容为原来的1.5倍，但如果仍小于目标长度，则直接使用目标长度作为扩容后的长度；
* ArrayList中数据的长度最大不能超过Integer.MAX_VALUE，否则会造成OOM错误；
* ArrayList在每次进行结构化修改时都会将modCount递增，在迭代器和forEach()方法遍历列表时会校验此标识，如果在遍历过程中使用ArrayList内置的API修改了列表结构，则会抛出ConcurrentModificationException异常，要解决这个问题，可以考虑使用迭代器自身的相关方法进行结构化的修改；
* ArrayList提供了iterator()和listIterator()方法获取迭代器，前者只能正向遍历，后者可以正向反响遍历；
* 通过subList()方法创建的子列表SubList，其上的修改都会最终反应到ArrayList对象本身；
* ArrayList通过Serializable接口提供的writeObject()和readObject()签名对元素进行序列化，目的是避免elementData中的空元素造成时间和空间上的浪费；
* 让ArrayList支持线程安全的一种方法是使用Collections.synchronizedList()方法。
