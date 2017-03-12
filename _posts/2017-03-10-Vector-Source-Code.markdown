---
layout:     post
title:      "JDK底层实现源码分析系列(二) Vector源码分析"
subtitle:   "Vector the underlying implementation source code analyze"
date:       2017-03-09
author:     "Binux"
header-img: "img/in-post/JDK-Source-Code/blog.png"
catalog: true
tags:
    - JDK
    - Source Code
---

> “合抱之木 生于毫末 九层之台 起于累土 千里之行 始于足下”


## 前言

> JDK 版本1.7

### Collection 大家族

<img class="shadow" src="/img/in-post/JDK-Source-Code/Java-Collections.png" />
<small class="img-hint">Collection</small>

> 来源[Java Collections Framework](https://infinitescript.com/2014/10/java-collections-framework/)


### Vector 继承树
<img src="/img/in-post/JDK-Source-Code/Vector.png" />
<small class="img-hint">Vector</small>

---

## 代码分析

### 成员变量
```java
    /**
     * 序列化ID
     */
    private static final long serialVersionUID = -2767605614048989439L;

    /**
     * 容量增加时的数量
     * @serial
     */
    protected int capacityIncrement;

    /**
     * 存放数据的数组
     *
     * @serial
     */
    protected Object[] elementData;

    /**
     * 元素的个数 == size
     *
     * @serial
     */
    protected int elementCount;

    /**
     * 数组分配的最大大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

> **transient**为Java关键字 作用是序列化时 忽略修饰的对象

### 构造方法
```java
    /**
     * 默认构造方法 构造一个容量为10的Vector
     */
    public Vector() {
        this(10);
    }
    /**
     * 构造一个指定容量的Vector 增长速度为默认2倍
     *
     * @param   initialCapacity   the initial capacity of the vector
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public Vector(int initialCapacity) {
         this(initialCapacity, 0);
    }

    /**
     * 构造一个指定数组、增长容量的Vector
     *
     * @param   initialCapacity     初始化容量
     * @param   capacityIncrement   增长大小
     * @throws IllegalArgumentException
     */
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }

    /**
     * 将提供的集合转成数组返回给elementData（返回若不是Object[]将调用Arrays.copyOf方法将其转为Object[]）。
     *
     * @param c 提供的集合
     * @throws NullPointerException 如果指定的collection 为 null
     */
    public Vector(Collection<? extends E> c) {
        elementData = c.toArray();
        elementCount = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
    }
```

### 方法

> 只选取了ArrayList中没有的一些方法<br>
> Vector其余方法除了加了synchronized关键字 其他都差不多 就不贴出来了

#### <增>

##### void insertElementAt(E obj, int index)
```java
    /**
     * 在指定index插入一个元素
     *
     * @param      obj     the component to insert
     * @param      index   where to insert the new component
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index > size()})
     */
    public synchronized void insertElementAt(E obj, int index) {
        modCount++;
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index
                                                     + " > " + elementCount);
        }
        ensureCapacityHelper(elementCount + 1); // 确保数组容量够
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
        elementData[index] = obj;
        elementCount++;
    }
```

##### boolean addAll(int index, Collection<? extends E> c)
```java
    /**
     * 从index的位置插入c集合中的全部元素
     *
     * @param index 指定index
     * @param c 元素集合
     * @return boolean
     */
    public synchronized boolean addAll(int index, Collection<? extends E> c) {
        modCount++;
        if (index < 0 || index > elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityHelper(elementCount + numNew);

        int numMoved = elementCount - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        elementCount += numNew;
        return numNew != 0;
    }
```

#### <删>

##### boolean removeElement(Object obj)
```java
    /**
     * 删除指定元素
     *
     * @param   obj   元素对象
     * @return  boolean
     */
    public synchronized boolean removeElement(Object obj) {
        modCount++;
        int i = indexOf(obj);
        if (i >= 0) {
            removeElementAt(i);
            return true;
        }
        return false;
    }
```

##### void removeElementAt(int index)
```java
    /**
     * 删除指定index的元素
     *
     * @param      index   指定index
     * @throws ArrayIndexOutOfBoundsException
     *         ({@code index < 0 || index >= size()})
     */
    public synchronized void removeElementAt(int index) {
        modCount++;
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        elementCount--;
        elementData[elementCount] = null; /* to let gc do its work */
    }
```

##### void removeAllElements()
```java
    /**
     * 删除全部元素
     */
    public synchronized void removeAllElements() {
        modCount++;
        // Let gc do its work
        for (int i = 0; i < elementCount; i++)
            elementData[i] = null;

        elementCount = 0;
    }
```

#### <改>

##### E set(int index, E element)
```java
    /**
     * 修改指定index的元素
     *
     * @param index 指定index
     * @param element 需要替换的元素
     * @return 被替换的元素
     * @throws ArrayIndexOutOfBoundsException
     *         ({@code index < 0 || index >= size()})
     * @since 1.2
     */
    public synchronized E set(int index, E element) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```

##### void setElementAt(E obj, int index)
```java
    /**
     * 修改指定index的元素
     * @param      obj     需要替换的对象
     * @param      index   index
     * @throws ArrayIndexOutOfBoundsException
     *         ({@code index < 0 || index >= size()})
     */
    public synchronized void setElementAt(E obj, int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        elementData[index] = obj;
    }
```

##### void setSize(int newSize)
```java
    /**
     * 如果新的Size大于当前size 调整数组的容量
     *
     * @param  newSize   the new size of this vector
     * @throws ArrayIndexOutOfBoundsException if the new size is negative
     */
    public synchronized void setSize(int newSize) {
        modCount++;
        if (newSize > elementCount) {
            ensureCapacityHelper(newSize);
        } else {
            for (int i = newSize ; i < elementCount ; i++) {
                elementData[i] = null;
            }
        }
        elementCount = newSize;
    }

```

#### <查>

##### int indexOf(Object o, int index)
```java
    /**
     * 返回从index位置的出现的第一个指定的元素index
     *
     * @param o 需要查询的元素
     * @param index index
     * @return
     *          -1 表示无
     * @throws IndexOutOfBoundsException
     */
    public synchronized int indexOf(Object o, int index) {
        if (o == null) {
            for (int i = index ; i < elementCount ; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index ; i < elementCount ; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```


##### int capacity()
```java
    /**
     * 返回当前数组长度
     *
     * @return  the current capacity (the length of its internal
     *          data array, kept in the field {@code elementData}
     *          of this vector)
     */
    public synchronized int capacity() {
        return elementData.length;
    }
```


#### <其他>

##### List<E> subList(int fromIndex, int toIndex)
```java
    /**
     * 同ArrayList 只不过返回的是一个线程安全的list
     *
     */
    public synchronized List<E> subList(int fromIndex, int toIndex) {
        return Collections.synchronizedList(super.subList(fromIndex, toIndex),
                                            this);
    }
```


### list扩容(和ArrayList一样)

---

## 总结
Vector和ArrayList的实现上很类似 说白了就是每个会产生线程安全问题的方法上加synchronized的ArrayList

由于加了synchronized 导致Vector比ArrayList的性能低很多

而且Vector的默认扩容是2倍 ArrayList为1.5倍


---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
