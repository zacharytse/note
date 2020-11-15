[TOC]
给定一个以字符串表示的非负整数 num，移除这个数中的 k 位数字，使得剩下的数字最小。

注意:

- num 的长度小于 10002 且 ≥ k。
- num 不会包含任何前导零。

# 思路
要想使得某一个数字序列组成的数字最小，则要尽量使得该序列为单调递增序列。
因此对于这个问题，可以使用单调栈，从左到右，去掉k个不能使得该序列构成单调递增序列的数字
通过单调栈可以在遍历过程中去掉那些破坏单调性的元素
```java{.line-numbers}
class Solution {
    public String removeKdigits(String num, int k) {
        Deque<Character> queue = new LinkedList<>();
        for(int i = 0; i < num.length(); ++i){
            char digit = num.charAt(i);
            while(!queue.isEmpty() && k > 0 && queue.peekLast() > digit){
                queue.pollLast();
                --k;
            }
            queue.offerLast(digit);
        }
        for(int i = 0; i < k; ++i){
            queue.pollLast();
        }
        StringBuilder ret = new StringBuilder();
        boolean leadingZero = true;
        while(!queue.isEmpty()){
            char digit = queue.pollFirst();
            if(leadingZero && digit == '0'){
                continue;
            }
            leadingZero = false;
            ret.append(digit);
        }
        return ret.length() == 0 ? "0" : ret.toString();
    }
}
```