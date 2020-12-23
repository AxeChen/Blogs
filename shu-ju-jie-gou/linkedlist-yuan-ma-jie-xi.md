---
description: 'LinkedList相关源码解析，需要掌握LinkedList的核心部分，add(),get(),remove()等方法的实现。'
---

# LinkedList源码解析

## 1、LinkedList简介

LinkedList实现了List接口，是一个有序的容器（存放元素的顺序和添加顺序相同），存储数据为链表，允许添加为null的元素。

* 实现了List的接口，同样实现其接口的有LinkList，Vector；
* 存放元素的顺序和添加顺序相同；
* 添加的元素可以为null；
* 存储数据为双向链表。

## 2、源码解析

#### 1、类声明

```text
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
```

* Deque：可快速访问
* Cloneable：可克隆
* Serializable：可序列化

#### 2、成员变量

```text
// linkedList的size大小
transient int size = 0;

// 链表的第一个节点
transient Node<E> first;

// 链表的最后一个节点
transient Node<E> last;
```

LinkedList的核心为双向链表结构，所以这边会有Node成员变量：

```text
private static class Node<E> {
    E item;
    // 指向下一个节点
    Node<E> next;
    // 指向上一个节点
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

#### 3、构造函数

```text
// 声明一个新的LinkedList
public LinkedList() {
}

// 将一个集合传入生成一个新的集合
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

主要看是addAll\(\)方法

```text
public boolean addAll(int index, Collection<? extends E> c) {
    // 日常判断IndexOutofBoundsException
    checkPositionIndex(index);

    // 将传入的集合转成数组
    Object[] a = c.toArray();
    // 获取到传入集合的长度
    int numNew = a.length;
    if (numNew == 0)
        return false;

    // 声明两个节点一个指向上一个，一个指向下一个
    Node<E> pred, succ;
    if (index == size) {
    // 从最后一个节点往后添加元素。那么指向下一个为null
        succ = null;
        pred = last;
    } else {
    // 从中间的节点开始
        //获取索引 index 处的结点，那么下一个节点就指向索引处的节点
        succ = node(index);
        //上一个节点就是获取索引 index 处的结点指向的上一个节点
        pred = succ.prev;
    }

    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        // 遍历传入的集合
        Node<E> newNode = new Node<>(pred, e, null);
        // 如果pred为空，那就代表是第一个元素
        if (pred == null)
        // 那第一个元素就等于newNode
            first = newNode;
        else
        // 否则就指向下一个节点
            pred.next = newNode;
        // 将当前判断的节点指向下一个节点    
        pred = newNode;
    }

    // 代表从最后一个节点开始加
    if (succ == null) {
    // 最后一个节点就是last
        last = pred;
    } else {
    
        pred.next = succ;
        succ.prev = pred;
    }

    // 大小++
    size += numNew;
    modCount++;
    return true;
}
```

#### 4、添加元素

```text
public boolean add(E e) {
    linkLast(e);
    return true;
}
  
  // 将元素添加到最后
void linkLast(E e) {
        // 获取最后一个元素
        final Node<E> l = last;
        // 新建一个节点，上一个节点指向最后一个节点，下一个节点为null
        final Node<E> newNode = new Node<>(l, e, null);
         // 最后一个节点等于当前的节点
        last = newNode;
       // 假如最后一个节点为空，则代表这是第一个
        if (l == null)
            first = newNode;
        else
            l.next = newNode; //不是第一个，那上一个的节点的Next指向当前节点
            
            // 长度++
        size++;
        modCount++;
    }
```

稍微复杂点的来了：

```text
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
```

add\(index,value\) 将元素添加到指定的位置：

假如index==size这个意思就是从链表的最后一个添加，这个和add\(value\)一样的。

```text
/**
 * Inserts element e before non-null Node succ.
 */
void linkBefore(E e, Node<E> succ) {
    // 传入的这个succ，是传入的index位置的节点
    // assert succ != null;
    // 获取节点指向的下一个节点
    final Node<E> pred = succ.prev;
    // 生成一个新的节点，上一个节点是index节点的上一个节点，下一个节点是index的下一个节点。
    final Node<E> newNode = new Node<>(pred, e, succ);
    // 当然当前index的节点的上一个节点就是当前的节点
    succ.prev = newNode;
    if (pred == null)
    // 如果当一个节点为空，则是第一个
        first = newNode;
    else
    // 否则上一个节点的下一个节点是当前节点
        pred.next = newNode;
        
    // size++
    size++;
    modCount++;
}
```

#### addAll在构造函数中提到就不再做说明

#### 5、删除元素

```text
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

先看常用的remove\(index\)

```text
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

