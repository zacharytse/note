
[TOC]
#题目
n 皇后问题研究的是如何将 n 个皇后放置在 n×n 的棋盘上，并且使皇后彼此之间不能相互攻击。

给定一个整数 n，返回 n 皇后不同的解决方案的数量。

#基本思路
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