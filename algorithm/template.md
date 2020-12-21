[TOC]
# 二分的模板

查找最左边或者最右边的元素下标，只要考虑arr[mid]==target时，区间怎么移动即可 ，如果要取最左边的，则区间向左移动，如果要取右边的，则区间向右移动

## 查找最左边的大于等于target的元素下标
```java{.line-numbers}
public static void testBinary() {
        int[] arr = new int[]{1,3,5,5,5,6,7,9};
        int target = 5;
        int low = 0 , high = arr.length - 1;
        while(low <= high) {
            int mid = low + (high - low) / 2;
            if(target > arr[mid]) {
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
        System.out.println(low);
    }
```
## 查找最右边的大于等于target的元素下标
```java{.line-numbers}
public static void testBinary() {
        int[] arr = new int[]{1,3,5,5,5,6,7,9};
        int target = 5;
        int low = 0 , high = arr.length - 1;
        while(low <= high) {
            int mid = low + (high - low) / 2;
            if(target >= arr[mid]) {
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
        System.out.println(high);
    }
```