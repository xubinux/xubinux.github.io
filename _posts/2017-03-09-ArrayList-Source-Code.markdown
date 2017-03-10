---
layout:     post
title:      "JDK底层实现源码分析系列(一) ArrayList源码分析"
subtitle:   "ArrayList the underlying implementation source code analyze"
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


### ArrayList 继承树
<img src="/img/in-post/JDK-Source-Code/ArrayList.png" />
<small class="img-hint">ArrayList</small>

---

## 代码分析

### 成员变量 6个
```java
    /**
     * 序列化ID
     */
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * 默认的初始化容量
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 用于空实例的数组
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * 真正存放元素的数组
     */
    private transient Object[] elementData;

    /**
     * List的大小
     *
     * @serial
     */
    private int size;

    /**
     * 数组分配的最大大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

> **transient**为Java关键字 作用是序列化时 忽略修饰的对象

### 构造方法 3个
```java
    /**
     * 构造一个空的list 容量为指定的initialCapacity值大小
     *
     * @param  initialCapacity  list的容量
     * @throws IllegalArgumentException 如果指定的初始化容量小于0
     *
     */
    public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
    }

    /**
     * 构造一个空List 容量为10
     */
    public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
    }

    /**
     * 将提供的集合转成数组返回给elementData（返回若不是Object[]将调用Arrays.copyOf方法将其转为Object[]）。
     *
     * @param c 提供的集合
     * @throws NullPointerException 如果指定的collection 为 null
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        size = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }
```

### 方法


#### <增>

##### boolean add(E e)
```java
    /**
     * 增加一个元素
     *
     * @param e 追加这个元素到list的最后
     * @return true
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // 确保数组的容量并且Increments modCount!! fail-fast策略
        elementData[size++] = e;
        return true;
    }
```

##### void add(int index, E element)
```java
    /**
     * 在list的指定位置插入一个元素. 把指定位置右边的 (adds one to their indices).
     *
     * @param index 指定索引
     * @param element 插入的元素
     * @throws IndexOutOfBoundsException
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);           // 检查 (0 < index < size)?

        ensureCapacityInternal(size + 1);  // 确保数组的容量并且Increments modCount!! fail-fast策略
        System.arraycopy(elementData,      // src:源数组；
                         index,            // srcPos:源数组要复制的起始位置；
                         elementData,      // dest:目的数组；
                         index + 1,        // destPos:目的数组放置的起始位置；
                         size - index);    // length:复制的长度。
        // 把原来的数组从index位置向后移动一个位置
        elementData[index] = element;      // index 位置插入元素
        size++;                            // list.size++
    }
```

##### boolean addAll(Collection<? extends E> c)
```java
    /**
     * 添加一个集合至list
     *
     * @param c 集合对象
     * @return true
     * @throws NullPointerException 如果指定的集合为空
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        // 获取传入集合的Object[] elementData
        // 举例ArrayList重写的toArray()方法内容 Arrays.copyOf(elementData, size);
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // 确保数组的容量并且Increments modCount fail-fast策略
        System.arraycopy(a, 0, elementData, size, numNew);
        // 从elementData数组的size位置开始 添加a数组0到numNew个元素
        size += numNew;     // 重新计算size
        return numNew != 0;
    }


```

#### <删>

##### E remove(int index)
```java
    /**
     * 删除一个指定位置的元素 并把整个数组向index位置移动1
     *
     * @param 需要删除的元素索引
     * @return 返回删除的元素
     * @throws IndexOutOfBoundsException
     */
    public E remove(int index) {
        rangeCheck(index);
        // 判断(index >= size)? throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        modCount++; // fail-fast策略
        E oldValue = elementData(index);    // 获取删除的元素

        int numMoved = size - index - 1;    // 需要向前移动元素的数量
        if (numMoved > 0)   // numMoved=0 为正好是最后一个元素
            System.arraycopy(elementData,   // src:源数组；
                             index+1,       // srcPos:源数组要复制的起始位置；
                             elementData,   // dest:目的数组；
                             index,         // destPos:目的数组放置的起始位置；
                             numMoved);     // length:复制的长度。
        elementData[--size] = null;         // 数组的最后一个位置置为null 让GC回收

        return oldValue;
    }

