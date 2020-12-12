[TOC]
# 按奇偶排序数组II
## 题目
给定一个非负整数数组 A， A 中一半整数是奇数，一半整数是偶数。

对数组进行排序，以便当 A[i] 为奇数时，i 也是奇数；当 A[i] 为偶数时， i 也是偶数。

你可以返回任何满足上述条件的数组作为答案。

## 思路
### 方法一
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
### 方法二
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
# 比较含退格的字符串
## 题目
给定 S 和 T 两个字符串，当它们分别被输入到空白的文本编辑器后，判断二者是否相等，并返回结果。 # 代表退格字符。
注意：如果对空文本输入退格字符，文本继续为空。
## 思路
使用栈
```java{.line-numbers}
class Solution {
    public boolean backspaceCompare(String S, String T) {
        Stack<Character> stackS = new Stack<>();
        Stack<Character> stackT = new Stack<>();
        for(int i = 0; i < S.length(); ++i){
            if(S.charAt(i) == '#'){
                if(!stackS.empty()){
                    stackS.pop();
                }
            } else {
                stackS.push(S.charAt(i));
            }
        }
        for(int i = 0; i < T.length(); ++i){
            if(T.charAt(i) == '#'){
                if(!stackT.empty()){
                    stackT.pop();
                }
            }else{
                stackT.push(T.charAt(i));
            }
        }
        if(stackS.size() != stackT.size()){
            return false;
        }
        while(!stackS.empty()){
            Character chs = (Character)stackS.pop();
            Character cht = (Character)stackT.pop();
            if(!chs.equals(cht)){
                return false;
            }
        }
        return true;
    }
}
```
#  插入区间
## 题目
给出一个无重叠的 ，按照区间起始端点排序的区间列表。

在列表中插入一个新的区间，你需要确保列表中的区间仍然有序且不重叠（如果有必要的话，可以合并区间）。

## 思路
使用模拟的思路，在遍历数组的同时，将所有和新区间有交集的区间进行合并，直到碰到第一个不能和新区间合并的区间
```java{.line-numbers}
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        int left = newInterval[0];
        int right = newInterval[1];
        boolean placed = false;
        List<int[]> ansList = new ArrayList<int[]>();
        for (int[] interval : intervals) {
            if (interval[0] > right) {
                // 在插入区间的右侧且无交集
                if (!placed) {
                    ansList.add(new int[]{left, right});
                    placed = true;                    
                }
                ansList.add(interval);
            } else if (interval[1] < left) {
                // 在插入区间的左侧且无交集
                ansList.add(interval);
            } else {
                // 与插入区间有交集，计算它们的并集
                left = Math.min(left, interval[0]);
                right = Math.max(right, interval[1]);
            }
        }
        if (!placed) {
            ansList.add(new int[]{left, right});
        }
        int[][] ans = new int[ansList.size()][2];
        for (int i = 0; i < ansList.size(); ++i) {
            ans[i] = ansList.get(i);
        }
        return ans;
    }
}
```
# 查找常用字符
## 题目
给定仅有小写字母组成的字符串数组 A，返回列表中的每个字符串中都显示的全部字符（包括重复字符）组成的列表。例如，如果一个字符在每个字符串中出现 3 次，但不是 4 次，则需要在最终答案中包含该字符 3 次。

你可以按任意顺序返回答案。
## 基本思路
使用一个min和一个临时的dict数组，min数组用于记录所有字符串共有的字符的最小出现次数，例如字母'o'在字符串"abo","bcoo"中都出现了,且最小的出现次数为1。
dict数组用于记录单个字符串中的字符的数量，主要用于更新min数组
```java{.line-numbers}
class Solution {
    public List<String> commonChars(String[] A) {
        int []min = new int[26];
        for(int i = 0; i < min.length;++i){
            min[i] = Integer.MAX_VALUE;
        }
        List<String> res = new ArrayList<>();
        for(int i = 0; i < A.length;++i){
            int []dict = new int[26];
            for(int j = 0; j < A[i].length();++j){
                ++dict[A[i].charAt(j) - 'a'];
            }
            for(int j = 0; j < min.length;++j){
                if(dict[j] < min[j]){
                    min[j] = dict[j];
                }
            }
        }
        for(int i = 0; i < min.length;++i){
            for(int j = 0; j < min[i];++j){
                res.add(String.valueOf((char)(i + 'a')));
            }
        }
        return res;
    }
}
```
# 单词拆分II
## 题目
给定一个非空字符串 s 和一个包含非空单词列表的字典 wordDict，在字符串中增加空格来构建一个句子，使得句子中所有的单词都在词典中。返回所有这些可能的句子。

说明：

分隔时可以重复使用字典中的单词。
你可以假设字典中没有重复的单词。

## 思路
采用自顶向下的记忆化搜索
记忆化的实现是通过一个HashMap存储不同下标可以组成的不同语句，如果在回溯过程中遇到已经计算过的下标，则直接返回结果
```java{.line-numbers}
class Solution {
    public List<String> wordBreak(String s, List<String> wordDict) {
        Map<Integer,List<List<String>>> map = new HashMap<>();
        List<List<String>> wordBreaks = backtrack(s,s.length(),new HashSet<String>(wordDict),0,map);
        List<String> breakList = new LinkedList<>();
        for(List<String> wordBreak : wordBreaks){
            breakList.add(String.join(" ",wordBreak));
        }
        return breakList;
    }

    List<List<String>> backtrack(String s,int length,Set<String> dict,int index,
    Map<Integer,List<List<String>>> map){
        if(!map.containsKey(index)){
            List<List<String>> wordBreaks = new LinkedList<>();
            if(index == length){
                wordBreaks.add(new LinkedList<String>());
            }
            for(int i = index + 1; i <= length; ++i){
                String word = s.substring(index,i);
                if(dict.contains(word)){
                    List<List<String>> nextWordBreaks = backtrack(s,length,dict,i,map);
                    for(List<String> nextWordBreak : nextWordBreaks){
                        LinkedList<String> wordBreak = new LinkedList<String>(nextWordBreak);
                        wordBreak.offerFirst(word);
                        wordBreaks.add(wordBreak);
                    }
                }
            }
            map.put(index,wordBreaks);
        }
        return map.get(index);
    }
}
```
# 单词接龙
## 题目
给定两个单词（beginWord 和 endWord）和一个字典，找到从 beginWord 到 endWord 的最短转换序列的长度。转换需遵循如下规则：

每次转换只能改变一个字母。
转换过程中的中间单词必须是字典中的单词。

## 思路
### 方法一
找最短序列一般是使用bfs。同时这里为了构造图方便，采用了虚拟节点的方式，例如对于单词hit,会为其构造3个虚拟节点\*it,h\*t,ht\*。
```java{.line-numbers}
class Solution {
    
    List<List<Integer>> edge = new LinkedList<>();
    Map<String,Integer> map = new HashMap<>();
    int nodeNum = 0;

    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        for(String word : wordList){
            addEdge(word);
        }
        addEdge(beginWord);
        if(!map.containsKey(endWord)){
            return 0;
        }
        int[] dist = new int[edge.size()];
        Arrays.fill(dist,Integer.MAX_VALUE);
        int beginId = map.get(beginWord);
        int endId = map.get(endWord);
        dist[beginId] = 0;
        Queue<Integer> q = new LinkedList();
        q.offer(beginId);
        while(!q.isEmpty()){
            int queueSize = q.size();
            for(int i = 0; i < queueSize; ++i){
                int x = q.poll();
                if(x == endId){
                    return dist[x] / 2 + 1;
                }
                List<Integer> neighbors = edge.get(x);
                for(Integer neighbor : neighbors){
                    if(dist[neighbor] == Integer.MAX_VALUE){
                        dist[neighbor] = dist[x] + 1;
                        q.offer(neighbor);
                    }
                }
            }
        }
        return 0;
    }

    private void addEdge(String word){
        addWord(word);
        int id1 = map.get(word);
        char[] array = word.toCharArray();
        int length = array.length;
        for(int i = 0; i < length; ++i){
            char tmp = array[i];
            array[i] = '*';
            String newWord = new String(array);
            addWord(newWord);
            int id2 = map.get(newWord);
            edge.get(id1).add(id2);
            edge.get(id2).add(id1);
            array[i] = tmp;
        }
    }

    private void addWord(String word){
        if(!map.containsKey(word)){
            map.put(word,nodeNum++);
            edge.add(new LinkedList<Integer>());
        }
    }
}
```
### 方法二
双向dfs
```java{.line-numbers}
class Solution {
    Map<String,Integer> map = new HashMap<>();
    List<List<Integer>> edge = new LinkedList<>();
    int nodeNum = 0;

    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        for(String word : wordList){
            addEdge(word);
        }
        addEdge(beginWord);
        if(!map.containsKey(endWord)){
            return 0;
        }
        int[] beginDist = new int[edge.size()];
        int[] endDist = new int[edge.size()];
        int beginId = map.get(beginWord);
        int endId = map.get(endWord);
        Arrays.fill(beginDist,Integer.MAX_VALUE);
        Arrays.fill(endDist,Integer.MAX_VALUE);
        Queue<Integer> beginQueue = new LinkedList<>();
        Queue<Integer> endQueue = new LinkedList<>();
        beginDist[beginId] = 0;
        endDist[endId] = 0;
        beginQueue.offer(beginId);
        endQueue.offer(endId);
        while(!beginQueue.isEmpty() && !endQueue.isEmpty()){
            int beginSize = beginQueue.size();
            for(int i = 0; i < beginSize; ++i){
                int x = beginQueue.poll();
                if(endDist[x] != Integer.MAX_VALUE){
                    return (endDist[x] + beginDist[x]) / 2 + 1;
                }
                List<Integer> elements = edge.get(x);
                for(Integer e : elements){
                    if(beginDist[e] == Integer.MAX_VALUE){
                        beginDist[e] = beginDist[x] + 1;
                        beginQueue.offer(e);
                    }
                }
            }
            int endSize = endQueue.size();
            for(int i = 0; i < endSize; ++i){
                int x = endQueue.poll();
                if(beginDist[x] != Integer.MAX_VALUE){
                    return (endDist[x] + beginDist[x]) / 2 + 1;
                }
                List<Integer> elements = edge.get(x);
                for(Integer e : elements){
                    if(endDist[e] == Integer.MAX_VALUE){
                        endDist[e] = endDist[x] + 1;
                        endQueue.offer(e);
                    }
                }
            }
        }
        return 0;
    }

    private void addEdge(String word){
        addWord(word);
        int id1 = map.get(word);
        char[] array = word.toCharArray();
        int length = array.length;
        for(int i = 0; i < length; ++i){
            char tmp = array[i];
            array[i] = '*';
            String newWord = new String(array);
            addWord(newWord);
            int id2 = map.get(newWord);
            edge.get(id1).add(id2);
            edge.get(id2).add(id1);
            array[i] = tmp;
        }
    }

    private void addWord(String word){
        if(!map.containsKey(word)){
            map.put(word,nodeNum++);
            edge.add(new LinkedList<Integer>());
        }
    }
}
```
# 岛屿的周长
## 题目
给定一个包含 0 和 1 的二维网格地图，其中 1 表示陆地 0 表示水域。

网格中的格子水平和垂直方向相连（对角线方向不相连）。整个网格被水完全包围，但其中恰好有一个岛屿（或者说，一个或多个表示陆地的格子相连组成的岛屿）。

岛屿中没有“湖”（“湖” 指水域在岛屿内部且不和岛屿周围的水相连）。格子是边长为 1 的正方形。网格为长方形，且宽度和高度均不超过 100 。计算这个岛屿的周长。

## 基本思路
**方法一**
迭代，从左到右找，如果当前格子为陆地。则记为4条边。再判断该格子上下左右是否是陆地，如果是陆地，就要减去一条边
```java{.line-numbers}
class Solution {
    public int islandPerimeter(int[][] grid) {
        int sum = 0;
        int[][] dir = new int[][]{{-1,0},{1,0},{0,1},{0,-1}};
        for(int i = 0; i < grid.length;++i){
            for(int j = 0; j < grid[0].length;++j){
                int total = 4;
                if(grid[i][j] == 0){
                    continue;
                }
                for(int k = 0; k < 4;++k){
                    int x = i + dir[k][0];
                    int y = j + dir[k][1];
                    if(x < 0 || x >= grid.length
                    || y < 0 || y >= grid[0].length){
                        continue;
                    }
                    total -= grid[x][y];
                }
                sum += total;
            }
        }
        return sum;
    }
}
```
**方法二**
使用dfs
```java{.line-numbers}
class Solution {
    static int [] dirx = {-1,0,1,0};
    static int [] diry = {0,1,0,-1};

    public int islandPerimeter(int[][] grid) {
        int n = grid.length, m = grid[0].length;
        int ans = 0;
        for(int i = 0; i < n; ++i){
            for(int j = 0; j < m; ++j){
                if(grid[i][j] == 1){
                    ans += dfs(i,j,grid,n,m);
                }
            }
        }
        return ans;
    }

    int dfs(int x,int y,int[][] grid,int n,int m){
        if(x < 0 || x >= n || y < 0 || y >= m || grid[x][y] == 0){
            return 1;
        }
        if(grid[x][y] == 2){
            return 0;
        }
        grid[x][y] = 2;
        int res = 0;
        for(int i = 0; i < 4; ++i){
            int tx = x + dirx[i];
            int ty = y + diry[i];
            res += dfs(tx,ty,grid,n,m);
        }
        return res;
    }
}
```
# 独一无二的出现次数
## 题目
给你一个整数数组 arr，请你帮忙统计数组中每个数的出现次数。

如果每个数的出现次数都是独一无二的，就返回 true；否则返回 false。

 1 <= arr.length <= 1000
-1000 <= arr[i] <= 1000

## 基本思路
**方法一**
使用一次hash记录所有数据出现的次数，然后再使用set，把这些统计出现的次数加入到set中，判断加入set的数据总量和set的size大小是否一致，如果一致，则说明没有重复，否则返回false
```java{.line-numbers}
class Solution {
    public boolean uniqueOccurrences(int[] arr) {
        int[] counts = new int[2001];
        for(int i = 0; i < arr.length;++i){
            ++counts[arr[i] + 1000];
        }
        Set<Integer> set = new HashSet<>();
        int sum = 0;
        for(int i = 0; i < counts.length;++i){
            if(counts[i] == 0){
                continue;
            }
            set.add(counts[i]);
            ++sum;
        } 
        return set.size() == sum ? true : false;
    }
}
```
**方法二**
先对数组进行排序，然后用一个flag数组记录每一个数据出现的次数，比如某一个数出现了2次，则flag[2] = true。然后在遍历过程中，如果发现flag[i]是true，则说明这个次数已经出现过了，此时返回false。在最后的时候，因为最后一个元素循环之后，会跳出循环，所以最后不能直接返回true，而应该返回!flag[j - i]
```java{.line-numbers}
class Solution {
    public boolean uniqueOccurrences(int[] arr) {
        Arrays.sort(arr);
        boolean[] flag = new boolean[arr.length + 1];
        int i = 0, j = i + 1;
        for(;j < arr.length;++j){
            if(arr[j] != arr[i]){
                if(flag[j - i]){
                    return false;
                }
                flag[j - i] = true;
                i = j;
            }
        }
        return !flag[j - i];
    }
}
```
# 对链表进行插入排序
## 题目
对链表进行插入排序
## 思路
### 优化前
```java{.line-numbers}
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode insertionSortList(ListNode head) {
        if(head == null){
            return null;
        }
        ListNode newHead = head;
        head = head.next;
        newHead.next = null;
        while(head != null){
            ListNode cur = newHead;
            ListNode pre = null;
            ListNode next = head.next;
            head.next = null;
            while(cur != null && cur.val < head.val){
                pre = cur;
                cur = cur.next;
            }
            if(pre == null){
                head.next = cur;
                newHead = head;
            } else {
                pre.next = head;
                head.next = cur;
            }
            head = next;
        }
        return newHead;
    }
}
```
### 优化后
可以加一个lastSorted指针指向有序序列的最后一个元素，在找插入位置时，可以先判断是不是直接插入到有序序列尾部，如果是的话，则直接插入到尾部。避免了一次从有序序列头部到尾部的扫描
```java{.line-numbers}
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode insertionSortList(ListNode head) {
        if(head == null){
            return null;
        }
        ListNode newHead = head;
        ListNode lastSorted = newHead;
        head = head.next;
        newHead.next = null;
        while(head != null){
            ListNode cur = newHead;
            ListNode pre = null;
            ListNode next = head.next;
            head.next = null;
            if(lastSorted.val < head.val){
                lastSorted.next = head;
                lastSorted = head;
                head = next;
                continue;
            }
            while(cur != null && cur.val < head.val){
                pre = cur;
                cur = cur.next;
            }
            if(pre == null){
                head.next = cur;
                newHead = head;
            } else {
                pre.next = head;
                head.next = cur;
            }
            head = next;
        }
        return newHead;
    }
}
```
# 二叉树的前序遍历
## 题目
给定一个二叉树，返回它的 前序 遍历。
## 基本思路
- 方法一
  使用递归
