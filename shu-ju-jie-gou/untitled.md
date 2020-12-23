---
description: 'ArrayList相关源码解析，需要掌握ArrayList的核心部分，自动扩容，以及add(),get(),remove()等方法的实现。'
---

# ArrayList源码解析

ArrayList可以说是随处可见，作为一个安卓开发，从一开始的ListView开始就习惯使用ArrayList了。而且不少的面试题也会拿ArrayList和LinkList作比较。那个add，remove速度快，谁获取元素比较快等等。

### 1、ArrayList的简介

ArrayList实现了List接口，是一个有序的容器（存放元素的顺序和添加顺序相同），保存数据为一个object数组，允许添加重复的元素，允许添加为null的元素。可以自动扩容。

* 实现了List的接口，同样实现其接口的有LinkList，Vector；
* 存放元素的顺序和添加顺序相同；
* 保存数据为一个object数组；
* 允许添加重复元素；
* 可以自动扩容；

### 2、源码解析

#### 1、类的声明

```text
public class ArrayList<E> extends 
                                AbstractList<E> 
                         implements 
                                List<E>, RandomAccess, Cloneable, Serializable {
// ... ...
}
```

* RandomAccess，可快速随机访问（搞不懂，说是For比迭代器要）
* Cloneable，可克隆。
* Serializable，可序列化。

#### 2、成员变量

```text
// 序列化ID
private static final long serialVersionUID = 8683452581122892189L;

// 默认的大小
/**
 * Default initial capacity.
 */
private static final int DEFAULT_CAPACITY = 10;

// 当初始化一个为0的ArrayList(0)时的默认值
/**
 * Shared empty array instance used for empty instances.
 */
private static final Object[] EMPTY_ELEMENTDATA = {};

// 当初始化一个为0的ArrayList()时的默认值
/**
 * Shared empty array instance used for default sized empty instances. We
 * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
 * first element is added.
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

// 保存元素的数组
/**
 * The array buffer into which the elements of the ArrayList are stored.
 * The capacity of the ArrayList is the length of this array buffer. Any
 * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
 * will be expanded to DEFAULT_CAPACITY when the first element is added.
 */
// Android-note: Also accessed from java.util.Collections
transient Object[] elementData; // non-private to simplify nested class access

// 集合的大小
/**
 * The size of the ArrayList (the number of elements it contains).
 *
 * @serial
 */
private int size;
```

使用object数组，表示ArrayList可以存放任何数据对象，有和数组一样的特性。

#### 3、构造函数

```text
// 初始化一个默认带大小的ArrayList，initialCapacity不能为负数，
/**
 * Constructs an empty list with the specified initial capacity.
 *
 * @param  initialCapacity  the initial capacity of the list
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

// 初始化一个空的
/**
 * Constructs an empty list with an initial capacity of ten.
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// 传入另一个集合类型 比如 ArrayList<Int>(ArrayList<Int>())
/**
 * Constructs a list containing the elements of the specified
 * collection, in the order they are returned by the collection's
 * iterator.
 *
 * @param c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

#### 4、获取元素

```text
public E get(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

    return (E) elementData[index];
}
```

可以看到这里有个常见的IndexOutofBoundsException，数组越界异常。这里通过索引直接访问，说明ArrayList获取元素的效率非常高。

#### 5、添加元素

```text
public boolean add(E e) {
    // 计算是不是要扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
 
// 计算是不是要扩容
private void ensureCapacityInternal(int minCapacity) {
        // 假如一开始数组为空数组
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 从传入的size和默认的数组相比，取最大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        // 去
        ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    // 当传入的长度（就是添加这个字段的长度）减去当前长度大于0，则说明超了，需要扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

// 扩容
 private void grow(int minCapacity) {
        // overflow-conscious code
        // 未扩容前的长度
        int oldCapacity = elementData.length;
        // newCapacity的长度为旧长度+旧长度的一半，也就是1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 如果长度达不到最小长度，就把新的长度定位最小长度
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 如果长度超过了最大的长度，则把新的长度定位Integer.MAX_VALUE
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        //构建符合容量大小的新数组并复制原数组的数据
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

可以从添加元素的代码中看出，有非常多的操作，假如数组比较长，那么复制的操作必然耗时，所有ArrayList的添加元素是耗时的操作。

当然还有另一种的add\(index,value\)的方法.

```text
public void add(int index, E element) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```

首先第一步判断是不是数组越界的日常操作，第二步上面已经讲到了是不是要扩充。第三步、将index后面的所有数值向后推移一位。第四步：将index位置的值设置为传入的值。

所以add\(index,value\)是比直接add\(value\)更耗时的操作。

还有更复杂的addAll操作

```text
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```

复制数据这块是将新来的数组添加到原来数组的末端。当然在这之前也少不了判断是否需要扩容的操作。

更复杂的来了：从指定索引处添加数组。

```text
public boolean addAll(int index, Collection<? extends E> c) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

首先日常的扩容操作，然后判断需要移动多少元素，那个判断numMoved&gt;0的操作就是判断是不是index是数组的最后一个，假如是最后一个，那么就不需要扩容。

#### 6、移出元素

```text
public E remove(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

    modCount++;
    E oldValue = (E) elementData[index];

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}

public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

先看remove\(int\)，首先日常数组越界判断。然后判断移出这个元素之后需要移动多少个元素。同样有个numMoved的这个判断，假如移出的最后一个元素，那么就不需要移动元素了。

然后remove\(object\)，因为不知道这个object在哪个位置，所以这里需要先遍历获取这个index，然后后面的操作就是和remove\(int\)一样了，根据index然后去移动数组的这些逻辑。所以remove\(object\)是要比remove\(int\)要耗时的。

#### 7、set元素

```text
    public E set(int index, E element) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        E oldValue = (E) elementData[index];
        elementData[index] = element;
        return oldValue;
    }
```

修改元素就比较简单了，先获取，然后直接修改即可。

#### 8、和MutableList的关系

```text
public inline fun <T> mutableListOf(): MutableList<T> = ArrayList()
```



