## Java中的数据结构-LinkedList源码解析

> &emsp;&emsp;`LinkedList`是一种双向链表的数据结构，具有`List`和`Deque`接口的所有特性。<br/>
> &emsp;&emsp;**此类是线程不安全的。**<br/>
> &emsp;&emsp;所有需要定位某个元素的操作，都会触发LinkedList正向或逆向的遍历所有元素，具体遍历方向与元素距离两端的距离有关。

### 1、基础结构
```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;

    transient Node<E> first;

    transient Node<E> last;

    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
}
```
从上述代码可知，LinkedList具有列表List、双向队列Deque的全部特性，且支持克隆和序列化。

由于LinkedList是链式存储的，每个节点`Node`中除了存储元素数据之外，还需要提供指向其前后元素的指针`prev`和`next`；同时，LinkedList中也提供了头尾节点的指针`first`和`last`。

### 2、添加/删除元素
LinkedList中实现了List接口和Deque接口中的方法，LinkedList中可以被外界调用的部分常用方法如下：
```java
// 在链表的最后追加一个元素
public boolean add(E e) {}
// 从链表中移除一个元素
public boolean remove(Object o) {}
// 移除链表中的第一个元素
public E poll() {}
// 在链表的最后追加一个元素
public boolean offer(E e) {}
// 在链表的头部添加一个元素
public void push(E e) {}
// 移除链表中的第一个元素
public E pop() {}

public E removeFirst() {}
public E removeLast() {}
public void addFirst(E e) {}
public void addLast(E e) {}
public boolean offerFirst(E e) {}
public boolean offerLast(E e) {}
public E pollFirst() {}
public E pollLast() {}
```

### 3、查找元素
这里主要介绍根据下标index查找元素。LinkedList中提供了一个`node()`方法，用于根据index定位元素：
```java
Node<E> node(int index) {
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```
在这个方法中会判断index与size的大小关系，如果index大于size的一半，则表示这个元素在链表的后半段，此时从后向前遍历以降低时间复杂度；否则从前向后遍历即可。从源码中看，这个方法用在了以下方法中：
```java
public boolean addAll(int index, Collection<? extends E> c) {}
public E get(int index) {}
public E set(int index, E element) {}
public void add(int index, E element) {}
public E remove(int index) {}
```

### 4、迭代器
LinkedList没有`iterator()`方法，只有`listIterator()`方法。listIterator()方法生成的ListItr可以向前遍历，也可以向后遍历。

### 5、序列化
LinkedList的序列化过程与ArrayList相似，都是直接将列表中的每个元素序列化到ObjectOutputStream中。