```java{.line-numbers}
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    private List<Integer> ans = new LinkedList<>();
    public List<Integer> preorderTraversal(TreeNode root) {
        dfs(root);
        return ans;
    }

    void dfs(TreeNode root){
        if(root == null){
            return;
        }
        ans.add(root.val);
        dfs(root.left);
        dfs(root.right);
    }
}
```
- 方法二
使用非递归
```java{.line-numbers}
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> ans = new LinkedList<>();
        Stack<TreeNode> stack = new Stack<>();
        while(!stack.empty() || root != null){
            while(root != null){
                ans.add(root.val);
                stack.push(root);
                root = root.left;
            }
            root = stack.pop();
            root = root.right;
        }
        return ans;
    }
}
```
# 二叉搜索树的最小绝对差
##题目
给定一个只包含正整数的非空数组。是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。
#基本思路

### 方法1 直接遍历
因为给定的是一颗二叉搜索树，是有序的，所以每一个节点的前序以及后继与他本身相差的是最小的，所以只需要逐个验证每一个节点和它的前序以及后序的插值即可

```java{.line-numbers}
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int getMinimumDifference(TreeNode root) {
        if(root == null ||
        (root.left == null && root.right == null)){
            return Integer.MAX_VALUE;
        }
        int preValue = getPreValue(root);
        int postValue = getPostValue(root);
        int min = Math.min(Math.abs(root.val - preValue),
        Math.abs(root.val - postValue));
        int childMin = Math.min(getMinimumDifference(root.left),
        getMinimumDifference(root.right));
        return Math.min(childMin,min);
    }

    int getPreValue(TreeNode node){
        if(node.left == null){
            return Integer.MAX_VALUE;
        }
        node = node.left;
        while(node!= null && node.right != null){
            node = node.right;
        }
        return node.val;
    }

    int getPostValue(TreeNode node){
        if(node.right == null){
            return Integer.MAX_VALUE;
        }
        node = node.right;
        while(node != null && node.left != null){
            node = node.left;
        }
        return node.val;
    }
}
```
### 方法2 中序遍历
方法1想的时候想麻烦了，有不少重复计算，其实只要计算当前节点和其前驱节点的差值的绝对值就可以了，前驱节点是可以在中序遍历的过程中确定的
```java{.line-numbers}
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    int pre = -1;
    int ans = Integer.MAX_VALUE;
    public int getMinimumDifference(TreeNode root) {
        dfs(root);
        return ans;
    }

    public void dfs(TreeNode node){
        if(node == null){
            return;
        }
        dfs(node.left);
        if(pre == -1){
            pre = node.val;
        } else {
            int val = Math.abs(node.val - pre);
            ans = ans > val ? val : ans;
            pre = node.val;
        }
        dfs(node.right);
    }
}
```
# 翻转对
## 题目
给定一个数组 nums ，如果 i < j 且 nums[i] > 2*nums[j] 我们就将 (i, j) 称作一个重要翻转对。

你需要返回给定数组中的重要翻转对的数量。

示例:
```
输入: [1,3,2,3,1]
输出: 2
```
## 思路
使用归并排序，对于排完序的两部分数组，重要翻转对的个数是两部分数组中各自的重要翻转对的个数加上左右端点分布在两个数组之间的重要翻转对个数
```java{.line-numbers}
class Solution {
    public int reversePairs(int[] nums) {
        if(nums.length < 2) {
            return 0;
        }
        return countRecursizeReversePairs(nums,0,nums.length - 1);
    }

    private int countRecursizeReversePairs(int[] nums,int low,int high) {
        if(low >= high) {
            return 0;
        }
        int mid = low + (high - low) / 2;
        int n1 = countRecursizeReversePairs(nums,low,mid);
        int n2 = countRecursizeReversePairs(nums,mid+1,high);
        int ret = n1 + n2;
        int i = low , j = mid + 1;
        while(i <= mid) {
            while(j <= high && (long)nums[i] > 2 * (long)nums[j]) {
                ++j;
            }
            ret += j - mid - 1;
            ++i;
        }
        //合并排序数组
        int[] temp = new int[high - low + 1];
        int p1 = low , p2 = mid + 1;
        int p = 0;
        while(p1 <= mid || p2 <= high) {
            if(p1 > mid) {
                temp[p++] = nums[p2++];
            } else if(p2 > high) {
                temp[p++] = nums[p1++];
            } else if(nums[p1] < nums[p2]) {
                temp[p++] = nums[p1++];
            } else {
                temp[p++] = nums[p2++];
            }
        }
        for(i = low; i <= high; ++i) {
            nums[i] = temp[i - low];
        }
        return ret; 
    }
}
```
# 分割等和子集
## 题目
给定一个只包含正整数的非空数组。是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。
## 基本思路

### 使用动态规划

给定一个只包含正整数的非空数组。是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

注意:

每个数组中的元素不会超过 100
数组的大小不会超过 200

**问题转换**：原问题可以转换为从原数组中选取一些数，使得这些数的和等于原数组所有数和的一半($target = sum/2$)，即0-1背包问题

定义数组$dp[i][j]$表示从数组的$[0,i]$下标范围内选取若干个整数，使得他们的和等于j，所以最后的答案为$dp[n-1][target]$

还要考虑一些特殊情况。当数组元素个数小于2个时，显然是不可进行划分的，此时返回false。当数组中最大的元素大于数组和的一半时，也是不可能进行划分的，也返回false。如果数组的和是奇数的话也是不可能进行划分的，此时也应该返回false

考虑边界情况：

i=0或者j=0的情况

当j=0时，意味着target为0，不选择数即可达到目标，所以$dp[i][0]=true$。当i=0时，显然选择nums[0]是可以的，即$dp[0][nums[0]]=true$。

对于一般情况：

当$j>=nums[i]$时，有两种情况

1. 选取当前数,则$dp[i][j]=dp[i-1][j-nums[i]]$
2. 不选取当前数,则$dp[i][j]=dp[i-1][j]$

当$j<nums[i]$时，显然$nums[i]$是不能选取的,$dp[i][j]=dp[i-1][j]$

所以，最后的表达式为
$$
dp[i][j]=
\begin{cases}
dp[i-1][j] | dp[i-1][j-nums[i]],j>=nums[i] \\
dp[i-1][j]
\end{cases}
$$
**代码**

```c++{.line-numbers}
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        if(nums.size() < 2){
            return false;
        }
        int sum = 0;
        int maxNum = INT_MIN;
        for(int i = 0; i < nums.size();++i){
            sum += nums[i];
            if(maxNum < nums[i]){
                maxNum = nums[i];
            }
        }
        if(sum % 2 != 0){
            return false;
        }
        int target = sum / 2;
        if(maxNum > target){
            return false;
        }
        vector<vector<bool>> dp(nums.size(),vector<bool>(target + 1));
        for(int i = 0; i < nums.size();++i){
            dp[i][0] = true;
        }
        dp[0][nums[0]] = true;
        for(int i = 1; i < nums.size();++i){
            for(int j = 0; j <= target;++j){
                if(j >= nums[i]){
                    dp[i][j] = dp[i - 1][j] | dp[i - 1][j - nums[i]];
                }else{
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }
        return dp[nums.size() - 1][target];
    }
};
```

#### 优化策略

采用滚动数组做优化，因为每次更新使用的都是上一行的值，即$dp[i-1]$这一行的值

而且j要从大到小更新，如果从小到大更新，会提前把上一行的值覆盖，造成更新时，实际用的是这一行的值的问题

```c++{.line-numbers}
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        if(nums.size() < 2){
            return false;
        }
        int sum = 0;
        int maxNum = INT_MIN;
        for(int i = 0; i < nums.size();++i){
            sum += nums[i];
            if(maxNum < nums[i]){
                maxNum = nums[i];
            }
        }
        if(sum % 2 != 0){
            return false;
        }
        int target = sum / 2;
        if(maxNum > target){
            return false;
        }
        vector<bool> dp(target+1);
        dp[0]=true;
        dp[nums[0]] = true;
        for(int i = 1; i < nums.size();++i){
            for(int j = target; j>=1;--j){
                if(j >= nums[i]){
                    dp[j]=dp[j] | dp[j-nums[i]];
                }
            }
        }
        return dp[target];
    }
};
```
# 根据身高重建队列
## 题目
假设有打乱顺序的一群人站成一个队列。 每个人由一个整数对(h, k)表示，其中h是这个人的身高，k是排在这个人前面且身高大于或等于h的人数。 编写一个算法来重建这个队列。

注意：
总人数少于1100人。

