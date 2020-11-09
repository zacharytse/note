[TOC]
# 题目
我们有一个由平面上的点组成的列表 points。需要从中找出 K 个距离原点 (0, 0) 最近的点。

（这里，平面上两点之间的距离是欧几里德距离。）

你可以按任何顺序返回答案。除了点坐标的顺序之外，答案确保是唯一的。

# 思路
使用优先队列，构造大顶堆
```java{.line-numbers}
class Solution {
    public int[][] kClosest(int[][] points, int K) {
        PriorityQueue<int[]> queue = new PriorityQueue<int[]>
        (new Comparator<int[]>(){
            public int compare(int[] array1,int[] array2){
                return array2[0] - array1[0];
            }
        });
        for(int i = 0; i < K; ++i){
            queue.offer(new int[]{points[i][0] * points[i][0] 
            + points[i][1] * points[i][1],i});
        }
        for(int i = K; i < points.length; ++i){
            int dist = points[i][0] * points[i][0] +
                    points[i][1] * points[i][1];
            if(dist < queue.peek()[0]){
                queue.poll();
                queue.offer(new int[]{dist,i});
            }
        }
        int[][] ans = new int[K][2];
        for(int i = 0; i < K; ++i){
            int[] e = queue.poll();
            ans[i][0] = points[e[1]][0];
            ans[i][1] = points[e[1]][1];
        }
        return ans;
    }
}
```