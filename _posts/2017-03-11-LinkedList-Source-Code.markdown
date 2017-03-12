---
layout:     post
title:      "JDK底层实现源码分析系列(三) LinkedList源码分析"
subtitle:   "LinkedList the underlying implementation source code analyze"
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


### LinkedList 继承树
<img src="/img/in-post/JDK-Source-Code/LinkedList.png" />
<small class="img-hint">LinkedList</small>

LinkedList从数据结构层面上说算是一个双向链表 继承于AbstractSequentialList实现了List、Deque、Cloneable、Serializable

有三个重要的成员变量first(链表的头节点)、last(链表的尾节点)、size(链表的长度)

Node为存储数据的类 共有3个变量item(元素)、next(此元素的下一个节点)、prev(此元素的上一个节点)


---

## 分析

### 链表和数组
* 数组：<br>
数组是将元素在内存中连续存放，由于每个元素占用内存相同，可以通过下标迅速访问数组中任何元素。但是如果要在数组中增加一个元素，需要移动大量元素，在内存中空出一个元素的空间，然后将要增加的元素放在其中。同样的道理，如果想删除一个元素，同样需要移动大量元素去填掉被移动的元素。如果应用需要快速访问数据，很少或不插入和删除元素，就应该用数组。

* 链表：<br>
链表恰好相反，链表中的元素在内存中不是顺序存储的，而是通过存在元素中的指针联系到一起。比如：上一个元素有个指针指到下一个元素，以此类推，直到最后一个元素。如果要访问链表中一个元素，需要从第一个元素开始，一直找到需要的元素位置。但是增加和删除一个元素对于链表数据结构就非常简单了，只要修改元素中的指针就可以了。如果应用需要经常插入和删除元素你就需要用链表数据结构了。

* 区别：<br>
数组静态分配内存，链表动态分配内存；<br>
数组在内存中连续，链表不连续；<br>
数组元素在栈区，链表元素在堆区；<br>
数组利用下标定位，时间复杂度为O(1)，链表定位元素时间复杂度O(n)；<br>
数组插入或删除元素的时间复杂度O(n)，链表的时间复杂度O(1)。<br>

### new
LinkedList一共有俩个构造方法，一个是无参的，新new一个size为空的LinkedList，另一个是传入一个Collection类型的集合使用addAll(size,c)把集合中的全部数据加入到新建的LinkedList

```java
 /**
     * Inserts all of the elements in the specified collection into this
     * list, starting at the specified position.  Shifts the element
     * currently at that position (if any) and any subsequent elements to
     * the right (increases their indices).  The new elements will appear
     * in the list in the order that they are returned by the
     * specified collection's iterator.
     *
     * @param index index at which to insert the first element
     *              from the specified collection
     * @param c collection containing elements to be added to this list
     * @return {@code true} if this list changed as a result of the call
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @throws NullPointerException if the specified collection is null
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        // index >= 0 && index <= size;
        // false throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0) // 传入集合为空
            return false;

        Node<E> pred, succ; // pred为被插入节点前一个节点 succ为index节点
        if (index == size) { // 没有后继节点
            succ = null;
            pred = last; //
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked")
            E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
```

---

## 总结


---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