示例
> 输入:
> \[[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]
> 输出:
> \[[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]

## 思路
### 方法一 从低到高考虑
先假设所有人的身高都不相同，然后按身高从低到高进行排序。如果有n个人的话，排序结束后的序列为$h_0,h_1,...,h_{n-1}$。对于第i个人来说，有两种情况
- 前面的i-1个人身高都比第i个人矮，这i-1个人先插入到最后的结果后，不会影响第i个人在最终队列中的位置
- i+1到第n-1个人身高都比第i个人高，他们插入到队列中之后，必然会影响第i个人的位置。

所以在第i个人插入后，前面必然要留出$k_i$个空给身高比第i个人高的人插入。因此第i个人在队列中的最终位置应该是$k_i+1$个空位

接下来考虑身高相同的情况。如果两个人身高相同，也就是$h_i=h_j$,如果$k_i>k_j$,说明i在队列中的位置应该在j的后面。所以可以认为i不会影响j最终的位置，因此应该把$h_i$适当缩小，也就是让第i个人在排序完成后在j的前面。实现时，可以让k排逆序，使得当$k_i>k_j$时，排序结果为第i个人在第j个人的前面

总结下来，就是以h为第一关键字排正序，以k为第二关键字排逆序，最后每个人的位置为[0,i]中的第$h_i+1$个空位
```java{.line-numbers}
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        Arrays.sort(people, new Comparator<int[]>() {
            public int compare(int[] person1, int[] person2) {
                if (person1[0] != person2[0]) {
                    return person1[0] - person2[0];
                } else {
                    return person2[1] - person1[1];
                }
            }
        });
        int n = people.length;
        int[][] ans = new int[n][];
        for (int[] person : people) {
            int spaces = person[1] + 1;
            for (int i = 0; i < n; ++i) {
                if (ans[i] == null) {
                    --spaces;
                    if (spaces == 0) {
                        ans[i] = person;
                        break;
                    }
                }
            }
        }
        return ans;
    }
}
```
### 方法二 从高到低考虑
和方法一类似，身高从高到低排完序后，对于第i个人，要考虑两种情况
- 前面的i-1个人身高都比第i个人要高，所以他们的插入位置会影响到第i个人的位置
- i+1到n-1个人的身高都比第i个人要矮，所以他们的插入位置对于第i个人来说，没有任何影响

前面的i-1个人都插入到队列后，因为后面的人的位置不会影响到第i个人，所以第i个人的位置是确定的，应该插在队列的$k_i+1$处,这样一定能保证第i个人前面有$k_i$个人身高比他高

考虑身高相同的情况，当$h_i=h_j$时，如果$k_i>k_j$，第i个人在队列中的位置应该在第j个人后面，所以第i个人的插入不应该影响到第j个人，所以希望在排序之后，第i个人可以直接排到第j个人后面。

综上，应该以h为第一关键字倒序排序，以k为第二关键字正序排序，最后对于每个人i,应该插入到队列的$k_i$处
```java{.line-numbers}
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        Arrays.sort(people, new Comparator<int[]>() {
            public int compare(int[] person1, int[] person2) {
                if (person1[0] != person2[0]) {
                    return person2[0] - person1[0];
                } else {
                    return person1[1] - person2[1];
                }
            }
        });
        List<int[]> ans = new ArrayList<int[]>();
        for (int[] person : people) {
            ans.add(person[1], person);
        }
        return ans.toArray(new int[ans.size()][]);
    }
}
```
# 根据数字二进制下1的数目排序
## 题目
给你一个整数数组 arr 。请你将数组中的元素按照其二进制表示中数字 1 的数目升序排序。

如果存在多个数字二进制中 1 的数目相同，则必须将它们按照数值大小升序排列。

请你返回排序后的数组。

**提示**
- $1 <= arr.length <= 500$
- $0 <= arr[i] <= 10^4$

## 思路
### 方法一
暴力求解，因为每一个元素的大小是在$[0,10^4]$之间，所以可以直接用数组记录每个元素含有二进制1的个数，然后定义Comparator接口对原数组进行排序
```java{.line-numbers}
class Solution {
    public int[] sortByBits(int[] arr) {
        List<Integer> list = new ArrayList<>();
        int[] bits = new int[100001];
        for(Integer x : arr){
            list.add(x);
            bits[x] = getOneNum(x);
        }
        Collections.sort(list,new Comparator<Integer>(){
            public int compare(Integer x,Integer y){
                if(bits[x] != bits[y]){
                    return bits[x] - bits[y];
                } else {
                    return x - y;
                }
            }
        });
        for(int i = 0; i < list.size();++i){
            arr[i] = list.get(i);
        }
        return arr;
    }

    private int getOneNum(int x){
        int sum = 0;
        while(x != 0){
            sum += (x & 1);
            x >>= 1;
        }
        return sum;
    }
}
```
### 方法二
基于递推式$bits[i] = bits[i >> 1] + (i \& 1)$
```java{.line-numbers}
class Solution {
    public int[] sortByBits(int[] arr) {
        int[] bits = new int[10001];
        List<Integer> list = new ArrayList<>();
        for(int i = 1; i < bits.length; ++i){
            bits[i] = bits[i >> 1] + (i & 1);
        }
        for(Integer x : arr){
            list.add(x);
        }
        Collections.sort(list,new Comparator<Integer>(){
            public int compare(Integer x,Integer y){
                if(bits[x] != bits[y]){
                    return bits[x] - bits[y];
                } else {
                    return x - y;
                }
            }
        });
        for(int i = 0; i < arr.length; ++i){
            arr[i] = list.get(i);
        }
        return arr;
    }
}
```
# 划分字母区间
## 题目
字符串 S 由小写字母组成。我们要把这个字符串划分为尽可能多的片段，同一个字母只会出现在其中的一个片段。返回一个表示每个字符串片段的长度的列表。
## 基本思路
使用贪心的思想，首先记录每个字母最后出现的位置。然后记录当前遍历的区间，如果在该区间中，有字母最后出现的位置在区间外，则扩大该区间至那个字母最后出现的位置。
一直遍历下去，直到到达区间尾部，然后把区间的长度记录到答案数组中，即可
```java{.line-numbers}
class Solution {
    public List<Integer> partitionLabels(String S) {
        int[] map = new int[26];
        List<Integer> ans = new ArrayList<>();
        for(int i = 0; i < S.length();++i){
            map[S.charAt(i) - 'a'] = i;
        }
        int maxPos = 0;
        int last = 0;
        for(int i = 0; i < S.length(); ++i){
            if(map[S.charAt(i) - 'a'] > maxPos){
                //区间需要扩张
                maxPos = map[S.charAt(i) - 'a'];
            } else if(i == maxPos){//到达区间尾部
                ans.add(maxPos - last + 1);
                ++maxPos;
                last = maxPos;
            }
        }
        return ans;
    }
}
```
# 回文链表
## 题目
请判断一个链表是否为回文链表。
## 基本思路
先获取到链表的后半段，然后将后半段逆置，最后遍历后半段的链表，与前半段的值进行比较，若同一位置的节点值不一致，则不是回文链表
```java{.line-numbers}
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean isPalindrome(ListNode head) {
        ListNode mid = getMid(head);
        mid = reverseList(mid);
        while(mid != null){
            if(mid.val != head.val){
                return false;
            }
            mid = mid.next;
            head = head.next;
        }
        return true;
    }

    private ListNode getMid(ListNode head){
        ListNode slow = head ,quick = head;
        while(slow != null && quick != null){
            slow = slow.next;
            quick = quick.next;
            if(quick != null){
                quick = quick.next;
            } else {
                break;
            }
        }
        return slow;
    }

    private ListNode reverseList(ListNode head){
        ListNode newHead = new ListNode(0);
        while(head != null){
            ListNode next = head.next;
            head.next = newHead.next;
            newHead.next = head;
            head = next;
        }
        return newHead.next;
    }
}
```
# 计数质数
## 题目
统计所有**小于非负整数 n** 的质数的数量。

## 思路
### 方法一(暴力枚举)
```java{.line-numbers}
class Solution {
    public int countPrimes(int n) {
        int cnt = 0;
        for(int i = 2; i < n; ++i) {
            if(isPrime(i)) {
                ++cnt;
            }
        }
        return cnt;
    }

    private boolean isPrime(int n) {
        int i = 2;
        int max = (int)Math.sqrt(n);
        while(i <= max) {
            if(n % i == 0) {
                return false;
            }
            ++i;
        }
        return true;
    }
}
```
### 方法二(埃氏筛)
设isPrime[i]为质数，则大于i的i的整数倍肯定不是质数，简单的，从i*i开始，isPrime[i]都不为质数
```java{.line-numbers}
class Solution {
    public int countPrimes(int n) {
        boolean[] isPrime = new boolean[n];
        Arrays.fill(isPrime,true);
        int ans = 0;
        for(int i = 2 ; i < n; ++i) {
            if(isPrime[i]) {
                ++ans;
                if((long)i * i < n) {
                    for(int j = i * i; j < n; j += i) {
                        isPrime[j] = false;
                    }
                }
            }
        }
        return ans;
    }
}
```
# 加油站
## 题目
在一条环路上有 N 个加油站，其中第 i 个加油站有汽油 gas[i] 升。

你有一辆油箱容量无限的的汽车，从第 i 个加油站开往第 i+1 个加油站需要消耗汽油 cost[i] 升。你从其中的一个加油站出发，开始时油箱为空。

如果你可以绕环路行驶一周，则返回出发时加油站的编号，否则返回 -1。

示例：
```
输入: 
gas  = [1,2,3,4,5]
cost = [3,4,5,1,2]

输出: 3

解释:
从 3 号加油站(索引为 3 处)出发，可获得 4 升汽油。此时油箱有 = 0 + 4 = 4 升汽油
开往 4 号加油站，此时油箱有 4 - 1 + 5 = 8 升汽油
开往 0 号加油站，此时油箱有 8 - 2 + 1 = 7 升汽油
开往 1 号加油站，此时油箱有 7 - 3 + 2 = 6 升汽油
开往 2 号加油站，此时油箱有 6 - 4 + 3 = 5 升汽油
开往 3 号加油站，你需要消耗 5 升汽油，正好足够你返回到 3 号加油站。
因此，3 可为起始索引。
```
## 思路
### 方法一
直接暴力求解
```java{.line-numbers}
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int n = gas.length;
        for(int start = 0; start < n; ++ start){
            int end = start;
            int oil = gas[start];
            int count = 0;
            while(count < n){
                int next = (end + 1) % n;
                oil -= cost[end];
                if(oil < 0){
                    break;
                }
                oil += gas[next]; 
                end = next;
                ++count;
            }
            if(count == n) {
                return start;
            }
        }
        return -1;
    }
}
```
### 方法二
对方法一做的优化。
假设从加油站x出发，第一个无法到达的加油站为y，那么必须满足的式子如下
$$
\sum_{i=x}^ycost[i]>\sum_{i=x}^ygas[i]\\
\sum_{i=x}^jgas[i]>=\sum_{i=x}^jcost[i]\space(For\space all\space j \in [x,y))
$$
其中第一个式子表示x无法到达y，第二个式子表示x可以到达y之前所有的加油站
考虑从x到y之间任意一个加油站z，能否到达加油站y，即考虑$\sum_{i=z}^ygas[i]$与$\sum_{i=z}^ycost[i]$之间的关系
根据最上面的两个式子，我们可以得到
$$
\sum_{i=z}^ygas[i]=\sum_{i=x}^ygas[i]-\sum_{i=x}^{z-1}gas[i]\\
<\sum_{i=x}^ycost[i]-\sum_{i=x}^{z-1}gas[i]\\
<\sum_{i=x}^ycost[i]-\sum_{i=x}^{z-1}cost[i]\\=\sum_{i=z}^ycost[i]
$$
所以是首先检查第0个加油站，找到第一个无法到达的加油站x，然后下一次就从加油站x+1开始检查。
```java{.line-numbers}
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int n = gas.length;
        int i = 0;
        while(i < n){
            int cnt = 0;
            int sumOfGas = 0;
            int sumOfCost = 0;
            while(cnt < n){
                int j = (i + cnt) % n;
                sumOfCost += cost[j];
                sumOfGas += gas[j];
                if(sumOfCost > sumOfGas) {
                    break;
                }
                ++cnt;
            }
            if(cnt == n){
                return i;
            } else {
                i += cnt + 1;
            }
        }
        return -1;
    }
}
```
# 距离顺序排列矩阵单元格
# 题目
给出 R 行 C 列的矩阵，其中的单元格的整数坐标为 (r, c)，满足 0 <= r < R 且 0 <= c < C。

另外，我们在该矩阵中给出了一个坐标为 (r0, c0) 的单元格。

返回矩阵中的所有单元格的坐标，并按到 (r0, c0) 的距离从最小到最大的顺序排，其中，两单元格(r1, c1) 和 (r2, c2) 之间的距离是曼哈顿距离，|r1 - r2| + |c1 - c2|。（你可以按任何满足此条件的顺序返回答案。）

示例
```
输入：R = 1, C = 2, r0 = 0, c0 = 0
输出：[[0,0],[0,1]]
解释：从 (r0, c0) 到其他单元格的距离为：[0,1]
```
## 思路
### 方法一
先保存所有的点，再按照和(r0,c0)之间的曼哈顿距离进行排序
```java{.line-numbers}
class Solution {
    public int[][] allCellsDistOrder(int R, int C, int r0, int c0) {
        int[][] ans = new int[R * C][2];
        int idx = 0;
        for(int i = 0; i < R; ++i){
            for(int j = 0; j < C; ++j){
                ans[idx][0] = i;
                ans[idx++][1] = j;
            }
        }
        Arrays.sort(ans,new Comparator<int[]>(){
            public int compare(int[] data1,int[] data2){
                int dist1 = Math.abs(data1[0] - r0) + Math.abs(data1[1] - c0);
                int dist2 = Math.abs(data2[0] - r0) + Math.abs(data2[1] - c0);
                return dist1 - dist2;
            }
        });
        return ans;
    }
}
```
### 方法二
利用桶排序的思想
```java{.line-numbers}
class Solution {
    public int[][] allCellsDistOrder(int R, int C, int r0, int c0) {
        int maxDist = Math.max(r0,R-1-r0) + Math.max(c0,C - 1 - c0);
        List<List<int[]>> bucket = new ArrayList<>();
        for(int i = 0; i <= maxDist; ++i){
            bucket.add(new ArrayList<int[]>());
        }
        for(int i = 0; i < R; ++i){
            for(int j = 0; j < C; ++j){
                int d = dist(r0,c0,i,j);
                bucket.get(d).add(new int[]{i,j});
            }
        }
        int[][] ans = new int[R * C][];
        int idx = 0;
        for(int i = 0; i <= maxDist; ++i){
            for(int[] it : bucket.get(i)){
                ans[idx++] = it;
            }
        }
        return ans;
    }

    private int dist(int r1, int c1,int r2,int c2){
        return Math.abs(r1 - r2) + Math.abs(c1 - c2);
    }
}
```
### 方法三
几何法
![](https://gitee.com/zacharytse/image/raw/master/img/20201117103033.png)
可以发现虚线正方形上的点到(r0,c0)的曼哈顿距离都是相同的，所以只要把所有这样的正方形都遍历一遍，将正方形上的所有点都加进去即可。

将(r0-1,c0)作为遍历正方形的第一个起始点 
```java{.line-numbers}
class Solution {
    int[] dr = {1,1,-1,-1};
    int[] dc = {1,-1,-1,1};

    public int[][] allCellsDistOrder(int R, int C, int r0, int c0) {
        int maxDist = Math.max(r0,R - r0 - 1) + Math.max(c0,C - c0 - 1);
        int[][] ans = new int[R * C][];
        ans[0] = new int[]{r0,c0};
        int row = r0;
        int col = c0;
        int idx = 1;
        for(int d = 1; d <= maxDist; ++d){
            --row;
            for(int i = 0; i < 4; ++i){
                //遍历边
                while((i % 2 == 0 && row != r0) || (i % 2 != 0 && col != c0)){
                    if(row >= 0 && row < R && col >= 0 && col < C){
                        ans[idx++] = new int[]{row,col};
                    }
                    row += dr[i];
                    col += dc[i];
                }
            }
        }
        return ans;
    }
}
```
# 两个数组的交集
## 题目
给定两个数组，编写一个函数来计算它们的交集。

## 思路
使用两个HashSet分别存储两个数组，然后遍历其中一个set，判断这个set中的元素是否存在于另一个set中，如果存在，则将该元素加入到结果集中
```java{.line-numbers}
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        List<Integer> ans = null;
        int[] nums = null;
        Set<Integer> set1 = initSet(nums1);
        Set<Integer> set2 = initSet(nums2);
        if(set1.size() < set2.size()){
            ans = getIntersection(set1,set2);
        } else {
            ans = getIntersection(set2,set1);
        }
        nums = new int[ans.size()];
        for(int i = 0; i < ans.size(); ++i){
            nums[i] = ans.get(i);
        }
        return nums;
    }

    List<Integer> getIntersection(Set<Integer> set1,Set<Integer> set2){
        List<Integer> ans = new ArrayList<>();
        Iterator<Integer> iter = set1.iterator();
        while(iter.hasNext()){
            int val = iter.next();
            if(set2.contains(val)){
                ans.add(val);
            }
        }
        return ans;
    }

    Set<Integer> initSet(int[] nums){
        Set<Integer> set = new HashSet<>();
        for(int val : nums){
            set.add(val);
        }
        return set;
    }
}
```
# 两两交换链表中的节点
## 题目
给定一个链表，两两交换其中相邻的节点，并返回交换后的链表。

**你不能只是单纯的改变节点内部的值**，而是需要实际的进行节点交换。
## 基本思路
采用递归的思想
分解子问题，首先是当前的子节点要进行交换，其次后面的子链表也要完成两两节点之间的互换
```ditaa {cmd=true args=["-E"] hide=true}
                      +------------------------+
+------+    +------+  | +------+    +------+   | 
|      |    |      |  | |      |    |      |   |
|  A   |--->|  B   |--+ |  C   |--->|  D   |...|
|      |    |      |  | |      |    |      |   |
+------+    +------+  | +------+    +------+   | 
    ^            ^    +------------------------+                      
    |            |
    +------------+
```
由上图可以看到，A,B交换完之后，C,D以及后面的链表仍然需要进行交换，也就是方框框起来的就代表是后面的子问题，后面子问题完成之后，只要把A(A和B互换位置后,A在B的后面)和后面的链表连接起来即可
```ditaa{cmd=true args=["-E"] hide=true}
                      +-----------------------+
+------+    +------+  |+------+    +------+   |
|      |    |      |  ||      |    |      |   |
|  B   |--->|  A   |..+|  D   |--->|  C   |...|
|      |    |      |  ||      |    |      |   |
+------+    +------+  |+------+    +------+   |
                      +-----------------------+
```
方框中的即为交换过后的子问题
```java{.line-numbers}
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode swapPairs(ListNode head) {
        if(head == null || head.next == null){
            return head;
        }
        ListNode tail = head.next;
        ListNode next = tail.next;
        tail.next = head;
        head.next = swapPairs(next);
        return tail;
    }
}
```
# 买卖股票的最佳时机
## 题目
给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。

注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

## 思路
### 方法一
贪心，如果现在的价格比前一天的价格高，就卖出
由于股票的购买没有限制，所以整个问题等价于寻找x个**不相交**的区间$(l_i,r_i]$,使得如下等式最大化
$$
    \sum_{i=1}^x a[r_i] - a[l_i]
$$
其中$l_i$表示在第$l_i$天买入，$r_i$表示在第$r_i$天卖出。
从数学上来说$a[r_i]-a[l_i]$等价于
$$
(a[r_i] - a[r_i-1]) + (a[r_i-1] -a[r_i-1]) +...+(a[l_i+1] - a[l_i])
$$
因此最后问题可以简化为找x个长度为1的区间$(l_i,l_i+1]$,使得$\sum_{i=1}^xa[l_i+1]-a[l_i]$最大
从贪心角度来考虑，只要使得每次选择的区间最后的差值大于0，那么就可以使得答案最大化
所以最后答案为
$$
ans=\sum_{i=1}^{n-1}max\{0,a[i]-a[i-1]\}
$$
```java{.line-numbers}
class Solution {
    public int maxProfit(int[] prices) {
        int ans = 0;
        for(int i = 1; i < prices.length; ++i){
            if(prices[i] > prices[i - 1]){
                ans += prices[i] - prices[i - 1];
            }
        }
        return ans;
    }
}
```
### 方法二
动态规划，定义$dp[i][0]$是第i天交易完股票不持有股票的最大收益，$dp[i][1]$是第i天交易完股票持有股票的最大收益。对于$dp[i][0]$,可能的转移状态是前一天不持有股票和前一天持有股票。如果前一天不持有股票，那么$dp[i][0] = dp[i-1][0]$,如果持有股票，那么$dp[i][0]=dp[i-1][1]+prices[i]$,所以$dp[i][0]$的转移方程如下
$$
dp[i][0] = max(dp[i-1][0],dp[i-1][1] + prices[i])
$$
同样地，对于$dp[i][1]$也是两种转移状态，即前一天不持有股票和前一天持有股票。所以$dp[i][1]$对应的转移方程如下
$$
dp[i][1] = max(dp[i-1][1],dp[i-1][0] - prices[i])
$$
```java{.line-numbers}
class Solution {
    public int maxProfit(int[] prices) {
        int[][] dp = new int[prices.length][2];
        dp[0][0] = 0;
        dp[0][1] = -prices[0];
        for(int i = 1; i < prices.length; ++i){
            dp[i][0] = Math.max(dp[i - 1][0],dp[i-1][1] + prices[i]);
            dp[i][1] = Math.max(dp[i - 1][1],dp[i - 1][0] - prices[i]);
        }
        return dp[prices.length - 1][0];
    }
}
```
# 排序链表
## 题目
给你链表的头结点 head ，请将其按 升序 排列并返回 排序后的链表 。
## 思路
### 自顶向下排序
找中间节点可以借助快慢指针
```java{.line-numbers}
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode sortList(ListNode head) {
        return mergeSort(head);
    }

    private ListNode mergeSort(ListNode node){
        if(node == null || node.next == null){
            return node;
        }
        ListNode mid = getMid(node);
        ListNode head1 = mergeSort(node);
        ListNode head2 = mergeSort(mid);
        return merge(head1,head2);
    }

    private ListNode merge(ListNode head1,ListNode head2){
        ListNode newHead = new ListNode();
        ListNode tail = newHead;
        while(head1 != null || head2 != null){
            if(head1 == null){
                tail.next = head2;
                break;
            } else if(head2 == null){
                tail.next = head1;
                break;
            }
            if(head1.val < head2.val){
                ListNode temp = head1.next;
                head1.next = null;
                tail.next = head1;
                head1 = temp;
            } else {
                ListNode temp = head2.next;
                head2.next = null;
                tail.next = head2;
                head2 = temp;
            }
            tail = tail.next;
        }
        return newHead.next;
    }

    private ListNode getMid(ListNode head){
        ListNode slow = head,quick = head;
        ListNode pre = null;
        while(slow != null && quick != null){
            pre = slow;
            slow = slow.next;
            quick = quick.next;
            if(quick != null){
                quick = quick.next;
            }
        }
        if(pre != null){
            pre.next = null;
        }
        return slow;
    }
}
```
这种方法空间复杂度是o(logn),这里产生的空间是栈空间
### 方法二 自底向上
```java{.line-numbers}
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode sortList(ListNode head) {
        if(head == null){
            return null;
        }
        int length = getLength(head);
        ListNode dumyHead = new ListNode(0,head);
        for(int subLength = 1; subLength <= length; subLength <<= 1){
            ListNode pre = dumyHead , curr = dumyHead.next;
            while(curr != null){
                ListNode head1 = curr;
                for(int i = 1; i < subLength && curr != null; ++i){
                    curr = curr.next;
                }
                ListNode head2 = null;
                if(curr != null){
                    head2 = curr.next;
                    curr.next = null;
                }
                curr = head2;
                for(int i = 1; i < subLength 
                    && curr != null && curr.next != null; ++i){
                    curr = curr.next;
                }
                if(curr != null){
                    ListNode temp = curr.next;
                    curr.next = null;
                    curr = temp;
                }
                ListNode newHead = merge(head1,head2);
                pre.next = newHead;
                while(pre != null && pre.next != null){
                    pre = pre.next;
                }
            }
        }
        return dumyHead.next;
    }

    private ListNode merge(ListNode head1,ListNode head2){
        ListNode dummyHead = new ListNode();
        ListNode tail = dummyHead;
        while(head1 != null || head2 != null){
            if(head1 == null){
                tail.next = head2;
                break;
            } else if(head2 == null){
                tail.next = head1;
                break;
            }
            if(head1.val < head2.val){
                ListNode temp = head1.next;
                head1.next = null;
                tail.next = head1;
                head1 = temp;
            } else {
                ListNode temp = head2.next;
                head2.next = null;
                tail.next = head2;
                head2 = temp;
            }
            tail = tail.next;
        }
        return dummyHead.next;
    }
    private int getLength(ListNode head){
        ListNode node = head;
        int length = 0;
        while(node != null){
            ++length;
            node = node.next;
        }
        return length;
    }
}
```
# 拼接最大数
## 题目
给定长度分别为 m 和 n 的两个数组，其元素由 0-9 构成，表示两个自然数各位上的数字。现在从这两个数组中选出 k (k <= m + n) 个数字拼接成一个新的数，要求从同一个数组中取出的数字保持其在原数组中的相对顺序。

求满足该条件的最大数。结果返回一个表示该最大数的长度为 k 的数组。

说明: 请尽可能地优化你算法的时间和空间复杂度。

示例:
```
输入:
nums1 = [3, 4, 6, 5]
nums2 = [9, 1, 2, 5, 8, 3]
k = 5
输出:
[9, 8, 6, 5, 3]
```
## 思路
先使用单调栈选出最大的子序列，然后把这个子序列合并，并和当前求出的最大序列做比较，然后更新最大序列
```java{.line-numbers}
class Solution {
    public int[] maxNumber(int[] nums1, int[] nums2, int k) {
        int m = nums1.length , n = nums2.length;
        int start = Math.max(0,k-n) , end = Math.min(k,m);
        int[] maxSubsequence = new int[k];
        for(int i = start; i <= end; ++i) {
            int[] curSubsequence1 = getSubSequence(nums1,i);
            int[] curSubsequence2 = getSubSequence(nums2,k - i);
            int[] curMaxSubsequence = merge(curSubsequence1,curSubsequence2);
            if(compare(curMaxSubsequence,0,maxSubsequence,0) > 0) {
                maxSubsequence = curMaxSubsequence;
            }
        }
        return maxSubsequence;
    }

    int[] getSubSequence(int[] nums,int k) {
        int[] stack = new int[k];
        int top = -1;
        //remain代表可以丢掉的元素个数
        int remain = nums.length - k;
        for(int i = 0; i < nums.length; ++i) {
            int num = nums[i];
            while(top >= 0 && stack[top] < num && remain > 0) {
                --top;
                --remain;
            }
            if(top < k - 1) {
                stack[++top] = nums[i];
            } else {
                //该元素不要了，已经满了
                remain--;
            }
        }
        return stack;
    }

    int[] merge(int[] nums1,int[] nums2) {
        if(nums1.length == 0) {
            return nums2;
        }
        if(nums2.length == 0) {
            return nums1;
        }
        int mergeLength = nums1.length + nums2.length;
        int[] merge = new int[mergeLength];
        int idx = 0;
        int p1 = 0 , p2 = 0;
        while(p1 < nums1.length || p2 < nums2.length) {
            if(p1 >= nums1.length) {
                merge[idx++] = nums2[p2++];
                continue;
            }
            if(p2 >= nums2.length) {
                merge[idx++] = nums1[p1++];
                continue;
            }
            //这里不直接用nums1[p1]和nums2[p2]比较
            //是因为要保证合并后的数是最大的
            //所以相等时，选择的策略应该是要保证选了这个数以后
            //后面剩下的数尽量最大
            if(compare(nums1,p1,nums2,p2) > 0) {
                merge[idx++] = nums1[p1++];
            } else {
                merge[idx++] = nums2[p2++];
            }
        }
        return merge;
    }

    int compare(int[] nums1,int idx1,int[] nums2,int idx2) {
        int n1 = nums1.length , n2 = nums2.length;
        while(idx1 < n1 && idx2 < n2) {
            int difference = nums1[idx1] - nums2[idx2];
            if(difference != 0) {
                return difference;
            }
            ++idx1;
            ++idx2;
        }
        return (n1 - idx1) - (n2 - idx2);
    }
}
```
# 奇偶链表
## 题目
给定一个单链表，把所有的奇数节点和偶数节点分别排在一起。请注意，这里的奇数节点和偶数节点指的是节点编号的奇偶性，而不是节点的值的奇偶性。

请尝试使用原地算法完成。你的算法的空间复杂度应为 O(1)，时间复杂度应为 O(nodes)，nodes 为节点总数。

## 思路
使用两个链表分别保存奇数节点和偶数节点，最后把这两个链表连接到一起
```java{.line-numbers}
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode oddEvenList(ListNode head) {
        ListNode odd =  new ListNode();
        ListNode even = new ListNode();
        ListNode oddTail = odd;
        ListNode evenTail = even;
        while(head != null){
            ListNode temp = head.next;
            head.next = null;
            oddTail.next = head;
            oddTail = oddTail.next;
            head = temp;
            if(head == null){
                break;
            }
            temp = head.next;
            head.next = null;
            evenTail.next = head;
            evenTail = evenTail.next;
            head = temp;
        }
        oddTail.next = even.next;
        return odd.next;
    }
}
```
# 求根到叶子节点数字之和
## 题目
给定一个二叉树，它的每个结点都存放一个 0-9 的数字，每条从根到叶子节点的路径都代表一个数字。

例如，从根到叶子节点路径 1->2->3 代表数字 123。

计算从根到叶子节点生成的所有数字之和。

## 基本思路
前序遍历
```java{.line-numbers}
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    private int sum = 0;
    public int sumNumbers(TreeNode root) {
        dfs(root,0);
        return sum;
    }

    private void dfs(TreeNode root,int val){
        if(root == null){
            return;
        }
        val = val * 10 + root.val;
        if(root.left == null && root.right == null){
            sum += val;
        }
        dfs(root.left,val);
        dfs(root.right,val);
    }
}
```
# 区间和的个数
## 题目
给定一个整数数组 nums，返回区间和在 [lower, upper] 之间的个数，包含 lower 和 upper。
区间和 S(i, j) 表示在 nums 中，位置从 i 到 j 的元素之和，包含 i 和 j (i ≤ j)。

## 思路
使用前缀和和归并排序。
> 前缀和在使用时，要在第一个位置加一个0，表示开始

求出前缀和preSum后，问题可以转换为找出所有满足下列条件的区间 [i,j],并统计他们的个数
$$
preSum[j] - preSum[i] \in [lower,upper]
$$
对于上面列出的表达式的求解，可以使用归并排序。考虑如下问题。对于有序数组n1和n2，需要统计满足下列条件的区间 [i,j] 个数
$$
n2[j] - n1[i] \in [lower,upper]
$$
在固定i不动的前提下，定义l，r指向n2的起始位置。要使得上述表达式满足，则必须使得$n2[l]-n1[i]>=lower$,所以要对l进行迭代，直到找到这么一个符合条件的l。同样的，也要满足表达式$n2[r] - n1[i]<=upper$,所以r也要向后迭代，直到找到一个r不满足这个表达式，那么此时$[l,r)$构成的区间即可以满足$n2[j] - n1[i] \in [lower,upper]$。
因此对于该问题来说，必须要保证preSum是有序的，所以可以考虑使用归并排序
```java{.line-numbers}
class Solution {
    public int countRangeSum(int[] nums, int lower, int upper) {
        long[] sum = new long[nums.length + 1];
        for(int i = 0; i < nums.length;++i){
            sum[i + 1] = sum[i] + nums[i];
        }
        return countRecursiveRangeSum(sum,lower,upper,0,sum.length - 1);
    }

    private int countRecursiveRangeSum(long[] sum,int lower,int upper,int left,int right){
        if(left == right){
            return 0;
        }
        int mid = (left + right) / 2;
        int n1 = countRecursiveRangeSum(sum,lower,upper,left,mid);
        int n2 = countRecursiveRangeSum(sum,lower,upper,mid + 1,right);
        int ret = n1 + n2;
        int i = left;
        int l = mid + 1;
        int r = mid + 1;
        while(i <= mid){
            while(l <= right && sum[l] - sum[i] < lower) ++l;
            while(r <= right && sum[r] - sum[i] <= upper) ++r;
            ret += r - l;
            ++i;
        }
        //合并两个排序数组
        int[] temp = new int[right - left + 1];
        int p1 = left, p2 = mid + 1;
        int p = 0;
        while(p1 <= mid || p2 <= right){
            if(p1 > mid){
                temp[p++] = (int)sum[p2++];
            } else if(p2 > right){
                temp[p++] = (int)sum[p1++];
            } else if(sum[p1] <= sum[p2]){
                temp[p++] = (int)sum[p1++];
            } else {
                temp[p++] = (int)sum[p2++];
            }
        }
        for(i = 0; i < temp.length;++i){
            sum[left + i] = temp[i];
        }
        return ret;
    }
}
```
# 三角形的最大周长
## 题目

给定由一些正数（代表长度）组成的数组 A，返回由其中三个长度组成的、面积不为零的三角形的最大周长。

如果不能形成任何面积不为零的三角形，返回 0。

示例:
```
输入：[2,1,2]
输出：5
```
## 思路
排序+贪心，排完序后，从最大的数开始遍历。设当前遍历的数为A[i]，如果A[i-1]+A[i-2] > A[i]，显然此时应该是组成了一个周长最大的三角形，如果不等式不成立，对于i-2前面的数显然更不可能成立。
```java{.line-numbers}
class Solution {
    public int largestPerimeter(int[] A) {
        Arrays.sort(A);
        int n = A.length;
        for(int i = n -1; i >= 2; --i) {
            int p1 = A[i - 2] , p2 = A[i - 1], p3 = A[i];
            if(p1 + p2 > p3) {
                return p1 + p2 + p3;
            }
        }
        return 0;
    }
}
```
# 删除链表的倒数第N个节点
## 题目
给定一个链表，删除链表的倒数第 n 个节点，并且返回链表的头结点。给定的n保证是一定有效的
## 思路
1. 先用一个节点遍历前n个节点，则剩下len-n个节点未遍历。
2. 然后再用一个指针指向头结点，并设定一个该指针的前驱指针。
3. 3个指针再同时遍历，直到第一个节点指针为null，此时前驱指针指向了待删除节点的前驱，它的后继指向了该删除的节点。
这里要考虑删除头节点的情况，所以最后要判断一下前驱节点是否为空，为空则代表是要删除头节点
```java{.line-numbers}
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode node = head;
        ListNode cur = head;
        ListNode pre = null;
        int i = 0;
        while(i++ < n){
            node = node.next;
        }
        while(node != null){
            pre = cur;
            cur = cur.next;
            node = node.next;
        }
        if(pre == null){
            head = cur.next;
        } else {
            pre.next = cur.next;
        }
        cur = null;
        return head;
    }
}
```
# 上升下降字符串
## 题目
给你一个字符串 s ，请你根据下面的算法重新构造字符串：

1. 从 s 中选出 最小 的字符，将它 接在 结果字符串的后面。
2. 从 s 剩余字符中选出 最小 的字符，且该字符比上一个添加的字符大，将它接在结果字符串后面。
3. 重复步骤 2 ，直到你没法从 s 中选择字符。
4. 从 s 中选出 最大 的字符，将它 接在 结果字符串的后面。
5. 从 s 剩余字符中选出 最大 的字符，且该字符比上一个添加的字符小，将它接在结果字符串后面。
6. 重复步骤 5 ，直到你没法从 s 中选择字符。
7. 重复步骤 1 到 6 ，直到 s 中所有字符都已经被选过。

在任何一步中，如果最小或者最大字符不止一个 ，你可以选择其中任意一个，并将其添加到结果字符串。

请你返回将 s 中字符重新排序后的 结果字符串 。

示例:
```
输入：s = "aaaabbbbcccc"
输出："abccbaabccba"
解释：第一轮的步骤 1，2，3 后，结果字符串为 result = "abc"
第一轮的步骤 4，5，6 后，结果字符串为 result = "abccba"
第一轮结束，现在 s = "aabbcc" ，我们再次回到步骤 1
第二轮的步骤 1，2，3 后，结果字符串为 result = "abccbaabc"
第二轮的步骤 4，5，6 后，结果字符串为 result = "abccbaabccba"
```

## 思路
### 方法一
先对原字符串排序，然后根据题目所给步骤进行模拟
```java{.line-numbers}
class Solution {
    public String sortString(String s) {
        char[] arr = s.toCharArray();
        Arrays.sort(arr);
        StringBuilder builder = new StringBuilder();
        int count = arr.length;
        while(count > 0) {
            int minIdx = 0;
            while(arr[minIdx] == (char)-1) {
                ++minIdx;
            }
            char last = arr[minIdx];
            builder.append(arr[minIdx]);
            arr[minIdx] = (char)-1;
            --count;
            int idx = minIdx + 1;
            while(idx < arr.length && count > 0) {
                if(arr[idx] != (char)-1 && arr[idx] != last) {
                    builder.append(arr[idx]);
                    last = arr[idx];
                    arr[idx] = (char)-1;
                    --count;
                }
                ++idx;
            }
            if(count <= 0) {
                break;
            }
            int maxIdx = arr.length - 1;
            while(arr[maxIdx] == (char)-1) {
                --maxIdx;
            }
            last = arr[maxIdx];
            builder.append(arr[maxIdx]);
            arr[maxIdx] = (char)-1;
            --count;
            idx = maxIdx - 1;
            while(idx >= 0 && count > 0) {
                if(arr[idx] != (char)-1 && last != arr[idx]) {
                    builder.append(arr[idx]);
                    last = arr[idx];
                    arr[idx] = (char)-1;
                    --count;
                }
                --idx;
            }
        }
        return builder.toString();
    }
}
```
### 方法二(桶排序)
因为不关心原有字符在字符串中的位置，所以只要用一个桶记录每个字母出现的次数，然后分别从左到右和从右到左进行遍历这个桶，从桶中取出一个字母，则该字母出现次数减1，最后直到新构造的字符串的长度和原有字符串一致，则停止遍历。
```java{.line-numbers}
class Solution {
    public String sortString(String s) {
        int[] dict = new int[26];
        for(int i = 0; i < s.length(); ++i) {
            ++dict[s.charAt(i) - 'a'];
        }
        StringBuilder builder = new StringBuilder();
        while(builder.length() < s.length()) {
            for(int i = 0; i < dict.length; ++i) {
                if(dict[i] != 0) {
                    builder.append((char)(i + 'a'));
                    --dict[i];
                }
            }
            for(int i = dict.length - 1; i >= 0; --i) {
                if(dict[i] != 0) {
                    builder.append((char)(i + 'a'));
                    --dict[i];
                }
            }
        }
        return builder.toString();
    }
}
```
# 视频拼接
## 题目
你将会获得一系列视频片段，这些片段来自于一项持续时长为 T 秒的体育赛事。这些片段可能有所重叠，也可能长度不一。
视频片段 clips[i] 都用区间进行表示：开始于 clips[i][0] 并于 clips[i][1] 结束。我们甚至可以对这些片段自由地再剪辑，例如片段 [0, 7] 可以剪切成 [0, 1] + [1, 3] + [3, 7] 三部分。

我们需要将这些片段进行再剪辑，并将剪辑后的内容拼接成覆盖整个运动过程的片段（[0, T]）。返回所需片段的最小数目，如果无法完成该任务，则返回 -1 。
## 基本思路
### 方法一
**动态规划**
定义dp[i]为覆盖区间[0,i)需要的最少片段数。对于区间$[a_j,b_j]$,如果$a_j < i <= b_j$,那么可以认为区间$[a_j,b_j]$覆盖了[0,i)的后半段$[a_j,i)$,那么只需要再加上前半段$[0,a_j)$的最优解，就可以求出覆盖区间[0,i)所需要的最少片段数。前半段需要的区间数可以用$dp[a_j]$来表示，那么最后的方程可以表示为
$$
dp[i] = min(dp[a_j]) + 1,a_j < i <= b_j
$$
```java{.line-numbers}
class Solution {
    public int videoStitching(int[][] clips, int T) {
        int[] dp = new int[T + 1];
        Arrays.fill(dp,Integer.MAX_VALUE - 1);
        dp[0] = 0;
        for(int i = 1; i <= T;++i){
            for(int[] clip : clips){
                if(clip[0] < i && clip[1] >= i){
                    dp[i] = Math.min(dp[i],dp[clip[0]] + 1);
                }
            }
        }
        return dp[T] == Integer.MAX_VALUE - 1 ? -1 : dp[T];
    }
}
```
### 方法二
**贪心**
首先求出[0,T)中每个点可以到达的最远位置，记为maxn[i]。再枚举每一个具体的位置，当枚举到位置i时，记左端点不大于i的所有区间中最远的右端点为last。每次枚举到一个新的位置就用maxn[i]来更新last，如果更新后，last==i,那么就说明[0,i)这个区间是跨不过去的，所以最后返回-1。同时要用一个pre记录上一个区间可到达的最远位置。如果$i==pre$,则说明现在已经越过了上一个区间可覆盖的位置，因此总共需要的区间数需要加1，同时要更新pre为当前位置可到达的最远位置
```java{.line-numbers}
class Solution {
    public int videoStitching(int[][] clips, int T) {
        int[] maxn = new int[T];
        int last = 0, pre = 0 , ret = 0;
        for(int[] clip : clips){
            if(clip[0] < T){
                maxn[clip[0]] = Math.max(maxn[clip[0]],clip[1]);
            }
        }
        for(int i = 0; i < T;++i){
            last = Math.max(last,maxn[i]);
            if(last == i){
                return - 1;
            }
            if(i == pre){
                ++ret;
                pre = last;
            }
        }
        return ret;
    }
}
```
# 数组的相对排序
## 题目
给你两个数组，arr1 和 arr2，

- arr2 中的元素各不相同
- arr2 中的每个元素都出现在 arr1 中

对 arr1 中的元素进行排序，使 arr1 中项的相对顺序和 arr2 中的相对顺序相同。未在 arr2 中出现过的元素需要按照升序放在 arr1 的末尾。

提示：

- arr1.length, arr2.length <= 1000
- 0 <= arr1[i], arr2[i] <= 1000
- arr2 中的元素 arr2[i] 各不相同
- arr2 中的每个元素 arr2[i] 都出现在 arr1 中

## 思路
由提示0 <= arr1[i], arr2[i] <= 1000可以想到使用计数排序。先用一个数组记录arr1中每一个数出现的个数，然后遍历arr2，将arr2中的元素按arr1中出现的个数填充到最后的结果中，并将遍历过的元素个数置为0。接着遍历记录出现次数的数组，将其中仍然不为0的元素对应的下标填充到最后结果中。
```java{.line-numbers}
class Solution {
    public int[] relativeSortArray(int[] arr1, int[] arr2) {
        int[] ans = new int[arr1.length];
        int[] counts = new int[1001];
        int idx = 0;
        for(int i = 0; i < arr1.length; ++i){
            ++counts[arr1[i]];
        }
        for(int i = 0; i < arr2.length; ++i){
            while(counts[arr2[i]] != 0){
                ans[idx++] = arr2[i];
                --counts[arr2[i]];
            }
        }
        for(int i = 0; i < counts.length; ++i){
            while(counts[i] != 0){
                ans[idx++] = i;
                --counts[i];
            }
        }
        return ans;
    }
}
```
# 数组中的最长山脉
## 题目
我们把数组 A 中符合下列属性的任意连续子数组 B 称为 “山脉”：

B.length >= 3
存在 0 < i < B.length - 1 使得 B[0] < B[1] < ... B[i-1] < B[i] > B[i+1] > ... > B[B.length - 1]
（注意：B 可以是 A 的任意子数组，包括整个数组 A。）

给出一个整数数组 A，返回最长 “山脉” 的长度。

如果不含有 “山脉” 则返回 0。

## 基本思路
通过状态定义的方法遍历整个数组。状态共有4种:INIT,UP0,UP1,DOWN，如果有一个区间能够完整的经历这4种状态，那么这个区间一定是符合题目条件的，最后求出所有满足条件的区间中长度最长的那个，并返回它的长度。
- INIT：区间的初始化状态
- UP0:区间第一次进入单调递增状态
- UP1:说明区间正式进入单调递增状态，只能从UP1转为DOWN，不能由UP0转为DOWN
- DOWN:区间的单调下降状态
这里设置两个递增状态主要是为了防止出现类似[3,2]这种情况，刚从INIT态转到UP态就直接DOWN了，是不符合题意的。
这里要注意从DOWN态切回INIT态时，i是要减2的，这是为了保证下次访问的是之前区间的最低谷的位置，而不是直接跳过这个位置,如[1,5,4,2,3],访问到3时，应该从DOWN切回INIT态，但是下次的访问位置应该是2，所以i要减2
```java{.line-numbers}
class Solution {
    public int longestMountain(int[] A) {
        final int UP0 = 0;
        final int UP1 = 1;
        final int DOWN = 2;
        final int INIT = 3;
        int STATE = INIT;
        int cur = 0 , maxDist = 0;
        for(int i = 0; i < A.length; ++i){
            switch(STATE){
                case INIT:
                    cur = i;
                    STATE = UP0;
                    break;
                case UP0:
                    if(A[i] > A[i - 1]){
                        STATE = UP1;
                    } else {
                        STATE = INIT;
                        --i;
                    }
                    break;
                case UP1:
                    if(A[i] < A[i - 1]){
                        STATE = DOWN;
                    } else if(A[i] == A[i - 1]){
                        STATE = INIT;
                        --i;
                    }
                    break;
                case DOWN:
                    if(A[i] >= A[i - 1]){
                        STATE = INIT;
                        maxDist = Math.max(maxDist,i - cur);
                        i -= 2;
                    }
                    break;
            }
        }
        if(STATE == DOWN){
            maxDist = Math.max(maxDist,A.length - cur);
        }
        return maxDist;
    }
}
```
# 四数相加
## 题目
给定四个包含整数的数组列表 A , B , C , D ,计算有多少个元组 (i, j, k, l) ，使得 A[i] + B[j] + C[k] + D[l] = 0。

为了使问题简单化，所有的 A, B, C, D 具有相同的长度 N，且 0 ≤ N ≤ 500 。所有整数的范围在 $-2^{28}$ 到 $2^{28} - 1$ 之间，最终结果不会超过 $2^{31} - 1$ 。

示例:
```
输入:
A = [ 1, 2]
B = [-2,-1]
C = [-1, 2]
D = [ 0, 2]

输出:
2

解释:
两个元组如下:
1. (0, 0, 0, 1) -> A[0] + B[0] + C[0] + D[1] = 1 + (-2) + (-1) + 2 = 0
2. (1, 1, 0, 0) -> A[1] + B[1] + C[0] + D[0] = 2 + (-1) + (-1) + 0 = 0
```
## 思路
### 方法一
用两个map分别记录A+B和C+D的和出现的次数，遍历其中一个map，如果发现这个map中有一个key是另一个map的key的相反数，则说明他们的和为0，总的元组数=两个map中key的出现次数的乘积
```java{.line-numbers}
class Solution {
    public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
        int n = A.length;
        if(n == 0) {
            return 0;
        }
        Map<Integer,Integer> mapAB = new HashMap<>();
        Map<Integer,Integer> mapCD = new HashMap<>();
        for(int i = 0; i < n; ++i) {
            for(int j = 0; j < n; ++j) {
                insertNewSum(mapAB,A[i] + B[j]);
                insertNewSum(mapCD,C[i] + D[j]);
            }
        }
        int ans = 0;
        for(Map.Entry<Integer,Integer> entry : mapAB.entrySet()) {
            if(mapCD.containsKey(-entry.getKey())) {
                ans += entry.getValue() * mapCD.get(-entry.getKey());
            }
        }
        return ans;
    }

    private void insertNewSum(Map<Integer,Integer> map,int sum) {
        int count = 0;
        if(map.containsKey(sum)) {
            count = map.get(sum);
        }
        map.put(sum,++count);
    }
}
```
### 优化
对于C+D不需要用map去存储了，直接做二重循环遍历就行了
```java{.line-numbers}
class Solution {
    public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
        int n = A.length;
        if(n == 0) {
            return 0;
        }
        Map<Integer,Integer> map = new HashMap<>();
        for(int i = 0; i < n; ++i) {
            for(int j = 0; j < n; ++j) {
                map.put(A[i] + B[j],map.getOrDefault(A[i] + B[j],0) + 1);
            }
        }
        int ans = 0;
        for(int i = 0; i < n; ++i) {
            for(int j = 0; j < n; ++j) {
                int sum = -(C[i] + D[j]);
                if(map.containsKey(sum)) {
                    ans += map.get(sum);
                }
            }
        }
        return ans;
    }
}
```
# 填充每个节点的下一个右侧节点指针
## 题目
给定一个完美二叉树，其所有叶子节点都在同一层，每个父节点都有两个子节点。二叉树定义如下：
```C
struct Node {
  int val;
  Node *left;
  Node *right;
  Node *next;
}
```
填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 NULL。
初始状态下，所有next指针都被设置为NULL。
![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/02/15/116_sample.png)
## 思路
做层次遍历，并且保存每一次遍历的前驱节点做连接。
```java{.line-numbers}
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;
    public Node next;

    public Node() {}
    
    public Node(int _val) {
        val = _val;
    }

    public Node(int _val, Node _left, Node _right, Node _next) {
        val = _val;
        left = _left;
        right = _right;
        next = _next;
    }
};
*/

class Solution {
    public Node connect(Node root) {
        if(root == null){
            return null;
        }
        int front = 0,rear = 1,last = 1;
        Node pre = null;
        Queue<Node> q = new LinkedList<>();
        q.offer(root);
        while(q.size() != 0){
            Node node = q.poll();
            ++front;
            if(node.left != null){
                q.offer(node.left);
                ++rear;
            }
            if(node.right != null){
                q.offer(node.right);
                ++rear;
            }
            if(pre != null){
                pre.next = node;
            }
            pre = node;
            if(front == last){
                pre = null;
                last = rear;
                continue;
            }
        }
        return root;
    }
}
```
# 完全二叉树的节点个数
## 题目
给出一个完全二叉树，求出该树的节点个数。
## 思路
### 方法一(暴力)
```java{.line-numbers}
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int countNodes(TreeNode root) {
        if(root == null){
            return 0;
        }
        return countNodes(root.left) + countNodes(root.right) + 1;
    }
}
```
### 方法二(二分查找+位运算)
对于一棵完全二叉树，如果最后一层只有一个节点，那么它的节点个数为$2^h$,如果最后一层是满节点，那么它的节点个数为$2^{h+1}-1$，也就是说对于h层的完全二叉树，它的节点个数范围是在$[2^{h},2^{h+1}-1]$中。可以使用二分查找，对于最后一层的节点k，如果k存在，说明节点个数一定是大于等于k，如果k不存在，则节点个数一定小于k。

**使用二分查找进行范围查找而不是某个数的查找时，定义mid可以向上取整，这样可以避免死循环，即mid=low + (high - low + 1)/2**

对于一棵完全二叉树，可以用二进制来表示最底层的k节点到根节点的路径。对于h层的完全二叉树，可以用h位的二进制来表示，最高位始终设为1，然后从根节点出发，如果子节点是父节点的左孩子，则为0，否则为1。可以根据这个二进制构成的路径判断k是否存在
```java{.line-numbers}
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int countNodes(TreeNode root) {
        if(root == null) {
            return 0;
        }
        int level = 0;
        TreeNode node = root;
        while(node.left != null){
            ++level;
            node = node.left;
        }
        int low = 1 << level;
        int high = (1 << (level + 1)) - 1;
        while(low < high) {
            int mid = low + (high - low + 1) / 2;
            if(exist(root,level,mid)) {
                low = mid;
            } else {
                high = mid - 1;
            }
        }
        return low;
    }

    private boolean exist(TreeNode node,int level,int k){
        TreeNode root = node;
        int bits = 1 << (level - 1);
        while(root != null && bits > 0) {
            if((bits & k) == 0) {
                root = root.left;
            } else {
                root = root.right;
            }
            bits >>= 1;
        }
        return root != null;
    }
}
```
# 下一个排列
## 题目
实现获取下一个排列的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。

如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。

必须原地修改，只允许使用额外常数空间。

## 思路
题目目标是为了找一个和当前排列相比较大，但相较于原来的值变化又不太大的排列。
可以先从右往左找一个较小的数(nums[i] < nums[i+1])，下标为i，然后再从右往左找一个比这个较小的数要大一点的数，下标为j，将两者进行交换。
此时区间[i+1,nums.length - 1]一定是一个递减区间，所以找到的j一定是这个区间中最小的数，和i交换后，也仍然保持递减的性质。同时为了保证与原来的值相比变化不大，所以要把区间[i+1,nums.length-1]进行逆转，使其递增
```java{.line-numbers}
class Solution {
    public void nextPermutation(int[] nums) {
        int i = nums.length - 2;
        while(i >= 0 && nums[i] >= nums[i + 1]){
            --i;
        }
        if(i >= 0){
            int j = nums.length - 1;
            while(j >= i+1 && nums[i] >= nums[j]){//要找一个完全比nums[i]大的数
                --j;
            }
            int temp = nums[i];
            nums[i] = nums[j];
            nums[j] = temp;
        }
        reverse(nums,i + 1, nums.length - 1);
    }

    private void reverse(int[] nums,int low ,int high){
        while(low < high){
            int temp = nums[low];
            nums[low] = nums[high];
            nums[high] = temp;
            ++low;
            --high;
        }
    }
}
```
# 移掉k位数字
## 题目
给定一个以字符串表示的非负整数 num，移除这个数中的 k 位数字，使得剩下的数字最小。

注意:

- num 的长度小于 10002 且 ≥ k。
- num 不会包含任何前导零。

## 思路
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
# 移动零
## 题目
给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。
示例:
```
输入: [0,1,0,3,12]
输出: [1,3,12,0,0]
```
说明:
1. 必须在原数组上操作，不能拷贝额外的数组。
2. 尽量减少操作次数。

## 思路
### 方法一
用一个指向数组第一个0元素的指针完成元素的交换。先找到数组中第一个0元素的位置，然后从该元素后面一个元素开始遍历，如果找到一个非0元素，则将该元素和指针指向的0进行交换，指针向后移动一格(这里因为是在遍历过程中进行交换，所以可以保证指针在后移后，一定还是指向了0)
```java{.line-numbers}
class Solution {
    public void moveZeroes(int[] nums) {
        int p = 0;
        while(p < nums.length){
            if(nums[p] == 0){
                break;
            }
            ++p;
        }
        for(int i = p + 1; i < nums.length; ++i){
            if(nums[i] != 0){
                nums[p] = nums[i];
                nums[i] = 0;
                ++p;
            }
        }
    }
}
```
### 方法二
双指针，整体思路和方法一差不多，只不过方法一是指针始终指向了数组第一个为0元素的位置。双指针是定义两个指针，左指针指向了处理好的序列尾部，右指针指向了待处理序列的头部。所以当右指针找到一个不为0的元素时，从该元素往前一直到左指针指向的位置都可以保证是0元素，直接将右指针放到该0元素位置即可。
但要注意在交换时，数组中只有一个非0元素的情况。为了避免0覆盖了原有的值，在交换时，要先把0交换到右指针的位置，再把右指针的值交换到左指针处
```java{.line-numbers}
class Solution {
    public void moveZeroes(int[] nums) {
        int n = nums.length;
        int left = 0,right = 0;
        while(right < n){
            if(nums[right] != 0){
                int temp = nums[right];
                nums[right] = 0;
                nums[left] = temp;
                ++left;
            }
            ++right;
        }
    }
}
```
# 用最少数量的箭引爆气球
## 题目
在二维空间中有许多球形的气球。对于每个气球，提供的输入是水平方向上，气球直径的开始和结束坐标。由于它是水平的，所以纵坐标并不重要，因此只要知道开始和结束的横坐标就足够了。开始坐标总是小于结束坐标。

一支弓箭可以沿着 x 轴从不同点完全垂直地射出。在坐标 x 处射出一支箭，若有一个气球的直径的开始和结束坐标为 xstart，xend， 且满足  xstart ≤ x ≤ xend，则该气球会被引爆。可以射出的弓箭的数量没有限制。 弓箭一旦被射出之后，可以无限地前进。我们想找到使得所有气球全部被引爆，所需的弓箭的最小数量。

给你一个数组 points ，其中 points [i] = [xstart,xend] ，返回引爆所有气球所必须射出的最小弓箭数。

示例:
```
输入：points = [[10,16],[2,8],[1,6],[7,12]]
输出：2
解释：对于该样例，x = 6 可以射爆 [2,8],[1,6] 两个气球，以及 x = 11 射爆另外两个气球
```
## 思路
先按每个区间的开始位置按升序排序，然后开始遍历，如果当前区间可以和之前区间得到的交集区间进行合并，则将该区间合并进去，否则重新求交集，并且此时需要多射一根箭。
```java{.line-numbers}
class Solution {
    public int findMinArrowShots(int[][] points) {
        if(points.length == 0){
            return 0;
        }
        if(points.length == 1){
            return 1;
        }
        Arrays.sort(points,new Comparator<int[]>(){
            public int compare(int[] point1,int[] point2){
                if(point1[0] > point2[0]){
                    return 1;
                } else if(point1[0] == point2[0]){
                    return 0;
                } else {
                    return -1;
                }
            }
        });
        int cnt = 1;
        int[] fightPoint = points[0];
        for(int i = 1; i < points.length; ++i){
            int[] intercation = getIntercation(fightPoint,points[i]);
            if(intercation == null){
                ++cnt;
                fightPoint = points[i];
            } else {
                fightPoint = intercation;
            }
        }
        return cnt;
    }

    //求交集时，默认point1在point2的左侧
    private int[] getIntercation(int[] point1,int[] point2){
        if(point1[1] < point2[0]){
            return null;
        }
        int[] intercation = new int[2];
        intercation[0] = Math.max(point1[0],point2[0]);
        intercation[1] = Math.min(point1[1],point2[1]);
        return intercation;
    }
}
```
# 有多少小于当前数字的数字
## 题目
给你一个数组 nums，对于其中每个元素 nums[i]，请你统计数组中比它小的所有数字的数目。

换而言之，对于每个 nums[i] 你必须计算出有效的 j 的数量，其中 j 满足 j != i 且 nums[j] < nums[i] 。

以数组形式返回答案。

## 基本思路
- 方法一
使用计数排序的方法，先用数组hash记录每一个数字出现的次数，则数组中每一个比当前数小的所有数的总和即为hash[0,....i-1]的总和
```java{.line-numbers}
class Solution {
    public int[] smallerNumbersThanCurrent(int[] nums) {
        int[] hash = new int[101];
        int[] ans = new int[nums.length];
        for(int i = 0; i < nums.length;++i){
            ++hash[nums[i]];
        }
        for(int i = 1; i < hash.length;++i){
            hash[i] += hash[i - 1];
        }
        for(int i = 0; i < nums.length;++i){
            ans[i] = (nums[i] == 0 ? 0 : hash[nums[i] - 1]);
        }
        return ans;
    }
}
```
- 方法二
使用快速排序的方法。记录下数组中每个元素的下标，然后对原数组进行排序，排完序后，每个元素左边元素的数量即为小于该元素的元素数量总和
```java{.line-numbers}
class Solution {
    public int[] smallerNumbersThanCurrent(int[] nums) {
        int[][] data = new int[nums.length][2];
        int[] ans = new int[nums.length];
        for(int i = 0; i < nums.length;++i){
            data[i][0] = nums[i];
            data[i][1] = i;
        }
        Arrays.sort(data,new Comparator<int[]>(){
            public int compare(int[] data1,int[] data2){
                return data1[0] - data2[0];
            }
        });
        int prev = -1;
        for(int i = 0; i < data.length;++i){
           if(prev == -1 || data[i][0] != data[i - 1][0]){
               //考虑i==0或者元素相同的情况
               prev = i;
           }
           ans[data[i][1]] = prev;
        }
        return ans;
    }
}
```
# 有效的山脉数组
## 题目
给定一个整数数组 A，如果它是有效的山脉数组就返回 true，否则返回 false。

让我们回顾一下，如果 A 满足下述条件，那么它是一个山脉数组：

A.length >= 3
在 0 < i < A.length - 1 条件下，存在 i 使得：
A[0] < A[1] < ... A[i-1] < A[i]
A[i] > A[i+1] > ... > A[A.length - 1]

## 思路
```java{.line-numbers}
class Solution {
    public boolean validMountainArray(int[] A) {
        final int UP0 = 1;
        final int UP1 = 3;
        final int DOWN = 2;
        final int INIT = 0;
        int STATE = INIT;
        for(int i = 0; i < A.length;++i){
            switch(STATE){
            case INIT:
                STATE = UP0;
                break;
            case UP0:
                if(A[i] > A[i - 1]){
                    STATE = UP1;
                } else {
                    return false;
                }
                break;
            case UP1:
                if(A[i] < A[i - 1]){
                    STATE = DOWN;
                } else if(A[i] == A[i - 1]){
                    return false;
                }
                break;
            case DOWN:
                if(A[i] >= A[i - 1]){
                    return false;
                }
                break;
            }
        }
        if(STATE == DOWN){
            return true;
        } else {
            return false;
        }
    }
}
```
# 有效的字母异位词
## 题目
给定两个字符串 s 和 t ，编写一个函数来判断 t 是否是 s 的字母异位词。

示例:
```
输入: s = "anagram", t = "nagaram"
输出: true
```
## 思路
hash
```java{.line-numbers}
class Solution {
    public boolean isAnagram(String s, String t) {
        int[] dict = new int[26];
        for(int i = 0; i < s.length();++i){
            ++dict[s.charAt(i) - 'a'];
        }
        for(int i = 0; i < t.length();++i){
            --dict[t.charAt(i) - 'a'];
        }
        for(int i = 0; i < 26; ++i){
            if(dict[i] != 0){
                return false;
            }
        }
        return true;
    }
}
```
# 有序数组的平方
##题目
给定一个按非递减顺序排序的整数数组 A，返回每个数字的平方组成的新数组，要求也按非递减顺序排序。
##思路
###方法1
先求出数组中每一个元素的绝对值，然后对这些绝对值进行排序。再对排序过后的数组求平方
```java{.line-numbers}
class Solution {
    public int[] sortedSquares(int[] A) {
        for(int i = 0; i < A.length;++i){
            A[i] = Math.abs(A[i]);
        }
        Arrays.sort(A);
        for(int i = 0; i < A.length;++i){
            A[i] = (int)Math.pow(A[i],2);
        }
        return A;
    }
}
```
###方法二
使用双指针。因为原数组是非递减的，对于数组的负数部分来说，求平方后是非递增的，对于非负数部分来说，平方后同样是非递减的。也就是说只要从非负数与负数的交界处对原数组进行分割，那么就相当于是两个有序数组在做合并
```java{.line-numbers}
class Solution {
    public int[] sortedSquares(int[] A) {
        int neg = -1;
        for(int i = 0; i < A.length;++i){
            if(A[i] < 0){
                neg = i;
            } else{
                break;
            }
        }
        int i = neg , j = neg + 1;
        int[] ans = new int[A.length];
        int pos = 0;
        while(i >= 0 || j < A.length){
            if(i < 0){
                ans[pos++] = (int)Math.pow(A[j],2);
                ++j;
                continue;
            }
            if(j >= A.length){
                ans[pos++] = (int)Math.pow(A[i],2);
                --i;
                continue;
            }
            if(A[i] * A[i] < A[j] * A[j]){
                ans[pos++] = A[i] * A[i];
                --i;
            } else{
                ans[pos++] = A[j] * A[j];
                ++j;
            }
        }
        return ans;
    }
}
```
###方法三
同样是使用双指针，但是不用对方法二中的i，j做边界判断了。由方法二的分析可知，是两个有序数组在做合并，对于前半部分的非递增数组，平方后是非递减数组，后半部分的数组平方后是个非递增数组，那么只要设置两个指针，一个指针指向前半部分数组的头，还有一个指针指向后半部分数组的尾，以此来进行合并。
```java{.line-numbers}
class Solution {
    public int[] sortedSquares(int[] A) {
        int i = 0 , j = A.length - 1;
        int pos = A.length - 1;
        int[] ans = new int[A.length];
        while(i <= j){
            if(A[i] * A[i] < A[j] * A[j]){
                ans[pos--] = A[j] * A[j];
                --j;
            } else {
                ans[pos--] = A[i] * A[i];
                ++i;
            }
        }
        return ans;
    }
}
```
# 在排序数组中查找元素的第一个和最后一个位置
## 题目
给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 target，返回 [-1, -1]。

进阶：

你可以设计并实现时间复杂度为 O(log n) 的算法解决此问题吗？
 
示例:
```java{.line-numbers}
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]
```
## 思路
二分查找即可
```java{.line-numbers}
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int[] ans = new int[2];
        ans[0] = findTarget(nums,target,true);
        ans[1] = findTarget(nums,target,false);
        return ans;
    }

    private int findTarget(int[] nums, int target,boolean isLower) {
        int low = 0 , high = nums.length - 1;
        while(low <= high) {
            int mid = low + (high - low) / 2;
            if(nums[mid] == target) {
                if(isLower) {
                    if(mid == low || nums[mid] > nums[mid - 1]) {
                        return mid;
                    } else {
                        high = mid - 1;
                    }
                } else {
                    if(mid == high || nums[mid] < nums[mid + 1]) {
                        return mid;
                    } else {
                        low = mid + 1;
                    }
                }
            } else if(nums[mid] > target) {
                high = mid - 1;
            } else {
                low = mid + 1;
            } 
        }
        return -1;
    }
}
```
# 长按键入
##长按键入
你的朋友正在使用键盘输入他的名字 name。偶尔，在键入字符 c 时，按键可能会被长按，而字符可能被输入 1 次或多次。

你将会检查键盘输入的字符 typed。如果它对应的可能是你的朋友的名字（其中一些字符可能被长按），那么就返回 True。
##基本思路
用一个指针指向typed,一个指针指向name。对于typed中的元素，如果要匹配上name中的字符，有且只能满足两个条件中的一个。要么typed中的元素和name中的元素相同，如果不同的话，那么该元素就必须和typed的前一个元素相同，也就是该元素是重复输入的
```java{.line-numbers}
class Solution {
    public boolean isLongPressedName(String name, String typed) {
        if(name.length() > typed.length()){
            return false;
        }
        int pos = 0;
        for(int i = 0; i < typed.length();++i){
            if(pos >= name.length() || typed.charAt(i) != name.charAt(pos)){
                if(i > 0 && typed.charAt(i) == typed.charAt(i - 1)){
                    continue;
                }
                return false;
            }
            if(typed.charAt(i) == name.charAt(pos)){
                ++pos;
            }
        }
        if(pos < name.length()){
            return false;
        }
        return true;
    }
}
```
# 重构字符串
## 题目
给定一个字符串S，检查是否能重新排布其中的字母，使得两相邻的字符不同。

若可行，输出任意可行的结果。若不可行，返回空字符串。

示例:
```
输入: S = "aab"
输出: "aba"
```
## 思路
要构成题目中的字符串，首先要先考虑原字符串需要满足什么样的条件。对于出现次数最多的字符，要保证它的相邻位置和它是不一样的字符，显然该字符的个数不能超过字符串长度的一半。
- 当字符串长度为偶数时，对于某一个字符，它允许出现的最大次数应该是n/2
- 当字符串长度为奇数时，对于某一个字符，它允许出现的最大次数应该是(n + 1) / 2

因为是向下取整的，所以当字符串长度为偶数时，n/2和(n+1)/2的值是相等的，所以字符串中的字符允许出现的最大次数为(n+1)/2，如果超过了这个次数，则不可能构成符合题目条件的字符串。
对于字符出现的次数，可以通过hash表来统计。统计完成后，就是要构造这个字符串

### 方法一
采用大顶堆。将每个字符根据它的出现次数放入大顶堆中。每次从堆中取2个字符，将它们拼接在一起，然后出现次数都各减一，如果剩余的出现次数不为0的话，则继续放入堆中。如果堆中元素个数仅剩一个的话，则将其取出，放到结果的末尾，并返回结果。
```java{.line-numbers}
class Solution {
    public String reorganizeString(String s) {
        Comparator<Pair<Integer,Character>> comparator = null;
        comparator = new Comparator<>() {
            public int compare(Pair<Integer,Character> p1,Pair<Integer,Character> p2) {
                return p2.getKey() - p1.getKey();
            }
        };
        PriorityQueue<Pair<Integer,Character>> queue =  
                    new PriorityQueue<>(comparator);
        int[] dict = countCharacter(s);
        int maxCount = (s.length() + 1) / 2;
        for(int i = 0; i < dict.length; ++i) {
            if(dict[i] != 0) {
                if(dict[i] > maxCount) {
                    return "";
                }
                Pair<Integer,Character> p = 
                        new Pair<>(dict[i],(char)(i + 'a'));
                queue.offer(p);
            }
        }
        StringBuilder builder = new StringBuilder();
        char last = 0;
        while(!queue.isEmpty()) {
            Pair<Integer,Character> p1 = queue.poll();
            Pair<Integer,Character> p2 = null;
            if(!queue.isEmpty()) {
                p2 = queue.poll();
            } else {
                builder.append(p1.getValue());
                return builder.toString();
            }
            builder.append(p1.getValue());
            builder.append(p2.getValue());
            int count1 = p1.getKey() - 1;
            int count2 = p2.getKey() - 1;
            if(count1 != 0) {
                queue.offer(
                    new Pair<Integer,Character>(count1,p1.getValue()));
            }
            if(count2 != 0) {
                queue.offer(
                    new Pair<Integer,Character>(count2,p2.getValue()));
            }
        }
        return builder.toString();
    }

    private int[] countCharacter(String s) {
        int[] dict = new int[26];
        for(int i = 0; i < s.length(); ++i) {
            ++dict[s.charAt(i) - 'a'];
        }
        return dict;
    }
}
```
### 方法二
计数法
当n是奇数且出现最多的字母次数为(n+1)/2时，则出现次数最多的这个字母必须放在偶数位下标，否则一定会出现相同字母相邻的情况(偶数位比奇数位多一个，放在奇数位，一定会有一个字母被放在偶数位，此时会出现相邻)

首先考虑能不能放在奇数位，只要字母出现的次数不超过字符串长度的一半(n/2),就可以放置在奇数下标。只有当字母出现次数超过字符串长度的一半时，才必须放在偶数下标(只有在n为奇数时才会出现这种情况)。这时也最多只有一个字母出现的次数会超过字符串的一半。

具体操作如下:
- 如果字母出现的次数小于等于n/2，且oddIndex没有超出数组的下标范围，则将字母放在oddIndex，并且oddIndex+2
- 如果字母出现次数大于n/2，或者oddIndex超出数组的下标范围，则将字母放在evenIndex，并且evenIndex+2

证明如下:
1. 当n为奇数，且字母出现的最大次数为(n+1)/2，则该字母可以占满所有的偶数位，此时一定不可能出现相邻的情况
2. 如果一个字母全部被放在偶数位或者奇数位，则不可能出现相邻的情况
3. 如果同一个字母先被放在了奇数位，当奇数位超出了数组范围后，才将其放到偶数位。
    - 当n为偶数时，如果该字母出现次数为n/2，奇数位下标个数为p，偶数位下标个数为q，p+q=n/2,且最小奇数位下标为n+1-2(p-1) = n-2p+1,最大的偶数位下标为2(q-1),两者之差为3
    - 当n为奇数时，如果该字母出现次数为(n-1)/2,奇数位下标个数为p，偶数位下标个数为q，p+q=(n-1)/2,且最小奇数位下标为n-1-2(p-1)=n-2p，最大偶数位下标为2(q-1),两者之差为3

    综上，最小的奇数位下标和最大的偶数位下标之差大于等于3，因此两者是不可能相邻的。
综上，算法成立
```java{.line-numbers}
class Solution {
    public String reorganizeString(String S) {
        int[] counts = new int[26];
        int n = S.length();
        int maxCount = 0;
        for(int i = 0; i < n; ++i) {
            ++counts[S.charAt(i) - 'a'];
            if(counts[S.charAt(i) - 'a'] > maxCount) {
                maxCount = counts[S.charAt(i) - 'a'];
            }
        }
        if(maxCount > (n + 1) / 2) {
            return "";
        }
        char[] arr = new char[n];
        int oddIndex = 1 , evenIndex = 0;
        int halfLength = n / 2;
        for(int i = 0; i < counts.length; ++i) {
            while(counts[i] > 0 && counts[i] <= halfLength && oddIndex < n) {
                arr[oddIndex] = (char)(i + 'a');
                --counts[i];
                oddIndex += 2;
            }
            while(counts[i] > 0) {
                arr[evenIndex] = (char)(i + 'a');
                --counts[i];
                evenIndex += 2;
            }
        }
        return new String(arr);
    }
}
```
# 重排链表
##题目
给定一个单链表 L：L0→L1→…→Ln-1→Ln ，
将其重新排列后变为： L0→Ln→L1→Ln-1→L2→Ln-2→…

你不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。
##基本思路
先把链表一分为二，然后把后半部分的链表逆置，最后合并两个链表
```java{.line-numbers}
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public void reorderList(ListNode head) {
        ListNode lastHead = head;
        ListNode cur = head;
        ListNode pre = null;
        int len = getLength(cur);
        int step = len - len / 2;
        while(step != 0){
            pre = lastHead;
            lastHead = lastHead.next;
            --step;
        }
        if(pre != null){
            pre.next = null;
        }else{
            return;
        }
        //reverse lasthead
        ListNode newHead = new ListNode();
        while(lastHead != null){
            ListNode next = lastHead.next;
            lastHead.next = newHead.next;
            newHead.next = lastHead;
            lastHead = next;
        }
        newHead = newHead.next;
        cur = head;
        while(cur != null && newHead != null){
            ListNode next = cur.next;
            ListNode lastNext = newHead.next;
            newHead.next = cur.next;
            cur.next = newHead;
            cur = next;
            newHead = lastNext;
        }
    }

    private int getLength(ListNode head){
        int len = 0;
        while(head != null){
            ++len;
            head = head.next;
        }
        return len;
    }
}
```
# 自由之路
## 题目
视频游戏“辐射4”中，任务“通向自由”要求玩家到达名为“Freedom Trail Ring”的金属表盘，并使用表盘拼写特定关键词才能开门。

给定一个字符串 ring，表示刻在外环上的编码；给定另一个字符串 key，表示需要拼写的关键词。您需要算出能够拼写关键词中所有字符的最少步数。

最初，ring 的第一个字符与12:00方向对齐。您需要顺时针或逆时针旋转 ring 以使 key 的一个字符在 12:00 方向对齐，然后按下中心按钮，以此逐个拼写完 key 中的所有字符。

旋转 ring 拼出 key 字符 key[i] 的阶段中：

您可以将 ring 顺时针或逆时针旋转一个位置，计为1步。旋转的最终目的是将字符串 ring 的一个字符与 12:00 方向对齐，并且这个字符必须等于字符 key[i] 。
如果字符 key[i] 已经对齐到12:00方向，您需要按下中心按钮进行拼写，这也将算作 1 步。按完之后，您可以开始拼写 key 的下一个字符（下一阶段）, 直至完成所有拼写。

### 示例
![](https://gitee.com/zacharytse/image/raw/master/img/20201111095903.png)
## 思路
使用动态规划。定义数组dp[i][j]表示拼接key的第i个字符，对于ring中的字符j需要旋转到12点钟方向的最小步数。
很显然，如果第i个字符要想和第j个字符进行拼接，两者必须是同一个字符。所以需要用一个数组pos来记录第i个字符在ring中的位置。
对于dp[i][j],每次都需要枚举上一次与12点方向对齐的k，然后根据k求一个最小值dp[i][j]。转移方程如下
$$
dp[i][j]=min_{k\in{pos[key[i-1]-'a']}}\{dp[i-1][k] + min\{abs(j - k),n - abs(j - k) + 1\}\}
$$
```java{.line-numbers}
class Solution {
    public int findRotateSteps(String ring, String key) {
        List<Integer>[] pos = new List[26];
        int m = key.length();
        int n = ring.length();
        int[][] dp = new int[m][n];
        for(int i = 0; i < pos.length; ++i){
            pos[i] = new ArrayList<Integer>();
        }
        for(int i = 0; i < n; ++i){
            pos[ring.charAt(i) - 'a'].add(i);
        }
        for(int i = 0; i < m; ++i){
            Arrays.fill(dp[i],0x3f3f3f);
        }
        for(Integer i : pos[key.charAt(0) - 'a']){
            dp[0][i] = Math.min(i,n-i) + 1;
        }
        for(int i = 1; i < m; ++i){
            for(Integer j : pos[key.charAt(i) - 'a']){
                for(Integer k : pos[key.charAt(i - 1) - 'a']){
                    dp[i][j] = Math.min(dp[i][j],dp[i-1][k] + Math.min(Math.abs(j - k),n - Math.abs(j - k)) + 1);
                }
            }
        }
        return Arrays.stream(dp[m-1]).min().getAsInt();
    }
}
```
# 最大间距
## 题目
给定一个无序的数组，找出数组在排序之后，相邻元素之间最大的差值。

如果数组元素个数小于 2，则返回 0。

示例:
```
输入: [3,6,9,1]
输出: 3
解释: 排序后的数组是 [1,3,6,9], 其中相邻元素 (3,6) 和 (6,9) 之间都存在最大差值 3。
```
## 思路
### 方法一(基数排序)
因为题目要求时间和空间复杂度为o(n)，所以可以考虑使用基数排序。
**基数排序在统计完每一个数字出现的次数后，应该要从后往前遍历数组将元素填充到对应位置上，这样才能保证排序的稳定性**
```java{.line-numbers}
class Solution {
    public int maximumGap(int[] nums) {
        if(nums.length < 2) {
            return 0;
        }
        int n = nums.length;
        int[] buf = new int[n];
        int maxVal = Arrays.stream(nums).max().getAsInt();
        int exp = 1;
        while(maxVal >= exp) {
            int[] cnt = new int[10];
            for(int i = 0; i < n; ++i) {
                int digit = (nums[i] / (int) exp) % 10;
                ++cnt[digit];
            }
            for(int i = 1; i < 10; ++i) {
                cnt[i] += cnt[i - 1];
            }
            /**
            *一定要从后往前，保证排序的稳定性
            */
            for(int i = n-1; i >= 0; --i) {
                int digit = (nums[i] / (int)exp) % 10;
                buf[cnt[digit] - 1] = nums[i];
                --cnt[digit];
            }
            System.arraycopy(buf,0,nums,0,n);
            exp *= 10;
        }
        int ret = 0;
        for(int i = 1; i < n; ++i) {
            ret = Math.max(ret,nums[i] - nums[i - 1]);
        }
        return ret;
    }
}
```
### 方法二(桶排序)
排序后的数组元素的最大间隔应该不小于$(maxVal - minVal) / (N - 1)$,使用反证法可以证明。证明如下
$$
A_n - A_1 = (A_n - A_{n-1}) +... + (A_2 - A_1)\newline
<(maxVal - minVal) / (N-1) +...+(maxVal-minVal)/(N-1)\newline
< maxVal - minVal
$$ 
而$A_n,A_1$显然是可以分别取maxVal和minVal，即不等式可以取等号，所以与事实不符。
可以将所有的元素分布在不同的桶中，每一个桶的容量是$(maxVal - minVal)/(N-1)$,由上述命题可知，同一个桶中的元素不可能出现最大间距，因此最大间距只有可能出现在相邻的两个桶中。所以只要遍历所有的桶，找到两个间隔最大的桶，即为最后的结果
```java{.line-numbers}
class Solution {
    public int maximumGap(int[] nums) {
        int n = nums.length;
        if(n < 2) {
            return 0;
        }
        int minVal = Arrays.stream(nums).min().getAsInt();
        int maxVal = Arrays.stream(nums).max().getAsInt();
        int d = Math.max(1,(maxVal - minVal) / (n - 1));
        int bucketSize = (maxVal - minVal) / d + 1;
        int[][] bucket = new int[bucketSize][2];
        for(int i = 0; i < bucketSize; ++i) {
            Arrays.fill(bucket[i],-1);
        }
        for(int i = 0; i < n; ++i) {
            int idx = (nums[i] - minVal) / d;
            if(bucket[idx][0] == -1) {
                bucket[idx][0] = bucket[idx][1] = nums[i];
            } else {
                bucket[idx][0] = Math.min(bucket[idx][0],nums[i]);
                bucket[idx][1] = Math.max(bucket[idx][1],nums[i]);
            }
        }
        int ret = 0;
        int prev = -1;
        for(int i = 0; i < bucketSize; ++i) {
            if(bucket[i][0] == -1) {
                continue;
            }
            if(prev != -1) {
                ret = Math.max(ret,bucket[i][0] - bucket[prev][1]);
            }
            prev = i;
        }
        return ret;
    }
}
```
# 最接近原点的k个点
## 题目
我们有一个由平面上的点组成的列表 points。需要从中找出 K 个距离原点 (0, 0) 最近的点。

（这里，平面上两点之间的距离是欧几里德距离。）

你可以按任何顺序返回答案。除了点坐标的顺序之外，答案确保是唯一的。

## 思路
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
# N皇后II
## 题目
n 皇后问题研究的是如何将 n 个皇后放置在 n×n 的棋盘上，并且使皇后彼此之间不能相互攻击。

给定一个整数 n，返回 n 皇后不同的解决方案的数量。

## 基本思路 
很简单的回溯，注意下检查是否合法的条件代码的编写，在检查左斜边和右斜边时，都是从当前行向上面的行做检查
```java{.line-numbers}
class Solution {
    char [][] q;
    int count = 0;
    public int totalNQueens(int n) {
        q = new char[n][n];
        for(int i = 0; i < n; ++i){
            for(int j = 0; j < n; ++j){
                q[i][j] = '.';
            }
        }
        dfs(0);
        return count;
    }

    private void dfs(int row){
        if(row == q.length){
            ++count;
            return;
        }
        for(int i = 0; i < q.length;++i){
            if(check(row,i) == true){
                q[row][i] = 'Q';
                dfs(row + 1);
                q[row][i] = '.';
            }
        }
    }

    private boolean check(int row,int col){
        //我写错了斜边的检查条件，应该倒着检查
        //因为是按行来遍历的，所以一行每次只会放一个皇后
        //检查同一列
        int y = 0;
        int x = 0;
        while(x < row){
            if(q[x][col] == 'Q'){
                return false;
            }
            ++x;
        }
        //检查左斜边
        x = row - 1;
        y = col - 1;
        while(x >= 0 && y >= 0){
            if(q[x][y] == 'Q'){
                return false;
            }
            --x;
            --y;
        }
        //检查右斜边
        x = row - 1;
        y = col + 1;
        while(x >= 0 && y < q.length){
            if(q[x][y] == 'Q'){
                return false;
            }
            --x;
            ++y;
        }
        return true;
    }
}
```
# O(1)时间插入、删除和获取随机元素
## 题目描述
设计一个支持在平均 时间复杂度 O(1) 下， 执行以下操作的数据结构。

注意: 允许出现重复元素。

insert(val)：向集合中插入元素 val。
remove(val)：当 val 存在时，从集合中移除一个 val。
getRandom：从现有集合中随机获取一个元素。每个元素被返回的概率应该与其在集合中的数量呈线性相关。

## 基本思路
使用数组保存所有的数据，同时维护一个HashMap，key是每一个元素，value是相同元素在数组中出现的所有下标
```java{.line-numbers}
class RandomizedCollection {

    private Map<Integer,Set<Integer>> idx;
    private List<Integer> nums;
    /** Initialize your data structure here. */
    public RandomizedCollection() {
        idx = new HashMap<>();
        nums = new ArrayList<>();
    }   
    
    /** Inserts a value to the collection. Returns true if the collection did not already contain the specified element. */
    public boolean insert(int val) {
        nums.add(val);
        Set<Integer> set = idx.getOrDefault(val,new HashSet<>());
        set.add(nums.size() - 1);
        idx.put(val,set);
        return set.size() == 1;
    }
    
    /** Removes a value from the collection. Returns true if the collection contained the specified element. */
    public boolean remove(int val) {
        if(!idx.containsKey(val)){
            return false;
        }
        Iterator<Integer> iter = idx.get(val).iterator();
        int i = iter.next();
        int lastNum = nums.get(nums.size() - 1);
        nums.set(i,lastNum);
        idx.get(val).remove(i);
        idx.get(lastNum).remove(nums.size() - 1);
        if(i < nums.size() - 1){
            idx.get(lastNum).add(i);
        }
        if(idx.get(val).size() == 0){
            idx.remove(val);
        }
        nums.remove(nums.size() - 1);
        return true;
    }
    
    /** Get a random element from the collection. */
    public int getRandom() {
        return nums.get((int)(Math.random() * nums.size()));
    }
}

/**
 * Your RandomizedCollection object will be instantiated and called as such:
 * RandomizedCollection obj = new RandomizedCollection();
 * boolean param_1 = obj.insert(val);
 * boolean param_2 = obj.remove(val);
 * int param_3 = obj.getRandom();
 */
```
# 分割数组为连续子序列
## 题目
给你一个按升序排序的整数数组 num（可能包含重复数字），请你将它们分割成一个或多个子序列，其中每个子序列都由连续整数组成且长度至少为 3 。

如果可以完成上述分割，则返回 true ；否则，返回 false 。

示例:
```
输入: [1,2,3,3,4,5]
输出: True
解释:
你可以分割出这样两个连续子序列 : 
1, 2, 3
3, 4, 5
```
## 思路
### 方法一
hash+最小堆
可以通过确定序列的长度以及序列的最后一位的数字x来确定整个序列，所以hash的key可以是这个数字x，value是一个最小堆，堆中存放着所有结尾为x的序列的长度。
最后判断所有的子序列长度是否大于等于3，如果有小于3的序列，则说明不能分割
```java{.line-numbers}
class Solution {
    public boolean isPossible(int[] nums) {
        if(nums.length < 3) {
            return false;
        }
        Map<Integer,PriorityQueue<Integer>> map = new HashMap<>();
        for(int x : nums) {
            if(!map.containsKey(x)) {
                map.put(x,new PriorityQueue<Integer>());
            }
            if(map.containsKey(x-1)){
                PriorityQueue<Integer> q = map.get(x-1);
                int length = q.poll();
                if(q.isEmpty()) {
                    map.remove(x-1);
                }
                map.get(x).offer(length+1);
            } else {
                map.get(x).offer(1);
            }
        }
        Set<Map.Entry<Integer,PriorityQueue<Integer>>> set = map.entrySet();
        for(Map.Entry<Integer,PriorityQueue<Integer>> entry : set) {
            if(entry.getValue().peek() < 3) {
                return false;
            }
        }
        return true;
    }
}
```
### 方法二
贪心
考虑使用两个hash表，其中一个表用来保存num数组中元素可以使用的次数，另一个hash表用来保存以x结尾的序列的数量。初始化时，第一个表初始化为元素的个数。
遍历整个数组，分两种情况考虑
- 当x-1在另一个hash表时，说明此时存在以x-1结尾的序列。此时把这个序列的数量减一，以x结尾的序列数量加一，并且减少第一个hash表中x的数量
- 当x-1不在另一个hash表时，此时x必然是序列的开端，因为要保证序列的长度至少为3，所以x+1,x+2的可用数量必须大于0。如果不大于0，则说明无法分割，直接返回flase。此时应该构成了序列[x,x+1,x+2]，所以要把x，x+1,x+2的数量都分别减1，同时增加x+2结尾的序列数量

最后如果没有出现不可分割的情况，则直接返回true

```java{.line-numbers}
class Solution {
    public boolean isPossible(int[] nums) {
        Map<Integer,Integer> counts = new HashMap<>();
        Map<Integer,Integer> countMap = new HashMap<>();
        for(int x : nums) {
            int count = counts.getOrDefault(x,0) + 1;
            counts.put(x,count); 
        }
        for(int x : nums) {
            int count = counts.get(x);
            if(count > 0) {
                int prevEndCount = countMap.getOrDefault(x - 1,0);
                if(prevEndCount > 0) {
                    //x-1结尾的序列是存在的
                    countMap.put(x-1,prevEndCount - 1);
                    countMap.put(x,countMap.getOrDefault(x,0) + 1);
                    counts.put(x,count - 1);
                } else {
                    //x-1结尾序列不存在
                    int count1 = counts.getOrDefault(x + 1,0);
                    int count2 = counts.getOrDefault(x + 2,0);
                    if(count1 > 0 && count2 > 0) {
                        countMap.put(x + 2,countMap.getOrDefault(x + 2,0) + 1);
                        counts.put(x,count-1);
                        counts.put(x + 1,count1-1);
                        counts.put(x+2,count2-1);
                    } else {
                        return false;
                    }
                }
            }
        }
         return true;
    }
}
```
# 杨辉三角
## 题目
给定一个非负整数 numRows，生成杨辉三角的前 numRows 行。
## 思路
数学方法
```java{.line-numbers}
class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> ret = new ArrayList<>(numRows);
        for(int i = 0; i < numRows; ++i) {
            List<Integer> row = new ArrayList<>(i + 1);
            for(int j = 0; j <= i ; ++j) {
                if(j == 0 || j == i) {
                    row.add(1);
                } else {
                    int left = ret.get(i - 1).get(j - 1);
                    int right = ret.get(i - 1).get(j);
                    row.add(left + right);
                }
            }
            ret.add(row);
        }
        return ret;
    }
}
```
# 翻转矩阵后的得分
## 题目
有一个二维矩阵 A 其中每个元素的值为 0 或 1 。

移动是指选择任一行或列，并转换该行或列中的每一个值：将所有 0 都更改为 1，将所有 1 都更改为 0。

在做出任意次数的移动后，将该矩阵的每一行都按照二进制数来解释，矩阵的得分就是这些数字的总和。

返回尽可能高的分数。

示例:
```
输入：[[0,0,1,1],[1,0,1,0],[1,1,0,0]]
输出：39
解释：
转换为 [[1,1,1,1],[1,0,0,1],[1,1,1,1]]
0b1111 + 0b1001 + 0b1111 = 15 + 9 + 15 = 39
```
## 思路
贪心，先把最高列全部置为1，然后从第二列开始，如果该列0的个数超过了一半，则把该列取反
```java{.line-numbers}
class Solution {
    public int matrixScore(int[][] A) {
        int n = A.length , m = A[0].length;
        //先把首行全部置为1
        for(int row = 0; row < n; ++row) {
            if(A[row][0] == 0) {
                reverse(A,row,true);
            }
        }
        //从第二列开始，如果1的个数小于0的个数，则翻转该列
        for(int col = 1; col < m; ++col) {
            int zeroCount = 0;
            for(int row = 0; row < n; ++row) {
                if(A[row][col] == 0) {
                    ++zeroCount;
                }
            }
            if(zeroCount >= (n + 1) / 2) {
                reverse(A,col,false);
            }
        }
        
        int ret = 0;
        for(int i = 0; i < n; ++i) {
            ret += getScore(A[i]);
        }
        return ret;
    }

    private void reverse(int[][] A,int start,boolean isRow) {
        if(isRow) {
            int m = A[0].length;
            for(int i = 0 ; i < m; ++i) {
                A[start][i] = 1 - A[start][i];
            }
        } else {
            int n = A.length;
            for(int i = 0; i < n; ++i) {
                A[i][start] = 1 - A[i][start];
            }
        }
    }

    private int getScore(int[] num) {
        int n = num.length;
        int ret = 0;
        for(int i = 0; i < n; ++i) {
            ret += (num[i] << (n - i - 1));
        }
        return ret;
    }
}
```
# 将数组拆分成斐波那契序列
## 题目
给定一个数字字符串 S，比如 S = "123456579"，我们可以将它分成斐波那契式的序列 [123, 456, 579]。

形式上，斐波那契式序列是一个非负整数列表 F，且满足：

0 <= F[i] <= 2^31 - 1，（也就是说，每个整数都符合 32 位有符号整数类型）；
F.length >= 3；
对于所有的0 <= i < F.length - 2，都有 F[i] + F[i+1] = F[i+2] 成立。
另外，请注意，将字符串拆分成小块时，每个块的数字一定不要以零开头，除非这个块是数字 0 本身。

返回从 S 拆分出来的任意一组斐波那契式的序列块，如果不能拆分则返回 []。

示例：
```
输入："123456579"
输出：[123,456,579]
```
## 思路
采取回溯加剪枝的方法。每次形成一个新的数字，然后将新的数字加入到队列中。剪枝策略如下:
- 如果当前串的开头已经为0，但该串的长度不为1，则可以直接返回false(不能以0开头)
- 如果该串表示的数字已经超过了int的范围，则直接返回false
- 如果答案列表中已经有两个或两个以上的数了，如果当前数比前面2个数的和还要大，也要直接剪枝

```java{.line-numbers}
class Solution {
    public List<Integer> splitIntoFibonacci(String S) {
        List<Integer> ans = new ArrayList<>();
        backtrack(S,ans,0,0,0);
        return ans;
    }

    private boolean backtrack(String s,List<Integer> ans,int index,int sum,int prev) {
        if(index >= s.length()) {
            return ans.size() >= 3;
        }
        long curr = 0l;
        for(int i = index; i < s.length();++i){
            if(i > index && s.charAt(index) == '0') {
                break;
            }
            curr = curr * 10 + (s.charAt(i) - '0');
            if(curr > Integer.MAX_VALUE) {
                break;
            }
            int cur = (int) curr;
            if(ans.size() >= 2) {
                if(cur < sum) {
                    continue;
                } else if(cur > sum) {
                    break;
                }
            }
            ans.add(cur);
            if(backtrack(s,ans,i + 1,prev + cur,cur)) {
                return true;
            } else {
                ans.remove(ans.size() - 1);
            }
        } 
        return false;
    }
}
```
# 不同路径
## 题目
一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

示例:
```
输入：m = 3, n = 7
输出：28
```
## 思路
可以用dfs，但会超时，所以用动态规划。用一个dp数组，dp[i][j]记录到(i,j)的路径总数，最后答案就是dp[m-1][n-1]
```java{.line-numbers}
class Solution {
    public int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];
        dp[0][0] = 1;
        for(int i = 0; i < m; ++i) {
            for(int j = 0; j < n; ++j) {
                if(i == 0 && j == 0) {
                    continue;
                }
                if(i == 0) {
                    dp[i][j] = dp[i][j - 1];
                } else if(j == 0) {
                    dp[i][j] = dp[i - 1][j];
                } else {
                    dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
                }
            }
        }
        return dp[m - 1][n - 1];
    }
}
```
滚动数组的优化:
```java{.line-numbers}
class Solution {
    public int uniquePaths(int m, int n) {
        int[] dp = new int[n];
        Arrays.fill(dp,1);
        for(int i = 1; i < m; ++i) {
            for(int j = 1; j < n; ++j) {
                dp[j] += dp[j - 1];
            }
        }
        return dp[n - 1];
    }
}
```
# 柠檬水找零
## 题目
在柠檬水摊上，每一杯柠檬水的售价为 5 美元。

顾客排队购买你的产品，（按账单 bills 支付的顺序）一次购买一杯。

每位顾客只买一杯柠檬水，然后向你付 5 美元、10 美元或 20 美元。你必须给每个顾客正确找零，也就是说净交易是每位顾客向你支付 5 美元。

注意，一开始你手头没有任何零钱。

如果你能给每位顾客正确找零，返回 true ，否则返回 false 。

示例:
```
输入：[5,5,5,10,20]
输出：true
解释：
前 3 位顾客那里，我们按顺序收取 3 张 5 美元的钞票。
第 4 位顾客那里，我们收取一张 10 美元的钞票，并返还 5 美元。
第 5 位顾客那里，我们找还一张 10 美元的钞票和一张 5 美元的钞票。
由于所有客户都得到了正确的找零，所以我们输出 true。
```

## 思路
正常模拟，对于20美元的找钱情况要分两种情况讨论，一种是找一个10美元和一个5美元，一种是找3个5美元
```java{.line-numbers}
class Solution {
    public boolean lemonadeChange(int[] bills) {
        int fiveMoney = 0 , tenMoney = 0;
        for(int i = 0; i < bills.length; ++i) {
            switch(bills[i]) {
                case 5 :
                    ++fiveMoney;
                    break;
                case 10:
                    if(fiveMoney == 0) {
                        return false;
                    }
                    --fiveMoney;
                    ++tenMoney;
                    break;
                case 20:
                    if(tenMoney != 0) {
                        if(fiveMoney != 0) {
                            --tenMoney;
                            --fiveMoney;
                        } else {
                            return false;
                        }
                    } else {
                        if(fiveMoney >= 3) {
                            fiveMoney -= 3;
                        } else {
                            return false;
                        }
                    }
                    break;
            }
        }
        return true;
    }
}
```
# Dota2参议院
## 题目
Dota2 的世界里有两个阵营：Radiant(天辉)和 Dire(夜魇)

Dota2 参议院由来自两派的参议员组成。现在参议院希望对一个 Dota2 游戏里的改变作出决定。他们以一个基于轮为过程的投票进行。在每一轮中，每一位参议员都可以行使两项权利中的一项：

1. 禁止一名参议员的权利：
    参议员可以让另一位参议员在这一轮和随后的几轮中丧失所有的权利。

2. 宣布胜利：
 如果参议员发现有权利投票的参议员都是同一个阵营的，他可以宣布胜利并决定在游戏中的有关变化。

 给定一个字符串代表每个参议员的阵营。字母 “R” 和 “D” 分别代表了 Radiant（天辉）和 Dire（夜魇）。然后，如果有 n 个参议员，给定字符串的大小将是 n。

以轮为基础的过程从给定顺序的第一个参议员开始到最后一个参议员结束。这一过程将持续到投票结束。所有失去权利的参议员将在过程中被跳过。

假设每一位参议员都足够聪明，会为自己的政党做出最好的策略，你需要预测哪一方最终会宣布胜利并在 Dota2 游戏中决定改变。输出应该是 Radiant 或 Dire。

示例:
```
输入："RD"
输出："Radiant"
解释：第一个参议员来自 Radiant 阵营并且他可以使用第一项权利让第二个参议员失去权力，因此第二个参议员将被跳过因为他没有任何权利。然后在第二轮的时候，第一个参议员可以宣布胜利，因为他是唯一一个有投票权的人
```
## 思路
### 方法一
采用类似约瑟夫环模拟的想法
```java{.line-numbers}
class Solution {
    public String predictPartyVictory(String senate) {
        int count = senate.length();
        int radCount = 0 , direCount = 0;
        char[] arr = senate.toCharArray();
        int n = arr.length;
        for(int i = 0; i < arr.length; ++i) {
            if(arr[i] == 'R') {
                ++radCount;
            } else {
                ++direCount;
            }
        }
        int idx = 0;
        while(count > 0) {
            if(arr[idx] == 0) {
                idx = (idx + 1) % n;
                continue;
            }
            if(radCount == 0) {
                return "Dire";
            }
            if(direCount == 0) {
                return "Radiant";
            }
            if(arr[idx] == 'R') {
                --direCount;
                arr[getNext(arr,(idx + 1) % n,'D')] = 0;
            } else {
                --radCount;
                arr[getNext(arr,(idx+1)%n,'R')] = 0;
            }
            idx = (idx + 1) % n;
            --count;
        }
        return "";
    }

    private int getNext(char[] arr,int start,char ch) {
        while(arr[start] != ch) {
            start = (start + 1) % arr.length;
        }
        return start;
    }
}
```
### 方法二
采用贪心，每次都把下一个要发言的对方阵营的人禁掉，然后为了保证相对顺序不变，该次发言的人下标要加上n
```java{.line-numbers}
class Solution {
    public String predictPartyVictory(String senate) {
        int n = senate.length();
        Queue<Integer> radiant = new LinkedList<>();
        Queue<Integer> dire = new LinkedList<>();
        for(int i = 0; i < senate.length(); ++i) {
            if(senate.charAt(i) == 'R') {
                radiant.offer(i);
            } else {
                dire.offer(i);
            }
        }

        while(!radiant.isEmpty() && !dire.isEmpty()) {
            int radiantIdx = radiant.poll() , direIdx = dire.poll();
            if(radiantIdx < direIdx) {
                radiant.offer(radiantIdx + n);
            } else {
                dire.offer(direIdx + n);
            }
        }
        return !radiant.isEmpty() ? "Radiant" : "Dire";
    }
}
```
# 摆动序列
## 题目
如果连续数字之间的差严格地在正数和负数之间交替，则数字序列称为摆动序列。第一个差（如果存在的话）可能是正数或负数。少于两个元素的序列也是摆动序列。

例如， [1,7,4,9,2,5] 是一个摆动序列，因为差值 (6,-3,5,-7,3) 是正负交替出现的。相反, [1,4,7,2,5] 和 [1,7,4,5,5] 不是摆动序列，第一个序列是因为它的前两个差值都是正数，第二个序列是因为它的最后一个差值为零。

给定一个整数序列，返回作为摆动序列的最长子序列的长度。 通过从原始序列中删除一些（也可以不删除）元素来获得子序列，剩下的元素保持其原始顺序。

示例:
```
输入: [1,7,4,9,2,5]
输出: 6 
解释: 整个序列均为摆动序列。
```
## 思路
### 动态规划
定义上升摆动序列为最后一个数字是大于倒数第二个数字的摆动序列，定义下降摆动序列为最后一个数字小于倒数第二个数字的摆动序列。定义两个数组up,down分别表示到i为止，上升摆动序列的最大长度和下降摆动序列的最大长度。

先考虑up[i]的情况
- 当nums[i] <= nums[i - 1]时，此时up[i] = up[i - 1]
- 当nums[i] > nums[i - 1]时，up[i-1]可以直接转移到up[i]。对于下降摆动序列，可以直接把nums[i]加入进去，变为上升摆动序列，所以up[i] = max{up[i - 1],down[i - 1] + 1}

down[i]和up[i]是镜像问题

综上，状态转移方程为:
![](https://gitee.com/zacharytse/image/raw/master/img/20201212103655.png)

```java{.line-numbers}
class Solution {
    public int wiggleMaxLength(int[] nums) {
        int n = nums.length;
        if(n  < 2) {
            return n;
        }
        int[] up = new int[n];
        int[] down = new int[n];
        up[0] = down[0] = 1;
        for(int i = 1; i < n; ++i) {
            if(nums[i] == nums[i - 1]) {
                up[i] = up[i - 1];
                down[i] = down[i - 1];
            } else if(nums[i] > nums[i - 1]) {
                up[i] = Math.max(up[i - 1],down[i - 1] + 1);
                down[i] = down[i - 1];
            } else {
                up[i] = up[i - 1];
                down[i] = Math.max(down[i - 1],up[i - 1] + 1);
            }
        }
        return Math.max(up[n - 1],down[n - 1]);
    }
}
```
**优化**
因为每次只用到前面一个的状态，所以可以把up和down只用一个变量来表示
```java{.line-numbers}
class Solution {
    public int wiggleMaxLength(int[] nums) {
        int n = nums.length;
        if(n  < 2) {
            return n;
        }
        int up = 1, down = 1;
        for(int i = 1; i < n; ++i) {
            if(nums[i] > nums[i - 1]) {
                up = Math.max(up,down + 1);
            } else if(nums[i] < nums[i - 1]) {
                down = Math.max(down,up + 1);
            }
        }
        return Math.max(up,down);
    }
}
```
### 方法三
贪心，可以贪心的认为峰或者谷的元素一定在最长的摇摆序列中。如果存在一个不是峰或者谷的过渡元素在摇摆序列中，则它所在的上升区间所对应的峰或者所在的下降区间所对应的谷也同样可以放到摇摆序列中，因此没有必要放一个过渡元素在最长的摇摆序列中。
```java{.line-numbers}
class Solution {
    public int wiggleMaxLength(int[] nums) {
        int n = nums.length;
        if(n < 2) {
            return n;
        }
        int prediff = nums[1] - nums[0];
        int ret = prediff != 0 ? 2 : 1;
        for(int i = 2; i < n; ++i) {
            int diff = nums[i] - nums[i - 1];
            if((diff > 0 && prediff <= 0) || (diff < 0 && prediff >= 0)) {
                ++ret;
                prediff = diff;
            }
        }
        return ret;
    }
}
```