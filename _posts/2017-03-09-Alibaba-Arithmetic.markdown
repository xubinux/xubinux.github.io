---
layout:     post
title:      "Alibaba 2018实习生招聘 在线编程题解答"
subtitle:   "数组四切分"
date:       2017-03-09
author:     "Binux"
header-img: "img/in-post/Alibaba-Arithmetic/blog.png"
catalog: true
tags:
    - Alibaba
    - Arithmetic
---

> “阿里巴巴 2018实习生 在线编程 题目解答”


## 题目

 对于一个长度为N的整型数组A， 数组里所有的数都是正整数，对于两个满足0<=X <= Y <N的整数，A\[X\], A\[X+1\] … A\[Y\]构成A的一个切片，记作(X, Y)。

 用三个下标 m1, m2, m3下标满足条件 0 < m1, m1 + 1 < m2, m2 +1 < m3 < N – 1。

 可以把这个整型数组分成(0, m1-1), (m1+1, m2-1), (m2+1, m3-1), (m3+1, N-1) 四个切片。如果这四个切片中的整数求和相等，称作“四等分”。

 编写一个函数，求一个给定的整型数组是否可以四等分，如果可以，返回一个布尔类型的true，如果不可以返回一个布尔类型的false。

 限制条件： 数组A最多有1,000,000项，数组中的整数取值范围介于-1,000,000到1,000,000之间。

 要求： 函数的计算复杂度为O(N)，使用的额外存储空间（除了输入的数组之外）最多为O(N)。

 例子：

 对于数组A=\[2, 5, 1, 1, 1, 1, 4, 1, 7, 3, 7\] 存在下标 2, 7, 9使得数组分成四个分片\[2, 5\], \[1, 1, 1, 4\], \[7\], \[7\]，这三个分片内整数之和相等，所以对于这个数组，函数应该返回true。

 对于数组 A=\[10, 2, 11, 13, 1, 1, 1, 1, 1\]， 找不到能把数组四等分的下标，所以函数应该返回false。

---

## 代码

```java
package cn.binux;

public class Main {

    /** 请完成下面这个函数，实现题目要求的功能 **/
    /** 当然，你也可以不按照这个模板来作答，完全按照自己的想法来 ^-^  **/
    static boolean resolve(int[] A) {
        if (A == null || A.length < (4 + 3)) {
            return false;
        }

        int indexl = 0;
        int indexr = A.length - 1;

        int suml = A[indexl];
        int sumr = A[indexr];

        while (indexl < indexr) {

            if (suml < sumr) {
                indexl++;
                suml += A[indexl];
            } else if (suml > sumr) {
                indexr--;
                sumr += A[indexr];
            } else {
                boolean check = check(indexl + 1, indexr - 1, suml, A);
                if (check) {
                    return check;
                } else {
                    indexl++;
                    suml += A[indexl];
                }
            }
        }

        return false;
    }

    private static boolean check(int start, int end, int sum, int[] A) {

        int indexl = start + 1;
        int indexr = end - 1;
        int suml = A[indexl];
        int sumr = A[indexr];
        while (indexr > indexl) {
            if (suml < sumr) {
                indexl++;
                suml += A[indexl];
            } else if (suml > sumr) {
                indexr--;
                sumr += A[indexr];
            } else if (suml == sumr) {
                if (indexr - indexl == 2 && suml == sum) {
                    return true;
                } else {
                    return false;
                }
            }
        }
        return false;
    }

    public static void main(String[] args){
        //ArrayList<Integer> inputs = new ArrayList<Integer>();
        //Scanner in = new Scanner(System.in);
        //String line = in.nextLine();
        //while(line != null && !line.isEmpty()) {
        //    int value = Integer.parseInt(line.trim());
        //    if(value == 0) break;
        //    inputs.add(value);
        //    line = in.nextLine();
        //}
        //int[] A = new int[inputs.size()];
        //for(int i=0; i<inputs.size(); i++) {
        //    A[i] = inputs.get(i).intValue();
        //}
        int[] A1 = { 2, 3, 3, 1, 5, 3, 1, 5, 3, 3, 5, 3 }; // true
        int[] A2 = { 11, 12, 13, 1, 2, 3, 4, 5 }; // false
        int[] A3 = { 4, 4, 1, 8, 1, 8, 1, 1, 8 }; // false
        Boolean res = resolve(A1);

        System.out.println(String.valueOf(res));
    }
}
```



---

## 总结
这道算法题 不是太难 毕竟是招实习生

并且阿里还说了 这道题目并不会影响最后的录用 只是作为一个参考

---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
