[TOC]
# 题目
给定一个非负整数数组 A， A 中一半整数是奇数，一半整数是偶数。

对数组进行排序，以便当 A[i] 为奇数时，i 也是奇数；当 A[i] 为偶数时， i 也是偶数。

你可以返回任何满足上述条件的数组作为答案。

# 思路
## 方法一
直接按照数组合并的思路来做，设定两个指针分别指向偶数位置和奇数位置
```java{.line-numbers}
class Solution {
    public int[] sortArrayByParityII(int[] A) {
        int[] ans = new int[A.length];
        int even = 0, odd = 1;
        for(int i = 0; i < A.length;++i){
            if(A[i] % 2 == 0){
                ans[even] = A[i];
                even += 2;
            } else {
                ans[odd] = A[i];
                odd += 2;
            }
        }
        return ans;
    }
}
```
## 方法二
在原数组可修改的情况下，设定两个指针odd和even分别指向奇数位置和偶数位置。如果A[even]当前为偶数，则even继续向前遍历，每次前进两个单位。如果A[even]当前为奇数，则遍历odd，找到一个为偶数的A[odd],两者进行交换。
**这种方法成立的前提是数组中偶数和奇数的数量是一样的。**
```java{.line-numbers}
class Solution {
    public int[] sortArrayByParityII(int[] A) {
        int odd = 1;
        for(int even = 0; even < A.length; even += 2){
            if(A[even] % 2 == 1){
                while(A[odd] % 2 == 1){
                    odd += 2;
                }
                int temp = A[even];
                A[even] = A[odd];
                A[odd] = temp;
            }
        }
        return A;
    }
}
```