```

##### boolean remove(Object o)
```java
    /**
     * 如果指定元素存在，删除list中第一个出现的指定元素
     *
     * @param o 需要删除的元素
     * @return boolean
     */
    public boolean remove(Object o) {
            if (o == null) {
                for (int index = 0; index < size; index++)
                    if (elementData[index] == null) {
                        fastRemove(index);
                        // 跳过rangeCheck(index)检查的remove(int index)方法
                        return true;
                    }
            } else {
                for (int index = 0; index < size; index++)
                    if (o.equals(elementData[index])) {
                        fastRemove(index);
                        // 跳过rangeCheck(index)检查的remove(int index)方法
                        return true;
                    }
            }
            return false;
    }

```

##### void fastRemove(int index)
```java
    /*
     * 私有的删除方法 跳过rangeCheck(index)检查的remove(int index)方法
     * 返回删除的元素
     */
    private void fastRemove(int index) {
            modCount++;
            int numMoved = size - index - 1;
            if (numMoved > 0)
                System.arraycopy(elementData, index+1, elementData, index,
                                 numMoved);
            elementData[--size] = null; // clear to let GC do its work
    }

```

##### boolean removeAll(Collection<?> c)
```java


    /**
     * 删除当前集合中所有包含在指定集合的元素
     *
     * @param c 指定集合
     * @return boolean
     */
    public boolean removeAll(Collection<?> c) {
        return batchRemove(c, false);
    }

```

##### boolean retainAll(Collection<?> c)
```java


    /**
     * b保留当前集合中所有包含在指定集合的元素
     *
     * @param c 指定集合
     * @return boolean
     */
     public boolean retainAll(Collection<?> c) {
            return batchRemove(c, true);
     }

```

##### boolean batchRemove(Collection<?> c, boolean complement)
```java
    /**
     * 批量删除
     *
     * @param c 指定集合
     * @param complement
     * @return boolean
     */
    private boolean batchRemove(Collection<?> c, boolean complement) {
           final Object[] elementData = this.elementData;
           // 获取当前list的elementData
           int r = 0, w = 0;
           boolean modified = false;
           try {
               for (; r < size; r++)
                   if (c.contains(elementData[r]) == complement)
                       elementData[w++] = elementData[r];
                   // false    elementData - c
                   // true     elementData ∩ c
                   // c.contains(elementData[r]) true or false
                   // complement == ture  --> retainAll(Collection<?> c)
                   //   elementData[]中存的是elementData和c共同有的元素集合
                   // complement == fales --> removeAll(Collection<?> c)
                   //   elementData[]中存的是elementData去除c中有的元素集合
           } finally {
               // Preserve behavioral compatibility with AbstractCollection,
               // even if c.contains() throws.
               if (r != size) { // c.contains() 抛出异常才会执行 正常r == size
                   System.arraycopy(elementData, r,
                                    elementData, w,
                                    size - r);
                   w += size - r;
               }
               if (w != size) { // w != size 证明elementData中和c不全是共同元素或者elementData中删除了c中的元素
                   // clear to let GC do its work
                   // 从w开始遍历 后面全置为null 让GC回收
                   for (int i = w; i < size; i++)
                       elementData[i] = null;
                   modCount += size - w;
                   size = w; // 出现计算size
                   modified = true; // 删除成功
               }
           }
           return modified;
    }
```

##### void removeRange(int fromIndex, int toIndex)
```java
    /**
     * 移除指定范围的元素
     *
     * @throws IndexOutOfBoundsException 如果fromIndex和toIndex不在这个范围内
     *         ({fromIndex < 0 || fromIndex >= size() || toIndex > size() || toIndex < fromIndex})
     */
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);
        // 把toIndex之后的元素整体移动到fromIndex后

        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex); // newSize后的元素置为null 让GC回收
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }
```

##### void clear()
```java
    /**
     * 移除list中全部元素
     */
    public void clear() {
        modCount++;

        // 让GC工作
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
```

#### <改>

##### E set(int index, E element)
```java
    /**
     * 替换指定索引位置的元素
     *
     * @param index 需要替换的元素索引
     * @param element 元素
     * @return 之前索引的元素
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E set(int index, E element) {
        rangeCheck(index); // 校验index

        E oldValue = elementData(index); // 获取指定位置的元素
        elementData[index] = element; // 替换成新元素
        return oldValue; // 返回新旧元素
    }

```

#### <查>

##### E get(int index)
```java
    /**
     * 从指定位置获取一个元素
     *
     * @param  需要返回元素的索引
     * @return 返回指定list指定位置的元素
     * @throws IndexOutOfBoundsException
     */
    public E get(int index) {
        rangeCheck(index); // 检查index

        return elementData(index); // 返回元素
    }
```

##### boolean contains(Object o)
```java
    /**
     * 返回true表示list中含有指定元素
     *
     * @param o 指定的元素
     * @return boolean
     */
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
```

##### int indexOf(Object o)
```java
    /**
     * 返回list中第一个出现指定元素的索引
     */
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

##### int lastIndexOf(Object o)
```java
    /**
     * 返回list中最后一个出现指定元素的索引
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

##### int size()
```java
    /**
     * 返回list的大小
     */
    public int size() {
        return size;
    }
```

##### boolean isEmpty()
```java
    /**
     * 判断list是否为空
     */
    public boolean isEmpty() {
        return size == 0;
    }
```

#### <其他>

##### Object clone()
```java
    /**
     * 返回一个浅复制的ArrayList实例
     *
     * @return 返回浅克隆的Object
     */
    public Object clone() {
        try {
            @SuppressWarnings("unchecked")
                ArrayList<E> v = (ArrayList<E>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError();
        }
    }
```

##### List<E> subList(int fromIndex, int toIndex)
```java
    /**
     * 它返回原来list的从[fromIndex  toIndex)之间这一部分的视图(为一个类部类)
     * 之所以说是视图 是因为实际上 返回的list是靠原来的list支持的
     * 你对原来的list和返回的list做的“非结构性修改”(non-structural changes) 都会影响到彼此对方
     * “非结构性修改” 是指不涉及到list的大小改变的修改 相反 结构性修改 指改变了list大小的修改
     *
     * 如果对返回的list线性修改 那么原来的list大小也会改变
     * 如果修改了原list的大小 那么之前产生的子list将会失效
     *
     */
    public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }
```

##### void trimToSize()
```java
    /**
     * 调整elementData的容量为size
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = Arrays.copyOf(elementData, size);
        }
    }
```

### list扩容
```java
    // list进行增加时执行
    ensureCapacityInternal(size + 1);

    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == EMPTY_ELEMENTDATA) { // EMPTY_ELEMENTDATA为空[]
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity); // DEFAULT_CAPACITY 10
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++; // Increments modCount!!

        // 对溢出进行考虑
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * 增加数组容量 确保可以容纳元素
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // oldCapacity >> 1 == oldCapacity / 2 (扩容1.5倍)
        // 如果oldCapacity + (oldCapacity >> 1)超过Integer.MAX_VALUE  newCapacity == -1
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0) // newCapacity > MAX_ARRAY_SIZE
            newCapacity = hugeCapacity(minCapacity); // 设置为最大容量
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
        // 扩大数组的大小
    }

    private static int hugeCapacity(int minCapacity) {
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            return (minCapacity > MAX_ARRAY_SIZE) ?
                Integer.MAX_VALUE :
                MAX_ARRAY_SIZE;
    }
```

### fail-fast机制

#### JDK ArrayList API
> 注意，迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败迭代器会尽最大努力抛出 ConcurrentModificationException。因此，为提高这类迭代器的正确性而编写一个依赖于此异常的程序是错误的做法：迭代器的快速失败行为应该仅用于检测 bug。

#### 出现原因
有两个线程（线程A，线程B）其中线程A负责遍历list 线程B修改list

线程A在遍历list过程的某个时候（此时expectedModCount = modCount = N）线程启动

同时线程B增加一个元素 这时modCount的值发生改变（modCount++ = N + 1）

线程A继续遍历执行next方法时，通告checkForComodification()方法发现expectedModCount = N

而modCount = N + 1 两者不等 这时就抛出ConcurrentModificationException 异常 从而产生fail-fast机制。

#### 解决办法

使用CopyOnWriteArrayList来替换ArrayList

---

## 总结
这是本人第一次看JDK源码 学到了挺多写代码的技巧 大牛写的代码就是不一样

这篇源码分析其实算是我自己笔记 因为我是复制一段段源码 来慢慢分析里面的流程的

就像System.arraycopy()的参数 说实话我真没怎么用过 自己也是查API 看他是怎么实现的

下一个会看Vector的源码 我所了解的 他们俩这是一个线程安全一个不安全 其他还有什么不一样就不知道了


---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
